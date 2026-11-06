+++
title = "Encoding"
weight = 60
description = "Explains how Protobuf encodes data to files or to the wire."
type = "docs"
+++

This document defines the Protobuf binary wire format.

It describes the serialization of structured data into a compact, binary
sequence of tag-payload pairs. This specification is intended to provide a
formal reference for implementers of Protobuf parsers and serializers.

## 1. Introduction

Protobuf is a language-neutral, platform-neutral, extensible mechanism for
serializing structured data. This document specifies the binary wire format for
Protobuf, defining how structured messages are represented as a compact sequence
of bytes.

The top design goals of the Protobuf binary wire format are supporting schema
evolvability in distributed systems, parser efficiency, reliable semantics, and
compactness.

### 1.1. Conformance

The requirements in this document define the behavior of a conforming Protobuf
parser and serializer. Existing implementations, including ones maintained by
Google, do not yet conform to every requirement as described here here. Google
intends to improve the conformance of its implementations over time; however,
because a conformance fix can change which inputs are accepted or how they are
interpreted, such fixes are often held until a breaking-change boundary in order
to reduce churn for customers. Readers SHOULD NOT infer that an implementation's
current behavior is permitted by this specification merely because it is widely
deployed.

Clarification about the behavior which is considered conformant in edge cases is
an ongoing effort; this document may be periodically updated correspondingly.

## 2. Terminology

This document uses the following terms:

*   **Varint**: A variable-width integer encoding format (specifically LEB128).
*   **Tag**: A combination of a field number and a wire type that identifies a
    field in the binary sequence.
*   **Payload**: The serialized data bytes associated with a specific tag.
*   **Record**: A single tag-payload pair in the sequence.
*   **Sequence**: The ordered series of zero or more records representing a
    serialized message.
*   **Reject**: The parser signals a parse failure of the entire sequence.
*   **Delimited message**: A nested message bounded by `DELIMITED_START` and
    `DELIMITED_END` tags. Historically these wire types were named
    `START_GROUP`/`END_GROUP` and the construct was called a "group".

## 3. Varint encoding: LEB128 {#varints}

Variable-width integers, or varints, are a fundamental encoding mechanism for
integers used in various positions in the Protobuf binary wire format. Protobuf
LEB128 allows encoding unsigned 64-bit integers using between one and ten bytes,
with smaller values consuming fewer bytes.

*Note on Bit Numbering*: Outside of the byte-level description of the encoding
algorithm in Section 3.1, when this document refers to a bit position in the
context of a varint (e.g. "bit 31", "bits 64 through 69"), it means the 0-based
position in the integer obtained by decoding the varint, with bit 0 being the
least significant. It does not refer to a position within the encoded bytes.

### 3.1. Encoding Algorithm

The varint encoding used by Protobuf is Little-Endian Base 128 (LEB128).

Each byte in the varint has a *continuation bit* (the most significant bit, MSB,
bit 7) that indicates if the byte that follows it is part of the varint. The
lower 7 bits (bits 0-6) carry the payload.

To encode an unsigned 64-bit integer *V* as a varint:

1.  Split *V* into 7-bit groups, starting from the least significant bits
    (little-endian order).
2.  For each 7-bit group, except the last one:
    *   Set the MSB (bit 7) to 1.
    *   Place the 7 bits of the group into the lower 7 bits (bits 0-6) of the
        byte.
3.  For the last 7-bit group (or the single byte if the value fits in 7 bits):
    *   Set the MSB (bit 7) to 0.
    *   Place the remaining bits into the lower 7 bits (bits 0-6) of the byte.

<span id="simple"></span>

#### Example:

*   The value 1 fits in a single 7-bit group, so the MSB is not set:

    ```
    0000 0001
    ^ MSB (0)
    ```

    On the wire, this is the single byte `0x01`.

*   The value 150 requires two 7-bit groups, so the MSB of the first byte is set
    to 1:

    ```
    10010110 00000001
    ^ MSB (1) ^ MSB (0)
    ```

    On the wire, this is the byte sequence `0x96 0x01`. To decode this back to
    150:

    1.  Drop the MSB from each byte (yielding `0010110` and `0000001`).
    2.  Since the 7-bit payloads are in little-endian order, reverse their order
        to big-endian (yielding `0000001` and `0010110`).
    3.  Concatenate the groups (yielding `00000010010110` in binary).
    4.  Interpret as an unsigned integer: 2<sup>7</sup> + 2<sup>4</sup> +
        2<sup>2</sup> + 2<sup>1</sup> = 128 + 16 + 4 + 2 = 150.

### 3.2. Decoding

A parser decodes a varint by reading bytes sequentially until it encounters a
byte with the MSB set to 0. The decoded value is reconstructed by concatenating
the 7-bit payloads of the bytes in little-endian order.

*   **Truncation**: If the input ends before a byte with the continuation bit
    clear has been read, the varint is truncated and the parser MUST reject.
    This applies equally to tags, length prefixes, and payload varints.
*   **Maximum Length**: No varint in the Protobuf wire format may be longer than
    10 bytes. Parsers MUST reject if the 10th byte of a varint has its
    continuation bit set. Tags (Section 4.2) and length prefixes (Section 5.3)
    are further limited to 5 bytes.
*   **Bit Range Enforcement**: 10 bytes of LEB128 can carry 70 bits, but
    Protobuf varints never represent more than 64 bits. Serializers MUST NOT set
    bits 64 through 69. Parsers MUST reject a varint in which any of bits 64
    through 69 is set, and MUST NOT produce a decoded value larger than
    `UINT64_MAX`.

