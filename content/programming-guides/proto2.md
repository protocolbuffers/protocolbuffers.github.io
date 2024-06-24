+++
title = "Language Guide (proto 2)"
weight = 30
description = "Covers how to use the version 2 of Protocol Buffers in your project."
aliases = "/programming-guides/proto/"
type = "docs"
+++

This guide describes how to use the protocol buffer language to structure your
protocol buffer data, including `.proto` file syntax and how to generate data
access classes from your `.proto` files. It covers the **proto2** version of the
protocol buffers language; for information on **proto3** syntax, see the
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
syntax = "proto2";

message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
}
```

*   The first line of the file specifies that you're using `proto2` syntax. This
    must be the first non-empty, non-comment line of the file.
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
    future reuse. This has been a very easy mistake to make with
    [extension fields](#extensions) for several reasons.
    [Extension Declarations](/programming-guides/extension_declarations)
    provide a mechanism for reserving extension fields.

The max field is 29 bits instead of the more-typical 32 bits because three lower
bits are used for the wire format. For more on this, see the
[Encoding topic](/programming-guides/encoding#structure).

<a id="specifying-rules"></a>

### Specifying Field Labels {#field-labels}

Message fields can be one of the following:

*   `optional`: An `optional` field is in one of two possible states:

    *   the field is set, and contains a value that was explicitly set or parsed
        from the wire. It will be serialized to the wire.
    *   the field is unset, and will return the default value. It will not be
        serialized to the wire.

    You can check to see if the value was explicitly set.

*   `repeated`: this field type can be repeated zero or more times in a
    well-formed message. The order of the repeated values will be preserved.

*   `map`: this is a paired key/value field type. See
    [Maps](/programming-guides/encoding#maps) for more on
    this field type.

*   `required`: **Do not use.** Required fields are so problematic they were
    removed from proto3. Semantics for required field should be implemented at
    the application layer. When it *is* used, a
    well-formed message must have exactly one of this field.

For historical reasons, `repeated` fields of scalar numeric types (for example,
`int32`, `int64`, `enum`) aren't encoded as efficiently as they could be. New
code should use the special option `[packed = true]` to get a more efficient
encoding. For example:

```proto
repeated int32 samples = 4 [packed = true];
repeated ProtoEnum results = 5 [packed = true];
```

You can find out more about `packed` encoding in
[Protocol Buffer Encoding](/programming-guides/encoding#packed).

{{% alert title="Important" color="warning" %}} **Required Is Forever**
As mentioned earlier **`required` must not be used for new fields**. Semantics
for required fields should be implemented at the application layer instead.
Existing `required` fields should be treated as permanent, immutable elements of
the message definition. It is nearly impossible to safely change a field from
`required` to `optional`. If there is any chance that a stale reader exists, it
will consider messages without this field to be incomplete and may reject or
drop them. {{% /alert %}}

A second issue with required fields appears when someone adds a value to an
enum. In this case, the unrecognized enum value is treated as if it were
missing, which also causes the required value check to fail.

#### Well-formed Messages {#well-formed}

The term "well-formed," when applied to protobuf messages, refers to the bytes
serialized/deserialized. The protoc parser validates that a given proto
definition file is parseable.

In the case of `optional` fields that have more than one value, the protoc
parser will accept the input, but only uses the last field. So, the "bytes" may
not be "well-formed" but the resulting message would have only one and would be
"well-formed" (but would not roundtrip the same).

### Adding More Message Types {#adding-types}

Multiple message types can be defined in a single `.proto` file. This is useful
if you are defining multiple related messages – so, for example, if you wanted
to define the reply message format that corresponds to your `SearchResponse`
message type, you could add it to the same `.proto`:

```proto
message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
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

To add comments to your `.proto` files, use C/C++-style `//` and `/* ... */`
syntax.

```proto
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;  // Which page number do we want?
  optional int32 results_per_page = 3;  // Number of results to return per page.
}
```

### Deleting Fields {#deleting}

Deleting fields can cause serious problems if not done properly.

**Do not delete** `required` fields. This is almost impossible to do safely.

