+++
title = "Language Guide (editions)"
weight = 40
description = "Covers how to use the edition 2023 revision of the Protocol Buffers language in your project."
type = "docs"
+++

This guide describes how to use the protocol buffer language to structure your
protocol buffer data, including `.proto` file syntax and how to generate data
access classes from your `.proto` files. It covers **edition 2023** of the
protocol buffers language. For information about how editions differ from proto2
and proto3 conceptually, see
[Protobuf Editions Overview](/editions/overview).

For information on the **proto2** syntax, see the
[Proto2 Language Guide](/programming-guides/proto2).

For information on **proto3** syntax, see the
[Proto3 Language Guide](/programming-guides/proto3).

This is a reference guide – for a step by step example that uses many of the
features described in this document, see the
[tutorial](/getting-started)
for your chosen language.

## Defining A Message Type {#simple}

First let's look at a very simple example. Let's say you want to define a search
request message format, where each search request has a query string, the
particular page of results you are interested in, and a number of results per
page. Here's the `.proto` file you use to define the message type.

```proto
edition = "2023";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
```

*   The first line of the file specifies that you're using edition 2023 of the
    protobuf language spec.

    *   The `edition` (or `syntax` for proto2/proto3) must be the first
        non-empty, non-comment line of the file.
    *   If no `edition` or `syntax` is specified, the protocol buffer compiler
        will assume you are using
        [proto2](/programming-guides/proto2).

*   The `SearchRequest` message definition specifies three fields (name/value
    pairs), one for each piece of data that you want to include in this type of
    message. Each field has a name and a type.

### Specifying Field Types {#specifying-types}

