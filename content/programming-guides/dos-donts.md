+++
title = "Proto Best Practices"
weight = 90
description = "Shares vetted best practices for authoring Protocol Buffers."
type = "docs"
+++

Clients and servers are never updated at exactly the same time - even when you
try to update them at the same time. One or the
other may get rolled back. Don’t assume that you can make a breaking change and
it'll be okay because the client and server are in sync.

<a id="dont-re-use-a-tag-number"></a>

## **Don't** Re-use a Tag Number {#reuse-number}

Never re-use a tag number. It messes up deserialization. Even if you think no
one is using the field, don’t re-use a tag number. If the change was live ever,
there could be serialized versions of your proto in a log
somewhere. Or there could be old code in another server that will break.

<a id="do-reserve-tag-numbers-for-deleted-fields"></a>

## **Do** Reserve Tag Numbers for Deleted Fields {#reserve-tag-numbers}

When you delete a field that's no longer used, reserve its tag number so that no
one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. No
type required (lets you trim dependencies!). You can also reserve names to avoid
recycling now-deleted field names: `reserved "foo", "bar";`.

<a id="do-reserve-numbers-for-deleted-enum-values"></a>

## **Do** Reserve Numbers for Deleted Enum Values {#reserve-deleted-numbers}

When you delete an enum value that's no longer used, reserve its number so that
no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough.
You can also reserve names to avoid recycling now-deleted value names: `reserved
"FOO", "BAR";`.

<a id="dont-change-the-type-of-a-field"></a>

## **Don't** Change the Type of a Field {#change-type}