### 3.3. Overlong Varints

Overlong varints occur when a value is encoded using more bytes than necessary
by padding the value with trailing zero bits (leading zero bits in the decoded
integer). In the binary representation, this is achieved by setting the
continuation bit (MSB=1) on bytes whose payload bits (0-6) are all zero. For
example, the value 1 can be represented as `0x01` (1 byte) or as the overlong
`0x81 0x00` (2 bytes), which decodes to the same value.

*   **Serializer Requirements**: Serializers SHOULD emit the shortest possible
    varint representation for a given value (the final byte SHOULD NOT be `0x00`
    unless it is the only byte, i.e. the value is zero).
*   **Parser Requirements**: Parsers MUST accept overlong varints and decode
    them to the same value as the shortest encoding, up to the maximum length
    permitted for the varint's position: 10 bytes for payload varints (Section
    5.1), 5 bytes for tags (Section 4.2) and length prefixes (Section 5.3).
    Parsers MUST reject a varint that exceeds the maximum for its position,
    regardless of whether the excess bytes carry non-zero payload bits.

<span id="cheat-sheet"></span>
<span id="cheat-sheet-key"></span>

## 4. Message Structure (Tag-Payload Sequence) {#structure}

A serialized Protobuf message is represented as a sequence of zero or more
records.

### 4.1. The Tag-Payload Sequence

Each record in the sequence is a Tag-Payload pair. The sequence is parsed
sequentially from start to end.

*   **No Header**: The wire format does not define any header or metadata for
    the sequence. The sequence starts directly with the first record's tag.
*   **Zero-Length Sequence**: A zero-length byte sequence is a valid sequence.

### 4.2. Tag Encoding

A Tag is a `uint32` where the low 3 bits are a `wire_type`, and the high 29 bits
represent the `field_number`. When encoded in the message sequence, a tag is
written as a varint-encoded value.

The tag value is computed as: `tag = (field_number << 3) | wire_type`

*   **Wire Type Space**: The wire type is stored in the lower 3 bits of the tag
    value, allowing for 8 possible wire types, though not all of these are used.
*   **Tag Varint Limits**:
    *   A tag is a `uint32`, leaving 29 bits for the field number; the maximum
        field number is therefore `2^29 - 1`. A 32-bit value needs at most 5
        varint bytes.
    *   Serializers MUST NOT emit a tag longer than 5 bytes, and MUST NOT set
        bits 32 through 34 of a 5-byte tag.
    *   Parsers MUST accept any tag of 1 to 5 bytes, including overlong
        encodings within that limit (e.g. `0x88 0x80 0x80 0x80 0x00` is a valid
        encoding of tag value 8).
    *   Parsers MUST reject a tag longer than 5 bytes, regardless of its decoded
        value (e.g. `0x88 0x80 0x80 0x80 0x80 0x00` MUST be rejected even though
        it would also decode to 8).
    *   Parsers MUST reject a tag in which any bit at index 32 or higher is set.
        Consequently a conforming parser never observes a field number greater
        than `2^29 - 1`.
    *   Parsers MUST accept every field number in the range 1 through
        `2^29 - 1`.

### 4.3. Field Number Restrictions

*   **Field number zero**: 0 is not allowed as a field number. When parsing a
    sequence with field number zero, the parser MUST reject.
*   **Reserved Range**: Field numbers 19000 through 19999 are reserved and are
    not allowed to be specified in `.proto` schemas. However, these field
    numbers are considered valid on the wire. Parsers MUST accept field numbers
    in this range if they appear in the sequence and MUST NOT be special cased
    (typically they should be handled the same as any other unknown field).

## 5. Wire Types and Payloads

The wire type specified in the tag determines the structure of the payload that
immediately follows.

### 5.1. Wire Type 0: VARINT {#int-types}

*   **Payload Structure**: A single varint, encoded as described in Section 3.
*   **Parser Requirements**:
    *   Parsers MUST accept any varint of 1 to 10 bytes, including overlong
        encodings within that limit, regardless of the field's declared type
        (`int32`, `bool`, and enum fields are held to exactly the same 10-byte
        limit as `uint64`).
    *   Parsers MUST reject if the varint continues past the 10th byte.
    *   Parsers MUST reject if any bit at index 64 or higher is set.
    *   Parsers MUST accept any combination of bits 0 through 63, regardless of
        the field's declared type. For interpretations narrower than 64 bits
        (`uint32`, `int32`, `sint32`, enums) the parser MUST silently discard
        the bits above the interpretation's width (see the per-type rules below)
        and MUST NOT reject.
    *   The accept/reject decision for a VARINT record MUST depend only on its
        encoding, never on how the schema interprets it.
*   **Serializer Requirements**:
    *   Serializers MUST NOT set any bit at index 64 or higher.
    *   Serializers SHOULD use the shortest possible varint representation.

A VARINT payload (wire type 0) is decoded as a 64-bit unsigned integer. This has
a number of different possible interpretations:

#### 5.1.1. Unknown Field Interpretation

If the field number is not defined in the parser's schema, the payload is
interpreted as a raw 64-bit bit pattern representing an unsigned 64-bit integer.

#### 5.1.2. Boolean {#bools-and-enums}

*   0 represents `false` and any non-zero value represents `true`.
*   Serializers MUST emit `1` for `true` (and `0` for `false` when serialized).
*   Parsers MUST decode the full 64-bit value and treat any non-zero value as
    `true` (including values whose only set bits are at index 32 or higher).

#### 5.1.3. Uint32

