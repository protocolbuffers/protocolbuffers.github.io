+++
title = "ProtoJSON Format"
weight = 62
description = "Covers how to use the Protobuf to JSON conversion utilities."
type = "docs"
+++

Protobuf supports a canonical encoding in JSON, making it easier to share data
with systems that do not support the standard protobuf binary wire format.

This page specifies the format, but a number of additional edge cases which
define a conformant ProtoJSON parser are covered in the Protobuf Conformance
Test Suite and are not exhaustively detailed here.

# Non-goals of the Format {#non-goals}

## Cannot Represent Some JSON schemas {#non-goals-arbitrary-json-schema}

The ProtoJSON format is designed to be a JSON representation of schemas which
are expressible in the Protobuf schema language.

It may be possible to represent many pre-existing JSON schemas as a Protobuf
schema and parse it using ProtoJSON, but it is not designed to be able to
represent arbitrary JSON schemas.

For example, there is no way to express in Protobuf schema to write types that
may be common in JSON schemas like `number[][]` or `number|string`.

It is possible to use `google.protobuf.Struct` and `google.protobuf.Value` types
to allow arbitrary JSON to be parsed into a Protobuf schema, but these only
allow you to capture the values as schemaless unordered key-value maps.

## Not as efficient as the binary wire format {#non-goals-highly-efficient}

ProtoJSON Format is not as efficient as binary wire format and never will be.

The converter uses more CPU to encode and decode messages and (except in rare
cases) encoded messages consume more space.

## Does not have as good schema-evolution guarantees as binary wire format {#non-goals-optimal-schema-evolution}

ProtoJSON format does not support unknown fields, and it puts field and enum
value names into encoded messages which makes it much harder to change those
names later. Removing fields is a breaking change that will trigger a parsing
error.

