+++
title = "Go Generated Code Guide"
weight = 610
linkTitle = "Generated Code Guide"
description = "Describes exactly what Go code the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

Any differences between
proto2 and proto3 generated code are highlighted - note that these differences
are in the generated code as described in this document, not the base API, which
are the same in both versions. You should read the
[proto2 language guide](/programming-guides/proto2)
and/or the
[proto3 language guide](/programming-guides/proto3)
before reading this document.

## Compiler Invocation {#invocation}

The protocol buffer compiler requires a plugin to generate Go code. Install it
using Go 1.16 or higher by running:

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

This will install a `protoc-gen-go` binary in `$GOBIN`. Set the `$GOBIN`
environment variable to change the installation location. It must be in your
`$PATH` for the protocol buffer compiler to find it.

The protocol buffer compiler produces Go output when invoked with the `go_out`
flag. The argument to the `go_out` flag is the directory where you want the
compiler to write your Go output. The compiler creates a single source file for
each `.proto` file input. The name of the output file is created by replacing
the `.proto` extension with `.pb.go`.

Where in the output directory the generated `.pb.go` file is placed depends on
the compiler flags. There are several output modes:

-   If the `paths=import` flag is specified, the output file is placed in a
    directory named after the Go package's import path (such as one provided by
    the `go_package` option within the `.proto` file). For example, an input
    file `protos/buzz.proto` with a Go import path of
    `example.com/project/protos/fizz` results in an output file at
    `example.com/project/protos/fizz/buzz.pb.go`. This is the default output
    mode if a `paths` flag is not specified.
-   If the `module=$PREFIX` flag is specified, the output file is placed in a
    directory named after the Go package's import path (such as one provided by
    the `go_package` option within the `.proto` file), but with the specified
    directory prefix removed from the output filename. For example, an input
    file `protos/buzz.proto` with a Go import path of
    `example.com/project/protos/fizz` and `example.com/project` specified as the
    `module` prefix results in an output file at `protos/fizz/buzz.pb.go`.
    Generating any Go packages outside the module path results in an error. This
    mode is useful for outputting generated files directly into a Go module.
-   If the `paths=source_relative` flag is specified, the output file is placed
    in the same relative directory as the input file. For example, an input file
    `protos/buzz.proto` results in an output file at `protos/buzz.pb.go`.

Flags specific to `protoc-gen-go` are provided by passing a `go_opt` flag when
invoking `protoc`. Multiple `go_opt` flags may be passed. For example, when
running:

```shell
protoc --proto_path=src --go_out=out --go_opt=paths=source_relative foo.proto bar/baz.proto
```

the compiler will read input files `foo.proto` and `bar/baz.proto` from within
the `src` directory, and write output files `foo.pb.go` and `bar/baz.pb.go` to
the `out` directory. The compiler automatically creates nested output
sub-directories if necessary, but will not create the output directory itself.

## Packages {#package}

In order to generate Go code, the Go package's import path must be provided for
every `.proto` file (including those transitively depended upon by the `.proto`
files being generated). There are two ways to specify the Go import path:

-   by declaring it within the `.proto` file, or
-   by declaring it on the command line when invoking `protoc`.

We recommend declaring it within the `.proto` file so that the Go packages for
`.proto` files can be centrally identified with the `.proto` files themselves
and to simplify the set of flags passed when invoking `protoc`. If the Go import
path for a given `.proto` file is provided by both the `.proto` file itself and
on the command line, then the latter takes precedence over the former.

The Go import path is locally specified in a `.proto` file by declaring a
`go_package` option with the full import path of the Go package. Example usage:

```proto
option go_package = "example.com/project/protos/fizz";
```

The Go import path may be specified on the command line when invoking the
compiler, by passing one or more `M${PROTO_FILE}=${GO_IMPORT_PATH}` flags.
Example usage:

```shell
protoc --proto_path=src \
  --go_opt=Mprotos/buzz.proto=example.com/project/protos/fizz \
  --go_opt=Mprotos/bar.proto=example.com/project/protos/foo \
  protos/buzz.proto protos/bar.proto
```

