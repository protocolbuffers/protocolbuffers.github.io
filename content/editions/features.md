+++
title = "Feature Settings for Editions"
weight = 43
description = "Protobuf Editions features and how they affect protobuf behavior."
type = "docs"
+++

This topic provides an overview of the features that are included in Edition
2023. Each subsequent edition's features will be added to this topic. We
announce new editions in [the News section](/news).

## Prototiller {#prototiller}

Prototiller is a command-line tool that converts proto2 and proto3 definition
files to Editions syntax. It hasn't been released yet, but is referenced
throughout this topic.

## Features {#features}

The following sections include all of the behaviors that are configurable using
features in Edition 2023. [Preserving proto2 or proto3 Behavior](#preserving)
shows how to override the default behaviors so that your proto definition files
act like proto2 or proto3 files. For more information on how Editions and
Features work together to set behavior, see
[Protobuf Editions Overview](/editions/overview).

Each of the following sections has an entry for what scope the feature applies
to. This can include file, enum, message, or field. The following sample shows a
mock feature applied to each scope:

```proto
edition = "2023";

// File-scope definition
option features.bar = BAZ;

enum Foo {
  // Enum-scope definition
  option features.bar = QUX;

  A = 1;
  B = 2;
}

message Corge {
  // Message-scope definition
  option features.bar = QUUX;

  // Field-scope definition
  Foo A = 1 [features.bar = GRAULT];
}
```

In this example, the setting `GRAULT` in the field-scope feature definition
overrides the message-scope QUUX setting.

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

**Default behavior in Edition 2023:** `OPEN`

**Behavior in proto2:** `CLOSED`

**Behavior in proto3:** `OPEN`

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
edition = "2023";

enum Foo {
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

**Default value in the Edition 2023:** `EXPLICIT`

**Behavior in proto2:** `EXPLICIT`

**Behavior in proto3:** `IMPLICIT` unless the field has the `optional` label, in
which case it behaves like `EXPLICIT`. See
[Presence in Proto3 APIs](/programming-guides/field_presence#presence-in-proto3-apis)
for more information.

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
edition = "2023";

message Foo {
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
edition = "2023";
option features.field_presence = IMPLICIT;

message Bar {
  int32 x = 1;
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

**Default behavior in Edition 2023:** `ALLOW`

**Behavior in proto2:** `LEGACY_BEST_EFFORT`

**Behavior in proto3:** `ALLOW`

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
edition = "2023";
features.json_format = LEGACY_BEST_EFFORT;

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

**Default behavior in Edition 2023:** `LENGTH_PREFIXED`

**Behavior in proto2:** `LENGTH_PREFIXED` except for groups, which default to
`DELIMITED`

**Behavior in proto3:** `LENGTH_PREFIXED`. Proto3 doesn't support `DELIMITED`.

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
edition = "2023";

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

**Default behavior in Edition 2023:** `PACKED`

**Behavior in proto2:** `EXPANDED`

**Behavior in proto3:** `PACKED`

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
edition = "2023";
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
edition = "2023";

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

**Default behavior in Edition 2023:** `VERIFY`

**Behavior in proto2:** `NONE`

**Behavior in proto3:** `VERIFY`

The following code sample shows a proto2 file:

```proto
syntax = "proto2";

message MyMessage {
  string foo = 1;
}
```

After running Prototiller, the equivalent code might look like this:

```proto
edition = "2023";

message MyMessage {
  string foo = 1 [features.utf8_validation = NONE];
}
```

### Language-specific Features {#lang-specific}

Some features apply to specific languages, and not to the same protos in other
languages. Using these features requires you to import the corresponding
*_features.proto file from the language's runtime. The examples in the following
sections show these imports.

#### `features.(pb.cpp/pb.java).legacy_closed_enum` {#legacy_closed_enum}

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

**Default behavior in Edition 2023:** `false`

**Behavior in proto2:** `true`

**Behavior in proto3:** `false`

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
edition = "2023";

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

#### `features.(pb.cpp).string_type` {#string_type}

**Languages:** C++

This feature determines how generated code should treat string fields. This
replaces the `ctype` option from proto2 and proto3, and offers a new
`string_view` feature. In Edition 2023, either `ctype` or `string_view` may be
specified on a field, but not both.

**Values available:**

*   `VIEW`: Generates `string_view` accessors for the field. This will be the
    default in a future edition.
*   `CORD`: Generates `Cord` accessors for the field.
*   `STRING`: Generates `string` accessors for the field.

**Applicable to the following scopes:** File, Field

**Default behavior in Edition 2023:** `STRING`

**Behavior in proto2:** `STRING`

**Behavior in proto3:** `STRING`

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
edition = "2023";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6;
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
edition = "2023";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6;
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

#### `features.(pb.java).utf8_validation` {#java-utf8_validation}

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

**Default behavior in Edition 2023:** `DEFAULT`

**Behavior in proto2:** `DEFAULT`

**Behavior in proto3:** `DEFAULT`

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
edition = "2023";

import "google/protobuf/java_features.proto";

option features.utf8_validation = NONE;
option features.(pb.java).utf8_validation = VERIFY;
message MyMessage {
  string foo = 1;
}
```

## Preserving proto2 or proto3 Behavior {#preserving}

You may want to move to the editions format but not deal with updates to the way
that generated code behaves yet. This section shows the changes that the
Prototiller tool makes to your .proto files to make Edition 2023 protos behave
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

```proto
// proto3 behaviors
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

### Caveats and Exceptions {#caveats}

This section shows the changes that you'll need to make manually if you choose
not to use Prototiller.

Setting the file-level defaults shown in the previous section sets the default
behaviors in most cases, but there are a few exceptions.

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