*   The value is the least significant 32 bits of the integer.
*   Serializers MUST NOT set bits at bit index 32 or higher.
*   Parsers MUST silently discard any bits at bit index 32 or higher when
    decoding `uint32`, and MUST NOT reject.

#### 5.1.4. Uint64

*   The value is the decoded integer itself.

#### 5.1.5. Int32 (Two's Complement) {#signed-ints}

*   The value is interpreted as the lower 32 bits with two's-complement
    interpretation.
*   For negative values, serializers MUST sign-extend the value to 64 bits and
    encode it as a 64-bit two's complement integer (which will always be
    represented as a 10-byte varint).
*   Parsers MUST behave as though the varint was parsed as `Uint64` and the low
    32 bits interpreted as a 32-bit two's complement signed integer. Parsers
    MUST NOT reject an `int32` record on account of bits 32 through 63 being
    set, whether or not they form a valid sign extension.

#### 5.1.6. Int64 (Two's Complement)

*   The value is interpreted as a signed integer using two's complement
    representation.
*   For negative values, serializers MUST encode the value as a 64-bit two's
    complement integer (which will always be represented as a 10-byte varint).

#### 5.1.7. Sint32 (Zigzag)

*   This type uses "ZigZag" encoding to map signed 32-bit integers to unsigned
    integers. This ensures that small negative numbers also result in small
    varint values (consuming fewer bytes on the wire) by mapping them to small
    positive integers, rather than the 10-byte representation used by standard
    negative `int32` two's complement sign extension.
*   **ZigZag Mapping**: Positive integers `p` are encoded as `2 * p` (even
    numbers), while negative integers `n` are encoded as `2 * |n| - 1` (odd
    numbers).
*   **Encoding Formula**: `z = (n << 1) ^ (n >> 31)` *(where `>>` is an
    arithmetic right shift, propagating the sign bit).*
*   **Decoding Formula**: `n = (z >> 1) ^ -(z & 1)` *(where `>>` is a logical
    right shift).*
*   **Truncation Order**: If a parser decodes a varint payload for a `sint32`
    field, and the decoded varint value *z* has bits set at bit index 32 or
    higher (0-indexed) (meaning it represents a value >= 2^32), the parser MUST
    truncate the value *z* to 32 bits (discarding all bits at index 32 and
    higher, leaving a 32-bit unsigned value) *before* applying the ZigZag
    decoding formula. Performing the ZigZag decode step first and then
    truncating the result is non-conformant and will yield incorrect values.
*   **Compatibility**: ZigZag encoding is performed before varint serialization.
    On the wire, `sint32` is indistinguishable from `uint32` of the same wire
    type 0; the interpretation MUST be driven by the schema.

#### 5.1.8. Sint64 (Zigzag)

*   This type uses "ZigZag" encoding to map signed 64-bit integers to unsigned
    integers.
*   **ZigZag Mapping**: Positive integers `p` are encoded as `2 * p` (even
    numbers), while negative integers `n` are encoded as `2 * |n| - 1` (odd
    numbers).
*   **Encoding Formula**: `z = (n << 1) ^ (n >> 63)` *(where `>>` is an
    arithmetic right shift, propagating the sign bit).*
*   **Decoding Formula**: `n = (z >> 1) ^ -(z & 1)` *(where `>>` is a logical
    right shift).*
*   **Compatibility**: ZigZag encoding is performed before varint serialization.
    On the wire, `sint64` is indistinguishable from `uint64` of the same wire
    type 0; the interpretation MUST be driven by the schema.

#### 5.1.9. Enums

Enums are encoded identically as `int32` on the wire as described in 5.1.5.

*   **Encoding**: The enum values are mapped to signed integers. Negative values
    MUST be sign-extended to 64 bits and encoded as a 10-byte varint on the
    wire, identical to `int32`. Parsers MUST decode enum values exactly as
    `int32` (Section 5.1.5), including truncation of bits 32 through 63, before
    applying open/closed enum handling.
*   **Unrecognized Values**: If a parser decodes an integer value that is not
    defined as a known enum constant in the schema, the parser MUST handle the
    value according to the schema's open or closed enum semantics (see Section
    6.11).

### 5.2. Wire Type 1: FIXED64 {#non-varints}

*   **Payload Structure**: Exactly 8 bytes of data.
*   **Parser Requirements**: Parsers MUST read exactly 8 bytes of payload. If
    the sequence ends before 8 bytes can be read, the parser MUST reject.

A FIXED64 payload (wire type 1) is exactly 8 bytes of data. It has a number of
different possible interpretations:

#### 5.2.1. Unknown Field Interpretation

If the field number is not defined in the parser's schema, the payload is
interpreted as a raw 64-bit bit pattern.

#### 5.2.2. fixed64

The 8 bytes are interpreted directly as a 64-bit unsigned integer in
little-endian order.

#### 5.2.3. sfixed64

The 8 bytes are interpreted as a two's-complement 64-bit integer in
little-endian order.

#### 5.2.4. double

The 8 bytes are interpreted as an IEEE-754 binary64 floating point number in
little-endian order.

### 5.3. Wire Type 2: LENGTH_PREFIXED {#length-types}

The `LENGTH_PREFIXED` payload is a varint *length* immediately followed by
exactly that many bytes. The length is an unsigned value that MUST be in the
range `[0, INT32_MAX]`; payloads larger than `INT32_MAX` bytes are not
representable.

*   Serializers MUST NOT emit a length prefix longer than 5 bytes, and MUST NOT
    set any bit at index 31 or higher.
