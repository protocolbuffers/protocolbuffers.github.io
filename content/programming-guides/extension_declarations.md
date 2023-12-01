+++
title = "Extension Declarations"
weight = 83
description = "Describes in detail what extension declarations are, why we need them, and how we use them."
type = "docs"
+++

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'shaod' reviewed: '2023-09-06' }
*-->

## Introduction {#intro}

This page describes in detail what extension declarations are, why we need them,
and how we use them.

**NOTE:** Extension declarations are mostly used in proto2, as proto3 does not
support extensions at this time (except for
[declaring custom options](/programming-guides/proto3/#customoptions)).

If you need an introduction to extensions, read this
[extensions guide](https://protobuf.dev/programming-guides/proto2/#extensions)

## Motivation {#motivation}

Extension declarations aim to strike a happy medium between regular fields and
extensions. Like extensions, they avoid creating a dependency on the message
type of the field, which therefore results in a leaner build graph and smaller
binaries in environments where unused messages are difficult or impossible to
strip. Like regular fields, the field name/number appear in the enclosing
message, which makes it easier to avoid conflicts and see a convenient listing
of what fields are declared.

Listing the occupied extension numbers with extension declarations makes it
easier for users to pick an available extension number and to avoid conflicts.

## Usage {#usage}

Extension declarations are an option of extension ranges. Like forward
declarations in C++, you can declare the field type, field name, and cardinality
(singular or repeated) of an extension field without importing the `.proto` file
containing the full extension definition:

```proto
syntax = "proto2";

message Foo {
  extensions 4 to 1000 [
    declaration = {
      number: 4,
      full_name: ".my.package.event_annotations",
      type: ".logs.proto.ValidationAnnotations",
      repeated: true },
    declaration = {
      number: 999,
      full_name: ".foo.package.bar",
      type: "int32"}];
}
```

This syntax has the following semantics:

*   Multiple `declaration`s with distinct extension numbers can be defined in a
    single extension range if the size of the range allows.
*   If there is any declaration for the extension range, *all* extensions of the
    range must also be declared. This prevents non-declared extensions from
    being added, and enforces that any new extensions use declarations for the
    range.
*   The given message type (`.logs.proto.ValidationAnnotations`) does not need
    to have been previously defined or imported. We check only that it is a
    valid name that could potentially be defined in another `.proto` file.
*   When this or another `.proto` file defines an extension of this message
    (`Foo`) with this name or number, we enforce that the number, type, and full
    name of the extension match what is forward-declared here.

**WARNING:** Avoid using declarations for extension range groups such as
`extensions 4, 999`. It is unclear which extension range the declarations apply
to, and it is currently unsupported.

The extension declarations expect two extension fields with different packages:

```proto
package my.package;
extend Foo {
  repeated logs.proto.ValidationAnnotations event_annotations = 4;
}
```

```proto
package foo.package;
extend Foo {
  optional int32 bar = 999;
}
```

### Reserved Declarations {#reserved}

An extension declaration can be marked `reserved: true` to indicate that it is
no longer actively used and the extension definition has been deleted. **Do not
delete the extension declaration or edit its `type` or `full_name` value**.

This `reserved` tag is separate from the reserved keyword for regular fields and
**does not require breaking up the extension range**.

```proto {highlight="context:reserved"}
syntax = "proto2";

message Foo {
  extensions 4 to 1000 [
    declaration = {
      number: 500,
      full_name: ".my.package.event_annotations",
      type: ".logs.proto.ValidationAnnotations",
      reserved: true }];
}
```

An extension field definition using a number that is `reserved` in the
declaration will fail to compile.

## Representation in descriptor.proto {#representation}

Extension declaration is  represented in descriptor.proto as fields in
`proto2.ExtensionRangeOptions`:

```proto
message ExtensionRangeOptions {
  message Declaration {
    optional int32 number = 1;
    optional string full_name = 2;
    optional string type = 3;
    optional bool reserved = 5;
    optional bool repeated = 6;
  }
  repeated Declaration declaration = 2;
}
```

## Reflection Field Lookup {#reflection}

Extension declarations are *not* returned from the normal field lookup functions
like `Descriptor::FindFieldByName()` or `Descriptor::FindFieldByNumber()`. Like
extensions, they are discoverable by extension lookup routines like
`DescriptorPool::FindExtensionByName()`. This is an explicit choice that
reflects the fact that declarations are not definitions and do not have enough
information to return a full `FieldDescriptor`.

Declared extensions still behave like regular extensions from the perspective of
TextFormat and JSON. It also means that migrating an existing field to a
declared extension will require first migrating any reflective use of that
field.

## Use Extension Declarations to Allocate Numbers {#recommendation}

Extensions use field numbers just like ordinary fields do, so it is important
for each extension to be assigned a number that is unique within the parent
message. We recommend using extension
declarations to declare the field
number and type for each extension in the parent message. The extension
declarations serve as a registry of all the parent message's extensions, and
protoc will enforce that there are no field number conflicts. When you add a new
extension, choose the number by just incrementing by one the previously added
extension number. Whenever you delete an extension, make sure to mark the field
number `reserved` to eliminate the risk of accidentally reusing
it.

This convention is only a recommendation--the protobuf team does not have the
ability or desire to force anyone to adhere to it for every extendable message.
If you as the owner of an extendable proto do not want to coordinate extension
numbers through extension declarations, you can choose to provide coordination
through other means. Be very careful, though,
because accidental reuse of an extension number can cause serious problems.

One way to sidestep the issue would be to avoid extensions entirely and use
[`google.protobuf.Any`](/programming-guides/proto3/#any)
instead. This could be a good choice for APIs that front storage or for
pass-through systems where the client cares about the contents of the proto but
the system receiving it does not.

### Consequences of Reusing an Extension Number {#reusing}

An extension is a field defined outside the container message; usually in a
separate .proto file. This distribution of definitions makes it easy for two
developers to accidentally create different definitions for the same extension
field number.

The consequences of changing an extension definition are the same for extensions
and standard fields. Reusing a field number introduces an ambiguity in how a
proto should be decoded from the wire format. The protobuf wire format is lean
and doesn’t provide a good way to detect fields encoded using one definition and
decoded using another.

This ambiguity can manifest in a short time frame, such as a client using one
extension definition and a server using another communicating
.

This ambiguity can also manifest over a longer time frame, such as storing data
encoded using one extension definition and later retrieving and decoding using
the second extension definition. This long-term case can be difficult to
diagnose if the first extension definition was deleted after the data was
encoded and stored.

The outcome of this can be:

1.  A parse error (best case scenario).
2.  Leaked PII / SPII – if PII or SPII is written using one extension definition
    and read using another extension definition.
3.  Data Corruption – if data is read using the “wrong” definition, modified and
    rewritten.

Data definition ambiguity is almost certainly going to cost someone time for
debugging at a minimum. It could also cause data leaks or corruption that takes
months to clean up.

## Usage Tips

### Never Delete an Extension Declaration {#never-delete}

Deleting an extension declaration opens the door to accidental reuse in the
future. If the extension is no longer processed and the definition is deleted,
the extension declaration can be [marked reserved](#reserved).

### Never Use a Field Name or Number from the `reserved` List for a New Extension Declaration {#never-reuse-reserved}

Reserved numbers may have been used for fields or other extensions in the past.

Using the `full_name` of a reserved field
is not recommended
due
to the possibility of ambiguity when using textproto.

### Never change the type of an existing extension declaration {#never-change-type}

Changing the extension field's type can result in data corruption.

If the extension field is of an enum or message type, and that enum or message
type is being renamed, updating the declaration name is required and safe. To
avoid breakages, the update of the type, the extension field definition, and
extension declaration should all happen in a single
commit.

### Use Caution When Renaming an Extension Field {#caution-renaming}

While renaming an extension field is fine for the wire format, it can break JSON
and TextFormat parsing.