Almost never change the type of a field; it'll mess up deserialization, same as
re-using a tag number. The
[protobuf docs](/programming-guides/proto2#updating)
outline a small number of cases that are okay (for example, going between
`int32`, `uint32`, `int64` and `bool`). However, changing a field’s message type
**will break** unless the new message is a superset of the old one.

<a id="dont-add-a-required-field"></a>

## **Don't** Add a Required Field {#add-required}

Never add a required field, instead add `// required` to document the API
contract. Required fields are considered harmful by so many they were
removed from proto3 completely. Make all fields
optional or repeated. You never know how long a message type is going to last
and whether someone will be forced to fill in your required field with an empty
string or zero in four years when it’s no longer logically required but the
proto still says it is.

For proto3 there are no `required` fields, so this advice does not apply.

<a id="dont-make-a-message-with-lots-of-fields"></a>

## **Don't** Make a Message with Lots of Fields {#lots-of-fields}

Don't make a message with “lots” (think: hundreds) of fields. In C++ every field
adds roughly 65 bits to the in-memory object size whether it’s populated or not
(8 bytes for the pointer and, if the field is declared as optional, another bit
in a bitfield that keeps track of whether the field is set). When your proto
grows too large, the generated code may not even compile (for example, in Java
there is a hard limit on the size of a method
).

<a id="do-include-an-unspecified-value-in-an-enum"></a>

## **Do** Include an Unspecified Value in an Enum {#unspecified-enum}

Enums should include a default `FOO_UNSPECIFIED` value as the first value in the
declaration . When new values
are added to a proto2 enum, old clients will see the field as unset and the
getter will return the default value or the first-declared value if no default
exists . For consistent behavior with [proto enums][proto-enums],
the first declared enum value should be a default `FOO_UNSPECIFIED` value and
should use tag 0. It may be tempting to declare this default as a semantically
meaningful value but as a general rule, do not, to aid in the evolution of your
protocol as new enum values are added over time. All enum values declared under
a container message are in the same C++ namespace, so prefix the unspecified
value with the enum’s name to avoid compilation errors. If you'll never need
cross-language constants, an `int32` will preserve unknown values and generates
less code. Note that [proto enums][proto-enums] require the first value to be
zero and can round-trip (deserialize, serialize) an unknown enum value.

[example-unspecified]: http://cs/#search/&q=file:proto%20%22_UNSPECIFIED%20=%200%22&type=cs
[proto-enums]: /programming-guides/proto2#enum

<a id="dont-use-cc-macro-constants-for-enum-values"></a>

## **Don't** Use C/C++ Macro Constants for Enum Values {#macro-constants}

Using words that have already been defined by the C++ language - specifically,
in its headers such as `math.h`, may cause compilation errors if the `#include`
statement for one of those headers appears before the one for `.proto.h`. Avoid
using macro constants such as "`NULL`," "`NAN`," and "`DOMAIN`" as enum values.

<a id="do-use-well-known-types-and-common-types"></a>

## **Do** Use Well-Known Types and Common Types {#well-known-common}

Using the following common, shared types is strongly encouraged. E.g., do not
use `int32 timestamp_seconds_since_epoch` or `int64 timeout_millis` in your code
when a perfectly suitable common type already exists!

<a id="well-known-types"></a><a id="common-types"></a>

*   [`duration`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/duration.proto)
    is a signed, fixed-length span of time (for example, 42s).
*   [`timestamp`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto)
    is a point in time independent of any time zone or calendar (for example,
    2017-01-15T01:30:15.01Z).
*   [`interval`](https://github.com/googleapis/googleapis/blob/master/google/type/interval.proto)
    is a time interval independent of time zone or calendar (for example,
    2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z).
*   [`date`](https://github.com/googleapis/googleapis/blob/master/google/type/date.proto)
    is a whole calendar date (for example, 2005-09-19).
*   [`month`](https://github.com/googleapis/googleapis/blob/master/google/type/month.proto)
    is a month of year (for example, April).
*   [`dayofweek`](https://github.com/googleapis/googleapis/blob/master/google/type/dayofweek.proto)
    is a day of week (for example, Monday).
*   [`timeofday`](https://github.com/googleapis/googleapis/blob/master/google/type/timeofday.proto)
    is a time of day (for example, 10:42:23).
*   [`field_mask`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/field_mask.proto)
    is a set of symbolic field paths (for example, f.b.d).
*   [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto)
    is a postal address (for example, 1600 Amphitheatre Parkway Mountain View,
    CA 94043 USA).
*   [`money`](https://github.com/googleapis/googleapis/blob/master/google/type/money.proto)
    is an amount of money with its currency type (for example, 42 USD).
*   [`latlng`](https://github.com/googleapis/googleapis/blob/master/google/type/latlng.proto)
    is a latitude/longitude pair (for example, 37.386051 latitude and
    -122.083855 longitude).
*   [`color`](https://github.com/googleapis/googleapis/blob/master/google/type/color.proto)
    is a color in the RGBA color space.

<a id="do-define-widely-used-message-types-in-separate-files"></a>

## **Do** Define Widely-used Message Types in Separate Files {#separate-files}

If you're defining message types or enums that you hope/fear/expect to be widely
used outside your immediate team, consider putting them in their own file with
no dependencies. Then it's easy for anyone to use those types without
introducing the transitive dependencies in your other proto files.

<a id="dont-change-the-default-value-of-a-field"></a>

## **Don't** Change the Default Value of a Field {#change-default-value}

Almost never change the default value of a proto field. This causes version skew
between clients and servers. A client reading an unset value will see a
different result than a server reading the same unset value when their builds
straddle the proto change. Proto3 removed the ability to
set default values.

<a id="dont-go-from-repeated-to-scalar"></a>

## **Don't** Go from Repeated to Scalar {#repeated-to-scalar}

Although it won't cause crashes, you'll lose data. For JSON, a mismatch in
repeatedness will lose the whole *message*. For numeric proto3 fields and proto2
`packed` fields, going from repeated to scalar will lose all data in that
*field*. For non-numeric proto3 fields and un-annotated proto2 fields, going
from repeated to scalar will result in the last deserialized value "winning."

Going from scalar to repeated is OK in proto2 and in proto3 with
`[packed=false]` because for binary serialization the scalar value becomes a
one-element list .

<a id="do-follow-the-style-guide-for-generated-code"></a>

## **Do** Follow the Style Guide for Generated Code {#follow-style-guide}

Proto generated code is referred to in normal code. Ensure that options in
`.proto` file do not result in generation of code which violate the style guide.
For example:

*   `java_outer_classname` should follow
    https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names

*   `java_package` and `java_alt_package` should follow
    https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names

*   `package`, although used for Java when `java_package` is not present, always
    directly corresponds to C++ namespace and thus should follow
    https://google.github.io/styleguide/cppguide.html#Namespace_Names.
    If these style guides conflict, use `java_package` for Java.

*   `ruby_package` should be in the form `Foo::Bar::Baz` rather than
    `Foo.Bar.Baz`.

<a id="never-use-text-format-messages-for-interchange"></a>

## **Don't** use Text Format Messages for Interchange {#text-format-interchange}

Text-based serialization formats like text format and
JSON represent fields and enum values as strings. As a result, deserialization
of protocol buffers in these formats using old code will fail when a field or
enum value is renamed, or when a new field or enum value or extension is added.
Use binary serialization when possible for data interchange, and use text format
for human editing and debugging only.

If you use protos converted to JSON in your API or for storing data, you may not
be able to safely rename fields or enums at all.

<a id="never-rely-on-serialization-stability-across-builds"></a>

## **Never** Rely on Serialization Stability Across Builds {#serialization-stability}

The stability of proto serialization is not guaranteed across binaries or across
builds of the same binary. Do not rely on it when, for example, building cache
keys.

<a id="dont-generate-java-protos-in-the-same-java-package-as-other-code"></a>

## **Don't** Generate Java Protos in the Same Java Package as Other Code {#generate-java-protos}

Generate Java proto sources into a separate package from your hand-written Java
sources. The `package`, `java_package` and `java_alt_api_package` options
control
[where the generated Java sources are emitted](/reference/java/java-generated#package).
Make sure hand-written Java source code does not also live in that same package.
A common practice is to generate your protos into a `proto` subpackage in your
project that **only** contains those protos (that is, no hand-written source
code).

## Avoid Using Language Keywords for Field Names {#avoid-keywords}

If the name of a message, field, enum, or enum value is a keyword in the
language that reads from/writes to that field, then protobuf may change the
field name, and may have different ways to access them than normal fields. For
example, see
[this warning about Python](/reference/python/python-generated#keyword-conflicts).

You should also avoid using keywords in your file paths, as this can also cause
problems.

## Appendix {#appendix}

### API Best Practices {#api-best-practices}

This document lists only changes that are extremely likely to cause breakage.
For higher-level guidance on how to craft proto APIs that grow gracefully see
[API Best Practices](/programming-guides/api).
