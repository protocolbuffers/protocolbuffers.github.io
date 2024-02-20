+++
title = "Protocol Buffers Well-Known Types"
weight = 830
linkTitle = "Well-Known Types"
description = "API documentation for the google.protobuf package."
type = "docs"
+++

## Index

-   [`Any`](#any) (message)
-   [`Api`](#api) (message)
-   [`BoolValue`](#bool-value) (message)
-   [`BytesValue`](#bytes-value) (message)
-   [`DoubleValue`](#double-value) (message)
-   [`Duration`](#duration) (message)
-   [`Empty`](#empty) (message)
-   [`Enum`](#enum) (message)
-   [`EnumValue`](#enum-value) (message)
-   [`Field`](#field) (message)
-   [`Field.Cardinality`](#field-cardinality) (enum)
-   [`Field.Kind`](#field-kind) (enum)
-   [`FieldMask`](#field-mask) (message)
-   [`FloatValue`](#float-value) (message)
-   [`Int32Value`](#int32-value) (message)
-   [`Int64Value`](#int64-value) (message)
-   [`ListValue`](#list-value) (message)
-   [`Method`](#method) (message)
-   [`Mixin`](#mixin) (message)
-   [`NullValue`](#null-value) (enum)
-   [`Option`](#option) (message)
-   [`SourceContext`](#source-context) (message)
-   [`StringValue`](#string-value) (message)
-   [`Struct`](#struct) (message)
-   [`Syntax`](#syntax) (enum)
-   [`Timestamp`](#timestamp) (message)
-   [`Type`](#type) (message)
-   [`UInt32Value`](#uint32-value) (message)
-   [`UInt64Value`](#uint64-value) (message)
-   [`Value`](#value) (message)

## Any {#any}

`Any` contains an arbitrary serialized message along with a URL that describes
the type of the serialized message.

#### JSON {#json}

The JSON representation of an `Any` value uses the regular representation of the
deserialized, embedded message, with an additional field `@type` which contains
the type URL. Example:

```proto
package google.profile;
message Person {
  string first_name = 1;
  string last_name = 2;
}
```

```json
{
  "@type": "type.googleapis.com/google.profile.Person",
  "firstName": <string>,
  "lastName": <string>
}
```

If the embedded message type is well-known and has a custom JSON representation,
that representation will be embedded adding a field `value` which holds the
custom JSON in addition to the `@type` field. Example (for message
`google.protobuf.Duration`):

```json
{
  "@type": "type.googleapis.com/google.protobuf.Duration",
  "value": "1.212s"
}
```

<table id="google.protobuf.Any.FIELDS">
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>type_url</code></td>
      <td><code class="apitype">string</code></td>
      <td><p>A URL/resource name whose content describes the type of the serialized message.</p><p>For URLs which use the schema <code>http</code>, <code>https</code>, or no schema, the following restrictions and interpretations apply:</p>
<ul>
  <li>If no schema is provided, <code>https</code> is assumed.</li>
  <li>The last segment of the URL's path must represent the fully  qualified name of the type (as in <code>path/google.protobuf.Duration</code>).</li>
  <li>An HTTP GET on the URL must yield a <code><a href="#type">google.protobuf.Type</a></code>  value in binary format, or produce an error.</li>
  <li>Applications are allowed to cache lookup results based on the  URL, or have them precompiled into a binary to avoid any  lookup. Therefore, binary compatibility needs to be preserved  on changes to types. (Use versioned type names to manage  breaking changes.)</li>
</ul><p>Schemas other than <code>http</code>, <code>https</code> (or the empty schema) might be used with implementation specific semantics.</p></td>
    </tr>
    <tr>
      <td><code>value</code></td>
      <td><code class="apitype">bytes</code></td>
      <td>Must be valid serialized data of the above specified type.</td>
    </tr>
  </tbody>
</table>

## Api {#api}

Api is a light-weight descriptor for a protocol buffer service.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>
        The fully qualified name of this api, including package name followed by
        the api's simple name.
      </td>
    </tr>
    <tr>
      <td><code>methods</code></td>
      <td>
        <code><a href="#method">Method</a></code>
      </td>
      <td>The methods of this api, in unspecified order.</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
<code><a href="#option">Option</a></code>
      </td>
      <td>Any metadata attached to the API.</td>
    </tr>
    <tr>
      <td><code>version</code></td>
      <td><code>string</code></td>
      <td>
        <p>
          A version string for this api. If specified, must have the form
          <code>major-version.minor-version</code>, as in <code>1.10</code>. If
          the minor version is omitted, it defaults to zero. If the entire
          version field is empty, the major version is derived from the package
          name, as outlined below. If the field is not empty, the version in the
          package name will be verified to be consistent with what is provided
          here.
        </p>
        <p>
          The versioning schema uses
          <a href="http://semver.org">semantic versioning</a> where the major
          version number indicates a breaking change and the minor version an
          additive, non-breaking change. Both version numbers are signals to
          users what to expect from different versions, and should be carefully
          chosen based on the product plan.
        </p>
        <p>
          The major version is also reflected in the package name of the API,
          which must end in <code>v&lt;major-version&gt;</code>, as in
          <code>google.feature.v1</code>. For major versions 0 and 1, the suffix
          can be omitted. Zero major versions must only be used for
          experimental, none-GA apis.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>source_context</code></td>
      <td>
        <code
          ><code
            ><a href="#source-context">SourceContext</a></code
          ></code
        >
      </td>
      <td>
        Source context for the protocol buffer service represented by this
        message.
      </td>
    </tr>
    <tr>
      <td><code>mixins</code></td>
      <td>
        <code
          ><code><a href="#mixin">Mixin</a></code></code
        >
      </td>
      <td>
        Included APIs. See
        <code><a href="#mixin">Mixin</a></code>.
      </td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>The source syntax of the service.</td>
    </tr>
  </tbody>
</table>

## BoolValue {#bool-value}

Wrapper message for `bool`.

The JSON representation for `BoolValue` is JSON `true` and `false`.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code class="apitype">bool</code></td>
      <td>The bool value.</td>
    </tr>
  </tbody>
</table>

## BytesValue {#bytes-value}

Wrapper message for `bytes`.

The JSON representation for `BytesValue` is JSON string.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>bytes</code></td>
      <td>The bytes value.</td>
    </tr>
  </tbody>
</table>

## DoubleValue {#double-value}

Wrapper message for `double`.

The JSON representation for `DoubleValue` is JSON number.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>double</code></td>
      <td>The double value.</td>
    </tr>
  </tbody>
</table>

## Duration {#duration}

A Duration represents a signed, fixed-length span of time represented as a count
of seconds and fractions of seconds at nanosecond resolution. It is independent
of any calendar and concepts like \"day\" or \"month\". It is related to
Timestamp in that the difference between two Timestamp values is a Duration and
it can be added or subtracted from a Timestamp. Range is approximately +-10,000
years.

Example 1: Compute Duration from two Timestamps in pseudo code.

```c
Timestamp start = ...;
Timestamp end = ...;
Duration duration = ...;

duration.seconds = end.seconds - start.seconds;
duration.nanos = end.nanos - start.nanos;

if (duration.seconds < 0 && duration.nanos > 0) {
  duration.seconds += 1;
  duration.nanos -= 1000000000;
} else if (duration.seconds > 0 && duration.nanos < 0) {
  duration.seconds -= 1;
  duration.nanos += 1000000000;
}
```

Example 2: Compute Timestamp from Timestamp + Duration in pseudo code.

```c
Timestamp start = ...;
Duration duration = ...;
Timestamp end = ...;

end.seconds = start.seconds + duration.seconds;
end.nanos = start.nanos + duration.nanos;

if (end.nanos < 0) {
  end.seconds -= 1;
  end.nanos += 1000000000;
} else if (end.nanos >= 1000000000) {
  end.seconds += 1;
  end.nanos -= 1000000000;
}
```

The JSON representation for `Duration` is a `String` that ends in `s` to
indicate seconds and is preceded by the number of seconds, with nanoseconds
expressed as fractional seconds.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>seconds</code></td>
      <td><code>int64</code></td>
      <td>
        Signed seconds of the span of time. Must be from -315,576,000,000 to
        +315,576,000,000 inclusive.
      </td>
    </tr>
    <tr>
      <td><code>nanos</code></td>
      <td><code>int32</code></td>
      <td>
        Signed fractions of a second at nanosecond resolution of the span of
        time. Durations less than one second are represented with a 0
        <code>seconds</code> field and a positive or negative
        <code>nanos</code> field. For durations of one second or more, a
        non-zero value for the <code>nanos</code> field must be of the same sign
        as the <code>seconds</code> field. Must be from -999,999,999 to
        +999,999,999 inclusive.
      </td>
    </tr>
  </tbody>
</table>

## Empty {#empty}

A generic empty message that you can re-use to avoid defining duplicated empty
messages in your APIs. A typical example is to use it as the request or the
response type of an API method. For instance:

```proto
service Foo {
  rpc Bar(google.protobuf.Empty) returns (google.protobuf.Empty);
}
```

The JSON representation for `Empty` is empty JSON object `{}`.

## Enum {#enum}

Enum type definition

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>Enum type name.</td>
    </tr>
    <tr>
      <td><code>enumvalue</code></td>
      <td>
        <code><a href="#enum-value">EnumValue</a></code>
      </td>
      <td>Enum value definitions.</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>Protocol buffer options.</td>
    </tr>
    <tr>
      <td><code>source_context</code></td>
      <td>
        <code><a href="#source-context">SourceContext</a></code>
      </td>
      <td>The source context.</td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>The source syntax.</td>
    </tr>
  </tbody>
</table>

## EnumValue {#enum-value}

Enum value definition.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>Enum value name.</td>
    </tr>
    <tr>
      <td><code>number</code></td>
      <td><code>int32</code></td>
      <td>Enum value number.</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>Protocol buffer options.</td>
    </tr>
  </tbody>
</table>

## Field {#field}

A single field of a message type.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>kind</code></td>
      <td>
        <code><a href="#field-kind">Kind</a></code>
      </td>
      <td>The field type.</td>
    </tr>
    <tr>
      <td><code>cardinality</code></td>
      <td>
        <code><a href="#field-cardinality">Cardinality</a></code>
      </td>
      <td>The field cardinality.</td>
    </tr>
    <tr>
      <td><code>number</code></td>
      <td><code>int32</code></td>
      <td>The field number.</td>
    </tr>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>The field name.</td>
    </tr>
    <tr>
      <td><code>type_url</code></td>
      <td><code>string</code></td>
      <td>
        The field type URL, without the scheme, for message or enumeration
        types. Example:
        <code>&quot;type.googleapis.com/google.protobuf.Timestamp&quot;</code>.
      </td>
    </tr>
    <tr>
      <td><code>oneof_index</code></td>
      <td><code>int32</code></td>
      <td>
        The index of the field type in <code>Type.oneofs</code>, for message or
        enumeration types. The first type has index 1; zero means the type is
        not in the list.
      </td>
    </tr>
    <tr>
      <td><code>packed</code></td>
      <td><code>bool</code></td>
      <td>Whether to use alternative packed wire representation.</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>The protocol buffer options.</td>
    </tr>
    <tr>
      <td><code>json_name</code></td>
      <td><code>string</code></td>
      <td>The field JSON name.</td>
    </tr>
    <tr>
      <td><code>default_value</code></td>
      <td><code>string</code></td>
      <td>
        The string value of the default value of this field. Proto2 syntax only.
      </td>
    </tr>
  </tbody>
</table>

## Cardinality {#field-cardinality}

Whether a field is optional, required, or repeated.

<table>
  <thead>
    <tr>
      <th>Enum value</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>CARDINALITY_UNKNOWN</code></td>
      <td>For fields with unknown cardinality.</td>
    </tr>
    <tr>
      <td><code>CARDINALITY_OPTIONAL</code></td>
      <td>For optional fields.</td>
    </tr>
    <tr>
      <td><code>CARDINALITY_REQUIRED</code></td>
      <td>For required fields. Proto2 syntax only.</td>
    </tr>
    <tr>
      <td><code>CARDINALITY_REPEATED</code></td>
      <td>For repeated fields.</td>
    </tr>
  </tbody>
</table>

## Kind {#field-kind}

Basic field types.

<table class="matchpre">
  <thead>
    <tr>
      <th>Enum value</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>TYPE_UNKNOWN</code></td>
      <td>Field type unknown.</td>
    </tr>
    <tr>
      <td><code>TYPE_DOUBLE</code></td>
      <td>Field type double.</td>
    </tr>
    <tr>
      <td><code>TYPE_FLOAT</code></td>
      <td>Field type float.</td>
    </tr>
    <tr>
      <td><code>TYPE_INT64</code></td>
      <td>Field type int64.</td>
    </tr>
    <tr>
      <td><code>TYPE_UINT64</code></td>
      <td>Field type uint64.</td>
    </tr>
    <tr>
      <td><code>TYPE_INT32</code></td>
      <td>Field type int32.</td>
    </tr>
    <tr>
      <td><code>TYPE_FIXED64</code></td>
      <td>Field type fixed64.</td>
    </tr>
    <tr>
      <td><code>TYPE_FIXED32</code></td>
      <td>Field type fixed32.</td>
    </tr>
    <tr>
      <td><code>TYPE_BOOL</code></td>
      <td>Field type bool.</td>
    </tr>
    <tr>
      <td><code>TYPE_STRING</code></td>
      <td>Field type string.</td>
    </tr>
    <tr>
      <td><code>TYPE_GROUP</code></td>
      <td>Field type group. Proto2 syntax only, and deprecated.</td>
    </tr>
    <tr>
      <td><code>TYPE_MESSAGE</code></td>
      <td>Field type message.</td>
    </tr>
    <tr>
      <td><code>TYPE_BYTES</code></td>
      <td>Field type bytes.</td>
    </tr>
    <tr>
      <td><code>TYPE_UINT32</code></td>
      <td>Field type uint32.</td>
    </tr>
    <tr>
      <td><code>TYPE_ENUM</code></td>
      <td>Field type enum.</td>
    </tr>
    <tr>
      <td><code>TYPE_SFIXED32</code></td>
      <td>Field type sfixed32.</td>
    </tr>
    <tr>
      <td><code>TYPE_SFIXED64</code></td>
      <td>Field type sfixed64.</td>
    </tr>
    <tr>
      <td><code>TYPE_SINT32</code></td>
      <td>Field type sint32.</td>
    </tr>
    <tr>
      <td><code>TYPE_SINT64</code></td>
      <td>Field type sint64.</td>
    </tr>
  </tbody>
</table>

## FieldMask {#field-mask}

`FieldMask` represents a set of symbolic field paths, for example:

```proto
paths: "f.a"
paths: "f.b.d"
```

Here `f` represents a field in some root message, `a` and `b` fields in the
message found in `f`, and `d` a field found in the message in `f.b`.

Field masks are used to specify a subset of fields that should be returned by a
get operation (a *projection*), or modified by an update operation. Field masks
also have a custom JSON encoding (see below).

#### Field Masks in Projections {#field-masks-projections}

When a `FieldMask` specifies a *projection*, the API will filter the response
message (or sub-message) to contain only those fields specified in the mask. For
example, consider this \"pre-masking\" response message:

```proto
f {
  a : 22
  b {
    d : 1
    x : 2
  }
  y : 13
}
z: 8
```

After applying the mask in the previous example, the API response will not
contain specific values for fields x, y, or z (their value will be set to the
default, and omitted in proto text output):

```proto
f {
  a : 22
  b {
    d : 1
  }
}
```

A repeated field is not allowed except at the last position of a field mask.

If a `FieldMask` object is not present in a get operation, the operation applies
to all fields (as if a FieldMask of all fields had been specified).

Note that a field mask does not necessarily apply to the top-level response
message. In case of a REST get operation, the field mask applies directly to the
response, but in case of a REST list operation, the mask instead applies to each
individual message in the returned resource list. In case of a REST custom
method, other definitions may be used. Where the mask applies will be clearly
documented together with its declaration in the API. In any case, the effect on
the returned resource/resources is required behavior for APIs.

#### Field Masks in Update Operations {#field-masks-updates}

A field mask in update operations specifies which fields of the targeted
resource are going to be updated. The API is required to only change the values
of the fields as specified in the mask and leave the others untouched. If a
resource is passed in to describe the updated values, the API ignores the values
of all fields not covered by the mask.

In order to reset a field's value to the default, the field must be in the mask
and set to the default value in the provided resource. Hence, in order to reset
all fields of a resource, provide a default instance of the resource and set all
fields in the mask, or do not provide a mask as described below.

If a field mask is not present on update, the operation applies to all fields
(as if a field mask of all fields has been specified). Note that in the presence
of schema evolution, this may mean that fields the client does not know and has
therefore not filled into the request will be reset to their default. If this is
unwanted behavior, a specific service may require a client to always specify a
field mask, producing an error if not.

As with get operations, the location of the resource which describes the updated
values in the request message depends on the operation kind. In any case, the
effect of the field mask is required to be honored by the API.

##### Considerations for HTTP REST {#http-rest}

The HTTP kind of an update operation which uses a field mask must be set to
PATCH instead of PUT in order to satisfy HTTP semantics (PUT must only be used
for full updates).

#### JSON Encoding of Field Masks {#json-encoding-field-masks}

In JSON, a field mask is encoded as a single string where paths are separated by
a comma. Fields name in each path are converted to/from lower-camel naming
conventions.

As an example, consider the following message declarations:

```proto
message Profile {
  User user = 1;
  Photo photo = 2;
}
message User {
  string display_name = 1;
  string address = 2;
}
```

In proto a field mask for `Profile` may look as such:

```proto
mask {
  paths: "user.display_name"
  paths: "photo"
}
```

In JSON, the same mask is represented as below:

```json
{
  mask: "user.displayName,photo"
}
```

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>paths</code></td>
      <td><code>string</code></td>
      <td>The set of field mask paths.</td>
    </tr>
  </tbody>
</table>

## FloatValue {#float-value}

Wrapper message for `float`.

The JSON representation for `FloatValue` is JSON number.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>float</code></td>
      <td>The float value.</td>
    </tr>
  </tbody>
</table>

## Int32Value {#int32-value}

Wrapper message for `int32`.

The JSON representation for `Int32Value` is JSON number.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>int32</code></td>
      <td>The int32 value.</td>
    </tr>
  </tbody>
</table>

## Int64Value {#int64-value}

Wrapper message for `int64`.

The JSON representation for `Int64Value` is JSON string.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>int64</code></td>
      <td>The int64 value.</td>
    </tr>
  </tbody>
</table>

## ListValue {#list-value}

`ListValue` is a wrapper around a repeated field of values.

The JSON representation for `ListValue` is JSON array.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>values</code></td>
      <td>
        <code><a href="#value">Value</a></code>
      </td>
      <td>Repeated field of dynamically typed values.</td>
    </tr>
  </tbody>
</table>

## Method {#method}

Method represents a method of an api.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>The simple name of this method.</td>
    </tr>
    <tr>
      <td><code>request_type_url</code></td>
      <td><code>string</code></td>
      <td>A URL of the input message type.</td>
    </tr>
    <tr>
      <td><code>request_streaming</code></td>
      <td><code>bool</code></td>
      <td>If true, the request is streamed.</td>
    </tr>
    <tr>
      <td><code>response_type_url</code></td>
      <td><code>string</code></td>
      <td>The URL of the output message type.</td>
    </tr>
    <tr>
      <td><code>response_streaming</code></td>
      <td><code>bool</code></td>
      <td>If true, the response is streamed.</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>Any metadata attached to the method.</td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>The source syntax of this method.</td>
    </tr>
  </tbody>
</table>

## Mixin {#mixin}

Declares an API to be included in this API. The including API must redeclare all
the methods from the included API, but documentation and options are inherited
as follows:

-   If after comment and whitespace stripping, the documentation string of the
    redeclared method is empty, it will be inherited from the original method.

-   Each annotation belonging to the service config (http, visibility) which is
    not set in the redeclared method will be inherited.

-   If an http annotation is inherited, the path pattern will be modified as
    follows. Any version prefix will be replaced by the version of the including
    API plus the `root` path if specified.

Example of a simple mixin:

```proto
package google.acl.v1;
service AccessControl {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v1/{resource=**}:getAcl";
  }
}

package google.storage.v2;
service Storage {
  //       rpc GetAcl(GetAclRequest) returns (Acl);

  // Get a data record.
  rpc GetData(GetDataRequest) returns (Data) {
    option (google.api.http).get = "/v2/{resource=**}";
  }
}
```

Example of a mixin configuration:

```
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
```

The mixin construct implies that all methods in `AccessControl` are also
declared with same name and request/response types in `Storage`. A documentation
generator or annotation processor will see the effective `Storage.GetAcl` method
after inheriting documentation and annotations as follows:

```proto
service Storage {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/{resource=**}:getAcl";
  }
  ...
}
```

Note how the version in the path pattern changed from `v1` to `v2`.

If the `root` field in the mixin is specified, it should be a relative path
under which inherited HTTP paths are placed. Example:

```
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
    root: acls
```

This implies the following inherited HTTP annotation:

```proto
service Storage {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/acls/{resource=**}:getAcl";
  }
  ...
}
```

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>The fully qualified name of the API which is included.</td>
    </tr>
    <tr>
      <td><code>root</code></td>
      <td><code>string</code></td>
      <td>
        If non-empty specifies a path under which inherited HTTP paths are
        rooted.
      </td>
    </tr>
  </tbody>
</table>

## NullValue {#null-value}

`NullValue` is a singleton enumeration to represent the null value for the
`Value` type union.

The JSON representation for `NullValue` is JSON `null`.

<table>
  <thead>
    <tr>
      <th>Enum value</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>NULL_VALUE</code></td>
      <td>Null value.</td>
    </tr>
  </tbody>
</table>

## Option {#option}

A protocol buffer option, which can be attached to a message, field,
enumeration, etc.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>
        The option's name. For example, <code>&quot;java_package&quot;</code>.
      </td>
    </tr>
    <tr>
      <td><code>value</code></td>
      <td>
        <code><a href="#any">Any</a></code>
      </td>
      <td>
        The option's value. For example,
        <code>&quot;com.google.protobuf&quot;</code>.
      </td>
    </tr>
  </tbody>
</table>

## SourceContext {#source-context}

`SourceContext` represents information about the source of a protobuf element,
like the file in which it is defined.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>file_name</code></td>
      <td><code>string</code></td>
      <td>
        The path-qualified name of the .proto file that contained the associated
        protobuf element. For example:
        <code>&quot;google/protobuf/source.proto&quot;</code>.
      </td>
    </tr>
  </tbody>
</table>

## StringValue {#string-value}

Wrapper message for `string`.

The JSON representation for `StringValue` is JSON string.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>string</code></td>
      <td>The string value.</td>
    </tr>
  </tbody>
</table>

## Struct {#struct}

`Struct` represents a structured data value, consisting of fields which map to
dynamically typed values. In some languages, `Struct` might be supported by a
native representation. For example, in scripting languages like JS a struct is
represented as an object. The details of that representation are described
together with the proto support for the language.

The JSON representation for `Struct` is JSON object.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>fields</code></td>
      <td>
        <code>map&lt;string, <a href="#value">Value</a>&gt;</code>
      </td>
      <td>Map of dynamically typed values.</td>
    </tr>
  </tbody>
</table>

## Syntax {#syntax}

The syntax in which a protocol buffer element is defined.

<table>
  <thead>
    <tr>
      <th>Enum value</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>SYNTAX_PROTO2</code></td>
      <td>Syntax <code>proto2</code>.</td>
    </tr>
    <tr>
      <td><code>SYNTAX_PROTO3</code></td>
      <td>Syntax <code>proto3</code>.</td>
    </tr>
  </tbody>
</table>

## Timestamp {#timestamp}

A Timestamp represents a point in time independent of any time zone or calendar,
represented as seconds and fractions of seconds at nanosecond resolution in UTC
Epoch time. It is encoded using the Proleptic Gregorian Calendar which extends
the Gregorian calendar backwards to year one. It is encoded assuming all minutes
are 60 seconds long, i.e. leap seconds are \"smeared\" so that no leap second
table is needed for interpretation. Range is from 0001-01-01T00:00:00Z to
9999-12-31T23:59:59.999999999Z. By restricting to that range, we ensure that we
can convert to and from RFC 3339 date strings. See
<https://www.ietf.org/rfc/rfc3339.txt>.

Example 1: Compute Timestamp from POSIX `time()`.

```cpp
Timestamp timestamp;
timestamp.set_seconds(time(NULL));
timestamp.set_nanos(0);
```

Example 2: Compute Timestamp from POSIX `gettimeofday()`.

```cpp
struct timeval tv;
gettimeofday(&tv, NULL);

Timestamp timestamp;
timestamp.set_seconds(tv.tv_sec);
timestamp.set_nanos(tv.tv_usec * 1000);
```

Example 3: Compute Timestamp from Win32 `GetSystemTimeAsFileTime()`.

```cpp
FILETIME ft;
GetSystemTimeAsFileTime(&ft);
UINT64 ticks = (((UINT64)ft.dwHighDateTime) << 32) | ft.dwLowDateTime;

// A Windows tick is 100 nanoseconds. Windows epoch 1601-01-01T00:00:00Z
// is 11644473600 seconds before Unix epoch 1970-01-01T00:00:00Z.
Timestamp timestamp;
timestamp.set_seconds((INT64) ((ticks / 10000000) - 11644473600LL));
timestamp.set_nanos((INT32) ((ticks % 10000000) * 100));
```

Example 4: Compute Timestamp from Java `System.currentTimeMillis()`.

```java
long millis = System.currentTimeMillis();

Timestamp timestamp = Timestamp.newBuilder().setSeconds(millis / 1000)
    .setNanos((int) ((millis % 1000) * 1000000)).build();
```

Example 5: Compute Timestamp from current time in Python.

```py
now = time.time()
seconds = int(now)
nanos = int((now - seconds) * 10**9)
timestamp = Timestamp(seconds=seconds, nanos=nanos)
```

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>seconds</code></td>
      <td><code>int64</code></td>
      <td>
        Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z.
        Must be from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z inclusive.
      </td>
    </tr>
    <tr>
      <td><code>nanos</code></td>
      <td><code>int32</code></td>
      <td>
        Non-negative fractions of a second at nanosecond resolution. Negative
        second values with fractions must still have non-negative nanos values
        that count forward in time. Must be from 0 to 999,999,999 inclusive.
      </td>
    </tr>
  </tbody>
</table>

## Type {#type}

A protocol buffer message type.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>The fully qualified message name.</td>
    </tr>
    <tr>
      <td><code>fields</code></td>
      <td>
        <code><a href="#field">Field</a></code>
      </td>
      <td>The list of fields.</td>
    </tr>
    <tr>
      <td><code>oneofs</code></td>
      <td><code>string</code></td>
      <td>
        The list of types appearing in <code>oneof</code> definitions in this
        type.
      </td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>The protocol buffer options.</td>
    </tr>
    <tr>
      <td><code>source_context</code></td>
      <td>
        <code><a href="#source-context">SourceContext</a></code>
      </td>
      <td>The source context.</td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>The source syntax.</td>
    </tr>
  </tbody>
</table>

## UInt32Value {#uint32-value}

Wrapper message for `uint32`.

The JSON representation for `UInt32Value` is JSON number.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>uint32</code></td>
      <td>The uint32 value.</td>
    </tr>
  </tbody>
</table>

## UInt64Value {#uint64-value}

Wrapper message for `uint64`.

The JSON representation for `UInt64Value` is JSON string.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>uint64</code></td>
      <td>The uint64 value.</td>
    </tr>
  </tbody>
</table>

## Value {#value}

`Value` represents a dynamically typed value which can be either null, a number,
a string, a boolean, a recursive struct value, or a list of values. A producer
of value is expected to set one of that variants, absence of any variant
indicates an error.

The JSON representation for `Value` is JSON value.

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="3">Union field, only one of the following:</td>
    </tr>
    <tr>
      <td><code>null_value</code></td>
      <td>
        <code><a href="#null-value">NullValue</a></code>
      </td>
      <td>Represents a null value.</td>
    </tr>
    <tr>
      <td><code>number_value</code></td>
      <td><code>double</code></td>
      <td>
        Represents a double value. Note that attempting to serialize NaN or
        Infinity results in error. (We can't serialize these as string "NaN" or
        "Infinity" values like we do for regular fields, because they would
        parse as string_value, not number_value).
      </td>
    </tr>
    <tr>
      <td><code>string_value</code></td>
      <td><code>string</code></td>
      <td>Represents a string value.</td>
    </tr>
    <tr>
      <td><code>bool_value</code></td>
      <td><code>bool</code></td>
      <td>Represents a boolean value.</td>
    </tr>
    <tr>
      <td><code>struct_value</code></td>
      <td>
        <code><a href="#struct">Struct</a></code>
      </td>
      <td>Represents a structured value.</td>
    </tr>
    <tr>
      <td><code>list_value</code></td>
      <td>
        <code><a href="#list-value">ListValue</a></code>
      </td>
      <td>Represents a repeated <code>Value</code>.</td>
    </tr>
  </tbody>
</table>
