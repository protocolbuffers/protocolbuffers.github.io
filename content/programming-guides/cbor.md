+++
title = "ProtoCBOR Format"
weight = 62
description = "Covers how to use the Protobuf to CBOR conversion utilities."
type = "docs"
+++

Protobuf supports a canonical encoding in CBOR, making it easier to share data
with systems that do not support the standard protobuf binary wire format and
have processing constraints that prefer more concise encoding.

ProtoCBOR Format is not as efficient as protobuf wire format.
Like ProtoJSON, the converter uses more CPU to encode and decode messages and encoded messages usually consume more space.
CBOR is a schemaless type-length-value (TLV) encoding, so it incurs more interpretive overhead than the standard wire format.

The encoding is described on a type-by-type basis in the table later in this topic.

When parsing CBOR-encoded data into a protocol buffer, if a value is missing or if its value is `null`, it will be interpreted as the corresponding [default value](/programming-guides/editions#default).

When generating CBOR-encoded output from a protocol buffer, if a protobuf field has the default value and if the field doesn't support field presence, it will be omitted from the output by default.
An implementation may provide options to include fields with default values in the output.

ProtoCBOR does support field presence.
A message type field in any edition of protobuf supports field presence and if set will appear in the output.
Proto3 implicit-presence scalar fields will only appear in the CBOR output if they are not set to the default value for that type.

<table>
  <tbody>
    <tr>
      <th>Protobuf</th>
      <th>CBOR</th>
      <th>CBOR example (diagnostic notation)</th>
      <th>Notes</th>
    </tr>
    <tr>
      <td>message</td>
      <td>map</td>
      <td><code>{/fooBar/ 0: v, /g/ 1: null, ...}</code></td>
      <td>Generates CBOR maps. Message field numbers are represented as CBOR `int` and become CBOR map keys.
        <code>null</code> is an accepted value for all field types and treated as the default value of the corresponding field type.
      </td>
    </tr>
    <tr>
      <td>enum</td>
      <td>int</td>
      <td><code>5</code></td>
      <td>The enum value as specified in proto is used and sent as a CBOR-encoded int.
      </td>
    </tr>
    <tr>
      <td>map&lt;K,V&gt;</td>
      <td>map</td>
      <td><code>{ * K => V}</code></td>
      <td>All keys are converted to their CBOR encodings.</td>
    </tr>
    <tr>
      <td>repeated V</td>
      <td>definite-length array</td>
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
      <td>bstr</td>
      <td><code>h'deadc0de'</code></td>
      <td>CBOR value will be the data encoded as a byte string.
      </td>
    </tr>
    <tr>
      <td>int32, int64, uint32, uint64</td>
      <td>int</td>
      <td><code>1, -10, 0</code></td>
      <td>CBOR value will use its variable length encodings for integers, which account for the first 60 entries of the initial byte jump table.
      Unsigned integers do not use the latter 32 entries of the initial 60 entries of the initial byte jump table.</td>
    </tr>
    <tr>
      <td>fixed32</td>
      <td>tagged little endian</td>
      <td><code>1, 10, 0</code></td>
      <td>CBOR value is <code>#6.70(bytes .size 4)</code> encoded as little-endian.</td>
      </td>
    </tr>
    <tr>
      <td>fixed64</td>
      <td>tagged little endian</td>
      <td><code>1, 10, 0</code></td>
      <td>CBOR value is <code>#6.71(bytes .size 8)</code> encoded as little-endian.</td>
      </td>
    </tr>
    <tr>
      <td>sfixed32</td>
      <td>tagged little endian</td>
      <td><code>1, -10, 0</code></td>
      <td>CBOR value is <code>#6.78(bytes .size 4)</code> encoded as little-endian.</td>
      </td>
    </tr>
    <tr>
      <td>sfixed64</td>
      <td>tagged little endian</td>
      <td><code>1, -10, 0</code></td>
      <td>CBOR value is <code>#6.79(bytes .size 8)</code> encoded as little-endian.</td>
      </td>
    </tr>
    <tr>
      <td>float, double</td>
      <td>float, double</td>
      <td><code>1.1, -10.0, 0, NaN, Infinity</code></td>
      <td>CBOR value will be the smallest lossless encoding of the floating point number, choosing between simple value, half-precision, single-precision, or double-precision.
      </td>
    </tr>
    <tr>
      <td>Any</td>
      <td><code>Generic value CBOR</code></td>
      <td><code>27([TypeURL, value])</code></td>
      <td>The URL of a binary `google.protobuf.Type` paired with the deserialized CBOR representation.
      Check <a href="/reference/protobuf/google.protobuf#cbor">google.protobuf.Any</a> for details.</td>
    </tr>
    <tr>
        <td>Timestamp</td>
        <td>#6.0(string)</td>
        <td><code>0("1972-01-01T10:00:20.021Z")</code></td>
        <td>Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits.
          Offsets other than "Z" are also accepted.
      </td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>#6.1002(map)</td>
      <td><code>1002({1: 17020, -9: 900000})</code></td>
      <td>A duration in RFC9581 format, limited to durations representable in `google.protobuf.Duration`.</td>
    </tr>
    <tr>
      <td>Struct</td>
      <td><code>map</code></td>
      <td><code>{ int => any }</code></td>
      <td>Any CBOR map with integer keys and respectively constrained values. See <code>struct.proto</code>.</td>
    </tr>
    <tr>
      <td>Wrapper types</td>
      <td>various types</td>
      <td><code>2, "2", "foo", true, "true", null, 0, ...</code></td>
      <td>Wrappers use the same representation in CBOR as the wrapped primitive
        type, except that <code>null</code> is allowed and preserved during data
        conversion and transfer.
      </td>
    </tr>
    <tr>
      <td>FieldMask</td>
      <td>string</td>
      <td><code>"f.fooBar,h"</code></td>
      <td>See <code>field_mask.proto</code>. TODO</td>
    </tr>
    <tr>
      <td>ListValue</td>
      <td>array</td>
      <td><code>[foo, bar, ...]</code></td>
      <td>Definite-length array.</td>
    </tr>
    <tr>
      <td>Value</td>
      <td>value</td>
      <td></td>
      <td>A CBOR value that is JSON-like, but structs have integer keys. Check
        <a href="/reference/protobuf/google.protobuf#value">google.protobuf.Value</a>
        for details.
      </td>
    </tr>
    <tr>
      <td>NullValue</td>
      <td>null</td>
      <td></td>
      <td>CBOR null</td>
    </tr>
    <tr>
      <td>Empty</td>
      <td>map</td>
      <td><code>{}</code></td>
      <td>An empty CBOR map, but represented differently as an Any value.</td>
    </tr>
  </tbody>
</table>

### CBOR Options {#json-options}

A conformant protobuf CBOR implementation may provide the following options:

*   **Ignore unknown fields**: The protobuf CBOR parser should reject unknown
    fields by default but may provide an option to ignore unknown fields in
    parsing.

*   **Emit validation code**: For registered CBOR tags that have further
    validation requirements, the protobuf CBOR message should have validation
    methods emitted.

