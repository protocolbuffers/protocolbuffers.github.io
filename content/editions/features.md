+++
title = "Feature Settings for Editions"
weight = 43
description = "Protobuf Editions features and how they affect protobuf behavior."
type = "docs"
+++

This topic provides an overview of the features that are included in the
released edition versions. Subsequent editions' features will be added to this
topic. We announce new editions in
[the News section](/news).

Before configuring feature settings in your new schema definition content, make
sure you understand why you are using them. Avoid
[cargo-culting](/best-practices/no-cargo-cults) with
features.

## Prototiller {#prototiller}

Prototiller is a command-line tool that updates proto schema configuration files
between syntax versions and editions. It hasn't been released yet, but is
referenced throughout this topic.

## Features {#features}

The following sections include all of the behaviors that are configurable using
features in editions. [Preserving proto2 or proto3 Behavior](#preserving) shows
how to override the default behaviors so that your proto definition files act
like proto2 or proto3 files. For more information on how editions and features
work together to set behavior, see
[Protobuf Editions Overview](/editions/overview).

<span id="cascading"> Feature settings apply at different levels:</span>

**File-level:** These settings apply to all elements (messages, fields, enums,
and so on) that don't have an overriding setting.

**Non-nested:** Messages, enums, and services can override settings made at the
file level. They apply to everything within them (message fields, enum values)
that aren't overridden, but don't apply to other parallel messages and enums.

**Nested:** Oneofs, messages, and enums can override settings from the message
they're nested in.

**Lowest-level:** Fields, extensions, enum values, extension ranges, and methods
are the lowest level at which you can override settings.

Each of the following sections has a comment that states what scope the feature
can be applied to. The following sample shows a mock feature applied to each
scope:

```proto
edition = "2024";

// File-level scope definition
option features.bar = BAZ;

enum Foo {
  // Enum (non-nested scope) definition
  option features.bar = QUX;

  A = 1;
  B = 2;
}

message Corge {
  // Message (non-nested scope) definition
  option features.bar = QUUX;

  message Garply {
    // Message (nested scope) definition
    option features.bar = WALDO;
    string id = 1;
  }

  // Field (lowest-level scope) definition
  Foo A = 1 [features.bar = GRAULT];
}
```

In this example, the setting "`GRAULT"` in the lowest-level scope feature
definition overrides the non-nested-scope "`QUUX`" setting. And within the
Garply message, "`WALDO`" overrides "`QUUX`."

### `features.default_symbol_visibility` {#symbol-vis}

This feature enables setting the default visibility for messages and enums,
making them available or unavailable when imported by other protos. Use of this
feature will reduce dead symbols in order to create smaller binaries.

In addition to setting the defaults for the entire file, you can use the `local`
and `export` keywords to set per-field behavior. Read more about this at
[`export` / `local` Keywords](/editions/overview#export-local).

**Values available:**

*   `EXPORT_ALL`: This is the default prior to Edition 2024. All messages and
    enums are exported by default.
*   `EXPORT_TOP_LEVEL`: All top-level symbols default to export; nested default
    to local.
*   `LOCAL_ALL`: All symbols default to local.
*   `STRICT`: All symbols local by default. Nested types cannot be exported,
    except for a special-case caveat for message `{ enum {} reserved 1 to max;
    }`. This is the recommended setting for new protos.

**Applicable to the following scope:** Enum, Message

**Added in:** Edition 2024

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | ------------------
2024           | `EXPORT_TOP_LEVEL`
2023           | `EXPORT_ALL`
proto3         | `EXPORT_ALL`
proto2         | `EXPORT_ALL`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following sample shows how you can apply the feature to elements in your
proto schema definition files:

```proto
// foo.proto
edition = "2024";

// Symbol visibility defaults to EXPORT_TOP_LEVEL. Setting
// default_symbol_visibility overrides these defaults
option features.default_symbol_visibility = LOCAL_ALL;

// Top-level symbols are exported by default in Edition 2024; applying the local
// keyword overrides this
export message LocalMessage {
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

### `features.enforce_naming_style` {#enforce-naming}

Introduced in Edition 2024, this feature enables strict naming style enforcement
as defined in
[the style guide](/programming-guides/style) to ensure
protos are round-trippable by default with a feature value to opt-out to use

**Values available:**

*   `STYLE2024`: Enforces strict adherence to the style guide for naming.
*   `STYLE_LEGACY`: Applies the pre-Edition 2024 level of style guide
    enforcement.

**Applicable to the following scope:** File

**Added in:** 2024

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | --------------
2024           | `STYLE2024`
2023           | `STYLE_LEGACY`
proto3         | `STYLE_LEGACY`
proto2         | `STYLE_LEGACY`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows an Edition 2023 file:

Edition 2023 defaults to `STYLE_LEGACY`, so a non-conformant field name is fine:

```proto
edition = "2023";

message Foo {
  // A non-conforming field name is not a problem
  int64 bar_1 = 1;
}
```

Edition 2025 defaults to `STYLE2024`, so an override is needed to keep the
non-conformant field name:

```proto
edition = "2024";

// To keep the non-conformant field name, override the STYLE2024 setting
option features.enforce_naming_style = "STYLE_LEGACY";

message Foo {
  int64 bar_1 = 1;
}
```

### `features.enum_type` {#enum_type}

This feature sets the behavior for how enum values that aren't contained within
the defined set are handled. See
[Enum Behavior](/programming-guides/enum) for more
information on open and closed enums.

This feature doesn't impact proto3 files, so this section doesn't have a before
and after of a proto3 file.

**Values available:**

*   `CLOSED:` Closed enums store enum values that are out of range in the
    unknown field set.
*   `OPEN:` Open enums parse out of range values into their fields directly.

**Applicable to the following scopes:** File, Enum

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | --------
2024           | `OPEN`
2023           | `OPEN`
proto3         | `OPEN`
proto2         | `CLOSED`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

enum Foo {
  A = 2;
  B = 4;
  C = 6;
}
```

After running [Prototiller](#prototiller), the equivalent code might look like
this:

```proto
edition = "2024";

enum Foo {
  // Setting the enum_type feature overrides the default OPEN enum
  option features.enum_type = CLOSED;
  A = 2;
  B = 4;
  C = 6;
}
```

### `features.field_presence` {#field_presence}

This feature sets the behavior for tracking field presence, or the notion of
whether a protobuf field has a value.

**Values available:**

*   `LEGACY_REQUIRED`: The field is required for parsing and serialization.
    Any explicitly-set value is
    serialized onto the wire (even if it is the same as the default value).
*   `EXPLICIT`: The field has explicit presence tracking. Any explicitly-set
    value is serialized onto the wire (even if it is the same as the default
    value). For singular primitive fields, `has_*` functions are generated for
    fields set to `EXPLICIT`.
*   `IMPLICIT`: The field has no presence tracking. The default value is not
    serialized onto the wire (even if it is explicitly set). `has_*` functions
    are not generated for fields set to `IMPLICIT`.

**Applicable to the following scopes:** File, Field

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | -----------
2024           | `EXPLICIT`
2023           | `EXPLICIT`
proto3         | `IMPLICIT`*
proto2         | `EXPLICIT`

\* proto3 is `IMPLICIT` unless the field has the `optional` label, in which case
it behaves like `EXPLICIT`. See
[Presence in Proto3 APIs](/programming-guides/field_presence#presence-in-proto3-apis)
for more information.

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message Foo {
  required int32 x = 1;
  optional int32 y = 2;
  repeated int32 z = 3;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

message Foo {
  // Setting the field_presence feature retains the proto2 required behavior
  int32 x = 1 [features.field_presence = LEGACY_REQUIRED];
  int32 y = 2;
  repeated int32 z = 3;
}
```

The following shows a proto3 file:

```proto
syntax = "proto3";

message Bar {
  int32 x = 1;
  optional int32 y = 2;
  repeated int32 z = 3;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";
// Setting the file-level field_presence feature matches the proto3 implicit default
option features.field_presence = IMPLICIT;

message Bar {
  int32 x = 1;
  // Setting the field_presence here retains the explicit state that the proto3
  // field has because of the optional syntax
  int32 y = 2 [features.field_presence = EXPLICIT];
  repeated int32 z = 3;
}
```

Note that the `required` and `optional` labels no longer exist in Editions, as
the corresponding behavior is set explicitly with the `field_presence` feature.

### `features.json_format` {#json_format}

This feature sets the behavior for JSON parsing and serialization.

This feature doesn't impact proto3 files, so this section doesn't have a before
and after of a proto3 file. Editions behavior matches the behavior in proto3.

**Values available:**

*   `ALLOW`: A runtime must allow JSON parsing and serialization. Checks are
    applied at the proto level to make sure that there is a well-defined mapping
    to JSON.
*   `LEGACY_BEST_EFFORT`: A runtime does the best it can to parse and serialize
    JSON. Certain protos are allowed that can result in unspecified behavior at
    runtime (such as many:1 or 1:many mappings).

**Applicable to the following scopes:** File, Message, Enum

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | --------------------
2024           | `ALLOW`
2023           | `ALLOW`
proto3         | `ALLOW`
proto2         | `LEGACY_BEST_EFFORT`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message Foo {
  // Warning only
  string bar = 1;
  string bar_ = 2;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";
option features.json_format = LEGACY_BEST_EFFORT;

message Foo {
  string bar = 1;
  string bar_ = 2;
}
```

### `features.message_encoding` {#message_encoding}

This feature sets the behavior for encoding fields when serializing.

This feature doesn't impact proto3 files, so this section doesn't have a before
and after of a proto3 file.

Depending on the language, fields that are "group-like" may have some unexpected
capitalization in generated code and in text-format, in order to provide
backwards compatibility with proto2. Message fields are "group-like" if all of
the following conditions are met:

*   Has `DELIMITED` message encoding specified
*   Message type is defined in the same scope as the field
*   Field name is exactly the lowercased type name

**Values available:**

*   `LENGTH_PREFIXED`: Fields are encoded using the LEN wire type described in
    [Message Structure](/programming-guides/encoding#structure).
*   `DELIMITED`: Message-typed fields are encoded as
    [groups](/programming-guides/proto2#groups).

**Applicable to the following scopes:** File, Field

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | -----------------
2024           | `LENGTH_PREFIXED`
2023           | `LENGTH_PREFIXED`
proto3         | `LENGTH_PREFIXED`
proto2         | `LENGTH_PREFIXED`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message Foo {
  group Bar = 1 {
    optional int32 x = 1;
    repeated int32 y = 2;
  }
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

message Foo {
  message Bar {
    int32 x = 1;
    repeated int32 y = 2;
  }
  Bar bar = 1 [features.message_encoding = DELIMITED];
}
```

### `features.repeated_field_encoding` {#repeated_field_encoding}

This feature is what the proto2/proto3
[`packed` option](/programming-guides/encoding#packed)
for `repeated` fields has been migrated to in Editions.

**Values available:**

*   `PACKED`: `Repeated` fields of a primitive type are encoded as a single LEN
    record that contains each element concatenated.
*   `EXPANDED`: `Repeated` fields are each encoded with the field number for
    each value.

**Applicable to the following scopes:** File, Field

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | ----------
2024           | `PACKED`
2023           | `PACKED`
proto3         | `PACKED`
proto2         | `EXPANDED`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message Foo {
  repeated int32 bar = 6 [packed=true];
  repeated int32 baz = 7;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";
option features.repeated_field_encoding = EXPANDED;

message Foo {
  repeated int32 bar = 6 [features.repeated_field_encoding=PACKED];
  repeated int32 baz = 7;
}
```

The following shows a proto3 file:

```proto
syntax = "proto3";

message Foo {
  repeated int32 bar = 6;
  repeated int32 baz = 7 [packed=false];
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

message Foo {
  repeated int32 bar = 6;
  repeated int32 baz = 7 [features.repeated_field_encoding=EXPANDED];
}
```

### `features.utf8_validation` {#utf8_validation}

This feature sets how strings are validated. It applies to all languages except
where there's a language-specific `utf8_validation` feature that overrides it.
See [`features.(pb.java).utf8_validation`](#java-utf8_validation) for the
Java-language-specific feature.

This feature doesn't impact proto3 files, so this section doesn't have a before
and after of a proto3 file.

**Values available:**

*   `VERIFY`: The runtime should verify UTF-8. This is the default proto3
    behavior.
*   `NONE`: The field behaves like an unverified `bytes` field on the wire.
    Parsers may handle this type of field in an unpredictable way, such as
    replacing invalid characters. This is the default proto2 behavior.

**Applicable to the following scopes:** File, Field

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | --------
2024           | `VERIFY`
2023           | `VERIFY`
proto3         | `VERIFY`
proto2         | `NONE`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message MyMessage {
  string foo = 1;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

message MyMessage {
  string foo = 1 [features.utf8_validation = NONE];
}
```

## Language-specific Features {#lang-specific}

Some features apply to specific languages, and not to the same protos in other
languages. Using these features requires you to import the corresponding
*_features.proto file from the language's runtime. The examples in the following
sections show these imports.

### `features.(pb.cpp).enum_name_uses_string_view` {#enum-name-string-view}

**Languages:** C++

Before Edition 2024, all generated enum types provide the following function to
obtain the label out of an enum value, which has some overhead to construct the
`std::string` instances at runtime:

```cpp
const std::string& Foo_Name(int);
```

The default feature value in Edition 2024 changes this signature to return
`absl::string_view` to allow for better storage decoupling and potential
memory/CPU savings. If you aren't ready to migrate, yet, you can override this
to set it back to its previous behavior. See
[string_view return type](/support/migration#string_view-return-type)
in the migration guide for more on this topic.

**Values available:**

*   `true`: The enum uses `string_view` for its values.
*   `false`: The enum uses `std::string` for its values.

**Applicable to the following scopes:** Enum, File

**Added in:** 2024

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | -------
2024           | `true`
2023           | `false`
proto3         | `false`
proto2         | `false`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

### `features.(pb.java).large_enum` {#java-large_enum}

**Languages:** Java

This language-specific feature enables you to adopt new functionality that
handles large enums in Java without causing compiler errors. Note that this
feature replicates enum-like behavior but has some notable differences. For
example, switch statements are not supported.

**Values available:**

*   `true`: Java enums will use the new functionality.
*   `false`: Java enums will continue to use Java enums.

**Applicable to the following scopes:** Enum

**Added in:** 2024

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | -------
2024           | `false`
2023           | `false`
proto3         | `false`
proto2         | `false`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

### `features.(pb.cpp/pb.java).legacy_closed_enum` {#legacy_closed_enum}

**Languages:** C++, Java

This feature determines whether a field with an open enum type should be behave
as if it was a closed enum. This allows editions to reproduce
[non-conformant behavior](/programming-guides/enum) in
Java and C++ from proto2 and proto3.

This feature doesn't impact proto3 files, and so this section doesn't have a
before and after of a proto3 file.

**Values available:**

*   `true`: Treats the enum as closed regardless of [`enum_type`](#enum_type).
*   `false`: Respect whatever is set in the `enum_type`.

**Applicable to the following scopes:** File, Field

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | -------
2024           | `false`
2023           | `false`
proto3         | `false`
proto2         | `true`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

import "myproject/proto3file.proto";

message Msg {
  myproject.proto3file.Proto3Enum name = 1;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

import "myproject/proto3file.proto";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

message Msg {
  myproject.proto3file.Proto3Enum name = 1 [
    features.(pb.cpp).legacy_closed_enum = true,
    features.(pb.java).legacy_closed_enum = true
  ];
}
```

### `features.(pb.java).nest_in_file_class` {#java-nest_in_file}

**Languages:** Java

This feature controls whether the Java generator will nest the generated class
in the Java generated file class. Setting this option to `Yes` is the equivalent
of setting `java_multiple_files = true` in proto2/proto3/edition 2023.

The default outer classname is also updated to always be the camel-cased .proto
filename suffixed with Proto by default (for example, `foo/bar_baz.proto`
becomes `BarBazProto`). You can still override this using the
`java_outer_classname` file option and replace the pre-Edition 2024 default of
`BarBaz` or `BarBazOuterClass` depending on the presence of conflicts.

**Values available:**

*   `NO`: Do not nest the generated class in the file class.
*   `YES`: Nest the generated class in the file class.
*   Legacy: An internal value used when the `java_multiple_files` option is set.

**Applicable to the following scopes:** Message, Enum, Service

**Added in:** 2024

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | --------
2024           | `NO`
2023           | `LEGACY`
proto3         | `LEGACY`
proto2         | `LEGACY`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

### `features.(pb.cpp).string_type` {#string_type}

**Languages:** C++

This feature determines how generated code should treat string fields. This
replaces the `ctype` option from proto2 and proto3, and offers a new
`string_view` feature. In Edition 2023, you can specify either `ctype` or
`string_type` on a field, but not both. In Edition 2024, the `ctype` option is
removed.

**Values available:**

*   `VIEW`: Generates `string_view` accessors for the field. This will be the
    default in a future edition.
*   `CORD`: Generates `Cord` accessors for the field.
*   `STRING`: Generates `string` accessors for the field.

**Applicable to the following scopes:** File, Field

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | --------
2024           | `VIEW`
2023           | `STRING`
proto3         | `STRING`
proto2         | `STRING`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message Foo {
  optional string bar = 6;
  optional string baz = 7 [ctype = CORD];
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6 [features.(pb.cpp).string_type = STRING];
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

The following shows a proto3 file:

```proto
syntax = "proto3"

message Foo {
  string bar = 6;
  string baz = 7 [ctype = CORD];
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6 [features.(pb.cpp).string_type = STRING];
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

### `features.(pb.java).utf8_validation` {#java-utf8_validation}

**Languages:** Java

This language-specific feature enables you to override the file-level settings
at the field level for Java only.

This feature doesn't impact proto3 files, and so this section doesn't have a
before and after of a proto3 file.

**Values available:**

*   `DEFAULT`: The behavior matches that set by
    [`features.utf8_validation`](#utf8_validation).
*   `VERIFY`: Overrides the file-level `features.utf8_validation` setting to
    force it to `VERIFY` for Java only.

**Applicable to the following scopes:** Field, File

**Added in:** 2023

**Default behavior per syntax/edition:**

Syntax/edition | Default
-------------- | ---------
2024           | `DEFAULT`
2023           | `DEFAULT`
proto3         | `DEFAULT`
proto2         | `DEFAULT`

**Note:** Feature settings on different schema elements
[have different scopes](#cascading).

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

option java_string_check_utf8=true;

message MyMessage {
  string foo = 1;
  string bar = 2;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2024";

import "google/protobuf/java_features.proto";

option features.utf8_validation = NONE;
option features.(pb.java).utf8_validation = VERIFY;
message MyMessage {
  string foo = 1;
  string bar = 2;
}
```

## Preserving proto2 or proto3 Behavior {#preserving}

You may want to move to the editions format but not deal with updates to the way
that generated code behaves yet. This section shows the changes that the
Prototiller tool makes to your .proto files to make editions-based protos behave
like a proto2 or proto3 file.

After these changes are made at the file level, you get the proto2 or proto3
defaults. You can override at lower levels (message level, field level) to
consider additional behavior differences (such as
[required, proto3 optional](#caveats)) or if you want your definition to only be
*mostly* like proto2 or proto3.

We recommend using Prototiller unless you have a specific reason not to. To
manually apply all of these instead of using Prototiller, add the content from
the following sections to the top of your .proto file.

### Proto2 Behavior {#proto2-behavior}

The following shows the settings to replicate proto2 behavior with Edition 2023.

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

option features.field_presence = EXPLICIT;
option features.enum_type = CLOSED;
option features.repeated_field_encoding = EXPANDED;
option features.json_format = LEGACY_BEST_EFFORT;
option features.utf8_validation = NONE;
option features.(pb.cpp).legacy_closed_enum = true;
option features.(pb.java).legacy_closed_enum = true;
```

### Proto3 Behavior {#proto3-behavior}

The following shows the settings to replicate proto3 behavior with Edition 2023.

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

option features.field_presence = IMPLICIT;
option features.enum_type = OPEN;
// `packed=false` needs to be transformed to field-level repeated_field_encoding
// features in Editions syntax
option features.json_format = ALLOW;
option features.utf8_validation = VERIFY;
option features.(pb.cpp).legacy_closed_enum = false;
option features.(pb.java).legacy_closed_enum = false;
```

### Edition 2023 to 2024 {#2023-2024}

The following shows the settings to replicate Edition 2023 behavior with Edition
2024.

```proto
edition = "2024";

import option "third_party/protobuf/cpp_features.proto";
import option "third_party/java/protobuf/java_features.proto";

option features.(pb.cpp).string_type = STRING;
option features.enforce_naming_style = STYLE_LEGACY;
option features.default_symbol_visibility = EXPORT_ALL;
option features.(pb.cpp).enum_name_uses_string_view = false;
option features.(pb.java).nest_in_file_class = LEGACY;
```

### Caveats and Exceptions {#caveats}

This section shows the changes that you'll need to make manually if you choose
not to use Prototiller.

Setting the file-level defaults shown in the previous section sets the default
behaviors in most cases, but there are a few exceptions.

**Edition 2023 and later**

*   `optional`: Remove all instances of the `optional` label and change the
    [`features.field_presence`](#field_presence) to `EXPLICIT` if the file
    default is `IMPLICIT`.
*   `required`: Remove all instances of the `required` label and add the
    [`features.field_presence=LEGACY_REQUIRED`](#field_presence) option at the
    field level.
*   `groups`: Unwrap the `groups` into a separate message and add the
    `features.message_encoding = DELIMITED` option at the field level. See
    [`features.message_encoding`](#message_encoding) for more on this.
*   `java_string_check_utf8`: Remove this file option and replace it with the
    [`features.(pb.java).utf8_validation`](#java-utf8_validation). You'll need
    to import Java features, as covered in
    [Language-specific Features](#lang-specific).
*   `packed`: For proto2 files converted to editions format, remove the `packed`
    field option and add `[features.repeated_field_encoding=PACKED]` at the
    field level when you don't want the `EXPANDED` behavior that you set in
    [Proto2 Behavior](#proto2-behavior). For proto3 files converted to editions
    format, add `[features.repeated_field_encoding=EXPANDED]` at the field level
    when you don't want the default proto3 behavior.

**Edition 2024 and later**

*   (C++) `ctype`: Remove all instances of the `ctype` option and set the
    [`features.(pb.cpp).string_type`](#string_type) value.
*   (C++ and Go) `weak`: Remove weak
    importsimports.. Use
    [`import option`](/editions/overview#import-option)
    instead.
*   (Java) `java_multiple_files`: Remove `java_multiple_files` and use
    [`features.(pb.java).nest_in_file_class`](#java-nest_in_file) instead.