Since the mapping of all `.proto` files to their Go import paths can be quite
large, this mode of specifying the Go import paths is generally performed by
some build tool (e.g., [Bazel](https://bazel.build/)) that has
control over the entire dependency tree. If there are duplicate entries for a
given `.proto` file, then the last one specified takes precedence.

For both the `go_package` option and the `M` flag, the value may include an
explicit package name separated from the import path by a semicolon. For
example: `"example.com/protos/foo;package_name"`. This usage is discouraged
since the package name will be derived by default from the import path in a
reasonable manner.

The import path is used to determine which import statements must be generated
when one `.proto` file imports another `.proto` file. For example, if `a.proto`
imports `b.proto`, then the generated `a.pb.go` file needs to import the Go
package which contains the generated `b.pb.go` file (unless both files are in
the same package). The import path is also used to construct output filenames.
See the \"Compiler Invocation\" section above for details.

There is no correlation between the Go import path and the
[`package` specifier](/programming-guides/proto3#packages)
in the `.proto` file. The latter is only relevant to the protobuf namespace,
while the former is only relevant to the Go namespace. Also, there is no
correlation between the Go import path and the `.proto` import path.

## Messages {#message}

Given a simple message declaration:

```proto
message Artist {}
```

the protocol buffer compiler generates a struct called `Artist`. An `*Artist`
implements the
[`proto.Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message)
interface.

The
[`proto` package](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc)
provides functions which operate on messages, including conversion to and from
binary format.

The `proto.Message` interface defines a `ProtoReflect` method. This method
returns a
[`protoreflect.Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message)
which provides a reflection-based view of the message.

The `optimize_for` option does not affect the output of the Go code generator.

### Nested Types

A message can be declared inside another message. For example:

```proto
message Artist {
  message Name {
  }
}
```

In this case, the compiler generates two structs: `Artist` and `Artist_Name`.

## Fields

The protocol buffer compiler generates a struct field for each field defined
within a message. The exact nature of this field depends on its type and whether
it is a singular, repeated, map, or oneof field.

Note that the generated Go field names always use camel-case naming, even if the
field name in the `.proto` file uses lower-case with underscores
([as it should](/programming-guides/style#message-field-names)).
The case-conversion works as follows:

1.  The first letter is capitalized for export. If the first character is an
    underscore, it is removed and a capital X is prepended.
2.  If an interior underscore is followed by a lower-case letter, the underscore
    is removed, and the following letter is capitalized.

Thus, the proto field `birth_year` becomes `BirthYear` in Go, and
`_birth_year_2` becomes `XBirthYear_2`.

### Singular Scalar Fields (proto2) {#singular-scalar-proto2}

For either of these field definitions:

```proto
optional int32 birth_year = 1;
required int32 birth_year = 1;
```

the compiler generates a struct with an `*int32` field named `BirthYear` and an
accessor method `GetBirthYear()` which returns the `int32` value in `Artist` or
the default value if the field is unset. If the default is not explicitly set,
the [zero value](https://golang.org/ref/spec#The_zero_value) of that
type is used instead (`0` for numbers, the empty string for strings).

For other scalar field types (including `bool`, `bytes`, and `string`), `*int32`
is replaced with the corresponding Go type according to the
[scalar value types table](/programming-guides/proto2#scalar).

### Singular Scalar Fields (proto3) {#singular-scalar-proto3}

For this field definition:

```proto
int32 birth_year = 1;
optional int32 first_active_year = 2;
```

The compiler will generate a struct with an `int32` field named `BirthYear` and
an accessor method `GetBirthYear()` which returns the `int32` value in
`birth_year` or the
[zero value](https://golang.org/ref/spec#The_zero_value) of that type
if the field is unset (`0` for numbers, the empty string for strings).

For other scalar field types (including `bool`, `bytes`, and `string`), `int32`
is replaced with the corresponding Go type according to the
[scalar value types table](/programming-guides/proto3#scalar).
Unset values in the proto will be represented as the
[zero value](https://golang.org/ref/spec#The_zero_value) of that type
(`0` for numbers, the empty string for strings).

### Singular Message Fields {#singular-message}

Given the message type:

```proto
message Band {}
```

For a message with a `Band` field:

```proto
// proto2
message Concert {
  optional Band headliner = 1;
  // The generated code is the same result if required instead of optional.
}

// proto3
message Concert {
  Band headliner = 1;
}
```

The compiler will generate a Go struct

```go
type Concert struct {
    Headliner *Band
}
```

Message fields can be set to `nil`, which means that the field is unset,
effectively clearing the field. This is not equivalent to setting the value to
an \"empty\" instance of the message struct.

The compiler also generates a `func (m *Concert) GetHeadliner() *Band` helper
function. This function returns a `nil` `*Band` if `m` is nil or `headliner` is
unset. This makes it possible to chain get calls without intermediate `nil`
checks:

```go
var m *Concert // defaults to nil
log.Infof("GetFoundingYear() = %d (no panic!)", m.GetHeadliner().GetFoundingYear())
```

### Repeated Fields {#repeated}

Each repeated field generates a slice of `T` field in the struct in Go, where
`T` is the field's element type. For this message with a repeated field:

```proto
message Concert {
  // Best practice: use pluralized names for repeated fields:
  // /programming-guides/style#repeated-fields
  repeated Band support_acts = 1;
}
```

the compiler generates the Go struct:

```go
type Concert struct {
    SupportActs []*Band
}
```

Likewise, for the field definition `repeated bytes band_promo_images = 1;` the
compiler will generate a Go struct with a `[][]byte` field named
`BandPromoImage`. For a repeated [enumeration](#enum) like `repeated MusicGenre
genres = 2;`, the compiler generates a struct with a `[]MusicGenre` field called
`Genre`.

The following example shows how to set the field:

```go
concert := &Concert{
  SupportActs: []*Band{
    {}, // First element.
    {}, // Second element.
  },
}
```

To access the field, you can do the following:

```go
support := concert.GetSupportActs() // support type is []*Band.
b1 := support[0] // b1 type is *Band, the first element in support_acts.
```

### Map Fields {#map}

Each map field generates a field in the struct of type `map[TKey]TValue` where
`TKey` is the field's key type and `TValue` is the field's value type. For this
message with a map field:

```proto
message MerchItem {}

message MerchBooth {
  // items maps from merchandise item name ("Signed T-Shirt") to
  // a MerchItem message with more details about the item.
  map<string, MerchItem> items = 1;
}
```

the compiler generates the Go struct:

```go
type MerchBooth struct {
    Items map[string]*MerchItem
}
```

### Oneof Fields {#oneof}

For a oneof field, the protobuf compiler generates a single field with an
interface type `isMessageName_MyField`. It also generates a struct for each of
the [singular fields](#singular-scalar-proto2) within the oneof. These all
implement this `isMessageName_MyField` interface.

For this message with a oneof field:

```proto
package account;
message Profile {
  oneof avatar {
    string image_url = 1;
    bytes image_data = 2;
  }
}
```

the compiler generates the structs:

```go
type Profile struct {
    // Types that are valid to be assigned to Avatar:
    //  *Profile_ImageUrl
    //  *Profile_ImageData
    Avatar isProfile_Avatar `protobuf_oneof:"avatar"`
}

type Profile_ImageUrl struct {
        ImageUrl string
}
type Profile_ImageData struct {
        ImageData []byte
}
```

Both `*Profile_ImageUrl` and `*Profile_ImageData` implement `isProfile_Avatar`
by providing an empty `isProfile_Avatar()` method.

The following example shows how to set the field:

```go
p1 := &account.Profile{
  Avatar: &account.Profile_ImageUrl{ImageUrl: "http://example.com/image.png"},
}

// imageData is []byte
imageData := getImageData()
p2 := &account.Profile{
  Avatar: &account.Profile_ImageData{ImageData: imageData},
}
```

To access the field, you can use a type switch on the value to handle the
different message types.

```go
switch x := m.Avatar.(type) {
case *account.Profile_ImageUrl:
    // Load profile image based on URL
    // using x.ImageUrl
case *account.Profile_ImageData:
    // Load profile image based on bytes
    // using x.ImageData
case nil:
    // The field is not set.
default:
    return fmt.Errorf("Profile.Avatar has unexpected type %T", x)
}
```

The compiler also generates get methods `func (m *Profile) GetImageUrl() string`
and `func (m *Profile) GetImageData() []byte`. Each get function returns the
value for that field or the zero value if it is not set.

## Enumerations {#enum}

Given an enumeration like:

```proto
message Venue {
  enum Kind {
    KIND_UNSPECIFIED = 0;
    KIND_CONCERT_HALL = 1;
    KIND_STADIUM = 2;
    KIND_BAR = 3;
    KIND_OPEN_AIR_FESTIVAL = 4;
  }
  Kind kind = 1;
  // ...
}
```

the protocol buffer compiler generates a type and a series of constants with
that type:

```go
type Venue_Kind int32

const (
    Venue_KIND_UNSPECIFIED       Venue_Kind = 0
    Venue_KIND_CONCERT_HALL      Venue_Kind = 1
    Venue_KIND_STADIUM           Venue_Kind = 2
    Venue_KIND_BAR               Venue_Kind = 3
    Venue_KIND_OPEN_AIR_FESTIVAL Venue_Kind = 4
)
```

For enums within a message (like the one above), the type name begins with the
message name:

```go
type Venue_Kind int32
```

For a package-level enum:

```proto
enum Genre {
  GENRE_UNSPECIFIED = 0;
  GENRE_ROCK = 1;
  GENRE_INDIE = 2;
  GENRE_DRUM_AND_BASS = 3;
  // ...
}
```

the Go type name is unmodified from the proto enum name:

```go
type Genre int32
```

This type has a `String()` method that returns the name of a given value.

The `Enum()` method initializes freshly allocated memory with a given value and
returns the corresponding pointer:

```go
func (Genre) Enum() *Genre
```

The protocol buffer compiler generates a constant for each value in the enum.
For enums within a message, the constants begin with the enclosing message's
name:

```go
const (
    Venue_KIND_UNSPECIFIED       Venue_Kind = 0
    Venue_KIND_CONCERT_HALL      Venue_Kind = 1
    Venue_KIND_STADIUM           Venue_Kind = 2
    Venue_KIND_BAR               Venue_Kind = 3
    Venue_KIND_OPEN_AIR_FESTIVAL Venue_Kind = 4
)
```

For a package-level enum, the constants begin with the enum name instead:

```go
const (
    Genre_GENRE_UNSPECIFIED   Genre = 0
    Genre_GENRE_ROCK          Genre = 1
    Genre_GENRE_INDIE         Genre = 2
    Genre_GENRE_DRUM_AND_BASS Genre = 3
)
```

The protobuf compiler also generates a map from integer values to the string
names and a map from the names to the values:

```go
var Genre_name = map[int32]string{
    0: "GENRE_UNSPECIFIED",
    1: "GENRE_ROCK",
    2: "GENRE_INDIE",
    3: "GENRE_DRUM_AND_BASS",
}
var Genre_value = map[string]int32{
    "GENRE_UNSPECIFIED":   0,
    "GENRE_ROCK":          1,
    "GENRE_INDIE":         2,
    "GENRE_DRUM_AND_BASS": 3,
}
```

Note that the `.proto` language allows multiple enum symbols to have the same
numeric value. Symbols with the same numeric value are synonyms. These are
represented in Go in exactly the same way, with multiple names corresponding to
the same numeric value. The reverse mapping contains a single entry for the
numeric value to the name which appears first in the .proto file.

## Extensions (proto2) {#extensions}

Given an extension definition:

```proto
extend Concert {
  optional int32 promo_id = 123;
}
```

The protocol buffer compiler will generate a
[`protoreflect.ExtensionType`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#ExtensionType)
value named `E_Promo_id`. This value may be used with the
[`proto.GetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#GetExtension),
[`proto.SetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#SetExtension),
[`proto.HasExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#HasExtension),
and
[`proto.ClearExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#ClearExtension)
functions to access an extension in a message. The `GetExtension` function and
`SetExtension` functions respectively return and accept an `interface{}` value
containing the extension value type.

For singular scalar extension fields, the extension value type is the
corresponding Go type from the
[scalar value types table](/programming-guides/proto3#scalar).

For singular embedded message extension fields, the extension value type is
`*M`, where `M` is the field message type.

For repeated extension fields, the extension value type is a slice of the
singular type.

For example, given the following definition:

```proto
extend Concert {
  optional int32 singular_int32 = 1;
  repeated bytes repeated_strings = 2;
  optional Band singular_message = 3;
}
```

Extension values may be accessed as:

```go
m := &somepb.Concert{}
proto.SetExtension(m, extpb.E_SingularInt32, int32(1))
proto.SetExtension(m, extpb.E_RepeatedString, []string{"a", "b", "c"})
proto.SetExtension(m, extpb.E_SingularMessage, &extpb.Band{})

v1 := proto.GetExtension(m, extpb.E_SingularInt32).(int32)
v2 := proto.GetExtension(m, extpb.E_RepeatedString).([][]byte)
v3 := proto.GetExtension(m, extpb.E_SingularMessage).(*extpb.Band)
```

Extensions can be declared nested inside of another type. For example, a common
pattern is to do something like this:

```proto
message Promo {
  extend Concert {
    optional int32 promo_id = 124;
  }
}
```

In this case, the `ExtensionType` value is named `E_Promo_Concert`.

## Services {#service}

The Go code generator does not produce output for services by default. If you
enable the [gRPC](https://www.grpc.io/) plugin (see the
[gRPC Go Quickstart guide](https://github.com/grpc/grpc-go/tree/master/examples))
then code will be generated to support gRPC.