*   Parsers MUST accept any length prefix of 1 to 5 bytes, including overlong
    encodings within that limit (e.g. `0x80 0x80 0x80 0x80 0x00` is a valid
    encoding of length 0).
*   Parsers MUST reject a length prefix longer than 5 bytes, regardless of its
    decoded value.
*   Parsers MUST reject a length prefix in which any bit at index 31 or higher
    is set. (Bit 31 would correspond to a negative `int32` length; bits 32
    through 34 exceed the `int32` range entirely.)
*   If the input ends before *length* bytes have been read, the parser MUST
    reject.
*   A length of `0` is valid.

It has a number of different possible interpretations:

#### 5.3.1. Unknown Field Interpretation

The payload is interpreted as a raw sequence of bytes.

#### 5.3.2. Bytes

The payload is interpreted as an arbitrary byte sequence (commonly represented
as a byte array).

#### 5.3.3. UTF-8 Strings

The payload is interpreted as UTF-8 encoded text. See Section 6.7 for discussion
on validation.

#### 5.3.4. Embedded Messages (Submessages) {#embedded}

The payload is interpreted as a nested Protobuf message (represented as a
sequence of tag-payload records). Parsers recursively parse this payload using
the corresponding schema of the nested message type. The length prefix of the
parent `LENGTH_PREFIXED` field defines the boundary of the submessage sequence.

#### 5.3.5. Packed Repeated Fields {#packed}

For repeated fields of a primitive type that would otherwise be encoded as a
`VARINT`, `FIXED32`, or `FIXED64` payload (including enums and bools), the
values may be packed into a single `LENGTH_PREFIXED` record. The payload is
interpreted as a sequence of values concatenated back-to-back.

Other payload types do not use packed representation: this means that messages
and strings are never considered to be in the packed representation.

Serializers SHOULD NOT emit 0-length payloads for empty repeated lists. Parsers
MUST decode a 0-length packed payload as an empty repeated list (0 elements),
and MUST NOT reject.

When representing a packed payload of FIXED32 or FIXED64, the payload length in
bytes must be a multiple of the element size. Parsers MUST reject if the payload
length is not an exact multiple.

For varint types, the parser decodes varints one after another until the payload
bytes are exhausted. Individual varints within the packed payload MUST NOT
exceed 10 bytes. Parsers MUST reject if the final byte of the payload has its
MSB set (implying a truncated varint).

### 5.4. Wire Type 3 and 4: DELIMITED_START and DELIMITED_END {#groups}

This encoding was originally used only for the proto2 `group` construct.
Starting in Edition 2023 (`features.message_encoding = DELIMITED`) it is
available for any message-typed field and is no longer deprecated.

*   **Structure**:
    *   A record with wire type 3 (`DELIMITED_START`) has no payload. It acts as
        a marker denoting the start of a nested delimited message.
    *   All subsequent records in the sequence are considered part of the nested
        message until a matching record with wire type 4 (`DELIMITED_END`) is
        encountered.
    *   A record with wire type 4 (`DELIMITED_END`) has no payload and marks the
        end of the nested message.
*   **Nesting and Validation**:
    *   Delimited messages may be nested. Parsers MUST track the field numbers
        of currently open delimited messages (conceptually a stack).
    *   A `DELIMITED_END` tag MUST have the same field number as the innermost
        open `DELIMITED_START`. If the field numbers differ, the parser MUST
        reject.
    *   If a `DELIMITED_END` tag is encountered when no delimited message is
        open in the current scope, the parser MUST reject.
    *   If the current scope ends (end of the top-level sequence, or end of the
        enclosing `LENGTH_PREFIXED` payload) while a delimited message is still
        open, the parser MUST reject.
    *   **Scoping**: Each `LENGTH_PREFIXED` submessage payload is parsed as an
        independent scope with an empty stack. A `DELIMITED_END` inside a
        `LENGTH_PREFIXED` payload can only close a `DELIMITED_START` that
        appeared within that same payload.
    *   Depth of delimited nesting counts toward the recursion limit in Section
        8.1, including while skipping unknown delimited fields.
    *   *Note*: Some implementations are known to not validate matching field
        numbers on end tags; conforming parsers MUST perform this validation.

A delimited field is interpreted as a nested message, similar to an embedded
message. However, instead of being bounded by a length prefix, it continues
until the corresponding `DELIMITED_END` tag is reached. `LENGTH_PREFIXED` and
`DELIMITED` message encodings are not wire-compatible with each other: a message
field declared with `LENGTH_PREFIXED` encoding expects wire type 2, while a
message field declared with `DELIMITED` encoding expects wire type 3 (mismatched
wire types are treated as unknown fields per Section 6.6).

#### 5.4.1. Unknown Delimited Field Interpretation

If the field number is not defined in the parser's schema (or is defined with a
different wire type, such as a `LENGTH_PREFIXED` message field), the parser MUST
consume all records up to and including the matching `DELIMITED_END`, applying
the validation rules above, and handle the whole delimited range as a single
unknown field per Section 6.6 (retained or discarded).

#### 5.4.2. Delimited Message Interpretation

If the field number is defined in the parser's schema as a delimited message
field (a proto2 `group` or a message field with
`features.message_encoding = DELIMITED`), the parser recursively parses the
sequence of tag-payload records. The field numbers used within the sequence are
scoped by the delimited range, and do not collide with the same field number
used in the outer sequence.

### 5.5. Wire Type 5: FIXED32

A FIXED32 payload (wire type 5) is exactly 4 bytes of data.

Parsers MUST read exactly 4 bytes of payload. If the sequence ends before 4
bytes can be read, the parser MUST reject.

