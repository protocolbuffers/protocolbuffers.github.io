+++
title = "Python Comparison"
weight = 751
linkTitle = "Python Comparison"
description = "Describes how Python compares objects."
type = "docs"
+++

Because of how proto data is serialized, you cannot rely on the wire
representation of a proto message instance to determine if its content is the
same as another instance. A subset of the ways that a wire-format proto message
instance can vary include the following:

*   The protobuf schema changes in certain ways.
*   A map field stores its values in a different order.
*   The binary is built with different flags (such as opt vs. debug).
*   The protobuf library is updated.

Because of these ways that serialized data can vary, determining equality
involves other methods.

## Comparison Methods {#methods}

You can compare protocol buffer messages for equality using the standard Python
`==` operator. Comparing two objects using the `==` operator compares with
`message.ListFields()`. When testing, you can use `self.assertEqual(msg1,
msg2)`.

Two messages are considered equal if they have the same type and all of their
corresponding fields are equal. The inequality operator `!=` is the exact
inverse of `==`.

Message equality is recursive: for two messages to be equal, any nested messages
must also be equal.

## Field Equality and Presence

The equality check for fields is value-based. For fields with
[explicit presence](#singular-explicit), the presence of a field is also taken
into account.

A field that is explicitly set to its default value is **not** considered equal
to a field that is unset.

For example, consider the following message which has an explicit presence
field:

```proto
edition = "2023";
message MyMessage {
  int32 value = 1; // 'explicit' presence by default in Editions
}
```

If you create two instances, one where `value` is unset and one where `value` is
explicitly set to `0`, they will not be equal:

```python
msg1 = MyMessage()
msg2 = MyMessage()
msg2.value = 0

assert not msg1.HasField("value")
assert msg2.HasField("value")
assert msg1 != msg2
```

This same principle applies to sub-message fields: an unset sub-message is not
equal to a sub-message that is present but empty (a default instance of the
sub-message class).

For fields with [implicit presence](#singular-implicit), since presence cannot
be tracked, the field is always compared by its value against the corresponding
field in the other message.

This behavior, where presence is part of the equality check, is different from
how some other languages or protobuf libraries might handle equality, where
unset fields and fields set to their default value are sometimes treated as
equivalent (often for wire-format compatibility). In Python, `==` performs a
stricter check.

## Other Field Types {#other-types}

*   **Repeated fields** are equal if they have the same number of elements and
    each corresponding element is equal. The order of elements matters.
*   **Map fields** are equal if they have the same set of key-value pairs. The
    order of pairs does not matter.
*   **Floating-point fields** (`float` and `double`) are compared by value. Be
    aware of the usual caveats with floating-point comparisons.
