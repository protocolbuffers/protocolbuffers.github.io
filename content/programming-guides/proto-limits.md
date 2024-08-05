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

Message with only singular proto fields (such as Boolean):

*   ~2100 fields (proto2)
*   ~3100 (proto3 without using optional fields)

Empty message extended by singular fields (such as Boolean):

*   ~4100 fields (proto2)

Extensions are supported
[only by proto2](/programming-guides/version-comparison#extensionsany).

To test this limitation, create a proto message with more than the upper bound
number of fields and compile using a Java proto rule. The limit comes from JVM
specs.

## Number of Values in an Enum {#enum}

The lowest limit is ~1700 values, in Java (fix available
go/java-large-enum-support). Other languages have different limits.

## Total Size of the Message {#total}

Any proto in serialized form must be <2GiB, as that is the maximum size
supported by all implementations. It's recommended to bound request and response
sizes.

## Depth Limit for Proto Unmarshaling {#depth}

*   Java: 100
*   C++: 100
*   Go:
    10000
    (there is a plan to reduce this to 100)

If you try to unmarshal a message that is nested deeper than the depth limit,
the unmarshaling will fail.