It has a number of different possible interpretations:

#### 5.5.1. Unknown Field Interpretation

If the field number is not defined in the parser's schema, the payload is
interpreted as a raw 32-bit bit pattern.

#### 5.5.2. fixed32

The 4 bytes are interpreted directly as a 32-bit unsigned integer in
little-endian order.

#### 5.5.3. sfixed32

The 4 bytes are interpreted as two's-complement signed 32-bit integer in
little-endian order.

#### 5.5.4. float

The 4 bytes are interpreted as an IEEE-754 binary32 in little-endian order.

### 5.6. Wire Type 6 and 7: Reserved and Unused

Wire types 6 and 7 are unused.

*   Serializers MUST NOT emit wire type 6 or 7.
*   Parsers MUST reject if they encounter wire type 6 or 7 in a tag.
    Unfortunately, parsers cannot be made graceful against the possibility of
    these tags being used in the future, since they cannot know the possible
    payload size to align the stream to continue parsing.

## 6. High-Level Message Semantics

This section defines how parsers and serializers map schemas to the binary wire
format, handling presence, field combinations, merging, and compatibility.

### 6.1. Field Presence Semantics {#optional}

Field presence defines the semantic meaning of a field being present zero times,
one time, or multiple times in the serialized sequence. The rules vary depending
on whether the schema defines the field with implicit presence or explicit
presence.

#### 6.1.1. Implicit Presence

Under implicit presence (sometimes referred to as "no presence"):

*   **Serializer Requirements**: Serializers SHOULD NOT emit fields that contain
    their default/zero value.
*   **Parser Requirements**:
    *   If a field is seen 0 times on the wire (absent from the sequence), the
        parser MUST treat it as having the default zero value (e.g., 0 for
        numerics, empty string for strings, empty list for repeated fields).
    *   Parsers MUST accept explicit default/zero values on the wire and decode
        them.
    *   If a field is seen multiple times in the sequence, the last-wins
        semantic applies (see Section 6.2), including when a later record
        carries an explicit default/zero value (which overwrites any earlier
        non-zero value in the same parse pass).

#### 6.1.2. Explicit Presence

Under explicit presence (such as `optional` fields, message-typed fields, or
legacy `required` / `features.field_presence = LEGACY_REQUIRED` fields):

*   **Serializer Requirements**: Serializers MUST emit the value if the field is
    explicitly set by the application, even if it contains its default/zero
    value. If the field is not set, serializers MUST NOT emit it.
*   **Parser Requirements**:
    *   Parsers MUST track whether a field was seen on the wire, distinguishing
        semantically between "seen 0 times" (not set) and "seen 1 or more times"
        (set), even if the decoded value is the default/zero value.
    *   If a field is seen multiple times in the sequence, the last-wins
        semantic applies (see Section 6.2), and the field is marked as present.
    *   For legacy `required` fields, the wire encoding is identical to other
        explicit-presence fields; checking whether all `required` fields were
        set is a higher-level validation concern separate from wire decoding.

#### 6.1.3. Repeated Fields and Maps

Repeated fields and maps do not have presence semantics.

*   If they appear 0 times in the sequence, they represent an empty list or map.
*   If they appear multiple times, elements are concatenated (for repeated
    fields) or keys are merged/overwritten (for maps), as described in Sections
    6.4 and 6.5.

### 6.2. Field Merge and Last-Wins Semantics {#last-one-wins}

When a record is encountered multiple times (N > 1) for the same field number in
a sequence, parsers apply the following rules to merge or overwrite the values:

#### 6.2.1. Singular Primitive and String/Bytes Fields

If a singular (non-repeated) primitive or string/bytes field is encountered
multiple times in the sequence, the parser MUST implement a "last-wins"
semantic. The value of the last record for that field number in the sequence
MUST overwrite any previously parsed values for the same field.

#### 6.2.2. Singular Embedded Message Fields

If an embedded message field (wire type 2, `LENGTH_PREFIXED` or wire type 3,
`DELIMITED_START`) is encountered multiple times, the parser MUST NOT overwrite
the previous message. Instead, it MUST merge the new instance into the existing
one. Merging is performed recursively:

*   Singular scalar fields in the newer instance replace those in the older
    instance.
*   Nested embedded messages are merged recursively.
*   Repeated fields are concatenated.

For sequences produced by conforming serializers using the same schema, this
merging behavior ensures that parsing the concatenation of two valid binary
sequences is semantically equivalent to parsing them separately and then merging
the resulting objects (e.g., using `Message::MergeFrom`).

### 6.3. Oneof Semantics {#oneofs}

A `oneof` is a schema-level constraint where at most one of the corresponding
fields may be set at a time. The fields are still considered to be directly on
the containing parent message and are otherwise encoded as though they were not
in a oneof.

*   **Parser Requirement (Last-Wins & Merging)**: Parsers MUST implement the
    following rules when decoding fields belonging to a `oneof`:
    *   If a field belonging to a `oneof` is encountered, the parser MUST clear
        all *other* fields belonging to that `oneof` from the in-memory message
        representation (setting the oneof case to the new field).
    *   If the *same* `oneof` member is encountered again and no *other* member
        of that `oneof` was encountered in between, standard singular-field
        rules apply (Section 6.2): primitives and string/bytes overwrite,
        message-typed members merge.
    *   If a different member of the `oneof` intervened, the earlier value has
        already been cleared; the new record starts from an empty value and MUST
        NOT be merged with the earlier one. For example, for message-typed
        members `A` and `B`, the sequence `A1, B, A2` yields `A2` alone, not
        `merge(A1, A2)`.
