+++
title = "Enum Behavior"
weight = 55
description = "Explains how enums currently work in Protocol Buffers vs. how they should work."
type = "docs"
+++

Enums behave differently in different language libraries. This topic covers the
different behaviors as well as the plans to move protobufs to a state where they
are consistent across all languages. If you're looking for information on how to
use enums in general, see the corresponding sections in the
[proto2](/programming-guides/proto2#enum) and
[proto3](/programming-guides/proto3#enum) language guide
topics.

## Definitions {#definitions}

Enums have two distinct flavors (*open* and *closed*). They behave identically
except in their handling of unknown values. Practically, this means that simple
cases work the same, but some corner cases have interesting implications.

For the purpose of explanation, let us assume we have the following `.proto`
file (we are deliberately not specifying if this is a `syntax = "proto2"` or
`syntax = "proto3"` file right now):

```
enum Enum {
  A = 0;
  B = 1;
}

message Msg {
  optional Enum enum = 1;
}
```

The distinction between *open* and *closed* can be encapsulated in a single
question:

> What happens when a program parses binary data that contains field 1 with the
> value `2`?

*   **Open** enums will parse the value `2` and store it directly in the field.
    Accessor will report the field as being *set* and will return something that
    represents `2`.
*   **Closed** enums will parse the value `2` and store it in the message's
    unknown field set. Accessors will report the field as being *unset* and will
    return the enum's default value.

## Implications of *Closed* Enums

The behavior of *closed* enums has unexpected consequences when parsing a
repeated field. When a `repeated Enum` field is parsed all unknown values will
be placed in the
[unknown field](/programming-guides/proto3/#unknowns)
set. When it is serialized those unknown values will be written again, *but not
in their original place in the list*. For example, given the `.proto` file:

```
enum Enum {
  A = 0;
  B = 1;
}

message Msg {
  repeated Enum r = 1;
}
```

A wire format containing the values `[0, 2, 1, 2]` for field 1 will parse so
that the repeated field contains `[0, 1]` and the value `[2, 2]` will end up
stored as an unknown field. After reserializing the message, the wire format
will correspond to `[0, 1, 2, 2]`.

Similarly, maps with *closed* enums for their value will place entire entries
(key and value) in the unknown fields whenever the value is unknown.

## History {#history}

Prior to the introduction of `syntax = "proto3"` all enums were *closed*. Proto3
introduced *open* enums specifically because of the unexpected behavior that
*closed* enums cause.

## Specification {#spec}

The following specifies the behavior of conformant implementations for protobuf.
Because this is subtle, many implementations are out of conformance. See
[Known Issues](#known-issues) for details on how different implementations
behave.

*   When a `proto2` file imports an enum defined in a `proto2` file, that enum
    should be treated as **closed**.
*   When a `proto3` file imports an enum defined in a `proto3` file, that enum
    should be treated as **open**.
*   When a `proto3` file imports an enum defined in a `proto2` file, the
    `protoc` compiler will produce an error.
*   When a `proto2` file imports an enum defined in a `proto3` file, that enum
    should be treated as **open**.

## Known Issues {#known-issues}

### C++ {#cpp}

All known C++ releases are out of conformance. When a `proto2` file imports an
enum defined in a `proto3` file, C++ treats that field as a **closed** enum.

### C&#35; {#csharp}

All known C# releases are out of conformance. C# treats all enums as **open**.

### Java {#java}

All known Java releases are out of conformance. When a `proto2` file imports an
enum defined in a `proto3` file, Java treats that field as a **closed** enum.

> **NOTE:** Java's handling of **open** enums has surprising edge cases. Given
> the following definitions:
>
> ```
> syntax = "proto3";
>
> enum Enum {
>   A = 0;
>   B = 1;
> }
>
> message Msg {
>   repeated Enum name = 1;
> }
> ```
>
> Java will generate the methods `Enum getName()` and `int getNameValue()`. The
> method `getName` will return `Enum.UNRECOGNIZED` for values outside the known
> set (such as `2`), whereas `getNameValue` will return `2`.
>
> Similarly, Java will generate methods `Builder setName(Enum value)` and
> `Builder setNameValue(int value)`. The method `setName` will throw an
> exception when passed `Enum.UNRECOGNIZED`, whereas `setNameValue` will accept
> `2`.

### Kotlin {#java}

All known Kotlin releases are out of conformance. When a `proto2` file imports
an enum defined in a `proto3` file, Kotlin treats that field as a **closed**
enum.

Kotlin is built on Java and shares all of its oddities.

### Go {#go}

All known Go releases are out of conformance. Go treats all enums as **open**.

### JSPB {#jspb}

All known JSPB releases are out of conformance. JSPB treats all enums as
**open**.

### PHP {#php}

PHP is conformant.

### Python {#python}

After 4.22.0, Python is conformant.

In 4.21.x, Python is conformant by default, but setting
`PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` will cause it to be out of
conformance.

Before 4.21.0, Python is out of conformance.

When a `proto2` file imports an enum defined in a `proto3` file, non-conformant
Python versions treat that field as a **closed** enum.

### Ruby {#ruby}

All known Ruby releases are out of conformance. Ruby treats all enums as
**open**.

### Objective-C {#obj-c}

After 22.0, Objective-C is conformant.

Prior to 22.0, Objective-C was out of conformance. When a `proto2` file imported
an enum defined in a `proto3` file, it would treat that field as a **closed**
enum.

### Swift {#swift}

Swift is conformant.

### Dart {#dart}

Dart treats all enums as **closed**.
