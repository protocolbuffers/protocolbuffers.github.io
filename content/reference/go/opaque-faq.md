+++
title = "Go Opaque API FAQ"
weight = 670
linkTitle = "Opaque API FAQ"
description = "A list of frequently asked questions about the Opaque API."
type = "docs"
+++

<style>
.good pre {
  border-left: 2px solid;
  border-left-color: #0b8043;
  background-color: #e2f3eb !important;
}
.bad pre {
  border-left: 2px solid;
  border-left-color: #c53929;
  background-color: #fbe9e7 !important;
}
</style>

The Opaque API is the latest version of the Protocol Buffers implementation for
the Go programming language. The old version is now called Open Struct API. See
the [Go Protobuf: The new Opaque API](https://go.dev/blog/protobuf-opaque) blog
post for an introduction.

This FAQ answers common questions about the new API and the migration process.

## Which API Should I Use When Creating a New .proto File? {#which}

We recommend you select the Opaque API for new development. Protobuf Edition
2024 (see
[Protobuf Editions Overview](/editions/overview/)) will
make the Opaque API the default.

## How Do I Enable the New Opaque API for My Messages? {#enable}

With Protobuf Edition 2023 (current at the time of writing), you can select the
Opaque API by setting the `api_level` editions feature to `API_OPAQUE` in your
`.proto` file. This can be set per file or per message:

```proto
edition = "2023";

package log;

import "google/protobuf/go_features.proto";
option features.(pb.go).api_level = API_OPAQUE;

message LogEntry { … }
```

Protobuf Edition 2024 will default to the Opaque API, meaning you will not need
extra imports or options anymore:

```proto
edition = "2024";

package log;

message LogEntry { … }
```

The release date estimate for Protobuf Edition 2024 is early 2025.

For your convenience, you can also override the default API level with a
`protoc` command-line flag:

```
protoc […] --go_opt=default_api_level=API_HYBRID
```

To override the default API level for a specific file (instead of all files),
use the `apilevelM` mapping flag (similar to
[the `M` flag for import paths](/reference/go/go-generated/#package)):

```
protoc […] --go_opt=apilevelMhello.proto=API_HYBRID
```

The command-line flags also work for `.proto` files still using proto2 or proto3
syntax, but if you want to select the API level from within the `.proto` file,
you need to migrate said file to editions first.

## How Do I Enable Lazy Decoding? {#lazydecoding}

1.  Migrate your code to use the opaque implementation.
1.  Set the `[lazy = true]` option on the proto submessage fields that should be
    lazily decoded.
1.  Run your unit and integration tests, and then roll out to a staging
    environment.

## Are Errors Ignored with Lazy Decoding? {#lazydecodingerrors}

No.
[`proto.Marshal`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Marshal)
will always validate the wire format data, even when decoding is deferred until
first access.

## Where Can I Ask Questions or Report Issues? {#questions}

If you found an issue with the `open2opaque` migration tool (such as incorrectly
rewritten code), please report it in the
[open2opaque issue tracker](https://github.com/golang/open2opaque/issues).

If you found an issue with Go Protobuf, please report it in the
[Go Protobuf issue tracker](https://github.com/golang/protobuf/issues/).

## What Are the Benefits of the Opaque API? {#benefits}

The Opaque API comes with numerous benefits:

*   It uses a more efficient memory representation, thereby reducing memory and
    Garbage Collection cost.
*   It makes lazy decoding possible, which can significantly improve
    performance.
*   It fixes a number of sharp edges. Bugs resulting from pointer address
    comparison, accidental sharing, or undesired use of Go reflection are all
    prevented when using the Opaque API.
*   It makes the ideal memory layout possible by enabling profile-driven
    optimizations.

See
[the Go Protobuf: The new Opaque API blog post](https://go.dev/blog/protobuf-opaque)
for more details on these points.

## Which Is Faster, Builders or Setters? {#builders-faster}

Generally, code using builders:

```go
_ = pb.M_builder{
  F: &val,
}.Build()
```

is slower than the following equivalent:

```go
m := &pb.M{}
m.SetF(val)
```

for the following reasons:

1.  The `Build()` call iterates over all fields in the message (even ones that
    are not explicitly set) and copies their values (if any) to the final
    message. This linear performance matters for messages with many fields.
1.  There's a potential extra heap allocation (`&val`).
1.  The builder can be significantly larger and use more memory in presence of
    oneof fields. Builders have a field per oneof union member while the message
    can store the oneof itself as a single field.

Aside from runtime performance, if binary size is a concern for you, avoiding
builders will result in less code.

## How Do I Use Builders? {#builders-how}

Builders are designed to be used as *values* and with an immediate `Build()`
call. Avoid using pointers to builders or storing builders in variables.

```go {.good}
m := pb.M_builder{
    // ...
}.Build()
```

```go {.bad}
// BAD: Avoid using a pointer
m := (&pb.M_builder{
    // ...
}).Build()
```

```go {.bad highlight="content:b.:= "}
// BAD: avoid storing in a variable
b := pb.M_builder{
    // ...
}
m := b.Build()
```

Proto messages are immutable in some other languages, hence users tend to pass
the builder type into function calls when constructing a proto message. Go proto
messages are mutable, hence there's no need for passing the builder into
function calls. Simply pass the proto message.

```go {.bad}
// BAD: avoid passing a builder around
func populate(mb *pb.M_builder) {
  mb.Field1 = proto.Int32(4711)
  //...
}
// ...
mb := pb.M_builder{}
populate(&mb)
m := mb.Build()
```

```go {.good}
func populate(mb *pb.M) {
  mb.SetField1(4711)
  //...
}
// ...
m := &pb.M{}
populate(m)
```

Builders are designed to imitate the composite literal construction of the Open
Struct API, not as an alternative representation of a proto message.

The recommended pattern is also more performant. The intended use of `Build()`
where it is called directly on the builder struct literal can be optimized well.
A separate call to `Build()` is much harder to optimize, as the compiler may not
easily identify which fields are populated. If the builder lives longer, there's
also a high chance that small objects like scalars have to be heap allocated and
later need to be freed by the garbage collector.

## Should I Use Builders or Setters? {#builders-vs-setters}

When constructing an empty protocol buffer, you should use `new` or an empty
composite literals. Both are equivalently idiomatic to construct a zero
initialized value in Go and are more performant than an empty builder.

```go {.good}
m1 := new(pb.M)
m2 := &pb.M{}
```

```go {.bad}
// BAD: avoid: unnecessarily complex
m1 := pb.M_builder{}.Build()
```

In cases where you need to construct non-empty protocol buffers, you have the
choice between using setters or using builders. Either is fine, but most people
will find builders more readable. If the code you are writing needs to perform
well,
[setters are generally slightly more performant than builders](#builders-faster).

```go {.good}
// Recommended: using builders
m1 := pb.M1_builder{
    Submessage: pb.M2_builder{
        Submessage: pb.M3_builder{
            String: proto.String("hello world"),
            Int:    proto.Int32(42),
        }.Build(),
        Bytes: []byte("hello"),
    }.Build(),
}.Build()
```

```go
// Also okay: using setters
m3 := &pb.M3{}
m3.SetString("hello world")
m3.SetInt(42)
m2 := &pb.M2{}
m2.SetSubmessage(m3)
m2.SetBytes([]byte("hello"))
m1 := &pb.M1{}
m1.SetSubmessage(m2)
```

You can combine the use of builder and setters if certain fields require
conditional logic before setting.

```go {.good}
m1 := pb.M1_builder{
    Field1: value1,
}.Build()
if someCondition() {
    m1.SetField2(value2)
    m1.SetField3(value3)
}
```

## How Can I Influence open2opaque’s Builder Behavior? {#builders-flags}

The `open2opaque` tool’s `--use_builders` flag can have the following values:

*   `--use_builders=everywhere`: always use builders, no exceptions.
*   `--use_builders=tests`: use builders only in tests, setters otherwise.
*   `--use_builders=nowhere`: never use builders.

## How Much Performance Benefit Can I Expect? {#performance-benefit}

This depends heavily on your workload. The following questions can guide your
performance exploration:

*   How big a percentage of your CPU usage is Go Protobuf? Some workloads, like
    logs analysis pipelines that compute statistics based on Protobuf input
    records, can spend about 50% of their CPU usage in Go Protobuf. Performance
    improvements will likely be clearly visible in such workloads. On the other
    end of the spectrum, in programs that only spend 3-5% of their CPU usage in
    Go Protobuf, performance improvements will often be insignificant compared
    to other opportunities.
*   How amenable to lazy decoding is your program? If large portions of the
    input messages are never accessed, lazy decoding can save a lot of work.
    This pattern is usually encountered in jobs like proxy servers (which pass
    through the input as-is), or logs analysis pipelines with high selectivity
    (which discard many records based on a high-level predicate).
*   Do your message definitions contain many elementary fields with explicit
    presence? The Opaque API uses a more efficient memory representation for
    elementary fields like integers, booleans, enums and floats, but not
    strings, repeated fields or submessages.

## How Does Proto2, Proto3, and Editions Relate to the Opaque API? {#proto23editions}

The terms proto2 and proto3 refer to different syntax versions in your `.proto`
files. [Protobuf Editions](/editions/overview) is the
successor to both proto2 and proto3.

The Opaque API affects only the generated code in `.pb.go` files, not what you
write in your `.proto` files.

The Opaque API works the same, independent of which syntax or edition your
`.proto` files use. However, if you want to select the Opaque API on a per-file
basis (as opposed to using a command-line flag when you are running `protoc`),
you must migrate the file to editions first. See
[How Do I Enable the New Opaque API for My Messages?](#enable) for details.

## Why Only Change the Memory Layout of Elementary Fields? {#memorylayout}

The
[announcement blog post’s “Opaque structs use less memory” section](https://go.dev/blog/protobuf-opaque#lessmemory)
explains:

> This performance improvement [modeling field presence more efficiently]
> depends heavily on your protobuf message shape: The change only affects
> elementary fields like integers, booleans, enums and floats, but not strings,
> repeated fields or submessages.

A natural follow-up question is why strings, repeated fields, and submessages
remain pointers in the Opaque API. The answer is twofold.

### Consideration 1: Memory Usage

Representing submessages as values instead of pointers would increase memory
usage: each Protobuf message type carries internal state, which would consume
memory even when the submessage is not actually set.

For strings and repeated fields, the situation is more nuanced. Let’s compare
the memory usage of using a string value compared to a string pointer:

Go variable type | set? | [word]s                  | #bytes
---------------- | ---- | ------------------------ | ------
`string`         | yes  | 2 (data, len)            | 16
`string`         | no   | 2 (data, len)            | 16
`*string`        | yes  | 1 (data) + 2 (data, len) | 24
`*string`        | no   | 1 (data)                 | 8

[word]: https://en.wikipedia.org/wiki/Word_(computer_architecture)

(The situation is similar for slices, but slice headers need 3 words: data, len,
cap.)

If your string fields are overwhelmingly not set, using a pointer saves RAM. Of
course, this saving comes at the cost of introducing more allocations and
pointers into the program, which increases load on the Garbage Collector.

The advantage of the Opaque API is that we can change the representation without
any changes to user code. The current memory layout was optimal for us when we
introduced it, but if we measured today or 5 years into the future, maybe we
would have chosen a different layout.

As described in the
[announcement blog post’s “Making the ideal memory layout possible” section](https://go.dev/blog/protobuf-opaque#idealmemory),
we aim to make these optimization decisions on a per-workload basis in the
future.

### Consideration 2: Lazy Decoding

Aside from the memory usage consideration, there is another restriction: fields
for which [lazy decoding](#lazydecoding) is enabled must be represented by
pointers.

Protobuf messages are safe for concurrent access (but not concurrent mutation),
so if two different goroutines trigger lazy decoding, they need to coordinate
somehow. This coordination is implemented through using the
[`sync/atomic` package](https://pkg.go.dev/sync/atomic), which can update
pointers atomically, but not slice headers (which exceed a [word]).

While `protoc` currently only permits lazy decoding for (non-repeated)
submessages, this reasoning holds for all field types.