When you no longer need a non-required field and all references have been
deleted from client code, you may delete the field definition from the message.
However, you **must** [reserve the deleted field number](#fieldreserved). If you
do not reserve the field number, it is possible for a developer to reuse that
number in the future.

You should also reserve the field name to allow JSON and TextFormat encodings of
your message to continue to parse.

<a id="fieldreserved"></a>

### Reserved Field Numbers {#reserved-field-numbers}

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
  reserved "foo", "bar";
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
    generates a `.kt` file for each message type, containing a DSL which can be
    used to simplify creating message instances.
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
*   For **Dart**, the compiler generates a `.pb.dart` file with a class for each
    message type in your file.

You can find out more about using the APIs for each language by following the
tutorial for your chosen language. For even more API
details, see the relevant [API reference](/reference/).

## Scalar Value Types {#scalar}

A scalar message field can have one of the following types – the table shows the
type specified in the `.proto` file, and the corresponding type in the
automatically generated class:

<div style="overflow:auto;width:100%;">
  <table style="width: 110%;">
    <tbody>
      <tr>
        <th>.proto Type</th>
        <th>Notes</th>
        <th>C++ Type</th>
        <th>Java/Kotlin Type<sup>[1]</sup></th>
        <th>Python Type<sup>[3]</sup></th>
        <th>Go Type</th>
        <th>Ruby Type</th>
        <th>C# Type</th>
        <th>Dart Type</th>
      </tr>
      <tr>
        <td>double</td>
        <td></td>
        <td>double</td>
        <td>double</td>
        <td>float</td>
        <td>*float64</td>
        <td>Float</td>
        <td>double</td>
        <td>double</td>
      </tr>
      <tr>
        <td>float</td>
        <td></td>
        <td>float</td>
        <td>float</td>
        <td>float</td>
        <td>*float32</td>
        <td>Float</td>
        <td>float</td>
        <td>double</td>
      </tr>
      <tr>
        <td>int32</td>
        <td>Uses variable-length encoding. Inefficient for encoding negative
        numbers – if your field is likely to have negative values, use sint32
        instead.</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>int</td>
        <td>*int32</td>
      </tr>
      <tr>
        <td>int64</td>
        <td>Uses variable-length encoding. Inefficient for encoding negative
        numbers – if your field is likely to have negative values, use sint64
        instead.</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>*int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>uint32</td>
        <td>Uses variable-length encoding.</td>
        <td>uint32</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>uint</td>
        <td>int</td>
      </tr>
      <tr>
        <td>uint64</td>
        <td>Uses variable-length encoding.</td>
        <td>uint64</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>sint32</td>
        <td>Uses variable-length encoding. Signed int value. These more
        efficiently encode negative numbers than regular int32s.</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>int</td>
        <td>*int32</td>
      </tr>
      <tr>
        <td>sint64</td>
        <td>Uses variable-length encoding. Signed int value. These more
        efficiently encode negative numbers than regular int64s.</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>*int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>fixed32</td>
        <td>Always four bytes. More efficient than uint32 if values are often
        greater than 2<sup>28</sup>.</td>
        <td>uint32</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>uint</td>
        <td>int</td>
      </tr>
      <tr>
        <td>fixed64</td>
        <td>Always eight bytes. More efficient than uint64 if values are often
        greater than 2<sup>56</sup>.</td>
        <td>uint64</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>sfixed32</td>
        <td>Always four bytes.</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>*int32</td>
        <td>Fixnum or Bignum (as required)</td>
        <td>int</td>
        <td>int</td>
      </tr>
      <tr>
        <td>sfixed64</td>
        <td>Always eight bytes.</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>*int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>bool</td>
        <td></td>
        <td>bool</td>
        <td>boolean</td>
        <td>bool</td>
        <td>*bool</td>
        <td>TrueClass/FalseClass</td>
        <td>bool</td>
        <td>bool</td>
      </tr>
      <tr>
        <td>string</td>
        <td>A string must always contain UTF-8 encoded<sup>[5]</sup> or 7-bit ASCII text, and
        cannot be longer than 2<sup>32</sup>.</td>
        <td>string</td>
        <td>String</td>
        <td>unicode (Python 2) or str (Python 3)</td>
        <td>*string</td>
        <td>String (UTF-8)</td>
        <td>string</td>
        <td>String</td>
      </tr>
      <tr>
        <td>bytes</td>
        <td>May contain any arbitrary sequence of bytes no longer than
        2<sup>32</sup>.</td>
        <td>string</td>
        <td>ByteString</td>
        <td>bytes</td>
        <td>[]byte</td>
        <td>String (ASCII-8BIT)</td>
        <td>ByteString</td>
        <td>List<int></td>
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

<sup>[5]</sup> Proto2 typically doesn't ever check the UTF-8 validity of string
fields. Behavior varies between languages though, and invalid UTF-8 data should
not be stored in string fields.

You can find out more about how these types are encoded when you serialize your
message in
[Protocol Buffer Encoding](/programming-guides/encoding).

## Optional Fields and Default Values {#optional}

As mentioned earlier, elements in a message description can be labeled
`optional`. A well-formed message may or may not contain an optional element.
When a message is parsed, if the encoded message does not contain an optional
element, accessing the corresponding field in the parsed object returns the
default value for that field. The default value can be specified as part of the
message description. For example, let's say you want to provide a default value
of 10 for a `SearchRequest`'s `result_per_page` value.

```proto
optional int32 result_per_page = 3 [default = 10];
```

If the default value is not specified for an optional element, a type-specific
default value is used instead:

*   For strings, the default value is the empty string.
*   For bytes, the default value is empty bytes.
*   For bools, the default value is false.
*   For numeric types, the default value is zero.
*   For enums, the default value is the **first defined enum value**.

Because the default value for enums is the first defined enum value, take care
when adding a value to the beginning of an enum value list. See the
[Updating a Message Type](#updating) section for guidelines on how to safely
change definitions.

## Enumerations {#enum}

When you're defining a message type, you might want one of its fields to only
have one of a predefined list of values. For example, let's say you want to add
a `corpus` field for each `SearchRequest`, where the corpus can be `UNIVERSAL`,
`WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` or `VIDEO`. You can do this very
simply by adding an `enum` to your message definition - a field with an `enum`
type can only have one of a specified set of constants as its value (if you try
to provide a different value, the parser will treat it like an unknown field).

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
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3 [default = 10];
  optional Corpus corpus = 4 [default = CORPUS_UNIVERSAL];
}
```

Because `SearchRequest` sets a default for the value of the `corpus` field, the
`CORPUS_UNSPECIFIED` value will not be used as a default. It will still be used
if a value of 0 is encountered on the wire. Other instances of the `Corpus` type
that **don't** set a default would use the `CORPUS_UNSPECIFIED` value as the
default.

You can define aliases by assigning the same value to different enum constants.
To do this you need to set the `allow_alias` option to `true`. Otherwise, the
protocol buffer compiler generates a warning message when aliases are
found. Though all alias values are valid during deserialization, the first value
is always used when serializing.

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

Enumerator constants must be in the range of a 32-bit integer. Since `enum`
values use
[varint encoding](/programming-guides/encoding) on the
wire, negative values are inefficient and thus not recommended. You can define
`enum`s within a message definition, as in the earlier example, or outside –
these `enum`s can be reused in any message definition in your `.proto` file. You
can also use an `enum` type declared in one message as the type of a field in a
different message, using the syntax `_MessageType_._EnumType_`.

When you run the protocol buffer compiler on a `.proto` that uses an `enum`, the
generated code will have a corresponding `enum` for Java, Kotlin, or C++, or a
special `EnumDescriptor` class for Python that's used to create a set of
symbolic constants with integer values in the runtime-generated class.

{{% alert title="Important" color="warning" %}} The
generated code may be subject to language-specific limitations on the number of
enumerators (low thousands for one language). Review the limitations for the
languages you plan to use. {{% /alert %}}

{{% alert title="Important" color="warning" %}} For
information on how enums should work contrasted with how they currently work in
different languages, see
[Enum Behavior](/programming-guides/enum).
{{% /alert %}}

Removing enum values is a breaking change for persisted protos. Instead of
removing a value, mark the value with the `reserved` keyword to prevent the enum
value from being code-generated, or keep the value but indicate that it will be
removed later by using the `deprecated` field option:

```proto
enum PhoneType {
  PHONE_TYPE_UNSPECIFIED = 0;
  PHONE_TYPE_MOBILE = 1;
  PHONE_TYPE_HOME = 2;
  PHONE_TYPE_WORK = 3 [deprecated=true];
  reserved 4,5;
}
```

For more information about how to work with message `enum`s in your
applications, see the [generated code guide](/reference/)
for your chosen language.

### Reserved Values {#reserved}

If you [update](#updating) an enum type by entirely removing an enum entry, or
commenting it out, future users can reuse the numeric value when making their
own updates to the type. This can cause severe issues if they later load old
versions of the same `.proto`, including data corruption, privacy bugs, and so
on. One way to make sure this doesn't happen is to specify that the numeric
values (and/or names, which can also cause issues for JSON serialization) of
your deleted entries are `reserved`. The protocol buffer compiler will complain
if any future users try to use these identifiers. You can specify that your
reserved numeric value range goes up to the maximum possible value using the
`max` keyword.

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
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
  optional string url = 1;
  optional string title = 2;
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

**Note that the public import functionality is not available in Java.**

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

### Using proto3 Message Types {#proto3}

It's possible to import
[proto3](/programming-guides/proto3) message types and
use them in your proto2 messages, and vice versa. However, proto2 enums cannot
be used in proto3 syntax.

## Nested Types {#nested}

You can define and use message types inside other message types, as in the
following example – here the `Result` message is defined inside the
`SearchResponse` message:

```proto
message SearchResponse {
  message Result {
    optional string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

If you want to reuse this message type outside its parent message type, you
refer to it as `_Parent_._Type_`:

```proto
message SomeOtherMessage {
  optional SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like. In the example below, note that the
two nested types named `Inner` are entirely independent, since they are defined
within different messages:

```proto
message Outer {       // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      optional int64 ival = 1;
      optional bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      optional int32  ival = 1;
      optional bool   booly = 2;
    }
  }
}
```

### Groups {#groups}

**Note that the groups feature is deprecated and should not be used when
creating new message types. Use nested message types instead.**

Groups are another way to nest information in your message definitions. For
example, another way to specify a `SearchResponse` containing a number of
`Result`s is as follows:

```proto
message SearchResponse {
  repeated group Result = 1 {
    optional string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
}
```

A group simply combines a nested message type and a field into a single
declaration. In your code, you can treat this message just as if it had a
`Result` type field called `result` (the latter name is converted to lower-case
so that it does not conflict with the former). Therefore, this example is
exactly equivalent to the `SearchResponse` earlier, except that the message has
a different [wire format](/programming-guides/encoding).

## Updating a Message Type {#updating}

If an existing message type no longer meets all your needs – for example, you'd
like the message format to have an extra field – but you'd still like to use
code created with the old format, don't worry! It's very simple to update
message types without breaking any of your existing code when you use the binary
wire format.

{{% alert title="Note" color="note" %}} If
you use JSON or
[proto text format](/reference/protobuf/textformat-spec)
to store your protocol buffer messages, the changes that you can make in your
proto definition are different. {{% /alert %}}

Check
[Proto Best Practices](/programming-guides/dos-donts) and
the following rules:

*   Don't change the field numbers for any existing fields. "Changing" the field
    number is equivalent to deleting the field and adding a new field with the
    same type. If you want to renumber a field, see the instructions for
    [deleting a field](#deleting).
*   Any new fields that you add should be `optional` or `repeated`. This means
    that any messages serialized by code using your "old" message format can
    still be parsed by your new generated code, as they won't be missing any
    `required` elements. You should keep in mind the [default values](#optional)
    for these elements so that new code can properly interact with messages
    generated by old code. Similarly, messages created by your new code can be
    parsed by your old code: old binaries simply ignore the new field when
    parsing. However, the unknown fields are not discarded, and if the message
    is later serialized, the unknown fields are serialized along with it – so if
    the message is passed on to new code, the new fields are still available.
    See the [Unknown Fields](#unknowns) section for details.
*   Non-required fields can be removed, as long as the field number is not used
    again in your updated message type. You may want to rename the field
    instead, perhaps adding the prefix "OBSOLETE_", or make the field number
    [reserved](#fieldreserved), so that future users of your `.proto` can't
    accidentally reuse the number.
*   A non-required field can be converted to an [extension](#extensions) and
    vice versa, as long as the type and number stay the same.
*   `int32`, `uint32`, `int64`, `uint64`, and `bool` are all compatible – this
    means you can change a field from one of these types to another without
    breaking forwards- or backwards-compatibility. If a number is parsed from
    the wire which doesn't fit in the corresponding type, you will get the same
    effect as if you had cast the number to that type in C++ (for example, if a
    64-bit number is read as an int32, it will be truncated to 32 bits).
*   `sint32` and `sint64` are compatible with each other but are *not*
    compatible with the other integer types.
*   `string` and `bytes` are compatible as long as the bytes are valid UTF-8.
*   Embedded messages are compatible with `bytes` if the bytes contain an
    encoded version of the message.
*   `fixed32` is compatible with `sfixed32`, and `fixed64` with `sfixed64`.
*   For `string`, `bytes`, and message fields, `optional` is compatible with
    `repeated`. Given serialized data of a repeated field as input, clients that
    expect this field to be `optional` will take the last input value if it's a
    scalar type field or merge all input elements if it's a message type field.
    Note that this is **not** generally safe for numeric types, including bools
    and enums. Repeated fields of numeric types can be serialized in the
    [packed](/programming-guides/encoding#packed) format,
    which will not be parsed correctly when an `optional` field is expected.
*   Changing a default value is generally OK, as long as you remember that
    default values are never sent over the wire. Thus, if a program receives a
    message in which a particular field isn't set, the program will see the
    default value as it was defined in that program's version of the protocol.
    It will NOT see the default value that was defined in the sender's code.
*   `enum` is compatible with `int32`, `uint32`, `int64`, and `uint64` in terms
    of wire format (note that values will be truncated if they don't fit).
    However, be aware that client code may treat them differently when the
    message is deserialized. Notably, unrecognized `enum` values are discarded
    when the message is deserialized, which makes the field's `has..` accessor
    return false and its getter return the first value listed in the `enum`
    definition, or the default value if one is specified. In the case of
    repeated enum fields, any unrecognized values are stripped out of the list.
    However, an integer field will always preserve its value. Because of this,
    you need to be very careful when upgrading an integer to an `enum` in terms
    of receiving out of bounds enum values on the wire.
*   In the current Java and C++ implementations, when unrecognized `enum` values
    are stripped out, they are stored along with other unknown fields. Note that
    this can result in strange behavior if this data is serialized and then
    reparsed by a client that recognizes these values. In the case of optional
    fields, even if a new value was written after the original message was
    deserialized, the old value will be still read by clients that recognize it.
    In the case of repeated fields, the old values will appear after any
    recognized and newly-added values, which means that order will not be
    preserved.
*   Changing a single `optional` field or extension into a member of a **new**
    `oneof` is binary compatible, however for some languages (notably, Go) the
    generated code's API will change in incompatible ways. For this reason,
    Google does not make such changes in its public APIs, as documented in
    [AIP-180](https://google.aip.dev/180#moving-into-oneofs). With
    the same caveat about source-compatibility, moving multiple fields into a
    new `oneof` may be safe if you are sure that no code sets more than one at a
    time. Moving fields into an existing `oneof` is not safe. Likewise, changing
    a single field `oneof` to an `optional` field or extension is safe.
*   Changing a field between a `map<K, V>` and the corresponding `repeated`
    message field is binary compatible (see [Maps](#maps), below, for the
    message layout and other restrictions). However, the safety of the change is
    application-dependent: when deserializing and reserializing a message,
    clients using the `repeated` field definition will produce a semantically
    identical result; however, clients using the `map` field definition may
    reorder entries and drop entries with duplicate keys.

## Unknown Fields {#unknowns}

Unknown fields are well-formed protocol buffer serialized data representing
fields that the parser does not recognize. For example, when an old binary
parses data sent by a new binary with new fields, those new fields become
unknown fields in the old binary.

Originally, proto3 messages always discarded unknown fields during parsing, but
in version 3.5 we reintroduced the preservation of unknown fields to match the
proto2 behavior. In versions 3.5 and later, unknown fields are retained during
parsing and included in the serialized output.

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
assert(1 == user_content.GetExtensionCount(kittens::kitten_videos));
user_content.GetExtension(kittens::kitten_videos, 0);
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
    optional int32 likes_count = 111;
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
    optional Photo puppy_photo = 127;
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
  optional Photo puppy_photo = 127;
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

```c++
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

**Currently the runtime libraries for working with `Any` types are under
development**.

If you want to limit contained messages to a small number of types and to
require permission before adding new types to the list, consider using
[extensions](#extensions) with
[extension declarations](/programming-guides/extension_declarations)
instead of `Any` message types.

## Oneof {#oneof}

If you have a message with many optional fields and where at most one field will
be set at the same time, you can enforce this behavior and save memory by using
the oneof feature.

Oneof fields are like optional fields except all the fields in a oneof share
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
any type except `map` fields, but you cannot use the `required`, `optional`, or
`repeated` keywords. If you need to add a repeated field to a oneof, you can use
a message containing the repeated field.

In your generated code, oneof fields have the same getters and setters as
regular `optional` fields. You also get a special method for checking which
value (if any) in the oneof is set. You can find out more about the oneof API
for your chosen language in the relevant
[API reference](/reference/).

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
    only the last member seen is used in the parsed message.

*   Extensions are not supported for oneof.

*   A oneof cannot be `repeated`.

*   Reflection APIs work for oneof fields.

*   If you set a oneof field to the default value (such as setting an int32
    oneof field to 0), the "case" of that oneof field will be set, and the value
    will be serialized on the wire.

*   If you're using C++, make sure your code doesn't cause memory crashes. The
    following sample code will crash because `sub_message` was already deleted
    by calling the `set_name()` method.

    ```c++
    SampleMessage message;
    SubMessage* sub_message = message.mutable_sub_message();
    message.set_name("name");      // Will delete sub_message
    sub_message->set_...            // Crashes here
    ```

*   Again in C++, if you `Swap()` two messages with oneofs, each message will
    end up with the other's oneof case: in the example below, `msg1` will have a
    `sub_message` and `msg2` will have a `name`.

    ```c++
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

*   **Move optional fields into or out of a oneof**: You may lose some of your
    information (some fields will be cleared) after the message is serialized
    and parsed. However, you can safely move a single field into a **new** oneof
    and may be able to move multiple fields if it is known that only one is ever
    set. See [Updating A Message Type](#updating) for further details.
*   **Delete a oneof field and add it back**: This may clear your currently set
    oneof field after the message is serialized and parsed.
*   **Split or merge oneof**: This has similar issues to moving regular
    `optional` fields.

## Maps {#maps}

If you want to create an associative map as part of your data definition,
protocol buffers provides a handy shortcut syntax:

```proto
map<key_type, value_type> map_field = N;
```

...where the `key_type` can be any integral or string type (so, any
[scalar](#scalar) type except for floating point types and `bytes`). Note that
enum is not a valid `key_type`. The `value_type` can be any type except another
map.

So, for example, if you wanted to create a map of projects where each `Project`
message is associated with a string key, you could define it like this:

```proto
map<string, Project> projects = 3;
```

### Maps Features {#maps-features}

*   Extensions are not supported for maps.
*   Maps cannot be `repeated`, `optional`, or `required`.
*   Wire format ordering and map iteration ordering of map values is undefined,
    so you cannot rely on your map items being in a particular order.
*   When generating text format for a `.proto`, maps are sorted by key. Numeric
    keys are sorted numerically.
*   When parsing from the wire or when merging, if there are duplicate map keys
    the last key seen is used. When parsing a map from text format, parsing may
    fail if there are duplicate keys.

The generated map API is currently available for all supported languages. You
can find out more about the map API for your chosen language in the relevant
[API reference](/reference/).

### Backwards Compatibility {#backwards}

The map syntax is equivalent to the following on the wire, so protocol buffers
implementations that do not support maps can still handle your data:

```proto
message MapFieldEntry {
  optional key_type key = 1;
  optional value_type value = 2;
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
  optional foo.bar.Open open = 1;
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

By default, the protocol compiler will then generate an abstract interface
called `SearchService` and a corresponding "stub" implementation. The stub
forwards all calls to an `RpcChannel`, which in turn is an abstract interface
that you must define yourself in terms of your own RPC system. For example, you
might implement an `RpcChannel` which serializes the message and sends it to a
server via HTTP. In other words, the generated stub provides a type-safe
interface for making protocol-buffer-based RPC calls, without locking you into
any particular RPC implementation. So, in C++, you might end up with code like
this:

```c++
using google::protobuf;

protobuf::RpcChannel* channel;
protobuf::RpcController* controller;
SearchService* service;
SearchRequest request;
SearchResponse response;

void DoSearch() {
  // You provide classes MyRpcChannel and MyRpcController, which implement
  // the abstract interfaces protobuf::RpcChannel and protobuf::RpcController.
  channel = new MyRpcChannel("somehost.example.com:1234");
  controller = new MyRpcController;

  // The protocol compiler generates the SearchService class based on the
  // definition given earlier.
  service = new SearchService::Stub(channel);

  // Set up the request.
  request.set_query("protocol buffers");

  // Execute the RPC.
  service->Search(controller, &request, &response,
                  protobuf::NewCallback(&Done));
}

void Done() {
  delete service;
  delete channel;
  delete controller;
}
```

All service classes also implement the `Service` interface, which provides a way
to call specific methods without knowing the method name or its input and output
types at compile time. On the server side, this can be used to implement an RPC
server with which you could register services.

```c++
using google::protobuf;

class ExampleSearchService : public SearchService {
 public:
  void Search(protobuf::RpcController* controller,
              const SearchRequest* request,
              SearchResponse* response,
              protobuf::Closure* done) {
    if (request->query() == "google") {
      response->add_result()->set_url("http://www.google.com");
    } else if (request->query() == "protocol buffers") {
      response->add_result()->set_url("http://protobuf.googlecode.com");
    }
    done->Run();
  }
};

int main() {
  // You provide class MyRpcServer.  It does not have to implement any
  // particular interface; this is just an example.
  MyRpcServer server;

  protobuf::Service* service = new ExampleSearchService;
  server.ExportOnPort(1234, service);
  server.Run();

  delete service;
  return 0;
}
```

If you don't want to plug in your own existing RPC system, you can use
[gRPC](https://github.com/grpc/grpc-common): a language- and platform-neutral
open source RPC system developed at Google. gRPC works particularly well with
protocol buffers and lets you generate the relevant RPC code directly from your
`.proto` files using a special protocol buffer compiler plugin. However, as
there are potential compatibility issues between clients and servers generated
with proto2 and proto3, we recommend that you use proto3 for defining gRPC
services. You can find out more about proto3 syntax in the
[Proto3 Language Guide](/programming-guides/proto3). If
you do want to use proto2 with gRPC, you need to use version 3.0.0 or higher of
the protocol buffers compiler and libraries.

In addition to gRPC, there are also a number of ongoing third-party projects to
develop RPC implementations for Protocol Buffers. For a list of links to
projects we know about, see the
[third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

## JSON Mapping {#json}

Proto2 supports a canonical encoding in JSON, making it easier to share data
between systems. The encoding is described on a type-by-type basis in the table
below.

When parsing JSON-encoded data into a protocol buffer, if a value is missing or
if its value is `null`, it will be interpreted as the corresponding
[default value](#optional).

A proto2 field that is defined with the `optional` keyword supports field
presence. Fields that have a value set and that support field presence always
include the field value in the JSON-encoded output, even if it is the default
value.

<table>
  <tbody>
    <tr>
      <th>proto2</th>
      <th>JSON</th>
      <th>JSON example</th>
      <th>Notes</th>
    </tr>
    <tr>
      <td>message</td>
      <td>object</td>
      <td><code>{"fooBar": v, "g": null, ...}</code></td>
      <td>Generates JSON objects. Message field names are mapped to
        lowerCamelCase and become JSON object keys. If the
        <code>json_name</code> field option is specified, the specified value
        will be used as the key instead. Parsers accept both the lowerCamelCase
        name (or the one specified by the <code>json_name</code> option) and the
        original proto field name. <code>null</code> is an accepted value for
        all field types and treated as the default value of the corresponding
        field type. However, <code>null</code> cannot be used for the
        <code>json_name</code> value. For more on why, see
        <a href="/news/2023-04-28#json-name">Stricter validation for json_name</a>.
      </td>
    </tr>
    <tr>
      <td>enum</td>
      <td>string</td>
      <td><code>"FOO_BAR"</code></td>
      <td>The name of the enum value as specified in proto is used. Parsers
        accept both enum names and integer values.
      </td>
    </tr>
    <tr>
      <td>map&lt;K,V&gt;</td>
      <td>object</td>
      <td><code>{"k": v, ...}</code></td>
      <td>All keys are converted to strings.</td>
    </tr>
    <tr>
      <td>repeated V</td>
      <td>array</td>
      <td><code>[v, ...]</code></td>
      <td><code>null</code> is accepted as the empty list <code>[]</code>.</td>
    </tr>
    <tr>
      <td>bool</td>
      <td>true, false</td>
      <td><code>true, false</code></td>
      <td></td>
    </tr>
    <tr>
      <td>string</td>
      <td>string</td>
      <td><code>"Hello World!"</code></td>
      <td></td>
    </tr>
    <tr>
      <td>bytes</td>
      <td>base64 string</td>
      <td><code>"YWJjMTIzIT8kKiYoKSctPUB+"</code></td>
      <td>JSON value will be the data encoded as a string using standard base64
        encoding with paddings. Either standard or URL-safe base64 encoding
        with/without paddings are accepted.
      </td>
    </tr>
    <tr>
      <td>int32, fixed32, uint32</td>
      <td>number</td>
      <td><code>1, -10, 0</code></td>
      <td>JSON value will be a decimal number. Either numbers or strings are
        accepted.
      </td>
    </tr>
    <tr>
      <td>int64, fixed64, uint64</td>
      <td>string</td>
      <td><code>"1", "-10"</code></td>
      <td>JSON value will be a decimal string. Either numbers or strings are
        accepted.
      </td>
    </tr>
    <tr>
      <td>float, double</td>
      <td>number</td>
      <td><code>1.1, -10.0, 0, "NaN", "Infinity"</code></td>
      <td>JSON value will be a number or one of the special string values "NaN",
        "Infinity", and "-Infinity". Either numbers or strings are accepted.
        Exponent notation is also accepted.  -0 is considered equivalent to 0.
      </td>
    </tr>
    <tr>
      <td>Any</td>
      <td><code>object</code></td>
      <td><code>{"@type": "url", "f": v, ... }</code></td>
      <td>If the <code>Any</code> contains a value that has a special JSON
        mapping, it will be converted as follows: <code>{"@type": xxx, "value":
        yyy}</code>. Otherwise, the value will be converted into a JSON object,
        and the <code>"@type"</code> field will be inserted to indicate the
        actual data type.
      </td>
    </tr>
    <tr>
      <td>Timestamp</td>
      <td>string</td>
      <td><code>"1972-01-01T10:00:20.021Z"</code></td>
      <td>Uses RFC 3339, where generated output will always be Z-normalized
          and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are
          also accepted.
      </td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>string</td>
      <td><code>"1.000340012s", "1s"</code></td>
      <td>Generated output always contains 0, 3, 6, or 9 fractional digits,
        depending on required precision, followed by the suffix "s". Accepted
        are any fractional digits (also none) as long as they fit into
        nano-seconds precision and the suffix "s" is required.
      </td>
    </tr>
    <tr>
      <td>Struct</td>
      <td><code>object</code></td>
      <td><code>{ ... }</code></td>
      <td>Any JSON object. See <code>struct.proto</code>.</td>
    </tr>
    <tr>
      <td>Wrapper types</td>
      <td>various types</td>
      <td><code>2, "2", "foo", true, "true", null, 0, ...</code></td>
      <td>Wrappers use the same representation in JSON as the wrapped scalar
        type, except that <code>null</code> is allowed and preserved during data
        conversion and transfer.
      </td>
    </tr>
    <tr>
      <td>FieldMask</td>
      <td>string</td>
      <td><code>"f.fooBar,h"</code></td>
      <td>See <code>field_mask.proto</code>.</td>
    </tr>
    <tr>
      <td>ListValue</td>
      <td>array</td>
      <td><code>[foo, bar, ...]</code></td>
      <td></td>
    </tr>
    <tr>
      <td>Value</td>
      <td>value</td>
      <td></td>
      <td>Any JSON value. Check
        <code><a href="/reference/protobuf/google.protobuf#value">google.protobuf.Value</a></code>
        for details.
      </td>
    </tr>
    <tr>
      <td>NullValue</td>
      <td>null</td>
      <td></td>
      <td>JSON null</td>
    </tr>
    <tr>
      <td>Empty</td>
      <td>object</td>
      <td><code>{}</code></td>
      <td>An empty JSON object</td>
    </tr>
  </tbody>
</table>

### JSON Options {#json-options}

A proto2 JSON implementation may provide the following options:

*   **Always emit fields without presence**: Fields that don't support presence
    and that have their default value are omitted by default in JSON output (for
    example, an implicit presence integer with a 0 value, implicit presence
    string fields that are empty strings, and empty repeated and map fields). An
    implementation may provide an option to override this behavior and output
    fields with their default values.

    As of v25.x, the C++, Java, and Python implementations are nonconformant, as
    this flag affects proto2 `optional` fields but not proto3 `optional` fields.
    A fix is planned for a future release.

*   **Ignore unknown fields**: Proto2 JSON parser should reject unknown fields
    by default but may provide an option to ignore unknown fields in parsing.

*   **Use proto field name instead of lowerCamelCase name**: By default proto2
    JSON printer should convert the field name to lowerCamelCase and use that as
    the JSON name. An implementation may provide an option to use proto field
    name as the JSON name instead. Proto2 JSON parsers are required to accept
    both the converted lowerCamelCase name and the proto field name.

*   **Emit enum values as integers instead of strings**: The name of an enum
    value is used by default in JSON output. An option may be provided to use
    the numeric value of the enum value instead.

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

*   `message_set_wire_format` (message option): If set to `true`, the message
    uses a different binary format intended to be compatible with an old format
    used inside Google called `MessageSet`. Users outside Google will probably
    never need to use this option. The message must be declared exactly as
    follows:

    ```proto
    message Foo {
      option message_set_wire_format = true;
      extensions 4 to max;
    }
    ```

*   `packed` (field option): If set to `true` on a repeated field of a basic
    numeric type, a more compact
    [encoding](/programming-guides/encoding#packed) is
    used. There is no downside to using this option. However, note that prior to
    version 2.3.0, parsers that received packed data when not expected would
    ignore it. Therefore, it was not possible to change an existing field to
    packed format without breaking wire compatibility. In 2.3.0 and later, this
    change is safe, as parsers for packable fields will always accept both
    formats, but be careful if you have to deal with old programs using old
    protobuf versions.

    ```proto
    repeated int32 samples = 4 [packed = true];
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
    optional int32 old_field = 6 [deprecated=true];
    ```

### Enum Value Options {#enum-value-options}

Enum value options are supported. You can use the `deprecated` option to
indicate that a value shouldn't be used anymore. You can also create custom
options using extensions.

The following example shows the syntax for adding these options:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.EnumValueOptions {
  optional string string_name = 123456789;
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
this is an **advanced feature** which most people don't need. Since options are
defined by the messages defined in `google/protobuf/descriptor.proto` (like
`FileOptions` or `FieldOptions`), defining your own options is simply a matter
of [extending](#extensions) those messages. For example:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}

message MyMessage {
  option (my_option) = "Hello world!";
}
```

Here we have defined a new message-level option by extending `MessageOptions`.
When we then use the option, the option name must be enclosed in parentheses to
indicate that it is an extension. We can now read the value of `my_option` in
C++ like so:

```proto
string value = MyMessage::descriptor()->options().GetExtension(my_option);
```

Here, `MyMessage::descriptor()->options()` returns the `MessageOptions` protocol
message for `MyMessage`. Reading custom options from it is just like reading any
other [extension](#extensions).

Similarly, in Java we would write:

```java
String value = MyProtoFile.MyMessage.getDescriptor().getOptions()
  .getExtension(MyProtoFile.myOption);
```

In Python it would be:

```python
value = my_proto_file_pb2.MyMessage.DESCRIPTOR.GetOptions()
  .Extensions[my_proto_file_pb2.my_option]
```

Custom options can be defined for every kind of construct in the Protocol
Buffers language. Here is an example that uses every kind of option:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.FileOptions {
  optional string my_file_option = 50000;
}
extend google.protobuf.MessageOptions {
  optional int32 my_message_option = 50001;
}
extend google.protobuf.FieldOptions {
  optional float my_field_option = 50002;
}
extend google.protobuf.OneofOptions {
  optional int64 my_oneof_option = 50003;
}
extend google.protobuf.EnumOptions {
  optional bool my_enum_option = 50004;
}
extend google.protobuf.EnumValueOptions {
  optional uint32 my_enum_value_option = 50005;
}
extend google.protobuf.ServiceOptions {
  optional MyEnum my_service_option = 50006;
}
extend google.protobuf.MethodOptions {
  optional MyMessage my_method_option = 50007;
}

option (my_file_option) = "Hello world!";

message MyMessage {
  option (my_message_option) = 1234;

  optional int32 foo = 1 [(my_field_option) = 4.5];
  optional string bar = 2;
  oneof qux {
    option (my_oneof_option) = 42;

    string quux = 3;
  }
}

enum MyEnum {
  option (my_enum_option) = true;

  FOO = 1 [(my_enum_value_option) = 321];
  BAR = 2;
}

message RequestType {}
message ResponseType {}

service MyService {
  option (my_service_option) = FOO;

  rpc MyMethod(RequestType) returns(ResponseType) {
    // Note:  my_method_option has type MyMessage.  We can set each field
    //   within it using a separate "option" line.
    option (my_method_option).foo = 567;
    option (my_method_option).bar = "Some string";
  }
}
```

Note that if you want to use a custom option in a package other than the one in
which it was defined, you must prefix the option name with the package name,
just as you would for type names. For example:

```proto
// foo.proto
import "google/protobuf/descriptor.proto";
package foo;
extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}
```

```proto
// bar.proto
import "foo.proto";
package bar;
message MyMessage {
  option (foo.my_option) = "Hello world!";
}
```

One last thing: Since custom options are extensions, they must be assigned field
numbers like any other field or extension. In the examples earlier, we have used
field numbers in the range 50000-99999\. This range is reserved for internal use
within individual organizations, so you can use numbers in this range freely for
in-house applications. If you intend to use custom options in public
applications, however, then it is important that you make sure that your field
numbers are globally unique. To obtain globally unique field numbers, send a
request to add an entry to
[protobuf global extension registry](https://github.com/protocolbuffers/protobuf/blob/master/docs/options.md).
Usually you only need one extension number. You can declare multiple options
with only one extension number by putting them in a sub-message:

```proto
message FooOptions {
  optional int32 opt1 = 1;
  optional string opt2 = 2;
}

extend google.protobuf.FieldOptions {
  optional FooOptions foo_options = 1234;
}

// usage:
message Bar {
  optional int32 a = 1 [(foo_options).opt1 = 123, (foo_options).opt2 = "baz"];
  // alternative aggregate syntax (uses TextFormat):
  optional int32 b = 2 [(foo_options) = { opt1: 123 opt2: "baz" }];
}
```

Also, note that each option type (file-level, message-level, field-level, etc.)
has its own number space, so, for example, you could declare extensions of
FieldOptions and MessageOptions with the same number.

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
  optional int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

It can also be set on a plain field, in which case it takes effect only when
that field appears inside an option:

```proto
message OptionsMessage {
  optional int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
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
  optional string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  optional int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
                                              targets = TARGET_TYPE_ENUM];
}

extend google.protobuf.FileOptions {
  optional MyOptions file_options = 50000;
}

extend google.protobuf.MessageOptions {
  optional MyOptions message_options = 50000;
}

extend google.protobuf.EnumOptions {
  optional MyOptions enum_options = 50000;
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
        [Go generated code reference](/reference/go/go-generated)
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
