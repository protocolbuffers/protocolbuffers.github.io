+++
title = "C# Generated Code Guide"
weight = 550
linkTitle = "Generated Code Guide"
description = "Describes exactly what C# code the protocol buffer compiler generates for protocol definitions using proto3 syntax."
type = "docs"
+++

You should
read the
[proto3 language guide](/programming-guides/proto3)
before reading this document.

{{% alert title="Note" color="note" %}}
The protobuf compiler can generate C\# interfaces for definitions using `proto2`
syntax starting from release 3.10. Refer to the
[proto2 language guide](/programming-guides/proto2) for
details of the semantics of `proto2` definitions, and see
`docs/csharp/proto2.md`
([view on GitHub](https://github.com/protocolbuffers/protobuf/blob/master/docs/csharp/proto2.md))
for details on the generated C\# code for proto2.
{{% /alert %}}

## Compiler Invocation {#invocation}

The protocol buffer compiler produces C\# output when invoked with the
`--csharp_out` command-line flag. The parameter to the `--csharp_out` option is
the directory where you want the compiler to write your C\# output, although
depending on [other options](#compiler_options) the compiler may create
subdirectories of the specified directory. The compiler creates a single source
file for each `.proto` file input, defaulting to an extension of `.cs` but
configurable via compiler options.

Only `proto3` messages are supported by the C\# code generator. Ensure that each
`.proto` file begins with a declaration of:

```proto
syntax = "proto3";
```

### C\#-specific Options {#compiler_options}

You can provide further C\# options to the protocol buffer compiler using the
`--csharp_opt` command-line flag. The supported options are:

-   **file\_extension**: Sets the file extension for generated code. This
    defaults to `.cs`, but a common alternative is `.g.cs` to indicate that the
    file contains generated code.

-   **base\_namespace**: When this option is specified, the generator creates a
    directory hierarchy for generated source code corresponding to the
    namespaces of the generated classes, using the value of the option to
    indicate which part of the namespace should be considered as the \"base\"
    for the output directory. For example, with the following command-line:

    ```shell
    protoc --proto_path=bar --csharp_out=src --csharp_opt=base_namespace=Example player.proto
    ```

    where `player.proto` has a `csharp_namespace` option of `Example.Game` the
    protocol buffer compiler generates a file `src/Game/Player.cs` being
    created. This option would usually correspond with the **default namespace**
    option in a C\# project in Visual Studio. If the option is specified but
    with an empty value, the full C\# namespace as used in the generated file
    will be used for the directory hierarchy. If the option is not specified at
    all, the generated files are simply written into the directory specified by
    `--csharp_out` without any hierarchy being created.

-   **internal\_access**: When this option is specified, the generator creates
    types with the `internal` access modifier instead of `public`.

-   **serializable**: When this option is specified, the generator adds the
    `[Serializable]` attribute to generated message classes.

Multiple options can be specified by separating them with commas, as in the
following example:

```shell
protoc --proto_path=src --csharp_out=build/gen --csharp_opt=file_extension=.g.cs,base_namespace=Example,internal_access src/foo.proto
```

## File structure {#structure}

The name of the output file is derived from the `.proto` filename by converting
it to Pascal-case, treating underscores as word separators. So, for example, a
file called `player_record.proto` will result in an output file called
`PlayerRecord.cs` (where the file extension can be specified using
`--csharp_opt`, [as shown above](#compiler_options)).

Each generated file takes the following form, in terms of public members. (The
implementation is not shown here.)

```csharp
namespace [...]
{
  public static partial class [... descriptor class name ...]
  {
    public static FileDescriptor Descriptor { get; }
  }

  [... Enums ...]
  [... Message classes ...]
}
```

The `namespace` is inferred from the proto's `package`, using the same
conversion rules as the file name. For example, a proto package of
`example.high_score` would result in a namespace of `Example.HighScore`. You can
override the default generated namespace for a particular .proto using the
`csharp_namespace`
[file option](/programming-guides/proto3#options).

Each top-level enum and message results in an enum or class being declared as
members of the namespace. Additionally, a single static partial class is always
generated for the file descriptor. This is used for reflection-based operations.
The descriptor class is given the same name as the file, without the extension.
However, if there is a message with the same name (as is quite common), the
descriptor class is placed in a nested `Proto` namespace to avoid colliding with
the message.

As an example of all of these rules, consider the `timestamp.proto` file which
is provided as part of Protocol Buffers. A cut down version of `timestamp.proto`
looks like this:

```proto
syntax = "proto3";
package google.protobuf;
option csharp_namespace = "Google.Protobuf.WellKnownTypes";

message Timestamp { ... }
```

The generated `Timestamp.cs` file has the following structure:

```csharp
namespace Google.Protobuf.WellKnownTypes
{
  namespace Proto
  {
    public static partial class Timestamp
    {
      public static FileDescriptor Descriptor { get; }
    }
  }

  public sealed partial class Timestamp : IMessage<Timestamp>
  {
    [...]
  }
}
```

## Messages {#message}

Given a simple message declaration:

```proto
message Foo {}
```

The protocol buffer compiler generates a sealed, partial class called `Foo`,
which implements the `IMessage<Foo>` interface, as shown below with member
declarations. See the inline comments for more information.

```csharp
public sealed partial class Foo : IMessage<Foo>
{
  // Static properties for parsing and reflection
  public static MessageParser<Foo> Parser { get; }
  public static MessageDescriptor Descriptor { get; }

  // Explicit implementation of IMessage.Descriptor, to avoid conflicting with
  // the static Descriptor property. Typically the static property is used when
  // referring to a type known at compile time, and the instance property is used
  // when referring to an arbitrary message, such as during JSON serialization.
  MessageDescriptor IMessage.Descriptor { get; }

  // Parameterless constructor which calls the OnConstruction partial method if provided.
  public Foo();
  // Deep-cloning constructor
  public Foo(Foo);
  // Partial method which can be implemented in manually-written code for the same class, to provide
  // a hook for code which should be run whenever an instance is constructed.
  partial void OnConstruction();

  // Implementation of IDeepCloneable<T>.Clone(); creates a deep clone of this message.
  public Foo Clone();

  // Standard equality handling; note that IMessage<T> extends IEquatable<T>
  public override bool Equals(object other);
  public bool Equals(Foo other);
  public override int GetHashCode();

  // Converts the message to a JSON representation
  public override string ToString();

  // Serializes the message to the protobuf binary format
  public void WriteTo(CodedOutputStream output);
  // Calculates the size of the message in protobuf binary format
  public int CalculateSize();

  // Merges the contents of the given message into this one. Typically
  // used by generated code and message parsers.
  public void MergeFrom(Foo other);

  // Merges the contents of the given protobuf binary format stream
  // into this message. Typically used by generated code and message parsers.
  public void MergeFrom(CodedInputStream input);
}
```

Note that all of these members are always present; the `optimize_for` option
does not affect the output of the C\# code generator.

### Nested Types

A message can be declared inside another message. For example:

```proto
message Foo {
  message Bar {
  }
}
```

In this case&mdash;or if a message contains a nested enum&mdash;the compiler
generates a nested `Types` class, and then a `Bar` class within the `Types`
class, so the full generated code would be:

```csharp
namespace [...]
{
  public sealed partial class Foo : IMessage<Foo>
  {
    public static partial class Types
    {
      public sealed partial class Bar : IMessage<Bar> { ... }
    }
  }
}
```

Although the intermediate `Types` class is inconvenient, it is required to deal
with the common scenario of a nested type having a corresponding field in the
message. You would otherwise end up with both a property and a type with the
same name nested within the same class&mdash;and that would be invalid C\#.

## Fields

The protocol buffer compiler generates a C\# property for each field defined
within a message. The exact nature of the property depends on the nature of the
field: its type, and whether it is singular, repeated, or a map field.

### Singular Fields {#singular}

Any singular field generates a read/write property. A `string` or `bytes` field
will generate an `ArgumentNullException` if a null value is specified; fetching
a value from a field which hasn't been explicitly set will return an empty
string or `ByteString`. Message fields can be set to null values, which is
effectively clearing the field. This is not equivalent to setting the value to
an \"empty\" instance of the message type.

### Repeated Fields {#repeated}

Each repeated field generates a read-only property of type
`Google.Protobuf.Collections.RepeatedField<T>` where `T` is the field's element
type. For the most part, this acts like `List<T>`, but it has an additional
`Add` overload to allow a collection of items to be added in one go. This is
convenient when populating a repeated field in an object initializer.
Additionally, `RepeatedField<T>` has direct support for serialization,
deserialization and cloning, but this is usually used by generated code instead
of manually-written application code.

Repeated fields cannot contain null values, even of message types, except for
the nullable wrapper types [explained below](#wrapper_types).

### Map Fields {#map}

Each map field generates a read-only property of type
`Google.Protobuf.Collections.MapField<TKey, TValue>` where `TKey` is the field's
key type and `TValue` is the field's value type. For the most part, this acts
like `Dictionary<TKey, TValue>`, but it has an additional `Add` overload to
allow another dictionary to be added in one go. This is convenient when
populating a repeated field in an object initializer. Additionally,
`MapField<TKey, TValue>` has direct support for serialization, deserialization
and cloning, but this is usually used by generated code instead of
manually-written application code. Keys in the map are not permitted to be null;
values may be if the corresponding singular field type would support null
values.

### Oneof Fields {#oneof}

Each field within a oneof has a separate property, like a regular
[singular field](#singular). However, the compiler also generates an additional
property to determine which field in the enum has been set, along with an enum
and a method to clear the oneof. For example, for this oneof field definition

```proto
oneof avatar {
  string image_url = 1;
  bytes image_data = 2;
}
```

The compiler will generate these public members:

```csharp
enum AvatarOneofCase
{
  None = 0,
  ImageUrl = 1,
  ImageData = 2
}

public AvatarOneofCase AvatarCase { get; }
public void ClearAvatar();
public string ImageUrl { get; set; }
public ByteString ImageData { get; set; }
```

If a property is the current oneof \"case\", fetching that property will return
the value set for that property. Otherwise, fetching the property will return
the default value for the property's type&mdash;only one member of a oneof can
be set at a time.

Setting any constituent property of the oneof will change the reported \"case\"
of the oneof. As with a regular [singular field](#singular), you cannot set a
oneof field with a `string` or `bytes` type to a null value. Setting a
message-type field to null is equivalent to calling the oneof-specific `Clear`
method.

### Wrapper Type Fields {#wrapper_types}

Most of the well-known types in proto3 do not affect code generation, but the
wrapper types (`StringWrapper`, `Int32Wrapper` etc) change the type and
behaviour of the properties.

All of the wrapper types that correspond to C\# value types (`Int32Wrapper`,
`DoubleWrapper`, `BoolWrapper` etc) are mapped to `Nullable<T>` where `T` is the
corresponding non-nullable type. For example, a field of type `DoubleValue`
results in a C\# property of type `Nullable<double>`.

Fields of type `StringWrapper` or `BytesWrapper` result in C\# properties of
type `string` and `ByteString` being generated, but with a default value of
null, and allowing null to be set as the property value.

For all wrapper types, null values are not permitted in a repeated field, but
are permitted as the values for map entries.

## Enumerations {#enum}

Given an enumeration definition like:

```proto
enum Color {
  COLOR_UNSPECIFIED = 0;
  COLOR_RED = 1;
  COLOR_GREEN = 5;
  COLOR_BLUE = 1234;
}
```

The protocol buffer compiler will generate a C\# enum type called `Color` with
the same set of values. The names of the enum values are converted to make them
more idiomatic for C\# developers:

-   If the original name starts with the upper-cased form of the enum name
    itself, that is removed
-   The result is converted into Pascal case

The `Color` proto enum above would therefore become the following C\# code:

```csharp
enum Color
{
  Unspecified = 0,
  Red = 1,
  Green = 5,
  Blue = 1234
}
```

This name transformation does not affect the text used within the JSON
representation of messages.

Note that the `.proto` language allows multiple enum symbols to have the same
numeric value. Symbols with the same numeric value are synonyms. These are
represented in C\# in exactly the same way, with multiple names corresponding to
the same numeric value.

A non-nested enumeration leads to a C\# enum being generated as a new namespace
member being generated; a nested enumeration lead to a C\# enum being generated
in the `Types` nested class within the class corresponding to the message the
enumeration is nested within.

## Services {#service}

The C\# code generator ignores services entirely.
