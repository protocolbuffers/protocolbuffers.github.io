+++
title = "Symbol Visibility"
weight = 85
linkTitle = "Symbol Visibility"
description = "Explains the terminology and functionality of visibility, introduced in edition 2024."
type = "docs"
+++

This document describes the terminology and functionality of the symbol
visibility system introduced in proto `edition = "2024"`

## Glossary {#glossary}

*   **Symbol**: Any of `message`, `enum`, `service` or `extend <type>`, the
    allowed **Top-Level** types in a `.proto` file.
*   **Top-Level**: A **Symbol** defined at the root of a `.proto` file. This
    includes all `service` definitions and any `message`, `enum`, or `extend`
    block not nested in `message`.
*   **Visibility**: Property of a **Symbol** that controls whether it can be
    imported into another `.proto` file. Either set explicitly or derived from
    file defaults.
*   **Entry-Point**: A **Symbol** in a `.proto` file, which acts as the root of
    a local sub-graph of **Symbol**s in that file, through which either
    generated code or other `.proto` files "enter" the file.
*   **Symbol Waste**: For a given **Symbol** 'X', the set of waste **Symbol**s
    are those unreachable from 'X', but defined in files which are transitively
    imported to satisfy the processing of the file in which 'X' is defined.

## Introduction {#introduction}

Visibility in Edition 2024 and later provides a way for authors to use access
modifiers for `message` and `enum` declarations. The visibility of a symbol
controls the ability to reference that symbol from another `.proto` file via
`import`. When a symbol is not visible outside of a given file, then references
to it from another `.proto` file will fail with a compiler error.

This feature allows authors to control access to their symbols from external
users and encourages smaller files, which may lead to a smaller code footprint
when only a subset of defined symbols are required.

Visibility applies to any `message` or `enum` referenced via:

*   `message` and `extend` field definitions.
*   `extend <symbol>`
*   `service` method request and response types.

Symbol visibility applies **only** to the proto language, controlling the proto
compiler's ability to reference that symbol from another proto file. Visibility
must not be reflected into any language-specific generated code.

## Detailed Usage {#syntax}

Visibility introduces a set of new file-level options and two new Protobuf
keywords, `local` and `export`, which can be prefixed to `message` and `enum`
types.

```proto
local message MyLocal {...}

export enum MyExported {...}
```

Each `.proto` file also has a default visibility controlled by the edition's
defaults or file-level `option features.default_symbol_visibility`. This
visibility impacts all `message` and `enum` definitions in the file.

**Values available:**

*   `EXPORT_ALL`: This is the default prior to Edition 2024. All messages and
    enums are exported by default.
*   `EXPORT_TOP_LEVEL`: All top-level symbols default to `export`; nested
    default to `local`.
*   `LOCAL_ALL`: All symbols default to `local`.
*   `STRICT`: All symbols default to `local` and visibility keywords are only
    valid on top-level `message` and `enum` types. Nested types can no longer
    use those keywords are are always treated as `local`

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | ------------------
2024           | `EXPORT_TOP_LEVEL`
2023           | `EXPORT_ALL`
proto3         | `EXPORT_ALL`
proto2         | `EXPORT_ALL`

Any top-level `message` and `enum` definitions can be annotated with explicit
`local` or `export` keywords to indicate their intended use. Symbols nested in
`message` may allow those keywords.

Example:

```proto
// foo.proto
edition = "2024";

// Symbol visibility defaults to EXPORT_TOP_LEVEL. Setting
// default_symbol_visibility overrides these defaults
option features.default_symbol_visibility = LOCAL_ALL;

// Top-level symbols are exported by default in Edition 2024; applying the local
// keyword overrides this
local message LocalMessage {
  int32 baz = 1;
  // Nested symbols are local by default in Edition 2024; applying the export
  // keyword overrides this
  enum ExportedNestedEnum {
    UNKNOWN_EXPORTED_NESTED_ENUM_VALUE = 0;
  }
}

// bar.proto
edition = "2024";

import "foo.proto";

message ImportedMessage {
  // The following is valid because the imported message explicitly overrides
  // the visibility setting in foo.proto
  LocalMessage bar = 1;

  // The following is not valid because default_symbol_visibility is set to
  // `LOCAL_ALL`
  // LocalMessage.ExportedNestedEnum qux = 2;
}
```

### STRICT default_symbol_visibility {#strict}

When `default_symbol_visibility` is set to `STRICT`, more restrictive visibility
rules are applied to the file. This mode is intended as a more optimal, but
invasive type of visibility where nested types are not to be used outside their
own file. In `STRICT` mode:

1.  All symbols default to `local`.
2.  `local` and `export` may be used only on top-level `message` and `enum`
    declarations.
3.  `local` and `export` keywords on nested symbols will result in a syntax
    error.

A single carve-out exception to nested visibility keywords is made for specific
wrapper `message` types used to address C++ namespace pollution. In this case a
`export enum` is supported iff:

1.  The top-level `message` is `local`
2.  All fields are `reserved`, preventing any field definitions, using `reserved
    1 to max;`

Example:

```proto
local message MyNamespaceMessage {
  export enum Enum {
    MY_VAL = 1;
  }

  // Ensure no fields are added to the message.
  reserved 1 to max;
}
```

## Purpose of Visibility {#purpose}