*   **Serializer Requirements**: Serializers SHOULD NOT emit multiple active
    arms of a `oneof` in a single serialization pass.
*   **Schema Restrictions**: The Protobuf schema definition language does not
    allow `repeated` fields to be members of a `oneof`.

### 6.4. Repeated Fields {#repeated}

*   **Interleaving**: Records for a `repeated` field do not need to appear
    consecutively in the sequence. They MAY be interleaved with records for
    other fields.
*   **Order Preservation**: Parsers MUST maintain the relative order of elements
    for the *same* repeated field as they appear in the sequence. The parser
    MUST NOT reorder elements of a repeated field.
*   **Non-Consecutive Records**: If a repeated field is serialized as
    non-packed, each element is a separate record. Parsers MUST append each
    encountered element to the list in order.
*   **Mixed Packed and Non-Packed**: If a repeated field is serialized as a mix
    of packed and non-packed records, parsers MUST decode each element and
    concatenate them in order.

### 6.5. Maps {#maps}

A map field is represented on the wire as though it were a repeated message with
a key field and a value field.

*   **Entry Schema**: For a `map<KeyType, ValueType> map_field = N;`, the wire
    representation is equivalent to the following:

    ```proto
    message MapEntry {
      KeyType key = 1;
      ValueType value = 2;
    }
    repeated MapEntry map_field = N;
    ```

    Each `MapEntry` record on the wire MUST be encoded as `LENGTH_PREFIXED`
    (wire type 2); `DELIMITED` encoding (wire types 3 and 4) is not used for map
    entries.

*   **Serialization Order**: Unlike normal repeated fields, the order of entries
    in a map is arbitrary, and may not be preserved in a parse-serialize round
    trip.

*   **Duplicate Keys (Last-Wins, No Merge)**: If the sequence contains multiple
    entries with the same key, the parser MUST keep only the value from the last
    such entry. This applies even when the value type is a message: the later
    value **replaces** the earlier one and is NOT merged into it (contrast with
    Section 6.2.2). Merging applies only *within* a single entry payload (see
    "Duplicate Fields within MapEntry" below).

*   **Missing Key or Value**: If a `MapEntry` payload on the wire is missing the
    `key` field (field number 1) or the `value` field (field number 2), the
    parser MUST treat it as though the default/zero value for the corresponding
    type (e.g., `0` for numerics, `""` for strings, or default messages).

*   **Presence of Keys and Values**: Parsers MAY retain or discard the presence
    information for the keys and values.

*   **Duplicate Fields within MapEntry**: If a `MapEntry` payload contains
    duplicate `key` fields or duplicate `value` fields (due to a non-conforming
    writer), the parser MUST apply the standard last-wins (or merging, for
    message values) rules *within* that `MapEntry` payload before inserting the
    parsed entry into the map.

*   **Unknown Fields within MapEntry**: If a `MapEntry` payload contains records
    with field numbers other than 1 or 2 (unknown fields), the parser MUST NOT
    reject. The parser MAY silently discard these unknown fields or it MAY
    preserve them (which differs from standard message unknown field
    preservation rules, see Section 6.6).

### 6.6. Unknown Field Handling

An unknown field is a field encountered in the sequence whose field number is
not defined in the parser's schema, or whose wire type does not match the
expected wire type for that field number.

*   **Parser Tolerance**: Parsers MUST NOT reject when they encounter an unknown
    field by default.
*   **Skipping Unknown Fields**: Parsers MUST be able to silently skip unknown
    fields by reading the wire type from the tag and using it to determine the
    size of the payload to skip:
    *   For wire type 0 (VARINT): Read and discard the varint (up to 10 bytes).
    *   For wire type 1 (FIXED64): Skip exactly 8 bytes.
    *   For wire type 5 (FIXED32): Skip exactly 4 bytes.
    *   For wire type 2 (LENGTH_PREFIXED): Read the length prefix varint, and
        skip exactly that many bytes.
    *   For wire type 3 (DELIMITED_START): Recursively parse the sequence
        (skipping fields) until a matching wire type 4 (DELIMITED_END) is
        encountered with the same field number. Parsers MUST maintain a stack of
        open delimited message field numbers during this skip operation to
        validate correct nested closure and prevent CPU/stack exploitation.
*   **Retention**: Parsers SHOULD preserve unknown fields and attach them to the
    parsed message representation, allowing them to be preserved during a
    round-trip (parse and re-serialize).
    *   **Configurability**: Parsers MAY be configurable to discard or reject
        when unknown fields are observed.
*   **Unexpected Wire Types**: If a parser encounters a known field number but
    with an unexpected wire type, it MUST treat it as an unknown field.

### 6.7. UTF-8 Validation Semantics

For `string` fields (wire type 2, `LENGTH_PREFIXED` interpreted as UTF-8):

*   **Validation Levels**: Individual fields may have different levels of UTF-8
    validation.
    *   *Enforced Validation (Reject)*: The parser MUST validate that the
        payload bytes are valid UTF-8. If any one record for a field with UTF-8
        validation set is not valid UTF-8, the parser MUST reject the entire
        sequence (this is the behavior for strings in proto3 and for Editions
        with `features.utf8_validation = VERIFY`, which is the default starting
        from Edition 2023).
    *   *Best Effort*: The parser accepts invalid UTF-8 bytes and passes them to
        the application (this is the behavior for strings in proto2 and opt-in
        behavior under Editions).
*   **Serializer Requirements**: Under enforced validation, serializers MUST NOT
    emit a `string` field whose bytes are not valid UTF-8; serialization MUST
    fail instead.
