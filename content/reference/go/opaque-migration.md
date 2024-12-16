+++
title = "Go Opaque API Migration"
weight = 650
linkTitle = "Opaque API Migration"
description = "Describes the automated migration to the Opaque API."
type = "docs"
+++

The Opaque API is the latest version of the Protocol Buffers implementation for
the Go programming language. The old version is now called Open Struct API. See
the [Go Protobuf: Releasing the Opaque API](https://go.dev/blog/protobuf-opaque)
blog post for an introduction.

The migration to the Opaque API happens incrementally, on a per-proto-message or
per-`.proto`-file basis, by setting the Protobuf Editions feature `api_level`
option to one of its possible values:

*   `API_OPEN` selects the Open Struct API; this was the only API before
    December 2024.
*   `API_HYBRID` is a step between Open and Opaque: The Hybrid API also includes
    accessor methods (so you can update your code), but still exports the struct
    fields as before. There is no performance difference; this API level only
    helps with the migration.
*   `API_OPAQUE` selects the Opaque API.

Today, the default is `API_OPEN`, but the upcoming
[Protobuf Edition 2024](/editions/overview) will change
the default to `API_OPAQUE`.

To use the Opaque API before Edition 2024, set the `api_level` like so:

```proto
edition = "2023";

package log;

import "google/protobuf/go_features.proto";
option features.(pb.go).api_level = API_OPAQUE;

message LogEntry { … }
```

Before you can change the `api_level` to `API_OPAQUE` for existing files, all
existing usages of the generated proto code need to be updated. The
`open2opaque` tool helps with this.

For your convenience, you can also override the default API level with a
`protoc` command-line flag:

```
protoc […] --go_opt=default_api_level=API_OPAQUE
```

To override the default API level for a specific file (instead of all files),
use the `apilevelM` mapping flag (similar to
[the `M` flag for import paths](/reference/go/go-generated/#package)):

```
protoc […] --go_opt=apilevelMhello.proto=API_OPAQUE
```

The command-line flags also work for `.proto` files still using proto2 or proto3
syntax, but if you want to select the API level from within the `.proto` file,
you need to migrate said file to editions first.

## Automated migration {#automated}

We try to make migrating existing projects to the Opaque API as easy as possible
for you: our open2opaque tool does most of the work!

To install the migration tool, use:

```
go install google.golang.org/open2opaque@latest
```

{{% alert title="Note" color="info" %}}If
you encounter any issues with the automated migration approach, refer to the
[Opaque API: Manual Migration](/reference/go/opaque-migration-manual)
guide. {{% /alert %}}

### Project Preparation {#projectprep}

Ensure your build environment and project are using recent-enough versions of
Protocol Buffers and Go Protobuf:

1.  Update the protobuf compiler (protoc) from
    [the protobuf release page](https://github.com/protocolbuffers/protobuf/releases/latest)
    to version 29.0 or newer.

1.  Update the protobuf compiler Go plugin (protoc-gen-go) to version 1.36.0 or
    newer:

    ```
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    ```

1.  In each project, update the `go.mod` file to use the protobuf module in
    version 1.36.0 or newer:

    ```
    go get google.golang.org/protobuf@latest
    ```

    **Note:** if you are not yet importing `google.golang.org/protobuf`, you
    might still be on an older module. See
    [the `google.golang.org/protobuf` announcement (from 2020)](https://go.dev/blog/protobuf-apiv2)
    and migrate your code before returning to this page.

### Step 1. Switch to the Hybrid API {#setup}

Use the `open2opaque` tool to switch your `.proto` files to the Hybrid API:

```
open2opaque setapi -api HYBRID $(find . -name "*.proto")
```

Your existing code will continue to build. The Hybrid API is a step between the
Open and Opaque API which adds the new accessor methods but keeps struct fields
visible.

### Step 2. `open2opaque rewrite` {#rewrite}

To rewrite your Go code to use the Opaque API, run the `open2opaque rewrite`
command:

```
open2opaque rewrite -levels=red github.com/robustirc/robustirc/...
```

You can specify one or more
[packages or patterns](https://pkg.go.dev/cmd/go#hdr-Package_lists_and_patterns).

As an example, if you had code like this:

```go
logEntry := &logpb.LogEntry{}
if req.IPAddress != nil {
    logEntry.IPAddress = redactIP(req.IPAddress)
}
logEntry.BackendServer = proto.String(host)
```

The tool would rewrite it to use accessors:

```go
logEntry := &logpb.LogEntry{}
if req.HasIPAddress() {
    logEntry.SetIPAddress(redactIP(req.GetIPAddress()))
}
logEntry.SetBackendServer(host)
```

Another common example is to initialize a protobuf message with a struct
literal:

```go
return &logpb.LogEntry{
    BackendServer: proto.String(host),
}
```

In the Opaque API, the equivalent is to use a Builder:

```go
return logpb.LogEntry_builder{
    BackendServer: proto.String(host),
}.Build()
```

The tool classifies its available rewrites into different levels. The
`-levels=red` argument enables all rewrites, including those that require human
review. The following levels are available:

*   <span style="background-color: lightgreen">green:</span> Safe rewrites (high
    confidence). Includes most changes the tool makes. These changes do not
    require a close look and could even be submitted by automation, without any
    human oversight.
*   <span style="background-color: yellow">yellow:</span> (reasonable
    confidence) These rewrites require human review. They should be correct, but
    please review them.
*   <span style="background-color: salmon">red:</span> Potentially dangerous
    rewrites, changing rare and complicated patterns. These require careful
    human review. For example, when an existing function takes a `*string`
    parameter, the typical fix of using `proto.String(msg.GetFoo())` does not
    work if the function meant to change the field value by writing to the
    pointer (`*foo = "value"`).

Many programs can be fully migrated with only green changes. Before you can
migrate a proto message or file to the Opaque API, you need to complete all
rewrites of all levels, at which point no direct struct access remains in your
code.

### Step 3. Migrate and Verify {#migrate-and-verify}

To complete the migration, use the `open2opaque` tool to switch your `.proto`
files to the Opaque API:

```
open2opaque setapi -api OPAQUE $(find . -name "*.proto")
```

Now, any remaining code that was not rewritten to the Opaque API yet will no
longer compile.

Run your unit tests, integration tests and other verification steps, if any.

## Questions? Issues?

First, check out the
[Opaque API FAQ](/reference/go/opaque-faq). If that
doesn't answer your question or resolve your issue, see
[Where can I ask questions or report issues?](/reference/go/opaque-faq#questions)
