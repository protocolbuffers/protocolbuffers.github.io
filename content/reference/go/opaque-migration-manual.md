+++
title = "Go Opaque API: Manual Migration"
weight = 660
linkTitle = "Opaque API: Manual Migration"
description = "Describes a manual migration to the Opaque API."
type = "docs"
+++

The Opaque API is the latest version of the Protocol Buffers implementation for
the Go programming language. The old version is now called Open Struct API. See
the [Go Protobuf: Releasing the Opaque API](https://go.dev/blog/protobuf-opaque)
blog post for an introduction.

This is a user guide for migrating Go Protobuf usages from the older Open Struct
API to the new Opaque API.

{{% alert title="Warning" color="warning" %}} You
are looking at the manual migration guide. Typically you’re better off using the
`open2opaque` tool to automate the migration. See
[Opaque API Migration](/reference/go/opaque-migration)
instead. {{% /alert %}}

The
[Generated Code Guide](/reference/go/go-generated-opaque)
provides more detail. This guide compares the old and new API side-by-side.

### Message Construction

Suppose there is a protobuf message defined like this:

```proto
message Foo {
  uint32 uint32 = 1;
  bytes bytes = 2;
  oneof union {
    string    string = 4;
    MyMessage message = 5;
  }
  enum Kind { … };
  Kind kind = 9;
}
```

Here is an example of how to construct this message from literal values:

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque API (new)</td>
        </tr>
<tr>
<td>

```go
m := &pb.Foo{
  Uint32: proto.Uint32(5),
  Bytes:  []byte("hello"),
}
```

</td>
<td>

```go
m := pb.Foo_builder{
  Uint32: proto.Uint32(5),
  Bytes:  []byte("hello"),
}.Build()
```

</td>
</tr>
</table>

As you can see, the builder structs allow for an almost 1:1 translation between
Open Struct API (old) and Opaque API (new).

Generally, prefer using builders for readability. Only in rare cases, like
creating Protobuf messages in a hot inner loop, might it be preferable to use
setters instead of builders. See
[the Opaque API FAQ: Should I use builders or setters?](/reference/go/opaque-faq#builders-vs-setters)
for more detail.

An exception to the above example is when working with [oneofs](#oneofs): The
Open Struct API (old) uses a wrapper struct type for each oneof case, whereas
the Opaque API (new) treats oneof fields like regular message fields:

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque API (new)</td>
        </tr>
<tr>
<td>

```go
m := &pb.Foo{
  Uint32: myScalar,  // could be nil
  Union:  &pb.Foo_String{myString},
  Kind:   pb.Foo_SPECIAL_KIND.Enum(),
}
```

</td>
<td>

```go
m := pb.Foo_builder{
  Uint32: myScalar,
  String: myString,
  Kind:   pb.Foo_SPECIAL_KIND.Enum(),
}.Build()
```

</td>
</tr>
</table>

For the set of Go struct fields associated with a oneof union, only one field
may be populated. If multiple oneof case fields are populated, the last one (in
field declaration order in your .proto file) wins.

### Scalar fields

Suppose there is a message defined with a scalar field:

```proto
message Artist {
  int32 birth_year = 1;
}
```

Protobuf message fields for which Go uses scalar types (bool, int32, int64,
uint32, uint64, float32, float64, string, []byte, and enum) will have `Get` and
`Set` accessor methods. Fields with
[explicit presence](/programming-guides/field_presence/)
will also have `Has` and `Clear` methods.

For a field of type `int32` named `birth_year`, the following accessor methods
will be generated for it:

```go
func (m *Artist) GetBirthYear() int32
func (m *Artist) SetBirthYear(v int32)
func (m *Artist) HasBirthYear() bool
func (m *Artist) ClearBirthYear()
```

`Get` returns a value for the field. If the field is not set or the message
receiver is nil, it returns the default value. The default value is the
[zero value](https://go.dev/ref/spec#The_zero_value), unless explicitly set with
the default option.

`Set` stores the provided value into the field. It panics when called on a nil
message receiver.

For bytes fields, calling `Set` with a nil []byte will be considered set. For
example, calling `Has` immediately after returns true. Calling `Get` immediately
after will return a zero-length slice (can either be nil or empty slice). Users
should use `Has` for determining presence and not rely on whether `Get` returns
nil.

`Has` reports whether the field is populated. It returns false when called on a
nil message receiver.

`Clear` clears the field. It panics when called on a nil message receiver.

**Example code snippets using a string field in:**

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque API (new)</td>
        </tr>
<tr>
<td>

```go
// Getting the value.
s := m.GetBirthYear()

// Setting the field.
m.BirthYear = proto.Int32(1989)

// Check for presence.
if s.BirthYear != nil { … }

// Clearing the field.
m.BirthYear = nil
```

</td>
<td>

```go
// Getting the field value.
s := m.GetBirthYear()

// Setting the field.
m.SetBirthYear(1989)

// Check for presence.
if m.HasBirthYear() { … }

// Clearing the field
m.ClearBirthYear()
```

</td>
</tr>
</table>

### Message fields

Suppose there is a message defined with a message-typed field:

```proto
message Band {}

message Concert {
  Band headliner = 1;
}
```

Protobuf message fields of type message will have `Get`, `Set`, `Has` and
`Clear` methods.

For a message-typed field named `headliner`, the following accessor methods will
be generated for it:

```go
func (m *Concert) GetHeadliner() *Band
func (m *Concert) SetHeadliner(*Band)
func (m *Concert) HasHeadliner() bool
func (m *Concert) ClearHeadliner()
```

`Get` returns a value for the field. It returns nil if not set or when called on
a nil message receiver. Checking if `Get` returns nil is equivalent to checking
if `Has` returns false.

`Set` stores the provided value into the field. It panics when called on a nil
message receiver. Calling `Set` with a nil pointer is equivalent to calling
`Clear`.

`Has` reports whether the field is populated. It returns false when called on a
nil message receiver.

`Clear` clears the field. It panics when called on a nil message receiver.

**Example code snippets**

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque (new)</td>
        </tr>
<tr>
<td>

```go
// Getting the value.
b := m.GetHeadliner()

// Setting the field.
m.Headliner = &pb.Band{}

// Check for presence.
if s.Headliner != nil { … }

// Clearing the field.
m.Headliner = nil
```

</td>
<td>

```go
// Getting the value.
s := m.GetHeadliner()

// Setting the field.
m.SetHeadliner(&pb.Band{})

// Check for presence.
if m.HasHeadliner() { … }

// Clearing the field
m.ClearHeadliner()
```

</td>
</tr>
</table>

### Repeated fields

Suppose there is a message defined with a repeated message-typed field:

```proto
message Concert {
  repeated Band support_acts = 2;
}
```

Repeated fields will have `Get` and `Set` methods.

`Get` returns a value for the field. It returns nil if the field is not set or
the message receiver is nil.

`Set` stores the provided value into the field. It panics when called on a nil
message receiver. `Set` will store a copy of the slice header that is provided.
Changes to the slice contents are observable in the repeated field. Hence, if
`Set` is called with an empty slice, calling `Get` immediately after will return
the same slice. For the wire or text marshaling output, a passed-in nil slice is
indistinguishable from an empty slice.

For a repeated message-typed field named `support_acts` on message `Concert`,
the following accessor methods will be generated for it:

```go
func (m *Concert) GetSupportActs() []*Band
func (m *Concert) SetSupportActs([]*Band)
```

**Example code snippets**

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque API (new)</td>
        </tr>
<tr>
<td>

```go
// Getting the entire repeated value.
v := m.GetSupportActs()

// Setting the field.
m.SupportActs = v

// Get an element in a repeated field.
e := m.SupportActs[i]

// Set an element in a repeated field.
m.SupportActs[i] = e

// Get the length of a repeated field.
n := len(m.GetSupportActs())

// Truncate a repeated field.
m.SupportActs = m.SupportActs[:i]

// Append to a repeated field.
m.SupportActs = append(m.GetSupportActs(), e)
m.SupportActs = append(m.GetSupportActs(), v...)

// Clearing the field.
m.SupportActs = nil
```

</td>
<td>

```go
// Getting the entire repeated value.
v := m.GetSupportActs()

// Setting the field.
m.SetSupportActs(v)

// Get an element in a repeated field.
e := m.GetSupportActs()[i]

// Set an element in a repeated field.
m.GetSupportActs()[i] = e

// Get the length of a repeated field.
n := len(m.GetSupportActs())

// Truncate a repeated field.
m.SetSupportActs(m.GetSupportActs()[:i])

// Append to a repeated field.
m.SetSupportActs(append(m.GetSupportActs(), e))
m.SetSupportActs(append(m.GetSupportActs(), v...))

// Clearing the field.
m.SetSupportActs(nil)
```

</td>
</tr>
</table>

### Maps

Suppose there is a message defined with a map-typed field:

```proto
message MerchBooth {
  map<string, MerchItems> items = 1;
}
```

Map fields will have `Get` and `Set` methods.

`Get` returns a value for the field. It returns nil if the field is not set or
the message receiver is nil.

`Set` stores the provided value into the field. It panics when called on a nil
message receiver. `Set` will store a copy of the provided map reference. Changes
to the provided map are observable in the map field.

For a map field named `items` on message `MerchBooth`, the following accessor
methods will be generated for it:

```go
func (m *MerchBooth) GetItems() map[string]*MerchItem
func (m *MerchBooth) SetItems(map[string]*MerchItem)
```

**Example code snippets**

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque API (new)</td>
        </tr>
<tr>
<td>

```go
// Getting the entire map value.
v := m.GetItems()

// Setting the field.
m.Items = v

// Get an element in a map field.
v := m.Items[k]

// Set an element in a map field.
// This will panic if m.Items is nil.
// You should check m.Items for nil
// before doing the assignment to ensure
// it won't panic.
m.Items[k] = v

// Delete an element in a map field.
delete(m.Items, k)

// Get the size of a map field.
n := len(m.GetItems())

// Clearing the field.
m.Items = nil
```

</td>
<td>

```go
// Getting the entire map value.
v := m.GetItems()

// Setting the field.
m.SetItems(v)

// Get an element in a map field.
v := m.GetItems()[k]

// Set an element in a map field.
// This will panic if m.GetItems() is nil.
// You should check m.GetItems() for nil
// before doing the assignment to ensure
// it won't panic.
m.GetItems()[k] = v

// Delete an element in a map field.
delete(m.GetItems(), k)

// Get the size of a map field.
n := len(m.GetItems())

// Clearing the field.
m.SetItems(nil)
```

</td>
</tr>
</table>

### Oneofs

For each oneof union grouping, there will be a `Which`, `Has` and `Clear` method
on the message. There will also be a `Get`, `Set`, `Has`, and `Clear` method on
each oneof case field in that union.

Suppose there is a message defined with oneof fields `image_url` and
`image_data` in oneof `avatar` like:

```proto
message Profile {
  oneof avatar {
    string image_url = 1;
    bytes image_data = 2;
  }
}
```

The generated Opaque API for this oneof will be:

```go
func (m *Profile) WhichAvatar() case_Profile_Avatar { … }
func (m *Profile) HasAvatar() bool { … }
func (m *Profile) ClearAvatar() { … }

type case_Profile_Avatar protoreflect.FieldNumber

const (
  Profile_Avatar_not_set_case case_Profile_Avatar = 0
  Profile_ImageUrl_case case_Profile_Avatar = 1
  Profile_ImageData_case case_Profile_Avatar = 2
)
```

`Which` reports which case field is set by returning the field number. It
returns 0 when none are set or when called on a nil message receiver.

`Has` reports whether any of the fields within the oneof is set. It returns
false when called on a nil message receiver.

`Clear` clears the currently set case field in the oneof. It panics on a nil
message receiver.

The generated Opaque API for each oneof case field will be:

```go
func (m *Profile) GetImageUrl() string { … }
func (m *Profile) GetImageData() []byte { … }

func (m *Profile) SetImageUrl(v string) { … }
func (m *Profile) SetImageData(v []byte) { … }

func (m *Profile) HasImageUrl() bool { … }
func (m *Profile) HasImageData() bool { … }

func (m *Profile) ClearImageUrl() { … }
func (m *Profile) ClearImageData() { … }
```

`Get` returns a value for the case field. It will return the zero value if the
case field is not set or when called on a nil message receiver.

`Set` stores the provided value into the case field. It also implicitly clears
the case field that was previously populated within the oneof union. Calling
`Set` on a oneof message case field with nil value will set the field to an
empty message. It panics when called on a nil message receiver.

`Has` reports whether the case field is set or not. It returns false when called
on a nil message receiver.

`Clear` clears the case field. If it was previously set, the oneof union is also
cleared. If the oneof union is set to different field, it will not clear the
oneof union. It panics when called on a nil message receiver.

**Example code snippets**

<table width="100%">
        <tr>
                <td width="50%">Open Struct API (old)</td>
                <td width="50%">Opaque API (new)</td>
        </tr>
<tr>
<td>

```go
// Getting the oneof field that is set.
switch m.GetAvatar().(type) {
case *pb.Profile_ImageUrl:
  … = m.GetImageUrl()
case *pb.Profile_ImageData:
  … = m.GetImageData()
}

// Setting the fields.
m.Avatar = &pb.Profile_ImageUrl{"http://"}
m.Avatar = &pb.Profile_ImageData{img}

// Checking whether any oneof field is set
if m.Avatar != nil { … }

// Clearing the field.
m.Avatar = nil

// Checking if a specific field is set.
_, ok := m.GetAvatar().(*pb.Profile_ImageUrl)
if ok { … }

// Clearing a specific field
_, ok := m.GetAvatar().(*pb.Profile_ImageUrl)
if ok {
  m.Avatar = nil
}

// Copy a oneof field.
m.Avatar = src.Avatar
```

</td>
<td>

```go
// Getting the oneof field that is set.
switch m.WhichAvatar() {
case pb.Profile_ImageUrl_case:
  … = m.GetImageUrl()
case pb.Profile_ImageData_case:
  … = m.GetImageData()
}

// Setting the fields.
m.SetImageUrl("http://")
m.SetImageData([]byte("…"))

// Checking whether any oneof field is set
if m.HasAvatar() { … }

// Clearing the field.
m.ClearAvatar()

// Checking if a specific field is set.
if m.HasImageUrl() { … }

// Clearing a specific field.
m.ClearImageUrl()

// Copy a oneof field
switch src.WhichAvatar() {
case pb.Profile_ImageUrl_case:
  m.SetImageUrl(src.GetImageUrl())
case pb.Profile_ImageData_case:
  m.SetImageData(src.GetImageData())
}
```

</td>
</tr>

</table>

### Reflection

Code that use Go `reflect` package on proto message types to access struct
fields and tags will no longer work when migrating away from the Open Struct
API. Code will need to migrate to use
[protoreflect](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect)

Some common libraries do use Go `reflect` under the hood, examples are:

*   [encoding/json](https://pkg.go.dev/encoding/json/)
    *   Use
        [protobuf/encoding/protojson](https://pkg.go.dev/google.golang.org/protobuf/encoding/protojson).
*   [pretty](https://pkg.go.dev/github.com/kr/pretty)
*   [cmp](https://pkg.go.dev/github.com/google/go-cmp/cmp)
    *   To use `cmp.Equal` properly with protobuf messages, use
        [protocmp.Transform](https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp#Transform)