In the earlier example, all the fields are [scalar types](#scalar): two integers
(`page_number` and `results_per_page`) and a string (`query`). You can also
specify [enumerations](#enum) and composite types like other message types for
your field.

### Assigning Field Numbers {#assigning}

You must give each field in your message definition a number between `1` and
`536,870,911` with the following restrictions:

-   The given number **must be unique** among all fields for that message.
-   Field numbers `19,000` to `19,999` are reserved for the Protocol Buffers
    implementation. The protocol buffer compiler will complain if you use one of
    these reserved field numbers in your message.
-   You cannot use any previously [reserved](#fieldreserved) field numbers or
    any field numbers that have been allocated to [extensions](#extensions).

This number **cannot be changed once your message type is in use** because it
identifies the field in the
[message wire format](/programming-guides/encoding).
"Changing" a field number is equivalent to deleting that field and creating a
new field with the same type but a new number. See [Deleting Fields](#deleting)
for how to do this properly.

Field numbers **should never be reused**. Never take a field number out of the
[reserved](#fieldreserved) list for reuse with a new field definition. See
[Consequences of Reusing Field Numbers](#consequences).

You should use the field numbers 1 through 15 for the most-frequently-set
fields. Lower field number values take less space in the wire format. For
example, field numbers in the range 1 through 15 take one byte to encode. Field
numbers in the range 16 through 2047 take two bytes. You can find out more about
this in
[Protocol Buffer Encoding](/programming-guides/encoding#structure).

#### Consequences of Reusing Field Numbers {#consequences}

Reusing a field number makes decoding wire-format messages ambiguous.

The protobuf wire format is lean and doesn't provide a way to detect fields
encoded using one definition and decoded using another.

Encoding a field using one definition and then decoding that same field with a
different definition can lead to:

-   Developer time lost to debugging
-   A parse/merge error (best case scenario)
-   Leaked PII/SPII
-   Data corruption

Common causes of field number reuse:

-   renumbering fields (sometimes done to achieve a more aesthetically pleasing
    number order for fields). Renumbering effectively deletes and re-adds all
    the fields involved in the renumbering, resulting in incompatible
    wire-format changes.
-   deleting a field and not [reserving](#fieldreserved) the number to prevent
    future reuse.

    -   This has been a very easy mistake to make with
        [extension fields](#extensions) for several reasons.
        [Extension Declarations](/programming-guides/extension_declarations)
        provide a mechanism for reserving extension fields.

The field number is limited to 29 bits rather than 32 bits because three bits
are used to specify the field's wire format. For more on this, see the
[Encoding topic](/programming-guides/encoding#structure).

<a id="specifying-field-rules"></a>

### Specifying Field Cardinality {#field-labels}

Message fields can be one of the following:

*   *Singular*:

    A singular field has no explicit cardinality label. It has two possible
    states:

    *   the field is set, and contains a value that was explicitly set or parsed
        from the wire. It will be serialized to the wire.
    *   the field is unset, and will return the default value. It will not be
        serialized to the wire.

    You can check to see if the value was explicitly set.

    Proto3 *implicit* fields that have been migrated to editions will use the
    `field_presence` feature set to the `IMPLICIT` value.

    Proto2 `required` fields that have been migrated to editions will also use
    the `field_presence` feature, but set to `LEGACY_REQUIRED`.

*   `repeated`: this field type can be repeated zero or more times in a
    well-formed message. The order of the repeated values will be preserved.

*   `map`: this is a paired key/value field type. See
    [Maps](/programming-guides/encoding#maps) for more on
    this field type.

#### Repeated Fields are Packed by Default {#use-packed}

In proto editions, `repeated` fields of scalar numeric types use `packed`
encoding by default.

You can find out more about `packed` encoding in
[Protocol Buffer Encoding](/programming-guides/encoding#packed).

#### Well-formed Messages {#well-formed}

The term "well-formed," when applied to protobuf messages, refers to the bytes
serialized/deserialized. The protoc parser validates that a given proto
definition file is parseable.

Singular fields can appear more than once in wire-format bytes. The parser will
accept the input, but only the last instance of that field will be accessible
through the generated bindings. See
[Last One Wins](/programming-guides/encoding#last-one-wins)
for more on this topic.

### Adding More Message Types {#adding-types}

Multiple message types can be defined in a single `.proto` file. This is useful
if you are defining multiple related messages – so, for example, if you wanted
to define the reply message format that corresponds to your `SearchResponse`
message type, you could add it to the same `.proto`:

```proto
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}

message SearchResponse {
 ...
}
```

**Combining Messages leads to bloat** While multiple message types (such as
message, enum, and service) can be defined in a single `.proto` file, it can
also lead to dependency bloat when large numbers of messages with varying
dependencies are defined in a single file. It's recommended to include as few
message types per `.proto` file as possible.

### Adding Comments {#adding-comments}

To add comments to your `.proto` files:

*   Prefer C/C++/Java line-end-style comments '//' on the line before the .proto
    code element
*   C-style inline/multi-line comments `/* ... */` are also accepted.

    *   When using multi-line comments, a margin line of '*' is preferred.

```proto
/**
 * SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response.
 */
message SearchRequest {
  string query = 1;

  // Which page number do we want?
  int32 page_number = 2;

  // Number of results to return per page.
  int32 results_per_page = 3;
}
```

### Deleting Fields {#deleting}

Deleting fields can cause serious problems if not done properly.

When you no longer need a field and all references have been deleted from client
code, you may delete the field definition from the message. However, you
**must** [reserve the deleted field number](#fieldreserved). If you do not
reserve the field number, it is possible for a developer to reuse that number in
the future.

You should also reserve the field name to allow JSON and TextFormat encodings of
your message to continue to parse.

<a id="fieldreserved"></a>

#### Reserved Field Numbers {#reserved-field-numbers}

If you [update](#updating) a message type by entirely deleting a field, or
commenting it out, future developers can reuse the field number when making
their own updates to the type. This can cause severe issues, as described in
[Consequences of Reusing Field Numbers](#consequences). To make sure this
doesn't happen, add your deleted field number to the `reserved` list.

The protoc compiler will generate error messages if any future developers try to
use these reserved field numbers.

```proto
message Foo {
  reserved 2, 15, 9 to 11;
}
```

Reserved field number ranges are inclusive (`9 to 11` is the same as `9, 10,
11`).

#### Reserved Field Names {#reserved-field-names}

Reusing an old field name later is generally safe, except when using TextProto
or JSON encodings where the field name is serialized. To avoid this risk, you
can add the deleted field name to the `reserved` list.

Reserved names affect only the protoc compiler behavior and not runtime
behavior, with one exception: TextProto implementations may discard unknown
fields (without raising an error like with other unknown fields) with reserved
names at parse time (only the C++ and Go implementations do so today). Runtime
JSON parsing is not affected by reserved names.

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved foo, bar;
}
```

Note that you can't mix field names and field numbers in the same `reserved`
statement.

### What's Generated from Your `.proto`? {#generated}

When you run the [protocol buffer compiler](#generating) on a `.proto`, the
compiler generates the code in your chosen language you'll need to work with the
message types you've described in the file, including getting and setting field
values, serializing your messages to an output stream, and parsing your messages
from an input stream.

*   For **C++**, the compiler generates a `.h` and `.cc` file from each
    `.proto`, with a class for each message type described in your file.
*   For **Java**, the compiler generates a `.java` file with a class for each
    message type, as well as a special `Builder` class for creating message
    class instances.
*   For **Kotlin**, in addition to the Java generated code, the compiler
    generates a `.kt` file for each message type with an improved Kotlin API.
    This includes a DSL that simplifies creating message instances, a nullable
    field accessor, and a copy function.
*   **Python** is a little different — the Python compiler generates a module
    with a static descriptor of each message type in your `.proto`, which is
    then used with a *metaclass* to create the necessary Python data access
    class at runtime.
*   For **Go**, the compiler generates a `.pb.go` file with a type for each
    message type in your file.
*   For **Ruby**, the compiler generates a `.rb` file with a Ruby module
    containing your message types.
*   For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file
    from each `.proto`, with a class for each message type described in your
    file.
*   For **C#**, the compiler generates a `.cs` file from each `.proto`, with a
    class for each message type described in your file.
*   For **PHP**, the compiler generates a `.php` message file for each message
    type described in your file, and a `.php` metadata file for each `.proto`
    file you compile. The metadata file is used to load the valid message types
    into the descriptor pool.
*   For **Dart**, the compiler generates a `.pb.dart` file with a class for each
    message type in your file.

You can find out more about using the APIs for each language by following the
tutorial for your chosen language. For even more API
details, see the relevant [API reference](/reference/).

## Scalar Value Types {#scalar}

A scalar message field can have one of the following types – the table shows the
type specified in the `.proto` file, and the corresponding type in the
automatically generated class:

<div>
  <table>
    <tbody>
      <tr>
        <th>Proto Type</th>
        <th>Notes</th>
      </tr>
      <tr>
        <td>double</td>
        <td></td>
      </tr>
      <tr>
        <td>float</td>
        <td></td>
      </tr>
      <tr>
        <td>int32</td>
        <td>Uses variable-length encoding. Inefficient for encoding negative
        numbers – if your field is likely to have negative values, use sint32
        instead.</td>
      </tr>
      <tr>
        <td>int64</td>
        <td>Uses variable-length encoding. Inefficient for encoding negative
        numbers – if your field is likely to have negative values, use sint64
        instead.</td>
      </tr>
      <tr>
        <td>uint32</td>
        <td>Uses variable-length encoding.</td>
      </tr>
      <tr>
        <td>uint64</td>
        <td>Uses variable-length encoding.</td>
      </tr>
      <tr>
        <td>sint32</td>
        <td>Uses variable-length encoding. Signed int value. These more
        efficiently encode negative numbers than regular int32s.</td>
      </tr>
      <tr>
        <td>sint64</td>
        <td>Uses variable-length encoding. Signed int value. These more
        efficiently encode negative numbers than regular int64s.</td>
      </tr>
      <tr>
        <td>fixed32</td>
        <td>Always four bytes. More efficient than uint32 if values are often
        greater than 2<sup>28</sup>.</td>
      </tr>
      <tr>
        <td>fixed64</td>
        <td>Always eight bytes. More efficient than uint64 if values are often
        greater than 2<sup>56</sup>.</td>
      </tr>
      <tr>
        <td>sfixed32</td>
        <td>Always four bytes.</td>
      </tr>
      <tr>
        <td>sfixed64</td>
        <td>Always eight bytes.</td>
      </tr>
      <tr>
        <td>bool</td>
        <td></td>
      </tr>
      <tr>
        <td>string</td>
        <td>A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot
        be longer than 2<sup>32</sup>.</td>
      </tr>
      <tr>
        <td>bytes</td>
        <td>May contain any arbitrary sequence of bytes no longer than 2<sup>32</sup>.</td>
      </tr>
    </tbody>
  </table>
</div>

<div>
  <table style="width: 100%;overflow-x: scroll;">
    <tbody>
      <tr>
        <th>Proto Type</th>
        <th>C++ Type</th>
        <th>Java/Kotlin Type<sup>[1]</sup></th>
        <th>Python Type<sup>[3]</sup></th>
        <th>Go Type</th>
        <th>Ruby Type</th>
        <th>C# Type</th>
        <th>PHP Type</th>
        <th>Dart Type</th>
        <th>Rust Type</th>
      </tr>
      <tr>
        <td>double</td>
        <td>double</td>
        <td>double</td>
        <td>float</td>
        <td>float64</td>
        <td>Float</td>
        <td>double</td>
        <td>float</td>
        <td>double</td>
        <td>f64</td>
      </tr>
      <tr>
        <td>float</td>
        <td>float</td>
        <td>float</td>
        <td>float</td>
        <td>float32</td>
        <td>Float</td>
        <td>float</td>
        <td>float</td>
        <td>double</td>
        <td>f32</td>
      </tr>
      <tr>
        <td>int32</td>
        <td>int32_t</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>int</td>
        <td>integer</td>
        <td>int</td>
        <td>i32</td>
      </tr>
      <tr>
        <td>int64</td>
        <td>int64_t</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
        <td>i64</td>
      </tr>
      <tr>
        <td>uint32</td>
        <td>uint32_t</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>uint</td>
        <td>integer</td>
        <td>int</td>
        <td>u32</td>
      </tr>
      <tr>
        <td>uint64</td>
        <td>uint64_t</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
        <td>u64</td>
      </tr>
      <tr>
        <td>sint32</td>
        <td>int32_t</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>int</td>
        <td>integer</td>
        <td>int</td>
        <td>i32</td>
      </tr>
      <tr>
        <td>sint64</td>
        <td>int64_t</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
        <td>i64</td>
      </tr>
      <tr>
        <td>fixed32</td>
        <td>uint32_t</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>uint</td>
        <td>integer</td>
        <td>int</td>
        <td>u32</td>
      </tr>
      <tr>
        <td>fixed64</td>
        <td>uint64_t</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
        <td>u64</td>
      </tr>
      <tr>
        <td>sfixed32</td>
        <td>int32_t</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>int</td>
        <td>integer</td>
        <td>int</td>
        <td>i32</td>
      </tr>
      <tr>
        <td>sfixed64</td>
        <td>int64_t</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
        <td>i64</td>
      </tr>
      <tr>
        <td>bool</td>
        <td>bool</td>
        <td>boolean</td>
        <td>bool</td>
        <td>bool</td>
        <td>TrueClass/FalseClass</td>
        <td>bool</td>
        <td>boolean</td>
        <td>bool</td>
        <td>bool</td>
      </tr>
      <tr>
        <td>string</td>
        <td>string</td>
        <td>String</td>
        <td>str/unicode<sup>[5]</sup></td>
        <td>string</td>
        <td>String (UTF-8)</td>
        <td>string</td>
        <td>string</td>
        <td>String</td>
        <td>ProtoString</td>
      </tr>
      <tr>
        <td>bytes</td>
        <td>string</td>
        <td>ByteString</td>
        <td>str (Python 2), bytes (Python 3)</td>
        <td>[]byte</td>
        <td>String (ASCII-8BIT)</td>
        <td>ByteString</td>
        <td>string</td>
        <td>List<int></td>
        <td>ProtoBytes</td>
      </tr>
    </tbody>
  </table>
</div>

<sup>[1]</sup> Kotlin uses the corresponding types from Java, even for unsigned
types, to ensure compatibility in mixed Java/Kotlin codebases.

<sup>[2]</sup> In Java, unsigned 32-bit and 64-bit integers are represented
using their signed counterparts, with the top bit simply being stored in the
sign bit.

<sup>[3]</sup> In all cases, setting values to a field will perform type
checking to make sure it is valid.

<sup>[4]</sup> 64-bit or unsigned 32-bit integers are always represented as long
when decoded, but can be an int if an int is given when setting the field. In
all cases, the value must fit in the type represented when set. See [2].

<sup>[5]</sup> Python strings are represented as unicode on decode but can be
str if an ASCII string is given (this is subject to change).

<sup>[6]</sup> Integer is used on 64-bit machines and string is used on 32-bit
machines.

You can find out more about how these types are encoded when you serialize your
message in
[Protocol Buffer Encoding](/programming-guides/encoding).

## Default Field Values {#default}

When a message is parsed, if the encoded message bytes do not contain a
particular field, accessing that field in the parsed object returns the default
value for that field. The default values are type-specific:

*   For strings, the default value is the empty string.
*   For bytes, the default value is empty bytes.
*   For bools, the default value is false.
*   For numeric types, the default value is zero.
*   For message fields, the field is not set. Its exact value is
    language-dependent. See the
    [generated code guide](/reference/) for details.
*   For enums, the default value is the **first defined enum value**, which must
    be 0. See [Enum Default Value](#enum-default).

The default value for repeated fields is empty (generally an empty list in the
appropriate language).

The default value for map fields is empty (generally an empty map in the
appropriate language).

### Overriding Default Scalar Values {#explicit-default}

In protobuf editions, you can specify explicit default values for singular
non-message fields. For example, let's say you want to provide a default value
of 10 for the `SearchRequest.result_per_page` field:

```proto
int32 result_per_page = 3 [default = 10];
```

If the sender does not specify `result_per_page`, the receiver will observe the
following state:

*   The result_per_page field is not present. That is, the
    `has_result_per_page()` (hazzer method) method would return `false`.
*   The value of `result_per_page` (returned from the "getter") is `10`.

If the sender does send a value for `result_per_page` the default value of 10 is
ignored and the sender's value is returned from the "getter".

See the [generated code guide](/reference/) for your
chosen language for more details about how defaults work in generated code.

Explicit default values cannot be specified for fields that have the
`field_presence` feature set to `IMPLICIT`.

## Enumerations {#enum}

When you're defining a message type, you might want one of its fields to only
have one of a predefined list of values. For example, let's say you want to add
a `corpus` field for each `SearchRequest`, where the corpus can be `UNIVERSAL`,
`WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` or `VIDEO`. You can do this very
simply by adding an `enum` to your message definition with a constant for each
possible value.

In the following example we've added an `enum` called `Corpus` with all the
possible values, and a field of type `Corpus`:

```proto
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
  Corpus corpus = 4;
}
```

### Enum Default Value {#enum-default}

The default value for the `SearchRequest.corpus` field is `CORPUS_UNSPECIFIED`
because that is the first value defined in the enum.

In edition 2023, the first value defined in an enum definition **must** have the
value zero and should have the name `ENUM_TYPE_NAME_UNSPECIFIED` or
`ENUM_TYPE_NAME_UNKNOWN`. This is because:

*   The zero value needs to be the first element for compatibility with
    [proto2](/programming-guides/proto2#enum-default)
    semantics, where the first enum value is the default unless a different
    value is explicitly specified.
*   There must be a zero value for compatibility with
    [proto3](/programming-guides/proto3#enum-default)
    semantics, where the zero value is used as the default value for all
    implicit-presence fields using this enum type.

It is also recommended that this first, default value have no semantic meaning
other than "this value was unspecified".

The default value for an enum field like `SearchRequest.corpus` field can be
explicitly overridden like this:

```
  Corpus corpus = 4 [default = CORPUS_UNIVERSAL];
```

If an enum type has been migrated from proto2 using `option features.enum_type =
CLOSED;` there is no restriction on the first value in the enum. It is not
recommended to change the first value of these types of enums because it will
change the default value for any fields using that enum type without an explicit
field default.

### Enum Value Aliases {#enum-aliases}

You can define aliases by assigning the same value to different enum constants.
To do this you need to set the `allow_alias` option to `true`. Otherwise, the
protocol buffer compiler generates a warning message when aliases are
found. Though all alias values are valid for serialization, only the first value
is used when deserializing.

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2;
}

enum EnumNotAllowingAlias {
  ENAA_UNSPECIFIED = 0;
  ENAA_STARTED = 1;
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message.
  ENAA_FINISHED = 2;
}
```

### Enum Constants {#enum-constants}

Enumerator constants must be in the range of a 32-bit integer. Since `enum`
values use
[varint encoding](/programming-guides/encoding) on the
wire, negative values are inefficient and thus not recommended. You can define
`enum`s within a message definition, as in the earlier example, or outside –
these `enum`s can be reused in any message definition in your `.proto` file. You
can also use an `enum` type declared in one message as the type of a field in a
different message, using the syntax `_MessageType_._EnumType_`.

### Language-specific Enum Implementations {#enum-language}

When you run the protocol buffer compiler on a `.proto` that uses an `enum`, the
generated code will have a corresponding `enum` for Java, Kotlin, or C++, or a
special `EnumDescriptor` class for Python that's used to create a set of
symbolic constants with integer values in the runtime-generated class.

{{% alert title="Important" color="warning" %}} The
generated code may be subject to language-specific limitations on the number of
enumerators (low thousands for one language). Review the
limitations for the languages you plan to use.
{{% /alert %}}

During deserialization, unrecognized enum values will be preserved in the
message, though how this is represented when the message is deserialized is
language-dependent. In languages that support open enum types with values
outside the range of specified symbols, such as C++ and Go, the unknown enum
value is simply stored as its underlying integer representation. In languages
with closed enum types such as Java, a case in the enum is used to represent an
unrecognized value, and the underlying integer can be accessed with special
accessors. In either case, if the message is serialized the unrecognized value
will still be serialized with the message.

{{% alert title="Important" color="warning" %}} For
information on how enums should work contrasted with how they currently work in
different languages, see
[Enum Behavior](/programming-guides/enum).
{{% /alert %}}

For more information about how to work with message `enum`s in your
applications, see the [generated code guide](/reference/)
for your chosen language.

### Reserved Values {#reserved}

If you [update](#updating) an enum type by entirely removing an enum entry, or
commenting it out, future users can reuse the numeric value when making their
own updates to the type. This can cause severe issues if they later load old
instances of the same `.proto`, including data corruption, privacy bugs, and so
on. One way to make sure this doesn't happen is to specify that the numeric
values (and/or names, which can also cause issues for JSON serialization) of
your deleted entries are `reserved`. The protocol buffer compiler will complain
if any future users try to use these identifiers. You can specify that your
reserved numeric value range goes up to the maximum possible value using the
`max` keyword.

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved FOO, BAR;
}
```

Note that you can't mix field names and numeric values in the same `reserved`
statement.

## Using Other Message Types {#other}

You can use other message types as field types. For example, let's say you
wanted to include `Result` messages in each `SearchResponse` message – to do
this, you can define a `Result` message type in the same `.proto` and then
specify a field of type `Result` in `SearchResponse`:

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### Importing Definitions {#importing}

In the earlier example, the `Result` message type is defined in the same file as
`SearchResponse` – what if the message type you want to use as a field type is
already defined in another `.proto` file?

You can use definitions from other `.proto` files by *importing* them. To import
another `.proto`'s definitions, you add an import statement to the top of your
file:

```proto
import "myproject/other_protos.proto";
```

By default, you can use definitions only from directly imported `.proto` files.
However, sometimes you may need to move a `.proto` file to a new location.
Instead of moving the `.proto` file directly and updating all the call sites in
a single change, you can put a placeholder `.proto` file in the old location to
forward all the imports to the new location using the `import public` notion.

**Note that the public import functionality is not available in Java, Kotlin,
TypeScript, JavaScript, GCL, as well as C++ targets that use protobuf static
reflection.**

`import public` dependencies can be transitively relied upon by any code
importing the proto containing the `import public` statement. For example:

```proto
// new.proto
// All definitions are moved here
```

```proto
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

```proto
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

The protocol compiler searches for imported files in a set of directories
specified on the protocol compiler command line using the `-I`/`--proto_path`
flag. If no flag was given, it looks in the directory in which the compiler was
invoked. In general you should set the `--proto_path` flag to the root of your
project and use fully qualified names for all imports.

### Using proto2 and proto3 Message Types {#proto2}

It's possible to import
[proto2](/programming-guides/proto2) and
[proto3](/programming-guides/proto3) message types and
use them in your editions 2023 messages, and vice versa.

## Nested Types {#nested}

You can define and use message types inside other message types, as in the
following example – here the `Result` message is defined inside the
`SearchResponse` message:

```proto
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

If you want to reuse this message type outside its parent message type, you
refer to it as `_Parent_._Type_`:

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like. In the example below, note that the
two nested types named `Inner` are entirely independent, since they are defined
within different messages:

```proto
message Outer {       // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## Updating A Message Type {#updating}

If an existing message type no longer meets all your needs – for example, you'd
like the message format to have an extra field – but you'd still like to use
code created with the old format, don't worry! It's very simple to update
message types without breaking any of your existing code when you use the binary
wire format.

{{% alert title="Note" color="note" %}} If
you use ProtoJSON or
[proto text format](/reference/protobuf/textformat-spec)
to store your protocol buffer messages, the changes that you can make in your
proto definition are different. The ProtoJSON wire format safe changes are
described
[here](/programming-guides/json#json-wire-safety).
{{% /alert %}}

Check
[Proto Best Practices](/best-practices/dos-donts) and the
following rules:

### Binary Wire-unsafe Changes {#wire-unsafe-changes}

Wire-unsafe changes are schema changes that will break if you use parse data
that was serialized using the old schema with a parser that is using the new
schema (or vice versa). Only make wire-unsafe changes if you know that all
serializers and deserializers of the data are using the new schema.

*   Changing field numbers for any existing field is not safe.
    *   Changing the field number is equivalent to deleting the field and adding
        a new field with the same type. If you want to renumber a field, see the
        instructions for [deleting a field](#deleting).
*   Moving fields into an existing `oneof` is not safe.

### Binary Wire-safe Changes {#wire-safe-changes}

Wire-safe changes are ones where it is fully safe to evolve the schema in this
way without risk of data loss or new parse failures.

Note that any wire-safe changes may be a breaking change to application code in
a given language. For example, adding a value to a preexisting enum would be a
compilation break for any code with an exhaustive switch on that enum. For that
reason, Google may avoid making some of these types of changes on public
messages: the AIPs contain guidance for which of these changes are safe to make
there.

*   Adding new fields is safe.
    *   If you add new fields, any messages serialized by code using your "old"
        message format can still be parsed by your new generated code. You
        should keep in mind the [default values](#default) for these elements so
        that new code can properly interact with messages generated by old code.
        Similarly, messages created by your new code can be parsed by your old
        code: old binaries simply ignore the new field when parsing. See the
        [Unknown Fields](#unknowns) section for details.
*   Removing fields is safe.
    *   The same field number must not used again in your updated message type.
        You may want to rename the field instead, perhaps adding the prefix
        "OBSOLETE_", or make the field number [reserved](#fieldreserved), so
        that future users of your `.proto` can't accidentally reuse the number.
*   Adding additional values to an enum is safe.
*   Changing a single explicit presence field or extension into a member of a
    **new** `oneof` is safe.
*   Changing a `oneof` which contains only one field to an explicit presence
    field is safe.
*   Changing a field into an extension of same number and type is safe.

### Binary Wire-compatible Changes (Conditionally Safe) {#conditionally-safe-changes}

Unlike Wire-safe changes, wire-compatible means that the same data can be parsed
both before and after a given change. However, a parse of the data may be lossy
under this shape of change. For example, changing an int32 to an int64 is a
compatible change, but if a value larger than INT32_MAX is written, a client
that reads it as an int32 will discard the high order bits of the number.

You can make compatible changes to your schema only if you manage the roll out
to your system carefully. For example, you may change an int32 to an int64 but
ensure you continue to only write legal int32 values until the new schema is
deployed to all endpoints, and then subsequently start writing larger values
after that.

If your schema is published outside of your organization, you should generally
not make wire-compatible changes, as you cannot manage the deployment of the new
schema to know when the different range of values may be safe to use.

*   `int32`, `uint32`, `int64`, `uint64`, and `bool` are all compatible.
    *   If a number is parsed from the wire which doesn't fit in the
        corresponding type, you will get the same effect as if you had cast the
        number to that type in C++ (for example, if a 64-bit number is read as
        an int32, it will be truncated to 32 bits).
*   `sint32` and `sint64` are compatible with each other but are *not*
    compatible with the other integer types.
    *   If the value written was between INT_MIN and INT_MAX inclusive it will
        parse as the same value with either type. If an sint64 value was written
        outside of that range and parsed as an sint32, the varint is truncated
        to 32 bits and then zigzag decoding occurs (which will cause a different
        value to be observed).
*   `string` and `bytes` are compatible as long as the bytes are valid UTF-8.
*   Embedded messages are compatible with `bytes` if the bytes contain an
    encoded instance of the message.
*   `fixed32` is compatible with `sfixed32`, and `fixed64` with `sfixed64`.
*   For `string`, `bytes`, and message fields, singular is compatible with
    `repeated`.
    *   Given serialized data of a repeated field as input, clients that expect
        this field to be singular will take the last input value if it's a
        primitive type field or merge all input elements if it's a message type
        field. Note that this is **not** generally safe for numeric types,
        including bools and enums. Repeated fields of numeric types are
        serialized in the
        [packed](/programming-guides/encoding#packed)
        format by default, which will not be parsed correctly when a singular
        field is expected.
*   `enum` is compatible with `int32`, `uint32`, `int64`, and `uint64`
    *   Be aware that client code may treat them differently when the message is
        deserialized: for example, unrecognized proto3 `enum` values will be
        preserved in the message, but how this is represented when the message
        is deserialized is language-dependent.
*   Changing a field between a `map<K, V>` and the corresponding `repeated`
    message field is binary compatible (see [Maps](#maps), below, for the
    message layout and other restrictions).
    *   However, the safety of the change is application-dependent: when
        deserializing and reserializing a message, clients using the `repeated`
        field definition will produce a semantically identical result; however,
        clients using the `map` field definition may reorder entries and drop
        entries with duplicate keys.

## Unknown Fields {#unknowns}

Unknown fields are well-formed protocol buffer serialized data representing
fields that the parser does not recognize. For example, when an old binary
parses data sent by a new binary with new fields, those new fields become
unknown fields in the old binary.

Editions messages preserve unknown fields and includes them during parsing and
in the serialized output, which matches proto2 and proto3 behavior.

### Retaining Unknown Fields {#retaining}

Some actions can cause unknown fields to be lost. For example, if you do one of
the following, unknown fields are lost:

*   Serialize a proto to JSON.
*   Iterate over all of the fields in a message to populate a new message.

To avoid losing unknown fields, do the following:

*   Use binary; avoid using text formats for data exchange.
*   Use message-oriented APIs, such as `CopyFrom()` and `MergeFrom()`, to copy data
    rather than copying field-by-field

TextFormat is a bit of a special case. Serializing to TextFormat prints unknown
fields using their field numbers. But parsing TextFormat data back into a binary
proto fails if there are entries that use field numbers.

## Extensions {#extensions}

An extension is a field defined outside of its container message; usually in a
`.proto` file separate from the container message's `.proto` file.

### Why Use Extensions? {#why-ext}

There are two main reasons to use extensions:

*   The container message's `.proto` file will have fewer imports/dependencies.
    This can improve build times, break circular dependencies, and otherwise
    promote loose coupling. Extensions are very good for this.
*   Allow systems to attach data to a container message with minimal dependency
    and coordination. Extensions are not a great solution for this because of
    the limited field number space and the
    [Consequences of Reusing Field Numbers](#consequences). If your use case
    requires very low coordination for a large number of extensions, consider
    using the
    [`Any` message type](/reference/protobuf/google.protobuf#any)
    instead.

### Example Extension {#ext-example}

Let's look at an example extension:

```proto
// file kittens/video_ext.proto

import "kittens/video.proto";
import "media/user_content.proto";

package kittens;

// This extension allows kitten videos in a media.UserContent message.
extend media.UserContent {
  // Video is a message imported from kittens/video.proto
  repeated Video kitten_videos = 126;
}
```

Note that the file defining the extension (`kittens/video_ext.proto`) imports
the container message's file (`media/user_content.proto`).

The container message must reserve a subset of its field numbers for extensions.

```proto
// file media/user_content.proto

package media;

// A container message to hold stuff that a user has created.
message UserContent {
  // Set verification to `DECLARATION` to enforce extension declarations for all
  // extensions in this range.
  extensions 100 to 199 [verification = DECLARATION];
}
```

The container message's file (`media/user_content.proto`) defines the message
`UserContent`, which reserves field numbers [100 to 199] for extensions. It is
recommended to set `verification = DECLARATION` for the range to require
declarations for all its extensions.

When the new extension (`kittens/video_ext.proto`) is added, a corresponding
declaration should be added to `UserContent` and `verification` should be
removed.

```
// A container message to hold stuff that a user has created.
message UserContent {
  extensions 100 to 199 [
    declaration = {
      number: 126,
      full_name: ".kittens.kitten_videos",
      type: ".kittens.Video",
      repeated: true
    },
    // Ensures all field numbers in this extension range are declarations.
    verification = DECLARATION
  ];
}
```

`UserContent` declares that field number `126` will be used by a `repeated`
extension field with the fully-qualified name `.kittens.kitten_videos` and the
fully-qualified type `.kittens.Video`. To learn more about extension
declarations see
[Extension Declarations](/programming-guides/extension_declarations).

Note that the container message's file (`media/user_content.proto`) **does not**
import the kitten_video extension definition (`kittens/video_ext.proto`)

There is no difference in the wire-format encoding of extension fields as
compared to a standard field with the same field number, type, and cardinality.
Therefore, it is safe to move a standard field out of its container to be an
extension or to move an extension field into its container message as a standard
field so long as the field number, type, and cardinality remain constant.

However, because extensions are defined outside of the container message, no
specialized accessors are generated to get and set specific extension fields.
For our example, the protobuf compiler **will not generate** `AddKittenVideos()`
or `GetKittenVideos()` accessors. Instead, extensions are accessed through
parameterized functions like: `HasExtension()`, `ClearExtension()`,
`GetExtension()`, `MutableExtension()`, and `AddExtension()`.

In C++, it would look something like:

```cpp
UserContent user_content;
user_content.AddExtension(kittens::kitten_videos, new kittens::Video());
assert(1 == user_content.GetRepeatedExtension(kittens::kitten_videos).size());
user_content.GetRepeatedExtension(kittens::kitten_videos)[0];
```

### Defining Extension Ranges {#defining-ranges}

If you are the owner of a container message, you will need to define an
extension range for the extensions to your message.

Field numbers allocated to extension fields cannot be reused for standard
fields.

It is safe to expand an extension range after it is defined. A good default is
to allocate 1000 relatively small numbers, and densely populate that space using
extension declarations:

```proto
message ModernExtendableMessage {
  // All extensions in this range should use extension declarations.
  extensions 1000 to 2000 [verification = DECLARATION];
}
```

When adding a range for extension declarations before the actual extensions, you
should add `verification = DECLARATION` to enforce that declarations are used
for this new range. This placeholder can be removed once an actual declaration
is added.

It is safe to split an existing extension range into separate ranges that cover
the same total range. This might be necessary for migrating a legacy message
type to
[Extension Declarations](/programming-guides/extension_declarations).
For example, before migration, the range might be defined as:

```proto
message LegacyMessage {
  extensions 1000 to max;
}
```

And after migration (splitting the range) it can be:

```proto
message LegacyMessage {
  // Legacy range that was using an unverified allocation scheme.
  extensions 1000 to 524999999 [verification = UNVERIFIED];
  // Current range that uses extension declarations.
  extensions 525000000 to max  [verification = DECLARATION];
}
```

It is not safe to increase the start field number nor decrease the end field
number to move or shrink an extension range. These changes can invalidate an
existing extension.

Prefer using field numbers 1 to 15 for standard fields that are populated in
most instances of your proto. It is not recommended to use these numbers for
extensions.

If your numbering convention might involve extensions having very large field
numbers, you can specify that your extension range goes up to the maximum
possible field number using the `max` keyword:

```proto
message Foo {
  extensions 1000 to max;
}
```

`max` is 2<sup>29</sup> - 1, or 536,870,911.

### Choosing Extension Numbers {#choosing}

Extensions are just fields that can be specified outside of their container
messages. All the same rules for [Assigning Field Numbers](#assigning) apply to
extension field numbers. The same
[Consequences of Reusing Field Numbers](#consequences) also apply to reusing
extension field numbers.

Choosing unique extension field numbers is simple if the container message uses
[extension declarations](/programming-guides/extension_declarations).
When defining a new extension, choose the lowest field number above all other
declarations from the highest extension range defined in the container message.
For example, if a container message is defined like this:

```proto
message Container {
  // Legacy range that was using an unverified allocation scheme
  extensions 1000 to 524999999;
  // Current range that uses extension declarations. (highest extension range)
  extensions 525000000 to max  [
    declaration = {
      number: 525000001,
      full_name: ".bar.baz_ext",
      type: ".bar.Baz"
    }
    // 525,000,002 is the lowest field number above all other declarations
  ];
}
```

The next extension of `Container` should add a new declaration with the number
`525000002`.

#### Unverified Extension Number Allocation (not recommended) {#unverified}

The owner of a container message may choose to forgo extension declarations in
favor of their own unverified extension number allocation strategy.

An unverified allocation scheme uses a mechanism external to the protobuf
ecosystem to allocate extension field numbers within the selected extension
range. One example could be using a monorepo's commit number. This system is
"unverified" from the protobuf compiler's point of view since there is no way to
check that an extension is using a properly acquired extension field number.

The benefit of an unverified system over a verified system like extension
declarations is the ability to define an extension without coordinating with the
container message owner.

The downside of an unverified system is that the protobuf compiler cannot
protect participants from reusing extension field numbers.

**Unverified extension field number allocation strategies are not recommended**
because the [Consequences of Reusing Field Numbers](#consequences) fall on all
extenders of a message (not just the developer that didn't follow the
recommendations). If your use case requires very low coordination, consider
using the
[`Any` message](/reference/protobuf/google.protobuf#any)
instead.

Unverified extension field number allocation strategies are limited to the range
1 to 524,999,999. Field numbers 525,000,000 and above can only be used with
extension declarations.

### Specifying Extension Types {#specifying-extension-types}

Extensions can be of any field type except `oneof`s and `map`s.

### Nested Extensions (not recommended) {#nested-exts}

You can declare extensions in the scope of another message:

```proto
import "common/user_profile.proto";

package puppies;

message Photo {
  extend common.UserProfile {
    int32 likes_count = 111;
  }
  ...
}
```

In this case, the C++ code to access this extension is:

```cpp
UserProfile user_profile;
user_profile.SetExtension(puppies::Photo::likes_count, 42);
```

In other words, the only effect is that `likes_count` is defined within the
scope of `puppies.Photo`.

This is a common source of confusion: Declaring an `extend` block nested inside
a message type *does not* imply any relationship between the outer type and the
extended type. In particular, the earlier example *does not* mean that `Photo`
is any sort of subclass of `UserProfile`. All it means is that the symbol
`likes_count` is declared inside the scope of `Photo`; it's simply a static
member.

A common pattern is to define extensions inside the scope of the extension's
field type - for example, here's an extension to `media.UserContent` of type
`puppies.Photo`, where the extension is defined as part of `Photo`:

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  extend media.UserContent {
    Photo puppy_photo = 127;
  }
  ...
}
```

However, there is no requirement that an extension with a message type be
defined inside that type. You can also use the standard definition pattern:

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  ...
}

// This can even be in a different file.
extend media.UserContent {
  Photo puppy_photo = 127;
}
```

This **standard (file-level) syntax is preferred** to avoid confusion. The
nested syntax is often mistaken for subclassing by users who are not already
familiar with extensions.

## Any {#any}

The `Any` message type lets you use messages as embedded types without having
their .proto definition. An `Any` contains an arbitrary serialized message as
`bytes`, along with a URL that acts as a globally unique identifier for and
resolves to that message's type. To use the `Any` type, you need to
[import](#other) `google/protobuf/any.proto`.

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

The default type URL for a given message type is
`type.googleapis.com/_packagename_._messagename_`.

Different language implementations will support runtime library helpers to pack
and unpack `Any` values in a typesafe manner – for example, in Java, the `Any`
type will have special `pack()` and `unpack()` accessors, while in C++ there are
`PackFrom()` and `UnpackTo()` methods:

```cpp
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const google::protobuf::Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

If you want to limit contained messages to a small number of types and to
require permission before adding new types to the list, consider using
[extensions](#extensions) with
[extension declarations](/programming-guides/extension_declarations)
instead of `Any` message types.

## Oneof {#oneof}

If you have a message with many singular fields and where at most one field will
be set at the same time, you can enforce this behavior and save memory by using
the oneof feature.

Oneof fields are like singular fields except all the fields in a oneof share
memory, and at most one field can be set at the same time. Setting any member of
the oneof automatically clears all the other members. You can check which value
in a oneof is set (if any) using a special `case()` or `WhichOneof()` method,
depending on your chosen language.

Note that if *multiple values are set, the last set value as determined by the
order in the proto will overwrite all previous ones*.

Field numbers for oneof fields must be unique within the enclosing message.

### Using Oneof {#using-oneof}

To define a oneof in your `.proto` you use the `oneof` keyword followed by your
oneof name, in this case `test_oneof`:

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

You then add your oneof fields to the oneof definition. You can add fields of
any type, except `map` fields and `repeated` fields. If you need to add a
repeated field to a oneof, you can use a message containing the repeated field.

In your generated code, oneof fields have the same getters and setters as
regular fields. You also get a special method for checking which value (if any)
in the oneof is set. You can find out more about the oneof API for your chosen
language in the relevant [API reference](/reference/).

### Oneof Features {#oneof-features}

*   Setting a oneof field will automatically clear all other members of the
    oneof. So if you set several oneof fields, only the *last* field you set
    will still have a value.

    ```cpp
    SampleMessage message;
    message.set_name("name");
    CHECK(message.has_name());
    // Calling mutable_sub_message() will clear the name field and will set
    // sub_message to a new instance of SubMessage with none of its fields set.
    message.mutable_sub_message();
    CHECK(!message.has_name());
    ```

*   If the parser encounters multiple members of the same oneof on the wire,
    only the last member seen is used in the parsed message. When parsing data
    on the wire, starting at the beginning of the bytes, evaluate the next
    value, and apply the following parsing rules:

    *   First, check if a *different* field in the same oneof is currently set,
        and if so clear it.

    *   Then apply the contents as though the field was not in a oneof:

        *   A primitive will overwrite any value already set
        *   A message will merge into any value already set

*   Extensions are not supported for oneof.

*   A oneof cannot be `repeated`.

*   Reflection APIs work for oneof fields.

*   If you set a oneof field to the default value (such as setting an int32
    oneof field to 0), the "case" of that oneof field will be set, and the value
    will be serialized on the wire.

*   If you're using C++, make sure your code doesn't cause memory crashes. The
    following sample code will crash because `sub_message` was already deleted
    by calling the `set_name()` method.

    ```cpp
    SampleMessage message;
    SubMessage* sub_message = message.mutable_sub_message();
    message.set_name("name");      // Will delete sub_message
    sub_message->set_...            // Crashes here
    ```

*   Again in C++, if you `Swap()` two messages with oneofs, each message will
    end up with the other's oneof case: in the example below, `msg1` will have a
    `sub_message` and `msg2` will have a `name`.

    ```cpp
    SampleMessage msg1;
    msg1.set_name("name");
    SampleMessage msg2;
    msg2.mutable_sub_message();
    msg1.swap(&msg2);
    CHECK(msg1.has_sub_message());
    CHECK(msg2.has_name());
    ```

### Backwards-compatibility issues {#backward}

Be careful when adding or removing oneof fields. If checking the value of a
oneof returns `None`/`NOT_SET`, it could mean that the oneof has not been set or
it has been set to a field in a different version of the oneof. There is no way
to tell the difference, since there's no way to know if an unknown field on the
wire is a member of the oneof.

#### Tag Reuse Issues {#reuse}

*   **Move singular fields into or out of a oneof**: You may lose some of your
    information (some fields will be cleared) after the message is serialized
    and parsed. However, you can safely move a single field into a **new** oneof
    and may be able to move multiple fields if it is known that only one is ever
    set. See [Updating A Message Type](#updating) for further details.
*   **Delete a oneof field and add it back**: This may clear your currently set
    oneof field after the message is serialized and parsed.
*   **Split or merge oneof**: This has similar issues to moving singular fields.

## Maps {#maps}

If you want to create an associative map as part of your data definition,
protocol buffers provides a handy shortcut syntax:

```proto
map<key_type, value_type> map_field = N;
```

...where the `key_type` can be any integral or string type (so, any
[scalar](#scalar) type except for floating point types and `bytes`). Note that
neither enum nor proto messages are valid for `key_type`.
The `value_type` can be any type except another map.

So, for example, if you wanted to create a map of projects where each `Project`
message is associated with a string key, you could define it like this:

```proto
map<string, Project> projects = 3;
```

### Maps Features {#maps-features}

*   Extensions are not supported for maps.
*   Map fields cannot be `repeated`.
*   Wire format ordering and map iteration ordering of map values is undefined,
    so you cannot rely on your map items being in a particular order.
*   When generating text format for a `.proto`, maps are sorted by key. Numeric
    keys are sorted numerically.
*   When parsing from the wire or when merging, if there are duplicate map keys
    the last key seen is used. When parsing a map from text format, parsing may
    fail if there are duplicate keys.
*   If you provide a key but no value for a map field, the behavior when the
    field is serialized is language-dependent. In C++, Java, Kotlin, and Python
    the default value for the type is serialized, while in other languages
    nothing is serialized.
*   No symbol `FooEntry` can exist in the same scope as a map `foo`, because
    `FooEntry` is already used by the implementation of the map.

The generated map API is currently available for all supported languages. You
can find out more about the map API for your chosen language in the relevant
[API reference](/reference/).

### Backwards Compatibility {#backwards}

The map syntax is equivalent to the following on the wire, so protocol buffers
implementations that do not support maps can still handle your data:

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

Any protocol buffers implementation that supports maps must both produce and
accept data that can be accepted by the earlier definition.

## Packages {#packages}

You can add an optional `package` specifier to a `.proto` file to prevent name
clashes between protocol message types.

```proto
package foo.bar;
message Open { ... }
```

You can then use the package specifier when defining fields of your message
type:

```proto
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

The way a package specifier affects the generated code depends on your chosen
language:

*   In **C++** the generated classes are wrapped inside a C++ namespace. For
    example, `Open` would be in the namespace `foo::bar`.
*   In **Java** and **Kotlin**, the package is used as the Java package, unless
    you explicitly provide an `option java_package` in your `.proto` file.
*   In **Python**, the `package` directive is ignored, since Python modules are
    organized according to their location in the file system.
*   In **Go**, the `package` directive is ignored, and the generated `.pb.go`
    file is in the package named after the corresponding `go_proto_library`
    Bazel rule. For open source projects, you **must** provide either a `go_package` option or set the Bazel `-M` flag.
*   In **Ruby**, the generated classes are wrapped inside nested Ruby
    namespaces, converted to the required Ruby capitalization style (first
    letter capitalized; if the first character is not a letter, `PB_` is
    prepended). For example, `Open` would be in the namespace `Foo::Bar`.
*   In **PHP** the package is used as the namespace after converting to
    PascalCase, unless you explicitly provide an `option php_namespace` in your
    `.proto` file. For example, `Open` would be in the namespace `Foo\Bar`.
*   In **C#** the package is used as the namespace after converting to
    PascalCase, unless you explicitly provide an `option csharp_namespace` in
    your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.

Note that even when the `package` directive does not directly affect the
generated code, for example in Python, it is still strongly recommended to
specify the package for the `.proto` file, as otherwise it may lead to naming
conflicts in descriptors and make the proto not portable for other languages.

### Packages and Name Resolution {#name-resolution}

Type name resolution in the protocol buffer language works like C++: first the
innermost scope is searched, then the next-innermost, and so on, with each
package considered to be "inner" to its parent package. A leading '.' (for
example, `.foo.bar.Baz`) means to start from the outermost scope instead.

The protocol buffer compiler resolves all type names by parsing the imported
`.proto` files. The code generator for each language knows how to refer to each
type in that language, even if it has different scoping rules.

## Defining Services {#services}

If you want to use your message types with an RPC (Remote Procedure Call)
system, you can define an RPC service interface in a `.proto` file and the
protocol buffer compiler will generate service interface code and stubs in your
chosen language. So, for example, if you want to define an RPC service with a
method that takes your `SearchRequest` and returns a `SearchResponse`, you can
define it in your `.proto` file as follows:

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

The most straightforward RPC system to use with protocol buffers is
[gRPC](https://grpc.io): a language- and platform-neutral open source RPC system
developed at Google. gRPC works particularly well with protocol buffers and lets
you generate the relevant RPC code directly from your `.proto` files using a
special protocol buffer compiler plugin.

If you don't want to use gRPC, it's also possible to use protocol buffers with
your own RPC implementation. You can find out more about this in the
[Proto2 Language Guide](/programming-guides/proto2#services).

There are also a number of ongoing third-party projects to develop RPC
implementations for Protocol Buffers. For a list of links to projects we know
about, see the
[third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

## JSON Mapping {#json}

The standard protobuf binary wire format is the preferred serialization format
for communication between two systems that use protobufs. For communicating with
systems that use JSON rather than protobuf wire format, Protobuf supports a
canonical encoding in
[ProtoJSON](/programming-guides/json).

## Options {#options}

Individual declarations in a `.proto` file can be annotated with a number of
*options*. Options do not change the overall meaning of a declaration, but may
affect the way it is handled in a particular context. The complete list of
available options is defined in [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto).

Some options are file-level options, meaning they should be written at the
top-level scope, not inside any message, enum, or service definition. Some
options are message-level options, meaning they should be written inside message
definitions. Some options are field-level options, meaning they should be
written inside field definitions. Options can also be written on enum types,
enum values, oneof fields, service types, and service methods; however, no
useful options currently exist for any of these.

Here are a few of the most commonly used options:

*   `java_package` (file option): The package you want to use for your generated
    Java/Kotlin classes. If no explicit `java_package` option is given in the
    `.proto` file, then by default the proto package (specified using the
    "package" keyword in the `.proto` file) will be used. However, proto
    packages generally do not make good Java packages since proto packages are
    not expected to start with reverse domain names. If not generating Java or
    Kotlin code, this option has no effect.

    ```proto
    option java_package = "com.example.foo";
    ```

*   `java_outer_classname` (file option): The class name (and hence the file
    name) for the wrapper Java class you want to generate. If no explicit
    `java_outer_classname` is specified in the `.proto` file, the class name
    will be constructed by converting the `.proto` file name to camel-case (so
    `foo_bar.proto` becomes `FooBar.java`). If the `java_multiple_files` option
    is disabled, then all other classes/enums/etc. generated for the `.proto`
    file will be generated *within* this outer wrapper Java class as nested
    classes/enums/etc. If not generating Java code, this option has no effect.

    ```proto
    option java_outer_classname = "Ponycopter";
    ```

*   `java_multiple_files` (file option): If false, only a single `.java` file
    will be generated for this `.proto` file, and all the Java
    classes/enums/etc. generated for the top-level messages, services, and
    enumerations will be nested inside of an outer class (see
    `java_outer_classname`). If true, separate `.java` files will be generated
    for each of the Java classes/enums/etc. generated for the top-level
    messages, services, and enumerations, and the wrapper Java class generated
    for this `.proto` file won't contain any nested classes/enums/etc. This is a
    Boolean option which defaults to `false`. If not generating Java code, this
    option has no effect.

    ```proto
    option java_multiple_files = true;
    ```

*   `optimize_for` (file option): Can be set to `SPEED`, `CODE_SIZE`, or
    `LITE_RUNTIME`. This affects the C++ and Java code generators (and possibly
    third-party generators) in the following ways:

    *   `SPEED` (default): The protocol buffer compiler will generate code for
        serializing, parsing, and performing other common operations on your
        message types. This code is highly optimized.
    *   `CODE_SIZE`: The protocol buffer compiler will generate minimal classes
        and will rely on shared, reflection-based code to implement
        serialization, parsing, and various other operations. The generated code
        will thus be much smaller than with `SPEED`, but operations will be
        slower. Classes will still implement exactly the same public API as they
        do in `SPEED` mode. This mode is most useful in apps that contain a very
        large number of `.proto` files and do not need all of them to be
        blindingly fast.
    *   `LITE_RUNTIME`: The protocol buffer compiler will generate classes that
        depend only on the "lite" runtime library (`libprotobuf-lite` instead of
        `libprotobuf`). The lite runtime is much smaller than the full library
        (around an order of magnitude smaller) but omits certain features like
        descriptors and reflection. This is particularly useful for apps running
        on constrained platforms like mobile phones. The compiler will still
        generate fast implementations of all methods as it does in `SPEED` mode.
        Generated classes will only implement the `MessageLite` interface in
        each language, which provides only a subset of the methods of the full
        `Message` interface.

    ```proto
    option optimize_for = CODE_SIZE;
    ```

*   `cc_generic_services`, `java_generic_services`, `py_generic_services` (file
    options): **Generic services are deprecated.** Whether or not the protocol
    buffer compiler should generate abstract service code based on
    [services definitions](#services) in C++, Java, and Python, respectively.
    For legacy reasons, these default to `true`. However, as of version 2.3.0
    (January 2010), it is considered preferable for RPC implementations to
    provide
    [code generator plugins](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)
    to generate code more specific to each system, rather than rely on the
    "abstract" services.

    ```proto
    // This file relies on plugins to generate service code.
    option cc_generic_services = false;
    option java_generic_services = false;
    option py_generic_services = false;
    ```

*   `cc_enable_arenas` (file option): Enables
    [arena allocation](/reference/cpp/arenas) for C++
    generated code.

*   `objc_class_prefix` (file option): Sets the Objective-C class prefix which
    is prepended to all Objective-C generated classes and enums from this
    .proto. There is no default. You should use prefixes that are between 3-5
    uppercase characters as
    [recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4).
    Note that all 2 letter prefixes are reserved by Apple.

*   `packed` (field option): In protobuf editions, this option is locked to
    `true`. To use unpacked wireformat, you can override this option using an
    editions feature. This provides compatibility with parsers prior to version
    2.3.0 (rarely needed) as shown in the following example:

    ```proto
    repeated int32 samples = 4 [features.repeated_field_encoding = EXPANDED];
    ```

*   `deprecated` (field option): If set to `true`, indicates that the field is
    deprecated and should not be used by new code. In most languages this has no
    actual effect. In Java, this becomes a `@Deprecated` annotation. For C++,
    clang-tidy will generate warnings whenever deprecated fields are used. In
    the future, other language-specific code generators may generate deprecation
    annotations on the field's accessors, which will in turn cause a warning to
    be emitted when compiling code which attempts to use the field. If the field
    is not used by anyone and you want to prevent new users from using it,
    consider replacing the field declaration with a [reserved](#fieldreserved)
    statement.

    ```proto
    int32 old_field = 6 [deprecated = true];
    ```

### Enum Value Options {#enum-value-options}

Enum value options are supported. You can use the `deprecated` option to
indicate that a value shouldn't be used anymore. You can also create custom
options using extensions.

The following example shows the syntax for adding these options:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.EnumValueOptions {
  string string_name = 123456789;
}

enum Data {
  DATA_UNSPECIFIED = 0;
  DATA_SEARCH = 1 [deprecated = true];
  DATA_DISPLAY = 2 [
    (string_name) = "display_value"
  ];
}
```

The C++ code to read the `string_name` option might look something like this:

```cpp
const absl::string_view foo = proto2::GetEnumDescriptor<Data>()
    ->FindValueByName("DATA_DISPLAY")->options().GetExtension(string_name);
```

See [Custom Options](#customoptions) to see how to apply custom options to enum
values and to fields.

### Custom Options {#customoptions}

Protocol Buffers also allows you to define and use your own options. Note that
this is an **advanced feature** which most people don't need. If you do think
you need to create your own options, see the
[Proto2 Language Guide](/programming-guides/proto2#customoptions)
for details. Note that creating custom options uses
[extensions](/programming-guides/proto2#extensions).

### Option Retention {#option-retention}

Options have a notion of *retention*, which controls whether an option is
retained in the generated code. Options have *runtime retention* by default,
meaning that they are retained in the generated code and are thus visible at
runtime in the generated descriptor pool. However, you can set `retention =
RETENTION_SOURCE` to specify that an option (or field within an option) must not
be retained at runtime. This is called *source retention*.

Option retention is an advanced feature that most users should not need to worry
about, but it can be useful if you would like to use certain options without
paying the code size cost of retaining them in your binaries. Options with
source retention are still visible to `protoc` and `protoc` plugins, so code
generators can use them to customize their behavior.

Retention can be set directly on an option, like this:

```proto
extend google.protobuf.FileOptions {
  int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

It can also be set on a plain field, in which case it takes effect only when
that field appears inside an option:

```proto
message OptionsMessage {
  int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

You can set `retention = RETENTION_RUNTIME` if you like, but this has no effect
since it is the default behavior. When a message field is marked
`RETENTION_SOURCE`, its entire contents are dropped; fields inside it cannot
override that by trying to set `RETENTION_RUNTIME`.

{{% alert title="Note" color="note" %}} As
of Protocol Buffers 22.0, support for option retention is still in progress and
only C++ and Java are supported. Go has support starting from 1.29.0. Python
support is complete but has not made it into a release yet.
{{% /alert %}}

### Option Targets {#option-targets}

Fields have a `targets` option which controls the types of entities that the
field may apply to when used as an option. For example, if a field has
`targets = TARGET_TYPE_MESSAGE` then that field cannot be set in a custom option
on an enum (or any other non-message entity). Protoc enforces this and will
raise an error if there is a violation of the target constraints.

At first glance, this feature may seem unnecessary given that every custom
option is an extension of the options message for a specific entity, which
already constrains the option to that one entity. However, option targets are
useful in the case where you have a shared options message applied to multiple
entity types and you want to control the usage of individual fields in that
message. For example:

```proto
message MyOptions {
  string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
                                     targets = TARGET_TYPE_ENUM];
}

extend google.protobuf.FileOptions {
  MyOptions file_options = 50000;
}

extend google.protobuf.MessageOptions {
  MyOptions message_options = 50000;
}

extend google.protobuf.EnumOptions {
  MyOptions enum_options = 50000;
}

// OK: this field is allowed on file options
option (file_options).file_only_option = "abc";

message MyMessage {
  // OK: this field is allowed on both message and enum options
  option (message_options).message_and_enum_option = 42;
}

enum MyEnum {
  MY_ENUM_UNSPECIFIED = 0;
  // Error: file_only_option cannot be set on an enum.
  option (enum_options).file_only_option = "xyz";
}
```

## Generating Your Classes {#generating}

To generate the Java, Kotlin, Python, C++, Go, Ruby, Objective-C, or C# code
that you need to work with the message types defined in a `.proto` file, you
need to run the protocol buffer compiler `protoc` on the `.proto` file. If you
haven't installed the compiler,
[download the package](/downloads) and follow the
instructions in the README. For Go, you also need to install a special code
generator plugin for the compiler; you can find this and installation
instructions in the [golang/protobuf](https://github.com/golang/protobuf/)
repository on GitHub.

The Protocol Compiler is invoked as follows:

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

*   `IMPORT_PATH` specifies a directory in which to look for `.proto` files when
    resolving `import` directives. If omitted, the current directory is used.
    Multiple import directories can be specified by passing the `--proto_path`
    option multiple times; they will be searched in order. `-I=_IMPORT_PATH_`
    can be used as a short form of `--proto_path`.

**Note:** File paths relative to their `proto_path` must be globally unique in a
given binary. For example, if you have `proto/lib1/data.proto` and
`proto/lib2/data.proto`, those two files cannot be used together with
`-I=proto/lib1 -I=proto/lib2` because it would be ambiguous which file `import
"data.proto"` will mean. Instead `-Iproto/` should be used and the global names
will be `lib1/data.proto` and `lib2/data.proto`.

If you are publishing a library and other users may use your messages directly,
you should include a unique library name in the path that they are expected to
be used under to avoid file name collisions. If you have multiple directories in
one project, it is best practice to prefer setting one `-I` to a top level
directory of the project.

*   You can provide one or more *output directives*:

    *   `--cpp_out` generates C++ code in `DST_DIR`. See the
        [C++ generated code reference](/reference/cpp/cpp-generated)
        for more.
    *   `--java_out` generates Java code in `DST_DIR`. See the
        [Java generated code reference](/reference/java/java-generated)
        for more.
    *   `--kotlin_out` generates additional Kotlin code in `DST_DIR`. See the
        [Kotlin generated code reference](/reference/kotlin/kotlin-generated)
        for more.
    *   `--python_out` generates Python code in `DST_DIR`. See the
        [Python generated code reference](/reference/python/python-generated)
        for more.
    *   `--go_out` generates Go code in `DST_DIR`. See the
        [Go generated code reference](/reference/go/go-generated-opaque)
        for more.
    *   `--ruby_out` generates Ruby code in `DST_DIR`. See the
        [Ruby generated code reference](/reference/ruby/ruby-generated)
        for more.
    *   `--objc_out` generates Objective-C code in `DST_DIR`. See the
        [Objective-C generated code reference](/reference/objective-c/objective-c-generated)
        for more.
    *   `--csharp_out` generates C# code in `DST_DIR`. See the
        [C# generated code reference](/reference/csharp/csharp-generated)
        for more.
    *   `--php_out` generates PHP code in `DST_DIR`. See the
        [PHP generated code reference](/reference/php/php-generated)
        for more.

    As an extra convenience, if the `DST_DIR` ends in `.zip` or `.jar`, the
    compiler will write the output to a single ZIP-format archive file with the
    given name. `.jar` outputs will also be given a manifest file as required by
    the Java JAR specification. Note that if the output archive already exists,
    it will be overwritten.

*   You must provide one or more `.proto` files as input. Multiple `.proto`
    files can be specified at once. Although the files are named relative to the
    current directory, each file must reside in one of the `IMPORT_PATH`s so
    that the compiler can determine its canonical name.

## File location {#location}

Prefer not to put `.proto` files in the same
directory as other language sources. Consider
creating a subpackage `proto` for `.proto` files, under the root package for
your project.

### Location Should be Language-agnostic {#location-language-agnostic}

When working with Java code, it's handy to put related `.proto` files in the
same directory as the Java source. However, if any non-Java code ever uses the
same protos, the path prefix will no longer make sense. So in
general, put the protos in a related language-agnostic directory such as
`//myteam/mypackage`.

The exception to this rule is when it's clear that the protos will be used only
in a Java context, such as for testing.

## Supported Platforms {#platforms}

For information about:

*   the operating systems, compilers, build systems, and C++ versions that are
    supported, see
    [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
*   the PHP versions that are supported, see
    [Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions).
