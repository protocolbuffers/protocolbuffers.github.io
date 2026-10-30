

+++
title = "We Prefer Extensions Over Any"
weight = 93
description = "Covers why the Protobuf team regrets Any as a replacement for extensions, and why extensions are preferred for most use cases"
type = "docs"
+++

When proto3 was introduced, it removed general-purpose
[extensions](/programming-guides/editions#extensions)
(keeping them only for custom options) and introduced
[`google.protobuf.Any`](/programming-guides/editions#any)
as their replacement.

Unfortunately, `Any` has enough design flaws that that this is a decision the
Protobuf team regrets. We have no plans to formally deprecate `Any`, and it
remains supported, but we recommend using extensions instead of `Any` for nearly
all use cases.

*   [Protobuf Editions](/editions/overview) restored full
    support for extensions starting in Edition 2023.
*   For new schemas, **prefer extensions over `Any`** whenever extensions can
    satisfy the use case (see [Proto Best
    Practices](/best-practices/dos-donts#avoid-any)).
*   Reserve `Any` for the uncommon case where horizontal infrastructure needs to
    propagate completely arbitrary messages and declaring an extension for each
    possible type is infeasible.

## What Extensions and `Any` Are {#what}

Both mechanisms solve the same dependency problem: letting a container message
carry a submessage whose `.proto` file the container does not import. Instead of
the container depending on every possible payload type, each payload depends on
the container.

### Extensions {#what-extensions}

An [extension](/programming-guides/editions#extensions)
is a field defined outside of its container message, using a field number from
an `extensions` range reserved by the container:

```proto
// media/user_content.proto
edition = "2024";

package media;

message UserContent {
  string id = 1;

  extensions 100 to 199 [
    declaration = {
      number: 100,
      full_name: ".kittens.kitten_video",
      type: ".kittens.Video"
    }
  ];
}
```

```proto
// kittens/video_ext.proto
edition = "2024";

package kittens;

import "kittens/video.proto";
import "media/user_content.proto";

extend media.UserContent {
  Video kitten_video = 100;
}
```

In generated code, the extension is accessed as a typed submessage stored
directly on the parent message:

```cpp
media::UserContent content;
content.MutableExtension(kittens::kitten_video)->set_title("Cute kittens");

if (content.HasExtension(kittens::kitten_video)) {
  const kittens::Video& video = content.GetExtension(kittens::kitten_video);
}
```

On the wire, an extension is identical to a regular field with the same field
number, type, and cardinality. With [extension
declarations](/programming-guides/extension_declarations),
the container can list the number, name, and type name of each extension in one
place without importing the payload `.proto` file.

### `google.protobuf.Any` {#what-any}

[`google.protobuf.Any`](/reference/protobuf/google.protobuf#any)
is a wrapper message defined in `google/protobuf/any.proto`:

```proto
syntax = "proto3";

package google.protobuf;

message Any {
  string type_url = 1;
  bytes value = 2;
}
```

Instead of extending the container message, the container declares a field of
type `google.protobuf.Any`. Packing a message into `Any` serializes the inner
message into `value` and writes a URI containing the inner message's fully
qualified name into `type_url`:

```cpp
media::UserContent content;
kittens::Video video;
video.set_title("Cute kittens");
content.mutable_payload()->PackFrom(video);

if (content.payload().Is<kittens::Video>()) {
  kittens::Video unpacked;
  if (content.payload().UnpackTo(&unpacked)) {
    // use unpacked
  }
}
```

## Why Proto3 Replaced Extensions with `Any` {#why-introduced}

In proto2, extensions had two real problems:

1.  There was no built-in way to coordinate extension numbers on a container
    message without importing the extensions. Because `extend` blocks lived in
    separate `.proto` files, two teams could pick the same field number on a
    shared container message and silently collide on the wire.
2.  Proto3 tried to simplify the language so that new runtimes could represent
    messages as plain [open
    structs](/design-decisions/opaque-apis) with minimal
    runtime machinery. Extensions required an `ExtensionRegistry`, extension
    storage on messages, and parameterized accessor APIs in every language.

`Any` avoided both: on the binary wire it is an ordinary message with a `string`
and a `bytes` field, its discriminator is the globally unique message name
rather than a coordinated field number, and runtime support could be limited to
`PackFrom` / `UnpackTo` helpers plus special-case handling in
[ProtoJSON](/programming-guides/json#any) and TextFormat.

The open-struct premise was abandoned (see [We Prefer Opaque
APIs](/design-decisions/opaque-apis)).

Risks of extension number collisions are still a concern with extensions, but
can be mitigated in most cases by using [extension
declarations](/programming-guides/extension_declarations).

## Why We Regret `Any` as a Replacement for Extensions {#flaws}

### Type Names on the Binary Wire Prevent Renaming and Repackaging {#type-url-lock-in}

A core property of the Protobuf binary wire format is that message names, field
names, and package names do not appear on the wire. Because the binary format
identifies fields by numeric tag, you can rename a field, rename a message, or
move a message to a different `.proto` package without breaking wire
compatibility with deployed binaries or stored data.

`Any` breaks this property. Because `PackFrom()` writes the fully qualified
message name into `type_url` on the binary wire, **any message packed into an
`Any` cannot safely be renamed or moved to a different `.proto` package**:

*   If a message is renamed or moved to a new package, `Is<T>()` and
    `UnpackTo()` compare the new descriptor's `full_name()` against the old
    `type_url` string and reject existing payloads, even though the serialized
    bytes in `value` are wire-compatible.
*   At the same time, new binaries start writing the new `type_url` string,
    which older binaries and rollback targets do not recognize.

Worse, **`Any` is not opt-in**. A message author does not declare that their
message may be packed into an `Any`. Any caller in another repository or service
can pass an arbitrary message to `PackFrom()` without the message owner's
knowledge, and because packing can happen through generic or reflective code
paths, static analysis cannot prove that a message type is never stored in an
`Any`. This is a very spooky property that makes it difficult to know if
renaming a message might not be safe.

Extensions identify fields on the wire by their numeric tag, just like regular
fields. Renaming an extension, renaming the message type it holds, or moving
either across `.proto` packages is safe on the binary wire.

### Wire Incompatibility with Regular Fields {#wire-incompatibility}

An extension has the exact same binary wire encoding as a regular field with the
same field number, type, and cardinality. That makes the boundary between
regular fields and extensions reversible:

*   If a regular field on a container message is causing unwanted build
    dependencies, it can be moved out into an `extend` block with the same field
    number and type without changing a single byte on the wire.
*   Conversely, a widely used extension can be promoted directly into a regular
    field on the container message.

`Any` wraps its payload inside a submessage containing `type_url` (field 1) and
`value` (field 2). It is wire-incompatible with a normal submessage field, so a
field cannot be migrated into or out of an `Any` without breaking the wire
format.

### Wire Size Overhead {#wire-size}

An extension tag on the wire is a single varint encoding the field number and
wire type: 2 bytes for field numbers up to 2047, and at most 5 bytes for the
largest valid field numbers.

Every `Any` message carries the full `type.googleapis.com/package.MessageName`
string on the wire in addition to the nested length prefixes. A typical
`type_url` adds 40 to 80 bytes per instance. For small messages or `repeated
google.protobuf.Any` fields, the repeated `type_url` strings often take more
space than the payload itself.

### In-Memory Cost and Eager Serialization {#serialization}

Because `Any` stores its payload as a `bytes value` field rather than as a typed
submessage in the message tree, working with `Any` in memory is clumsy and
expensive:

*   `PackFrom()` immediately serializes the inner message into a `bytes` buffer,
    even if the outer message is only passed around in memory.
*   In most languages (including C++, Go, Python, and open-source Java), calling
    `UnpackTo()` allocates a separate message object and parses `value` from
    scratch on every read, unless the caller maintains a separate cache by hand.
    By contrast, an extension is stored directly inside the parent message (and
    shares its arena in C++), so reading `GetExtension()` is a pointer lookup.

A common misconception is that `Any` is cheaper for intermediate proxies that do
not care about the payload because `value` stays unparsed until `UnpackTo()` is
called. Extensions already have this property: if a binary does not link in or
register an extension, the bytes remain unparsed in the unknown field set and
are forwarded intact on serialization. Even when a binary *does* link in the
extension, runtimes that support lazy fields defer parsing the extension until
the first `GetExtension()` call and then cache the parsed submessage in place.

### Broken `MergeFrom`, Equality, and Reflection Semantics {#semantics}

To Protobuf, an `Any` is just a two-field message (`string type_url = 1; bytes
value = 2;`), not a typed container. Standard message operations act on those
two scalar fields rather than on the packed message inside:

#### `MergeFrom` {#merging}

When two messages with a submessage extension are merged via `MergeFrom()`, the
extension submessages are merged recursively field by field, and distinct
extensions on the same container coexist.

When two `Any` messages are merged, `MergeFrom()` merges `type_url` and `value`
as ordinary scalar fields, overwriting `value` instead of merging the inner
message. Worse, because `google/protobuf/any.proto` is defined with `syntax =
"proto3"`, both `type_url` and `value` have [implicit
presence](/design-decisions/implicit-presence#merging):

1.  Suppose `A` is an `Any` packed with a non-empty `Bar` message (`type_url =
    "type.googleapis.com/Bar"`, `value = "\x08\x01"`).
2.  Suppose `B` is an `Any` packed with a default, empty `Quux` message. Because
    an all-default message serializes to 0 bytes, `B` has `type_url =
    "type.googleapis.com/Quux"` and `value = ""`.
3.  Calling `A.MergeFrom(B)` overwrites `A.type_url` with
    `"type.googleapis.com/Quux"`, but because `B.value` is empty (`""`),
    implicit presence treats `B.value` as unset and does not overwrite
    `A.value`.

The merged `Any` now has `Quux`'s `type_url` paired with `Bar`'s serialized wire
bytes.

#### Equality {#equality}

Standard message equality on `Any` compares the raw `value` byte string rather
than the logical inner message. As explained in [Serialization Is Not
Canonical](/programming-guides/serialization-not-canonical),
two logically equal Protobuf messages can serialize to different byte sequences
(for example, due to map entry ordering or unknown fields), causing two `Any`
messages holding equivalent payloads to compare unequal.

#### Reflection and Generic Traversal {#reflection}

Unless a tool links in the packed message's descriptor and writes custom
`Any`-unpacking code, reflection walkers, field-path utilities, and redaction
passes only see a `string type_url` and an opaque `bytes value`. Extensions have
normal `FieldDescriptor` entries and are visited by standard reflection walkers
without special cases.

### Poor Fit for Mobile and Lite Runtimes {#lite}

In mobile, embedded, and web client builds (`LITE_RUNTIME`), Protobuf strips
descriptors and symbol names to reduce binary size and avoid leaking unreleased
feature names in shipped client binaries.

`Any` relies on runtime type names to construct and check `type_url`:

*   In **JavaLite**, `MessageLite` does not carry descriptor names, so
    `Any.pack()`, `Any.unpack()`, and `Any.is()` are omitted from the generated
    `Any` class altogether. Callers must construct and compare `type_url`
    strings manually.
*   In **C++ Lite**, `Any::PackFrom<T>()` and `UnpackTo<T>()` only work because
    the code generator emits a `FullMessageName()` string literal on every
    message class, which forces the binary to retain the `.proto` package and
    message name for every type used with `Any`.

Extensions use numeric tags and generated extension identifiers, which work in
`LITE_RUNTIME` builds and support tree shaking without retaining descriptor
strings.

### Type URLs Do Not Provide Dynamic Schema Discovery {#type-urls}

The `type_url` field was originally formatted as a URL
(`type.googleapis.com/...`) on the theory that runtimes might fetch unknown
schemas over HTTP at parse time. Protobuf never implemented remote type fetching
and explicitly warns against contacting `type_url` endpoints in `any.proto` as a
security problem. Every parser simply strips everything up to the last `/`, and
unpacking an `Any` in JSON, TextFormat, or reflection still requires the target
descriptor to be linked or loaded into the local binary's descriptor pool, just
like an extension.

## What to Use Instead {#instead}

### Regular Submessage Fields or `oneof` {#oneof}

When the set of possible payload types is bounded, known to the container
author, and importing them directly into the container `.proto` does not cause
dependency problems, use a regular submessage field or a `oneof`. A `oneof`
makes the set of alternatives explicit in one place and generates the cleanest
API in every language.

### Extensions with Extension Declarations {#use-extensions}

When payload definitions need to live in separate `.proto` files so the
container does not depend on them, use
[extensions](/programming-guides/editions#extensions):

*   Add [extension
    declarations](/programming-guides/extension_declarations)
    to the container's `extensions` range so all allocated extension numbers and
    type names are listed in one place and checked by `protoc` without importing
    the payload `.proto` files.
*   If central code-review ownership on the container `.proto` file would be a
    bottleneck for teams adding extensions, put the extendable container (or a
    dedicated extension-carrier submessage) in a separate `.proto` file with
    broader ownership.
*   If your container message is currently in a `syntax = "proto3"` file (which
    does not allow `extensions` ranges), migrate that file to [Protobuf
    Editions](/editions/overview). Moving a file from
    `proto3` to Edition 2023 is wire-compatible, and remaining `proto3` files
    can still import it.

### When `Any` Is Still Appropriate {#when-any}

The one case where `Any` is still the right tool is **unbounded horizontal
infrastructure**: systems that transport, log, or store completely arbitrary
messages across independent domains (for example, a generic RPC error detail
list like `google.rpc.Status.details`, a generic task queue payload, or a dead
letter queue) where declaring an extension on the envelope for every message
type in the ecosystem is infeasible.

Whenever you do use `Any`, treat the package name and message name of every
packed message as a permanently frozen part of your wire protocol.
