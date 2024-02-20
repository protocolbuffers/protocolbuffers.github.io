+++
title = "Dart Generated Code"
weight = 580
linkTitle = "Generated Code"
description = "Describes what Dart code the protocol buffer compiler generates for any given protocol definition."
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

The protocol buffer compiler requires a
[plugin to generate Dart](https://github.com/dart-lang/dart-protoc-plugin) code.
Installing it following the
[instructions](https://github.com/dart-lang/dart-protoc-plugin#how-to-build-and-use)
provides a `protoc-gen-dart` binary which `protoc` uses when invoked with the
`--dart_out` command-line flag. The `--dart_out` flag tells the compiler where
to write the Dart source files. For a `.proto` file input, the compiler produces
among others a `.pb.dart` file.

The name of the `.pb.dart` file is computed by taking the name of the `.proto`
file and making two changes:

-   The extension (`.proto`) is replaced with `.pb.dart`. For example, a file
    called `foo.proto` results in an output file called `foo.pb.dart`.
-   The proto path (specified with the `--proto_path` or `-I` command-line flag)
    is replaced with the output path (specified with the `--dart_out` flag).

For example, when you invoke the compiler as follows:

```shell
protoc --proto_path=src --dart_out=build/gen src/foo.proto src/bar/baz.proto
```

the compiler will read the files `src/foo.proto` and `src/bar/baz.proto`. It
produces: `build/gen/foo.pb.dart` and `build/gen/bar/baz.pb.dart`. The compiler
automatically creates the directory `build/gen/bar` if necessary, but it will
*not* create `build` or `build/gen`; they must already exist.

## Messages {#message}

Given a simple message declaration:

```proto
message Foo {}
```

The protocol buffer compiler generates a class called `Foo`, which extends the
class `GeneratedMessage`.

The class `GeneratedMessage` defines methods that let you check, manipulate,
read, or write the entire message. In addition to these methods, the `Foo` class
defines the following methods and constructors:

-   `Foo()`: Default constructor. Creates an instance where all singular fields
    are unset and repeated fields are empty.
-   `Foo.fromBuffer(...)`: Creates a `Foo` from serialized protocol buffer data
    representing the message.
-   `Foo.fromJson(...)`: Creates a `Foo` from a JSON string encoding the
    message.
-   `Foo clone()`: Creates a deep clone of the fields in the message.
-   `Foo copyWith(void Function(Foo) updates)`: Makes a writable copy of this
    message, applies the `updates` to it, and marks the copy read-only before
    returning it.
-   `static Foo create()`: Factory function to create a single `Foo`.
-   `static PbList<Foo> createRepeated()`: Factory function to create a List
    implementing a mutable repeated field of `Foo` elements.
-   `static Foo getDefault()`: Returns a singleton instance of `Foo`, which is
    identical to a newly-constructed instance of Foo (so all singular fields are
    unset and all repeated fields are empty).

### Nested Types

A message can be declared inside another message. For example:

```proto
message Foo {
  message Bar {
  }
}
```

In this case, the compiler generates two classes: `Foo` and `Foo_Bar`.

## Fields

In addition to the methods described in the previous section, the protocol
buffer compiler generates accessor methods for each field defined within the
message in the `.proto` file.

Note that the generated names always use camel-case naming, even if the field
name in the `.proto` file uses lower-case with underscores
([as it should](/programming-guides/style)). The
case-conversion works as follows:

1.  For each underscore in the name, the underscore is removed, and the
    following letter is capitalized.
2.  If the name will have a prefix attached (e.g. \"has\"), the first letter is
    capitalized. Otherwise, it is lower-cased.

Thus, for the field `foo_bar_baz`, the getter becomes `get fooBarBaz` and a
method prefixed with `has` would be `hasFooBarBaz`.

### Singular Primitive Fields (proto2)

For any of these field definitions:

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

The compiler will generate the following accessor methods in the message class:

-   `int get foo`: Returns the current value of the field. If the field is not
    set, returns the default value.
-   `bool hasFoo()`: Returns `true` if the field is set.
-   `set foo(int value)`: Sets the value of the field. After calling this,
    `hasFoo()` will return `true` and `get foo` will return `value`.
-   `void clearFoo()`: Clears the value of the field. After calling this,
    `hasFoo()` will return `false` and `get foo` will return the default value.

For other simple field types, the corresponding Dart type is chosen according to
the
[scalar value types table](/programming-guides/proto2#scalar).
For message and enum types, the value type is replaced with the message or enum
class.

### Singular Primitive Fields (proto3)

For this field definition:

```proto
int32 foo = 1;
```

The compiler will generate the following accessor methods in the message class:

-   `int get foo`: Returns the current value of the field. If the field is not
    set, returns the default value.
-   `set foo(int value)`: Sets the value of the field. After calling this, `get
    foo` will return `value`.
-   `void clearFoo()`: Clears the value of the field. After calling this,`get
    foo` will return the default value.

**NOTE:** Due to a quirk in the Dart proto3 implementation, the following
methods are generated even if the `optional` modifier, used to request
[presence semantics](/programming-guides/field_presence#presence-in-proto3-apis),
isn't in the proto definition.

-   `bool hasFoo()`: Returns `true` if the field is set.
-   `void clearFoo()`: Clears the value of the field. After calling this,
    `hasFoo()` will return `false` and `get foo` will return the default value.

### Singular Message Fields {#singular-message}

Given the message type:

```proto
message Bar {}
```

For a message with a `Bar` field:

```proto
// proto2
message Baz {
  optional Bar bar = 1;
  // The generated code is the same result if required instead of optional.
}

// proto3
message Baz {
  Bar bar = 1;
}
```

The compiler will generate the following accessor methods in the message class:

-   `Bar get bar`: Returns the current value of the field. If the field is not
    set, returns the default value.
-   `set bar(Bar value)`: Sets the value of the field. After calling this,
    `hasBar()` will return `true` and `get bar` will return `value`.
-   `bool hasBar()`: Returns `true` if the field is set.
-   `void clearBar()`: Clears the value of the field. After calling this,
    `hasBar()` will return `false` and `get bar` will return the default value.
-   `Bar ensureBar()`: Sets `bar` to the empty instance if `hasBar()` returns
    `false`, and then returns the value of `bar`. After calling this, `hasBar()`
    will return `true`.

### Repeated Fields

For this field definition:

```proto
repeated int32 foo = 1;
```

The compiler will generate:

-   `List<int> get foo`: Returns the list backing the field. If the field is not
    set, returns an empty list. Modifications to the list are reflected in the
    field.

### Int64 Fields

For this field definition:

```proto
// proto2
optional int64 bar = 1;

// proto3
int64 bar = 1;
```

The compiler will generate:

-   `Int64 get bar`: Returns an `Int64` object containing the field value.

Note that `Int64` is not built into the Dart core libraries. To work with these
objects, you may need to import the Dart `fixnum` library:

```dart
import 'package:fixnum/fixnum.dart';
```

### Map Fields

Given a [`map`](/programming-guides/proto3#maps) field
definition like this:

```proto
map<int32, int32> map_field = 1;
```

The compiler will generate the following getter:

-   `Map<int, int> get mapField`: Returns the Dart map backing the field. If the
    field is not set, returns an empty map. Modifications to the map are
    reflected in the field.

## Any

Given an [`Any`](/programming-guides/proto3#any) field
like this:

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

In our generated code, the getter for the `details` field returns an instance of
`com.google.protobuf.Any`. This provides the following special methods to pack
and unpack the `Any`'s values:

```dart
    /// Unpacks the message in [value] into [instance].
    ///
    /// Throws a [InvalidProtocolBufferException] if [typeUrl] does not correspond
    /// to the type of [instance].
    ///
    /// A typical usage would be `any.unpackInto(new Message())`.
    ///
    /// Returns [instance].
    T unpackInto<T extends GeneratedMessage>(T instance,
        {ExtensionRegistry extensionRegistry = ExtensionRegistry.EMPTY});

    /// Returns `true` if the encoded message matches the type of [instance].
    ///
    /// Can be used with a default instance:
    /// `any.canUnpackInto(Message.getDefault())`
    bool canUnpackInto(GeneratedMessage instance);

    /// Creates a new [Any] encoding [message].
    ///
    /// The [typeUrl] will be [typeUrlPrefix]/`fullName` where `fullName` is
    /// the fully qualified name of the type of [message].
    static Any pack(GeneratedMessage message,
        {String typeUrlPrefix = 'type.googleapis.com'});
```

## Oneof

Given a [`oneof`](/programming-guides/proto3#oneof)
definition like this:

```proto
message Foo {
  oneof test {
    string name = 1;
    SubMessage sub_message = 2;
  }
}
```

The compiler will generate the following Dart enum type:

```proto
 enum Foo_Test { name, subMessage, notSet }
```

In addition, it will generate these methods:

-   `Foo_Test whichTest()`: Returns the enum indicating which field is set.
    Returns `Foo_Test.notSet` if none of them is set.
-   `void clearTest()`: Clears the value of the oneof field which is currently
    set (if any), and sets the oneof case to `Foo_Test.notSet`.

For each field inside the oneof definition the regular field accessor methods
are generated. For instance for `name`:

-   `String get name`: Returns the current value of the field if the oneof case
    is `Foo_Test.name`. Otherwise, returns the default value.
-   `set name(String value)`: Sets the value of the field and sets the oneof
    case to `Foo_Test.name`. After calling this, `get name` will return `value`
    and `whichTest()` will return `Foo_Test.name`.
-   `void clearName()`: Nothing will be changed if the oneof case is not
    `Foo_Test.name`. Otherwise, clears the value of the field. After calling
    this, `get name` will return the default value and `whichTest()` will return
    `Foo_Test.notSet`.

## Enumerations {#enum}

Given an enum definition like:

```proto
enum Color {
  COLOR_UNSPECIFIED = 0;
  COLOR_RED = 1;
  COLOR_GREEN = 2;
  COLOR_BLUE = 3;
}
```

The protocol buffer compiler will generate a class called `Color`, which extends
the `ProtobufEnum` class. The class will include a `static const Color` for each
of the four values, as well as a `static const List<Color>` that contains the
values.

```dart
static const List<Color> values = <Color> [
  COLOR_UNSPECIFIED,
  COLOR_RED,
  COLOR_GREEN,
  COLOR_BLUE,
];
```

It will also include the following method:

-   `static Color? valueOf(int value)`: Returns the `Color` corresponding to the
    given numeric value.

Each value will have the following properties:

-   `name`: The enum's name, as specified in the .proto file.
-   `value`: The enum's integer value, as specified in the .proto file.

Note that the `.proto` language allows multiple enum symbols to have the same
numeric value. Symbols with the same numeric value are synonyms. For example:

```proto
enum Foo {
  BAR = 0;
  BAZ = 0;
}
```

In this case, `BAZ` is a synonym for `BAR` and will be defined like so:

```dart
static const Foo BAZ = BAR;
```

An enum can be defined nested within a message type. For instance, given an enum
definition like:

```proto
message Bar {
  enum Color {
    COLOR_UNSPECIFIED = 0;
    COLOR_RED = 1;
    COLOR_GREEN = 2;
    COLOR_BLUE = 3;
  }
}
```

The protocol buffer compiler will generate a class called `Bar`, which extends
`GeneratedMessage`, and a class called `Bar_Color`, which extends
`ProtobufEnum`.

## Extensions (proto2 only) {#extension}

Given a file `foo_test.proto` including a message with an
[extension range](/programming-guides/proto2#extensions)
and a top-level extension definition:

```proto
message Foo {
  extensions 100 to 199;
}

extend Foo {
  optional int32 bar = 101;
}
```

The protocol buffer compiler will generate, in addition to the `Foo` class, a
class `Foo_test` which will contain a `static Extension` for each extension
field in the file along with a method for registering all the extensions in an
`ExtensionRegistry` :

-   `static final Extension bar`
-   `static void registerAllExtensions(ExtensionRegistry registry)` : Registers
    all the defined extensions in the given registry.

The extension accessors of `Foo` can be used as follows:

```dart
Foo foo = Foo();
foo.setExtension(Foo_test.bar, 1);
assert(foo.hasExtension(Foo_test.bar));
assert(foo.getExtension(Foo_test.bar)) == 1);
```

Extensions can also be declared nested inside of another message:

```proto
message Baz {
  extend Foo {
    optional int32 bar = 124;
  }
}
```

In this case, the extension `bar` is instead declared as a static member of the
`Baz` class.

When parsing a message that might have extensions, you must provide an
`ExtensionRegistry` in which you have registered any extensions that you want to
be able to parse. Otherwise, those extensions will just be treated like unknown
fields. For example:

```dart
ExtensionRegistry registry = ExtensionRegistry();
registry.add(Baz.bar);
Foo foo = Foo.fromBuffer(input, registry);
```

If you already have a parsed message with unknown fields, you can use
`reparseMessage` on an `ExtensionRegistry` to reparse the message. If the set of
unknown fields contains extensions that are present in the registry, these
extensions are parsed and removed from the unknown field set. Extensions already
present in the message are preserved.

```dart
Foo foo = Foo.fromBuffer(input);
ExtensionRegistry registry = ExtensionRegistry();
registry.add(Baz.bar);
Foo reparsed = registry.reparseMessage(foo);
```

Be aware that this method to retrieve extensions is more expensive overall.
Where possible we recommend using `ExtensionRegistry` with all the needed
extensions when doing `GeneratedMessage.fromBuffer`.

## Services {#service}

Given a service definition:

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

The protocol buffer compiler can be invoked with the \`grpc\` option (e.g.
`--dart_out=grpc:output_folder`), in which case it will generate code to support
[gRPC](//www.grpc.io/). See the
[gRPC Dart Quickstart guide](https://grpc.io/docs/quickstart/dart.html)
for more details.
