+++
title = "Proto Serialization Is Not Canonical"
weight = 88
description = "Explains how serialization works and why it is not canonical."
type = "docs"
+++

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'esrauch' reviewed: '2025-01-09' }
*-->

Many people want a serialized proto to canonically represent the contents of
that proto. Use cases include:

*   using a serialized proto as a key in a hash table
*   taking a fingerprint or checksum of a serialized proto
*   comparing serialized payloads as a way of checking message equality

Unfortunately, *protobuf serialization is not (and cannot be) canonical*. There
are a few notable exceptions, such as MapReduce, but in general you should
generally think of proto serialization as unstable. This page explains why.

## Deterministic is not Canonical

Deterministic serialization is not canonical. The serializer can generate
different output for many reasons, including but not limited to the following
variations:

1.  The protobuf schema changes in any way.
1.  The application being built changes in any way.
1.  The binary is built with different flags (eg. opt vs. debug).
1.  The protobuf library is updated.

This means that hashes of serialized protos are fragile and not stable across
time or space.

There are many reasons why the serialized output can change. The above list is
not exhaustive. Some of them are inherent difficulties in the problem space that
would make it inefficient or impossible to guarantee canonical serialization
even if we wanted to. Others are things we intentionally leave undefined to
allow for optimization opportunities.

## Inherent Barriers to Stable Serialization

Protobuf objects preserve unknown fields to provide forward and backward
compatibility. The handling of unknown fields is a primary obstacle to canonical
serialization.

In the wire format, bytes fields and nested sub-messages use the same wire type.
This ambiguity makes it impossible to correctly canonicalize messages stored in
the unknown field set. Since the exact same contents may be either one, it is
impossible to know whether to treat it as a message and recurse down or not.

For efficiency, implementations typically serialize unknown fields after known
fields. Canonical serialization, however, would require interleaving unknown
fields with known fields according to field number. This would impose
significant efficiency and code size costs on all users, even those not
requiring this feature.

## Things Intentionally Left Undefined

Even if canonical serialization was feasible (that is, if we could solve the
unknown field problem), we intentionally leave serialization order undefined to
allow for more optimization opportunities:

1.  If we can prove a field is never used in a binary, we can remove it from the
    schema completely and process it as an unknown field. This saves substantial
    code size and CPU cycles.
2.  There may be opportunities to optimize by serializing vectors of the same
    field together, even though this would break field number order.

To leave room for optimizations like this, we want to intentionally scramble
field order in some configurations, so that applications do not inappropriately
depend on field order.