*   **Non-Validated Strings**: When validation is not enforced and malformed
    UTF-8 is observed, implementations MUST either expose the raw bytes
    unmodified or substitute U+FFFD for malformed sequences.

### 6.8. Packed/Non-Packed Compatibility

For repeated primitive fields (numeric scalar types), there are two legal
representations: the packed (wire type 2, `LENGTH_PREFIXED`) and non-packed
(multiple records with the field's wire type) representations.

*   **Parser Requirement**: Parsers MUST accept both packed and non-packed
    representations for any repeated primitive field, including a mix of each.
    If a parser encounters multiple records for the same repeated field, it MUST
    concatenate the values in the order they appear in the sequence.
*   **Serializer Requirement**: Serializers SHOULD emit the preferred
    representation as determined by the schema.

### 6.9. Extensions

Extensions (as defined in proto2 and Editions) allow defining fields for a
message outside the message's own definition.

*   **Wire Equivalence**: On the wire, extension fields are serialized as though
    they were normal fields. They use the standard tag-payload format, and their
    field numbers are allocated from the same valid field number range.
*   **Parser Behavior**: A parser decodes extension fields using the same rules
    as normal fields. If the parser has the extension definition, it interprets
    the payload accordingly; otherwise, it treats it as an unknown field.

### 6.10. Reserved Fields

Schemas can reserve field numbers or names to prevent them from being reused.

*   **Wire Behavior**: Reserved fields do not have a special representation on
    the wire.
*   **Parser Requirement**: If a parser encounters a field number on the wire
    that is marked as `reserved` in the schema, it MUST treat it as an unknown
    field. The parser MUST NOT reject due to the presence of a reserved field
    number.

### 6.11. Open and Closed Enums

When a parser decodes an integer for an enum field that does not match any known
enum constants defined in the schema, the behavior depends on whether the enum
is open or closed.

*   **Open Enums**: Default in proto3 and Edition 2023.
    *   **Parser Behavior**: The parser MUST preserve the unrecognized value
        directly in the field and expose it to the application (as a cast to the
        enum type).
    *   **Re-serialization**: The unrecognized enum value MUST be re-serialized
        in its original field position in the tag-payload sequence.
*   **Closed Enums**: Legacy behavior, default in proto2.
    *   **Parser Behavior**: The parser MUST treat the unrecognized value as an
        unknown field. The parser MUST NOT store the value in the enum field,
        and instead MUST record the unrecognized value in the message's unknown
        fields set as a `VARINT` (wire type 0) record for that field number
        (including when the unrecognized value was decoded from a packed
        `LENGTH_PREFIXED` repeated field). Because the unrecognized value is
        routed to unknown fields rather than modifying the field, any previously
        parsed known value for a singular closed enum field (or any currently
        active `oneof` arm) remains unchanged.
    *   **Re-serialization Side Effects**: Because unknown enum values are moved
        to unknown fields, they will be serialized at the end of the message
        (after all known fields), rather than in their original position. For
        repeated closed enums, if a sequence contains interleaved known and
        unknown values, parsing and re-serializing will reorder the values
        (unknown values will be grouped and written at the end).

## 7. Serialization Guarantees

This section defines the guarantees (and lack thereof) provided by the
serialization process.

<span id="implications"></span>

### 7.1. Non-Determinism {#order}

*   **No Byte Stability**: The binary representation of a serialized message is
    NOT guaranteed to be stable across different serializations, even on the
    same in-memory message instance. See
    [Proto Serialization Is Not Canonical](/programming-guides/serialization-not-canonical)
    for further details.
    *   **In-Instance Non-Determinism**: Repeated serialization invocations on
        the exact same message object instance in memory may not produce
        identical byte outputs.
    *   **Deterministic Serialization**: Deterministic serialization options
        (when available in libraries) only guarantee stable output for a
        specific compiled binary. The output remains subject to change across
        different compiler or library versions.
*   **No Order Guarantee**: Serializers MAY write fields in any order. The order
    of fields defined in a `.proto` schema file does NOT dictate serialization
    order. Parsers MUST be able to parse fields in any order.
*   **Implementation Variation**: Different language implementations or
    different versions of the same library may serialize fields in different
    orders.
*   **No Deterministic Comparison**: Applications MUST NOT rely on raw byte
    comparisons (e.g., direct string comparison, memory comparison `memcmp`, or
    cryptographic hashing of serialized bytes) to determine message equality.
    Semantically identical messages can serialize to different byte sequences.
    *   **Failed Assertions**: The following verification checks are NOT
        guaranteed to succeed and may fail for a message instance `foo`:
        *   `foo.SerializeAsString() == foo.SerializeAsString()`
        *   `Hash(foo.SerializeAsString()) == Hash(foo.SerializeAsString())`
*   **Scenarios for Different Serialized Outputs**: The following are common
    scenarios where logically equivalent messages `foo` and `bar` serialize to
    different byte outputs:
    *   One message contains unknown fields that the other does not (e.g., if
        one was parsed by an older schema version).
    *   The messages are serialized by implementations in different programming
        languages that write fields in a different order.
    *   The message has a map field (which has no serialization order
        guarantees).
    *   The message has a nested submessage that is itself serialized
        differently.
    *   The message contains a repeated field that is a concatenation of
        elements parsed from multiple interleaved records in a different order,
        or where one was packed and the other was expanded (since they parse to
        the same in-memory array but serialize to the preferred format).

### 7.2. Size Limits {#size-limit}

*   **INT32_MAX Limit**: The serialized size of a message MUST NOT exceed
    `INT32_MAX` (2^31 − 1) bytes. Serializers MUST fail serialization if the
    size would exceed this limit, and parsers MUST reject a sequence longer than
    this limit.
*   **Top-Level Sequence Limit**: Because the `LENGTH_PREFIXED` wire format
    (used for embedding submessages) uses a length prefix that is constrained to
    `INT32_MAX` (see Section 5.3), no message nested within another message can
    exceed this size. Consequently, parsers SHOULD enforce an `INT32_MAX` limit
    on top-level sequences as well, to ensure that a top-level message can
    always be legally nested as a submessage in future schemas.

## 8. Security Considerations

Implementations of Protobuf parsers must address several security risks inherent
in parsing untrusted binary sequences.

### 8.1. Unbounded Recursion (Deep Nesting)

An attacker can construct a malicious payload with deeply nested submessages
(either via `LENGTH_PREFIXED` wire type 2 or `DELIMITED_START`/`DELIMITED_END`
wire types 3/4) to cause stack overflow and Denial of Service (DoS).

*   **Depth Limit**: Parsers MUST implement a recursion depth limit. A default
    limit of 100 levels of nesting SHOULD be used for interoperability.
*   **Configuration**: Implementations MAY allow configuring the recursion depth
    limit.
*   **Validation of Delimited Fields**: For delimited fields (Section 5.4), the
    parser must track the depth stack even when skipping unknown delimited
    payloads to prevent recursion exploitation via malicious nested end tags.

### 8.2. Parser Differentials (Security Gateways)

If a system uses a security gateway (e.g., a proxy that filters fields) and a
backend service, differentials in how the gateway parser and the backend parser
handle the same binary sequence can lead to security bypasses.

*   **Interpretation Consistency**: There is intentionally no case where, within
    the limits of this specification, a valid binary sequence has more than one
    valid interpretation. However, parsers may differ in whether they accept or
    reject specific inputs (e.g., overlong varints, tags or length prefixes with
    set bits beyond their permitted range, or tags using 6 or more bytes on the
    wire).
*   **Gateway Disclaimer**: Security architectures SHOULD NOT rely solely on
    parser-level filtering for security guarantees unless both gateway and
    backend use identical parser implementations with identical schemas.
*   **Risks**:
    *   **Schema Skew**: Schema differences between the gateway and backend (for
        example, a field newly added to a pre-existing `oneof` in the backend
        but unknown to the gateway) can cause parser differentials where the
        gateway fails to clear/validate the field. This is working as designed
        behavior, and means that certain classes of parser differential
        behaviors in a distributed and evolving system will occur.
    *   **Practical Tolerances**: An attacker can craft a payload that is
        accepted by one parser (e.g., which tolerates overlong varints, allows
        6+ byte tags, or ignores high bits in tags) but rejected by another, or
        vice versa, potentially leading to security gateway bypass or backend
        Denial of Service (DoS) when a gateway forwards a payload that the
        backend rejects.
    *   **Out-of-Spec Behavior**: Implementations should not exhibit
        non-conformant behavior (such as tag wraparound due to truncation of
        bits above bit 31) when processing bit patterns that no conformant
        serializer will emit, but such behaviors are known to exist in practice
        and must not be relied on for security.

### 8.3. Truncated Messages (Memory Allocation Amplification)

An attacker can send a `LENGTH_PREFIXED` tag with a very large length prefix
(e.g., 2 GB) followed by a truncated sequence (few or no bytes).

*   **Amplification Attack**: If a parser eagerly allocates memory based on the
    declared `LENGTH_PREFIXED` prefix before reading the actual payload bytes,
    this can lead to memory exhaustion (DoS) with minimal attacker bandwidth.
*   **Allocation Safety**: Parsers SHOULD NOT eagerly allocate the full buffer
    space specified by a `LENGTH_PREFIXED` prefix unless they have verified that
    the sequence actually contains that many bytes, or they allocate memory
    incrementally as bytes are read.

## 9. Additional Topics

### 9.1. Message Streaming Formats (Length-Prefixed Streams)

For transmitting multiple messages over a stream (where the raw binary wire
format has no delimiting), a streaming protocol is required.

*   **Length-Prefixed Framing**: The standard streaming format prepends each
    message with its serialized size as a varint.
*   **Terminology Note**: In several APIs (e.g., Java's `writeDelimitedTo`),
    this streaming format is referred to as "delimited". However, to avoid
    confusion with the actual wire-level delimited format (see Section 5.4),
    this streaming protocol SHOULD be referred to as **Length-Prefixed
    Streaming**.
*   **Size Restriction**: Individual messages in the stream are subject to the
    `INT32_MAX` size limit.

### 9.2. Wire Compatible Schema Changes

When evolving a Protobuf schema, certain changes to field types are compatible
on the wire, meaning older parsers can read newer messages and vice versa.

*   **Reference**: For a comprehensive list of compatible and incompatible
    changes, refer to
    [Updating A Message Type](/programming-guides/editions#updating).
*   **General Rule**: Changes are generally compatible if the wire type remains
    the same (e.g., changing `int32` to `int64` since both are `VARINT`), or if
    the parser is designed to handle the transition (e.g., packed to non-packed
    repeated).
*   **Wire Type Mismatches**: Having the same wire type is a necessary but not
    sufficient condition for compatibility. For example, changing a field from
    `int32` to `sint32` is incompatible for negative numbers, even though both
    use wire type 0 (VARINT), because `int32` uses two's complement (10 bytes on
    wire for negative numbers) while `sint32` uses ZigZag encoding.
