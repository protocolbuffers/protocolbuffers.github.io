+++
title = "Proto Limits"
weight = 45
description = "Covers the limits to number of supported elements in proto schemas."
type = "docs"
+++

This topic documents the limits to the number of supported elements (fields,
enum values, and so on) in proto schemas.

This information is a collection of discovered limitations by many engineers,
but is not exhaustive and may be incorrect/outdated in some areas. As you
discover limitations in your work, contribute those to this document to help
others.

## Number of Fields {#fields}

All messages are limited to 65,535 fields.

Message with only singular proto fields (such as Boolean):

*   ~2100 fields (proto2)
*   ~3100 (proto3 without using optional fields)

Empty message extended by singular fields (such as Boolean):

*   ~4100 fields (proto2)

Extensions are not supported in proto3.

To test this limitation, create a proto message with more than the upper bound
number of fields and compile using a Java proto rule. The limit comes from JVM
specs.

## Field Numbers {#field-numbers}

Regular fields can use numbers from 1 to 2<sup>29</sup> - 1 (536,870,911),
because the field number shares a varint-encoded 32-bit tag with the 3-bit wire
type. For more on this, see the
[Encoding topic](/programming-guides/encoding#structure).

### MessageSet

Extensions of a message that uses `message_set_wire_format` (MessageSet) do not
have this limit, because their number is not packed into a tag: it is encoded as
a separate varint (`type_id`), so the wire format would allow numbers up to
2<sup>32</sup> - 1. Two limits of the descriptor representation apply instead:

1.  `FieldDescriptorProto.number` is an `int32`, so a number can be at most
    2<sup>31</sup> - 1 (`INT32_MAX`).
2.  `DescriptorProto.ExtensionRange.end` is an `int32` and is exclusive, so the
    largest number that any extension range can contain is 2<sup>31</sup> - 2
    (`INT32_MAX - 1`). `extensions 4 to max` on a MessageSet message resolves to
    this, and an explicit `extensions 4 to 2147483647` is rejected.

MessageSet is deprecated: do not use `message_set_wire_format` for new messages.

## Number of Values in an Enum {#enum}

The lowest limit is ~1700 values, in Java
. Other languages have different
limits.

## Total Size of the Message {#total}

Any proto in serialized form must be <2GiB, as that is the maximum size
supported by all implementations. It's recommended to bound request and response
sizes.

## Depth Limit for Proto Unmarshaling {#depth}

*   Java:
    100
*   C++:
    100
*   Go:
    10000
    (there is a plan to reduce this to 100)

If you try to unmarshal a message that is nested deeper than the depth limit,
the unmarshaling will fail.
