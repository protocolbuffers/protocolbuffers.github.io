+++
title = "Style Guide"
weight = 50
description = "Provides direction for how best to structure your proto definitions."
type = "docs"
+++

This document provides a style guide for `.proto` files. By following these
conventions, you'll make your protocol buffer message definitions and their
corresponding classes consistent and easy to read.

Enforcement of the following style guidelines is controlled via
[enforce_naming_style](/editions/features#enforce-naming).

## Standard File Formatting {#standard-file-formatting}

*   Keep the line length to 80 characters.
*   Use an indent of 2 spaces.
*   Prefer the use of double quotes for strings.

## File Structure {#file-structure}

Files should be named `lower_snake_case.proto`.

All files should be ordered in the following manner:

1.  License header (if applicable)
1.  File overview
1.  Syntax
1.  Package
1.  Imports (sorted)
1.  File options
1.  Everything else

## Identifier naming styles {#identifier}

Protobuf identifiers use one of the following naming styles:

1.  TitleCase
    *   Contains uppercase letters, lowercase letters, and numbers
    *   The initial character is an uppercase letter
    *   The initial letter of each word is capitalized
1.  lower_snake_case
    *   Contains lowercase letters, underscores, and numbers
    *   Words are separated by a single underscore
1.  UPPER_SNAKE_CASE
    *   Contains uppercase letters, underscores, and numbers
    *   Words are separated by a single underscore
1.  camelCase
    *   Contains uppercase letters, lowercase letters, and numbers
    *   The initial character is an lowercase letter
    *   The initial letter of each subsequent word is capitalized
    *   **Note:** The style guide below does not use camelCase for any
        identifier in .proto files; the terminology is only clarified here since
        some language's generated code may transform identifiers into this
        style.

In all cases, treat abbreviations as though they are single words: use
`GetDnsRequest` rather than `GetDNSRequest`, `dns_request` rather than
`d_n_s_request`.

#### Underscores in Identifiers {#underscores}

Don't use underscores as the initial or final character of a name. Any
underscore should always be followed by a letter (not a number or a second
underscore).

The motivation for this rule is that each protobuf language implementation may
convert identifiers into the local language style: a name of `song_id` in a
.proto file may end up having accessors for the field which are capitalized as
`SongId`, `songId` or `song_id` depending on the language.

By using underscores only before letters, it avoids situations where names may
be distinct in one style, but would collide after they are transformed into one
of the other styles.

For example, both `DNS2` and `DNS_2` would both transform into TitleCase as
`Dns2`. Allowing either of those names can be lead to painful situations when a
message is used only in some languages where the generated code keeps the
original UPPER_SNAKE_CASE style, becomes widely established, and then is only
later used in a language where names are transformed to TitleCase where they
collide.

When applied, this style rule means that you should use `XYZ2` or `XYZ_V2`
rather than `XYZ_2` or `XYZ_2V`.

## Packages {#packages}

Use dot-delimited lower_snake_case names as package names.

Multi-word package names may be lower_snake_case or dot.delimited (dot-delimited
package names are emitted as nested packages/namespaces in most languages).

Package names should attempt to be a short but unique name based on the project
name. Package names should not be Java packages (`com.x.y`); instead use `x.y`
as the package and use the `java_package` option as needed.

## Message Names {#message-names}

Use TitleCase for message names.

```proto
message SongRequest {
}
```

## Field Names {#field-names}

Use snake_case for field names, including extensions.

Use pluralized names for repeated fields.

```proto
string song_name = 1;
repeated Song songs = 2;
```

## Oneof Names {#oneof-names}

Use lower_snake_case for oneof names.

```proto
oneof song_id {
  string song_human_readable_id = 1;
  int64 song_machine_id = 2;
}
```

## Enums {#enums}

Use TitleCase for enum type names.

Use UPPER_SNAKE_CASE for enum value names.

```proto
enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

The first listed value should be a zero value enum and have the suffix of either
`_UNSPECIFIED` or `_UNKNOWN`. This value may be used as an unknown/default value
and should be distinct from any of the semantic values you expect to be
explicitly set. For more information on the unspecified enum value, see
[the Proto Best Practices page](/best-practices/dos-donts#unspecified-enum).

#### Enum Value Prefixing {#enum-value-prefixing}

Enum values are semantically considered to not be scoped by their containing
enum name, so the same name in two sibling enums is not allowed. For example,
the following would be rejected by protoc since the `SET` value defined in the
two enums are considered to be in the same scope:

```proto {.bad}
enum CollectionType {
  COLLECTION_TYPE_UNSPECIFIED = 0;
  SET = 1;
  MAP = 2;
  ARRAY = 3;
}

// Won't compile - `SET` enum name will clash
// with the one defined in `CollectionType` enum.
enum TennisVictoryType {
  TENNIS_VICTORY_TYPE_UNSPECIFIED = 0;
  GAME = 1;
  SET = 2;
  MATCH = 3;
}
```

Name collisions are a high risk when enums are defined at the top level of a
file (not nested inside a message definition); in that case the siblings include
enums defined in other files that set the same package, where protoc may not be
able to detect the collision has occurred at code generation time.

To avoid these risks, it is strongly recommended to do one of:

*   Prefix every value with the enum name (converted to UPPER_SNAKE_CASE)
*   Nest the enum inside a containing message

Either option is enough to mitigate collision risks, but prefer top-level enums
with prefixed values over creating a message simply to mitigate the issue. Since
some languages don't support an enum being defined inside a "struct" type,
preferring prefixed values ensures a consistent approach across binding
languages.

## Services {#services}

Use TitleCase for service names and method names.

```proto
service FooService {
  rpc GetSomething(GetSomethingRequest) returns (GetSomethingResponse);
  rpc ListSomething(ListSomethingRequest) returns (ListSomethingResponse);
}
```

## Things to Avoid {#avoid}

### Required Fields {#required}

Required fields are a way to enforce that a given field must be set when parsing
wire bytes, and otherwise refuse to parse the message. The required invariant is
generally not enforced on messages constructed in memory. Required fields were
removed in proto3. Proto2 `required` fields that have been migrated to editions
2023 can use the `field_presence` feature set to `LEGACY_REQUIRED` to
accommodate.

While enforcement of required fields at the schema level is intuitively
desirable, one of the primary design goals of protobuf is to support long term
schema evolution. No matter how obviously required a given field seems to be
today, there is a plausible future where the field should no longer be set (e.g.
an `int64 user_id` may need to migrate to a `UserId user_id` in the future).

Especially in the case of middleware servers that may forward messages that they
don't really need to process, the semantics of `required` has proven too harmful
for those long-term evolution goals, and so is now very strongly discouraged.

See
[Required is Strongly Deprecated](/programming-guides/proto2#required-deprecated).

### Groups {#groups}

Groups is an alternate syntax and wire format for nested messages. Groups are
considered deprecated in proto2, were removed from proto3, and are converted to
a delimited representation in edition 2023. You can use a nested message
definition and field of that type instead of using the group syntax, using the
[`message_encoding`](/editions/features#message_encoding)
feature for wire-compatibility.

See [groups](/programming-guides/proto2#groups).