See [JSON Wire Safety](#json-wire-safety) below for more details.

# Format Description {#format}

## Representation of each type {#field-representation}

The following table shows how data is represented in JSON files.

<table>
  <thead>
    <tr>
      <th>Protobuf type</th>
      <th>JSON</th>
      <th>JSON example</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>message</td>
      <td>object</td>
      <td><code>{"fooBar": v, "g": null, ...}</code></td>
      <td>Generates JSON objects.
        <p>
        Keys are serialized as lowerCamelCase of field name. See
        <a href="#field-names">Field Names</a> for more special cases regarding mapping of field names
        to object keys.
        <p>
        Well-known types have special representations, as described in the <a href="#wkt">Well-known types table</a>.
        <p>
        <code>null</code> is valid for any field and leaves the field unset. See <a href="null-values">Null Values</a> for clarification about the semantic behavior of null values.
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
      <td>All keys are converted to strings (object keys in JSON can only be strings).</td>
    </tr>
    <tr>
      <td>repeated V</td>
      <td>array</td>
      <td><code>[v, ...]</code></td>
      <td></td>
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
        accepted. Empty strings are invalid. Exponent notation (such as <code>1e2</code>) is
        accepted in both quoted and unquoted forms.
      </td>
    </tr>
    <tr>
      <td>int64, fixed64, uint64</td>
      <td>string</td>
      <td><code>"1", "-10"</code></td>
      <td>JSON value will be a decimal string. Either numbers or strings are
        accepted. Empty strings are invalid. Exponent notation (such as <code>1e2</code>) is
        accepted in both quoted and unquoted forms.
      </td>
    </tr>
    <tr>
      <td>float, double</td>
      <td>number</td>
      <td><code>1.1, -10.0, 0, "NaN", "Infinity"</code></td>
      <td>JSON value will be a number or one of the special string values "NaN",
        "Infinity", and "-Infinity". Either numbers or strings are accepted.
        Empty strings are invalid. Exponent notation is also accepted.
      </td>
    </tr>
  </tbody>
<table>

<!-- This section html instead of markdown as workaround to bug that escapes all html between the first and last table -->
<h3 id="wkt">Well-Known Types</h3>

<p>Some messages in the <code>google.protobuf</code> package have a special representation
when represented in JSON.

<p>No message type outside of the <code>google.protobuf</code> package has a special ProtoJSON
handling; for example, types in <code>google.types</code> package are represented with the
neutral representation.

<table>
  <thead>
    <tr>
      <th>Message type</th>
      <th>JSON</th>
      <th>JSON example</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Any</td>
      <td><code>object</code></td>
      <td><code>{"@type": "url", "f": v, ... }</code></td>
      <td>See <a href="#any">Any</a></td>
    </tr>
    <tr>
        <td>Timestamp</td>
        <td>string</td>
        <td><code>"1972-01-01T10:00:20.021Z"</code></td>
        <td>Uses RFC 3339 (see <a href="#rfc3339-timestamp">clarification</a>). Generated output will always be Z-normalized
          with 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are
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
        nanoseconds precision and the suffix "s" is required. This is <b>not</b> RFC 3339
        'duration' format (see <a href="#rfc3339-duration">Durations</a> for clarification).
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
      <td>Wrappers use the same representation in JSON as the wrapped primitive
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
        <a href="/reference/protobuf/google.protobuf#value">google.protobuf.Value</a>
        for details.
      </td>
    </tr>
    <tr>
      <td>NullValue</td>
      <td>null</td>
      <td></td>
      <td>JSON null. Special case of the <a href="#null-values">null parsing behavior</a>.</td>
    </tr>
    <tr>
      <td>Empty</td>
      <td>object</td>
      <td><code>{}</code> <b>(not special cased)</b></td>
      <td>An empty JSON object</td>
    </tr>
  </tbody>
</table>

## Field names as JSON keys {#field-names}

Message field names are mapped to lowerCamelCase to be used as JSON object keys.
If the `json_name` field option is specified, the specified value will be used
as the key instead.

Parsers accept both the lowerCamelCase name (or the one specified by the
`json_name` option) and the original proto field name. This allows for a
serializer option to choose to print using the original field name (see
[JSON Options](#json-options)) and have the resulting output still be parsed
back by all spec parsers.

`\0 (nul)` is not allowed within a `json_name` value. For more on why, see
<a href="/news/2023-04-28#json-name">Stricter validation
for json_name</a>. Note that `\0` is still considered a legal character within
the value of a `string` field.

## Presence and default-values {#presence}

When generating JSON-encoded output from a protocol buffer, if a field supports
presence, serializers must emit the field value if and only if the corresponding
hazzer would return true.

If the field doesn't support field presence and has the default value (for
example any empty repeated field) serializers should omit it from the output. An
implementation may provide options to include fields with default values in the
output.

## Null values {#null-values}

Serializers should not emit `null` values.

Parsers accept `null` as a legal value for any field, with the following
behavior:

*   Any key validity checking should still occur (disallowing unknown fields).
*   The field should remain unset, as though it was not present in the input at
    all (hazzers should still return false where applicable).

The implication of this is that a `null` value for an implicit presence field
will behave the identically to the behavior to the default value of that field,
since there are no hazzers for those fields. For example, a value of `null` or
`[]` for a repeated field will cause key-validation checks, but both will
otherwise behave the same as if the field was not present in the JSON at all.

`null` values are not allowed within repeated fields.

`google.protobuf.NullValue` is a special exception to this behavior: `null` is
handled as a sentinel-present value for this type, and so a field of this type
must be handled by serializers and parsers under the standard presence behavior.
This behavior correspondingly allows `google.protobuf.Struct` and
`google.protobuf.Value` to losslessly round trip arbitrary JSON.

## Duplicate values {#duplicate-values}

Serializers must never serialize the same field multiple times, nor multiple
different cases in the same oneof in the same JSON object.

Parsers should accept the same field being duplicated, and the last value
provided should be retained. This also applies to "alternate spellings" of the
same field name.

If implementations cannot maintain the necessary information about field order
it is preferred to reject inputs with duplicate keys rather than have an
arbitrary value win. In some implementations maintaining field order of objects
may be impractical or infeasible, so it is strongly recommended that systems
avoid relying on specific behavior for duplicate fields in ProtoJSON where
possible.

## Out of range numeric values

When parsing a numeric value, if the number that is is parsed from the wire
doesn't fit in the corresponding type, the parser should fail to parse.

This includes any negative number for `uint32`, and numbers less than `INT_MIN`
or larger than `INT_MAX` for `int32`.

Values with nonzero fractional portions are not allowed for integer-typed
fields. Zero fractional portions are accepted. For example `1.0` is valid for an
int32 field, but `1.5` is not.

## Any {#any}

### Normal messages {#any-normal-messages}

For any message that is not a well-known type with a special JSON
representation, the message contained inside the `Any` is turned into a JSON
object with an additional `"@type"` field inserted that contains the `type_url`
that was set on the `Any`.

For example, if you have this message definition:

```proto
package x;
message Child { int32 x = 1; string y = 2; }
```

When an instance of Child is packed into an `Any`, the JSON representation is:

```json
{
  "@type": "type.googleapis.com/x.Child",
  "x": 1,
  "y": "hello world"
}
```

### Special-cased well-known types {#any-special-cases}

If the `Any` contains a well-known type that has a special JSON mapping, the
message is converted into the special representation and set as a field with key
"value".

For example, a `google.protobuf.Duration` that represents 3.1 seconds will be
represented by the string `"3.1s"` in the special case handling. When that
`Duration` is packed into an `Any` it will be serialized as:

```
{
  "@type": "type.googleapis.com/google.protobuf.Duration",
  "value": "3.1s"
}
```

Note that `google.protobuf.Empty` is not considered to have any special JSON
mapping; it is simply a normal message that has zero fields. This means the
expected representation of an `Empty` packed into an `Any` is `{"@type":
"type.googleapis.com/google.protobuf.Empty"}` and not `{"@type":
"type.googleapis.com/google.protobuf.Empty", "value": {}}`.

## ProtoJSON Wire Safety {#json-wire-safety}

When using ProtoJSON, only some schema changes are safe to make in a distributed
system. This contrasts with the same concepts applied to the
[the binary wire format](/programming-guides/editions#updating).

### JSON Wire-unsafe Changes {#wire-unsafe}

Wire-unsafe changes are schema changes that will break if you parse data that
was serialized using the old schema with a parser that is using the new schema
(or vice versa). You should almost never do this shape of schema change.

*   Changing a field to or from an extension of same number and type is not
    safe.
*   Changing a field between `string` and `bytes` is not safe.
*   Changing a field between a message type and `bytes` is not safe.
*   Changing any field from `optional` to `repeated` is not safe.
*   Changing a field between a `map<K, V>` and the corresponding `repeated`
    message field is not safe.
*   Moving fields into an existing `oneof` is not safe.

### JSON Wire-safe Changes {#wire-safe}

Wire-safe changes are ones where it is fully safe to evolve the schema in this
way without risk of data loss or new parse failures.

Note that nearly all wire-safe changes may be a breaking change to application
code. For example, adding a value to a preexisting enum would be a compilation
break for any code with an exhaustive switch on that enum. For that reason,
Google may avoid making some of these types of changes on public messages. The
AIPs contain guidance for which of these changes are safe to make there.

*   Changing a single `optional` field into a member of a **new** `oneof` is
    safe.
*   Changing a `oneof` which contains only one field to an `optional` field is
    safe.
*   Changing a field between any of `int32`, `sint32`, `sfixed32`, `fixed32` is
    safe.
*   Changing a field between any of `int64`, `sint64`, `sfixed64`, `fixed64` is
    safe.
*   Changing a field number is safe (as the field numbers are not used in the
    ProtoJSON format), but still strongly discouraged since it is very unsafe in
    the binary wire format.
*   Adding values to an enum is safe if the "Emit enum values as integers" is
    set on all relevant clients (see [options](#json-options))

### JSON Wire-compatible Changes (Conditionally safe) {#conditionally-safe}

Unlike wire-safe changes, wire-compatible means that the same data can be parsed
both before and after a given change. However, a client that reads it will get
lossy data under this shape of change. For example, changing an int32 to an
int64 is a compatible change, but if a value larger than INT32_MAX is written, a
client that reads it as an int32 will discard the high order bits.

You can make compatible changes to your schema only if you manage the roll out
to your system carefully. For example, you may change an int32 to an int64 but
ensure you continue to only write legal int32 values until the new schema is
deployed to all endpoints, and then start writing larger values after that.

#### Compatible But With Unknown Field Handling Problems {#compatible-ish}

Unlike the binary wire format, ProtoJSON implementations generally do not
propagate unknown fields. This means that adding to schemas is generally
compatible but will result in parse failures if a client using the old schema
observes the new content.

This means you can add to your schema, but you cannot safely start writing them
until you know the schema has been deployed to the relevant client or server (or
that the relevant clients set an Ignore Unknown Fields flag, discussed
[below](#json-options)).

*   Adding and removing fields is considered compatible with this caveat.
*   Removing enum values is considered compatible with this caveat.

#### Compatible But Potentially Lossy {#compatible-lossy}

*   Changing between any of the 32-bit integers (`int32`, `uint32`, `sint32`,
    `sfixed32`, `fixed32`) and any of the 64-bit integers ( `int64`, `uint64`,
    `sint64`, `sfixed32`) is a compatible change.
    *   If a number is parsed from the wire that doesn't fit in the
        corresponding type, a parse failure will occur.
    *   Unlike binary wire format, `bool` is not compatible with integers.
    *   Note that the int64 and uint64 types are quoted by default to avoid
        precision loss when handled as a double or JavaScript number, and the 32
        bit types are unquoted by default. Conformant parsers will accept either
        quoted or unquoted for all integer types, but nonconformant
        implementations may mishandle this case and not handle quoted-int32s or
        unquoted-int64s, so caution should be taken.
*   `enum` may be conditionally compatible with `string`
    *   If "enums-as-ints" flag is used by any client, then enums will instead
        be compatible with the integer types instead.

## RFC 3339 Clarifications {#rfc3339}

### Timestamps {#rfc3339-timestamp}

ProtoJSON timestamps use the RFC 3339 timestamp format. Unfortunately, some
ambiguity in the RFC 3339 spec has created a few edge cases where various other
RFC 3339 implementations do not agree on whether or not the format is legal.

[RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) intends to declare a strict
subset of ISO-8601 format, and some additional ambiguity was created since RFC
3339 was published in 2002 and then ISO-8601 was subsequently revised without
any corresponding revisions of RFC 3339.

Most notably, ISO-8601-1988 contains this note:

> In date and time representations lower case characters may be used when upper
> case characters are not available.

It is ambiguous whether this note is suggesting that parsers should accept
lowercase letters in general, or if it is only suggesting that lowercase letters
may be used as a substitute in environments where uppercase cannot be
technically used. RFC 3339 contains a note that intends to clarify the
interpretation to be that lowercase letters should be accepted in general.

ISO-8601-2019 does not contain the corresponding note and is unambiguous that
lowercase letters are not allowed.

This created some confusion for all libraries that declare they support RFC
3339: today RFC 3339 declares it is a profile of ISO-8601 but contains a
clarifying note referencing text that is not present in the latest ISO-8601
spec.

ProtoJSON spec takes the decision that the timestamp format is the stricter
definition of "RFC 3339 as a profile of ISO-8601-2019". Some Protobuf
implementations may be non-conformant by using a timestamp parsing
implementation that is implemented as "RFC 3339 as a profile of ISO-8601-1988,"
which will accept a few additional edge cases.

For consistent interoperability, parsers should only accept the stricter subset
format where possible. When using a non-conformant implementation that accepts
the laxer definition, strongly avoid relying on the additional edge cases being
accepted.

### Durations {#rfc3339-duration}

RFC 3339 also defines a duration format, but unfortunately the RFC 3339 duration
format does not have any way to express sub-second resolution.

The ProtoJSON duration encoding is directly inspired by RFC 3339 `dur-seconds`
representation, but it is able to encode nanosecond precision. For integer
number of seconds the two representations may match (like `10s`), but the
ProtoJSON durations accept fractional values and conformant implementations must
precisely represent nanosecond precision (like `10.500000001s`).

## JSON Options {#json-options}

A conformant protobuf JSON implementation may provide the following options:

*   **Always emit fields without presence**: Fields that don't support presence
    and that have their default value are omitted by default in JSON output (for
    example, an implicit presence integer with a 0 value, implicit presence
    string fields that are empty strings, and empty repeated and map fields). An
    implementation may provide an option to override this behavior and output
    fields with their default values.

    As of v25.x, the C++, Java, and Python implementations are nonconformant, as
    this flag affects proto2 `optional` fields but not proto3 `optional` fields.
    A fix is planned for a future release.

*   **Ignore unknown fields**: The protobuf JSON parser should reject unknown
    fields by default but may provide an option to ignore unknown fields in
    parsing.

*   **Use proto field name instead of lowerCamelCase name**: By default the
    protobuf JSON printer should convert the field name to lowerCamelCase and
    use that as the JSON name. An implementation may provide an option to use
    proto field name as the JSON name instead. Protobuf JSON parsers are
    required to accept both the converted lowerCamelCase name and the proto
    field name.

*   **Emit enum values as integers instead of strings**: The name of an enum
    value is used by default in JSON output. An option may be provided to use
    the numeric value of the enum value instead.