There are two purposes for the visibility feature. The first introduces access
modifiers, a common feature of many popular programming languages, used to
communicate and enforce the intent of authors about the usage of a given API.
Visibility allows protos to have a limited set of proto-specific access control,
giving proto authors some additional controls about the parts of their API that
can be reused and defining the API's entry-points.

The second purpose is to encourage limiting the scope of a `.proto` file to a
narrow set of definitions to reduce the need to process unnecessary definitions.
The protobuf language definition requires that `Descriptor` type data be bundled
at the `FileDescriptorSet` and `FileDescriptor` level. This means that all
definitions in a single file, and its transitive dependencies via imports, are
unconditionally processed whenever that `.proto` file is processed. In large
definition sets, this can become a significant source of large unused blocks of
definitions that both slow down processing and generate a large set of unused
generated code. The best way to combat this sort of anti-pattern is to keep
`.proto` files narrowly scoped, containing only the symbols needed for a given
sub-graph located in the same file. With visibility added to `message` and
`enum` types, we can enumerate the set of entry-points in a given file and
determine where unrelated definitions can be split into different files to
provide fine-grained dependencies.

## Entry-Points {#entry-point}

A `.proto` file entry-point is any symbol that acts as a starting point for a
sub-graph of the full transitive closure of proto symbols processed for a given
file. That sub-graph represents the full transitive closure of types required
for the definition of that entry-point symbol, which is a subset of the symbols
required to process the file in which the entry-point is defined.

There are generally 3 types of entry-points.

### Simple-type Entry-Point {#simple-entry-point}

`message` and `enum` types that have a visibility of `export` both can act as
entry-points in a symbol graph. In the case of `enum`, that sub-graph is the
null set, which means any other type definitions in the same file as that
`export enum Type {}` can be considered 'waste' in reference to the symbol graph
required for importing the enum.

For `message` types the message acts as an entry-point for the full transitive
closure of types for the `message`'s field definitions and the recursive set of
field definitions of any sub-messages referenced by the entry-point.

Importantly for both `enum` and `message` any `service` or `extend` definitions
in the same file are unreferencable and can be considered 'waste' for users of
those `message` and `enum` definitions.

When a `.proto` file contains multiple `local` messages with no shared
dependencies, it is possible they can all act as independent entry-points when
used from generated code. The Protobuf compiler and static analysis system have
no way to determine the optimal entry-point layout of such a file. Without that
context we assume that `local` messages in the same file are optimal for the
intended generated code use-cases.

### Service Entry-Points {#service-entry-point}

`service` definitions act as an entry-point for all symbols referenced
transitively from that `service`, which includes the full transitive closure of
`message` and `enum` types for all `rpc` method definitions.

Service definitions cannot be referenced in any meaningful way from another
proto, which means when a `.proto` file is imported and it contains a `service`
definition, it and its transitive closure of method types can be considered a
waste. This is true even if all `method` input and output types are referenced
from the importing file.

### Extension Entry-Points {#extend-entry-point}

`extend <symbol>` is an Extend type entry-point. Similar to `service` there is
no way to reference an extension field from a `message` or `enum`. An extension
requires `import`-ing both the Extendee type for an `extend ExtendeeType` and
the Extender type for `ExtenderType my_ext = ...`, which requires depending on
the full transitive closure of symbols required for both `ExtendeeType` and
`ExtenderType`. The symbols for `ExtendeeType` can be considered waste to any
other entry-points in the same file as the `extend`.

Extensions exist to decouple the Extender type from the Extendee type, however
when extensions are defined in the same file as other entry-points this creates
a dependency inversion, forcing the other entry-points in the same file to
depend on the Extendee type.

This dependency inversion can be harmless when the Extendee type is trivial,
containing no imports or non-primitive types, but can also be a source of
tremendous waste if the Extendee has a large transitive dependency set.

In some generated code, nesting extensions inside of a `message` provides a
useful namespace. This can be accomplished safely with the use of visibility
keywords in one of two ways. Example:

```proto
import "expensive/extendee_type.proto";

// You can define a local message for your extension.
local message ExtenderType {
  extend expensive.ExtendeeType {
    ExtenderType ext = 12345;
  }

  string one = 1;
  int two = 2;
}
```

Alternatively, if you have a symbol you both want to use as an extension field
and have that same type be used as a normal field in another messages, then that
message should be defined in its own file and the `extend` defined in its own
isolated file. You can still provide a friendly namespace, which is especially
useful for languages like C++, for the extension with a `local` wrapper message
with no fields:

```proto
package my.pkg;

import "expensive/extendee_type.proto";
import "other/foo_type.proto";

// Exclusively used to bind other.FooType as an extension to
// expensive.ExtendeeType giving it a useful namespaced name
for `ext` as `my.pkg.WrapperForFooMsg.ext`
local message WrapperForFooMsg {
  extend expensive.ExtendeeType {
    other.FooMsg ext = 45678;
  }

  // reserve all fields to ensure nothing ever uses this except for wrapping the
  // extension.
  reserved 1 to max;
}

```

### Best-Practice: Maintain 1 Entry-Point per File

In general to avoid proto symbol waste striving for a 1:1 ratio between `.proto`
files and entry-points can ensure that no excess proto compiler processing or
code generation is being performed for any given symbol. This can be further
extended to build systems by ensuring that a build step is only processing a
single `.proto` file at a time. In Bazel, this would correspond to having only a
single `.proto` file per `proto_library` rule.
