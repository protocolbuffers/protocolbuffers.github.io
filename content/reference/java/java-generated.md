+++
title = "Java Generated Code Guide"
weight = 650
linkTitle = "Generated Code Guide"
description = "Describes exactly what Java code the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

Any
differences between proto2 and proto3 generated code are highlighted&mdash;note
that these differences are in the generated code as described in this document,
not the base message classes/interfaces, which are the same in both versions.
You should read the
[proto2 language guide](/programming-guides/proto2)
and/or
[proto3 language guide](/programming-guides/proto3)
before reading this document.

Note that no Java protocol buffer methods accept or return nulls unless
otherwise specified.

## Compiler Invocation {#invocation}

The protocol buffer compiler produces Java output when invoked with the
`--java_out=` command-line flag. The parameter to the `--java_out=` option is
the directory where you want the compiler to write your Java output. For each
`.proto` file input, the compiler creates a wrapper `.java` file containing a
Java class which represents the `.proto` file itself.

If the `.proto` file contains a line like the following:

```proto
option java_multiple_files = true;
```

Then the compiler will also create separate `.java` files for each of the
classes/enums which it will generate for each top-level message, enumeration,
and service declared in the `.proto` file.

Otherwise (i.e. when the `java_multiple_files` option is disabled; which is the
default), the aforementioned wrapper class will also be used as an outer class,
and the generated classes/enums for each top-level message, enumeration, and
service declared in the `.proto` file will all be nested within the outer
wrapper class. Thus the compiler will only generate a single `.java` file for
the entire `.proto` file.

The wrapper class's name is chosen as follows: If the `.proto` file contains a
line like the following:

```proto
option java_outer_classname = "Foo";
```

Then the wrapper class name will be `Foo`. Otherwise, the wrapper class name is
determined by converting the `.proto` file base name to camel case. For example,
`foo_bar.proto` will generate a class name of `FooBar`. If there's a service,
enum, or message (including nested types) in the file with the same name,
"OuterClass" will be appended to the wrapper class's name. Examples:

-   If `foo_bar.proto` contains a message called `FooBar`, the wrapper class
    will generate a class name of `FooBarOuterClass`.
-   If `foo_bar.proto` contains a service called `FooService`, and
    `java_outer_classname` is also set to the string `FooService`, then the
    wrapper class will generate a class name of `FooServiceOuterClass`.

**Note:** If you are using the deprecated v1 of the protobuf API, `OuterClass`
is added regardless of any collisions with message names.

In addition to any nested classes, the wrapper class itself will have the
following API (assuming the wrapper class is named `Foo` and was generated from
`foo.proto`):

```java
public final class Foo {
  private Foo() {}  // Not instantiable.

  /** Returns a FileDescriptor message describing the contents of {@code foo.proto}. */
  public static com.google.protobuf.Descriptors.FileDescriptor getDescriptor();
  /** Adds all extensions defined in {@code foo.proto} to the given registry. */
  public static void registerAllExtensions(com.google.protobuf.ExtensionRegistry registry);
  public static void registerAllExtensions(com.google.protobuf.ExtensionRegistryLite registry);

  // (Nested classes omitted)
}
```

The Java package name is chosen as described under [Packages](#package), below.

The output file is chosen by concatenating the parameter to `--java_out=`, the
package name (with `.`s replaced with `/`s), and the `.java` file name.

So, for example, let's say you invoke the compiler as follows:

```shell
protoc --proto_path=src --java_out=build/gen src/foo.proto
```

If `foo.proto`'s Java package is `com.example` and it doesn't enable
`java_multiple_files` and its outer classname is `FooProtos`, then the protocol
buffer compiler will generate the file `build/gen/com/example/FooProtos.java`.
The protocol buffer compiler will automatically create the `build/gen/com` and
`build/gen/com/example` directories if needed. However, it will not create
`build/gen` or `build`; they must already exist. You can specify multiple
`.proto` files in a single invocation; all output files will be generated at
once.

When outputting Java code, the protocol buffer compiler's ability to output
directly to JAR archives is particularly convenient, as many Java tools are able
to read source code directly from JAR files. To output to a JAR file, simply
provide an output location ending in `.jar`. Note that only the Java source code
is placed in the archive; you must still compile it separately to produce Java
class files.

For example, if the `.proto` file contains:

```proto
package foo.bar;
```

Then the resulting Java class will be placed in Java package
`foo.bar`. However, if the `.proto` file also
contains a `java_package` option, like so:

```proto
package foo.bar;
option java_package = "com.example.foo.bar";
```

Then the class is placed in the `com.example.foo.bar` package instead. The
`java_package` option is provided because normal `.proto` `package` declarations
are not expected to start with a backwards domain name.

## Messages {#message}

Given a simple message declaration:

```proto
message Foo {}
```

The protocol buffer compiler generates a class called `Foo`, which implements
the `Message` interface. The class is declared `final`; no further subclassing
is allowed. `Foo` extends `GeneratedMessage`, but this should be considered an
implementation detail. By default, `Foo` overrides many methods of
`GeneratedMessage` with specialized versions for maximum speed. However, if the
`.proto` file contains the line:

```proto
option optimize_for = CODE_SIZE;
```

then `Foo` will override only the minimum set of methods necessary to function
and rely on `GeneratedMessage`'s reflection-based implementations of the rest.
This significantly reduces the size of the generated code, but also reduces
performance. Alternatively, if the `.proto` file contains:

```proto
option optimize_for = LITE_RUNTIME;
```

then `Foo` will include fast implementations of all methods, but will implement
the `MessageLite` interface, which contains a subset of the methods of
`Message`. In particular, it does not support descriptors, nested builders, or
reflection. However, in this mode, the generated code only needs to link against
`libprotobuf-lite.jar` instead of `libprotobuf.jar`. The "lite" library is much
smaller than the full library, and is more appropriate for resource-constrained
systems such as mobile phones.

The `Message` interface defines methods that let you check, manipulate, read, or
write the entire message. In addition to these methods, the `Foo` class defines
the following static methods:

-   `static Foo getDefaultInstance()`: Returns the *singleton* instance of
    `Foo`. This instance's contents are identical to what you'd get if you
    called `Foo.newBuilder().build()` (so all singular fields are unset and all
    repeated fields are empty). Note that the default instance of a message can
    be used as a factory by calling its `newBuilderForType()` method.
-   `static Descriptor getDescriptor()`: Returns the type's descriptor. This
    contains information about the type, including what fields it has and what
    their types are. This can be used with the reflection methods of the
    `Message`, such as `getField()`.
-   `static Foo parseFrom(...)`: Parses a message of type `Foo` from the given
    source and returns it. There is one `parseFrom` method corresponding to each
    variant of `mergeFrom()` in the `Message.Builder` interface. Note that
    `parseFrom()` never throws `UninitializedMessageException`; it throws
    `InvalidProtocolBufferException` if the parsed message is missing required
    fields. This makes it subtly different from calling
    `Foo.newBuilder().mergeFrom(...).build()`.
-   `static Parser parser()`: Returns an instance of the `Parser`, which
    implements various `parseFrom()` methods.
-   `Foo.Builder newBuilder()`: Creates a new builder (described below).
-   `Foo.Builder newBuilder(Foo prototype)`: Creates a new builder with all
    fields initialized to the same values that they have in `prototype`. Since
    embedded message and string objects are immutable, they are shared between
    the original and the copy.

### Builders {#builders}

Message objects&mdash;such as instances of the `Foo` class described
above&mdash;are immutable, just like a Java `String`. To construct a message
object, you need to use a *builder*. Each message class has its own builder
class&mdash;so in our `Foo` example, the protocol buffer compiler generates a
nested class `Foo.Builder` which can be used to build a `Foo`. `Foo.Builder`
implements the `Message.Builder` interface. It extends the
`GeneratedMessage.Builder` class, but, again, this should be considered an
implementation detail. Like `Foo`, `Foo.Builder` may rely on generic method
implementations in `GeneratedMessage.Builder` or, when the `optimize_for` option
is used, generated custom code that is much faster.

`Foo.Builder` does not define any static methods. Its interface is exactly as
defined by the `Message.Builder` interface, with the exception that return types
are more specific: methods of `Foo.Builder` that modify the builder return type
`Foo.Builder`, and `build()` returns type `Foo`.

Methods that modify the contents of a builder&mdash;including field
setters&mdash;always return a reference to the builder (i.e. they "`return
this;`"). This allows multiple method calls to be chained together in one line.
For example: `builder.mergeFrom(obj).setFoo(1).setBar("abc").clearBaz();`

Note that builders are not thread-safe, so Java synchronization should be used
whenever it is necessary for multiple different threads to be modifying the
contents of a single builder.

### Sub-Builders {#sub-builders}

For messages containing sub-messages, the compiler also generates sub-builders.
This allows you to repeatedly modify deep-nested sub-messages without rebuilding
them. For example:

```proto
message Foo {
  optional int32 val = 1;
  // some other fields.
}

message Bar {
  optional Foo foo = 1;
  // some other fields.
}

message Baz {
  optional Bar bar = 1;
  // some other fields.
}
```

If you have a `Baz` message already, and want to change the deeply nested `val`
in `Foo`. Instead of:

```java
baz = baz.toBuilder().setBar(
    baz.getBar().toBuilder().setFoo(
        baz.getBar().getFoo().toBuilder().setVal(10).build()
    ).build()).build();
```

You can write:

```java
Baz.Builder builder = baz.toBuilder();
builder.getBarBuilder().getFooBuilder().setVal(10);
baz = builder.build();
```

### Nested Types {#nested}

A message can be declared inside another message. For example:

```proto
message Foo {
  message Bar { }
}
```

In this case, the compiler simply generates `Bar` as an inner class nested
inside `Foo`.

## Fields {#fields}

In addition to the methods described in the previous section, the protocol
buffer compiler generates a set of accessor methods for each field defined
within the message in the `.proto` file. The methods that read the field value
are defined both in the message class and its corresponding builder; the methods
that modify the value are defined in the builder only.

Note that method names always use camel-case naming, even if the field name in
the `.proto` file uses lower-case with underscores
([as it should](/programming-guides/style)). The
case-conversion works as follows:

*   For each underscore in the name, the underscore is removed, and the
    following letter is capitalized.
*   If the name will have a prefix attached (e.g. "get"), the first letter is
    capitalized. Otherwise, it is lower-cased.
*   The letter following the last digit in each number in a method name is
    capitalized.

Thus, the field `foo_bar_baz` becomes `fooBarBaz`. If prefixed with `get`, it
would be `getFooBarBaz`. And `foo_ba23r_baz` becomes `fooBa23RBaz`.

As well as accessor methods, the compiler generates an integer constant for each
field containing its field number. The constant name is the field name converted
to upper-case followed by `_FIELD_NUMBER`. For example, given the field
`optional int32 foo_bar = 5;`, the compiler will generate the constant `public
static final int FOO_BAR_FIELD_NUMBER = 5;`.

### Singular Fields (proto2) {#singular-proto2}

For any of these field definitions:

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

The compiler will generate the following accessor methods in both the message
class and its builder:

-   `boolean hasFoo()`: Returns `true` if the field is set.
-   `int getFoo()`: Returns the current value of the field. If the field is not
    set, returns the default value.

The compiler will generate the following methods only in the message's builder:

-   `Builder setFoo(int value)`: Sets the value of the field. After calling
    this, `hasFoo()` will return `true` and `getFoo()` will return `value`.
-   `Builder clearFoo()`: Clears the value of the field. After calling this,
    `hasFoo()` will return `false` and `getFoo()` will return the default value.

For other simple field types, the corresponding Java type is chosen according to
the
[scalar value types table](/programming-guides/proto2#scalar).
For message and enum types, the value type is replaced with the message or enum
class.

#### Embedded Message Fields {#embedded-message-proto2}

For message types, `setFoo()` also accepts an instance of the message's builder
type as the parameter. This is just a shortcut which is equivalent to calling
`.build()` on the builder and passing the result to the method.

If the field is not set, `getFoo()` will return a Foo instance with none of its
fields set (possibly the instance returned by `Foo.getDefaultInstance()`).

In addition, the compiler generates two accessor methods that allow you to
access the relevant sub-builders for message types. The following method is
generated in both the message class and its builder:

-   `FooOrBuilder getFooOrBuilder()`: Returns the builder for the field, if it
    already exists, or the message if not. Calling this method on builders will
    not create a sub-builder for the field.

The compiler generates the following method only in the message's builder.

-   `Builder getFooBuilder()`: Returns the builder for the field.

### Singular Fields (proto3) {#singular-proto3}

For this field definition:

```proto
int32 foo = 1;
```

The compiler will generate the following accessor method in both the message
class and its builder:

-   `int getFoo()`: Returns the current value of the field. If the field is not
    set, returns the default value for the field's type.

The compiler will generate the following methods only in the message's builder:

-   `Builder setFoo(int value)`: Sets the value of the field. After calling
    this, `getFoo()` will return `value`.
-   `Builder clearFoo()`: Clears the value of the field. After calling this,
    `getFoo()` will return the default value for the field's type.

For other simple field types, the corresponding Java type is chosen according to
the
[scalar value types table](/programming-guides/proto2#scalar).
For message and enum types, the value type is replaced with the message or enum
class.

#### Embedded Message Fields {#embedded-message-proto3}

For message field types, an additional accessor method is generated in both the
message class and its builder:

-   `boolean hasFoo()`: Returns `true` if the field has been set.

`setFoo()` also accepts an instance of the message's builder type as the
parameter. This is just a shortcut which is equivalent to calling `.build()` on
the builder and passing the result to the method.

If the field is not set, `getFoo()` will return a Foo instance with none of its
fields set (possibly the instance returned by `Foo.getDefaultInstance()`).

In addition, the compiler generates two accessor methods that allow you to
access the relevant sub-builders for message types. The following method is
generated in both the message class and its builder:

-   `FooOrBuilder getFooOrBuilder()`: Returns the builder for the field, if it
    already exists, or the message if not. Calling this method on builders will
    not create a sub-builder for the field.

The compiler generates the following method only in the message's builder.

-   `Builder getFooBuilder()`: Returns the builder for the field.

#### Enum Fields {#enum-proto3}

For enum field types, an additional accessor method is generated in both the
message class and its builder:

-   `int getFooValue()`: Returns the integer value of the enum.

The compiler will generate the following additional method only in the message's
builder:

-   `Builder setFooValue(int value)`: Sets the integer value of the enum.

In addition, `getFoo()` will return `UNRECOGNIZED` if the enum value is
unknown&mdash;this is a special additional value added by the proto3 compiler to
the generated [enum type](#enum).

### Repeated Fields {#repeated}

For this field definition:

```proto
repeated string foos = 1;
```

The compiler will generate the following accessor methods in both the message
class and its builder:

-   `int getFoosCount()`: Returns the number of elements currently in the field.
-   `String getFoos(int index)`: Returns the element at the given zero-based
    index.
-   `ProtocolStringList getFoosList()`: Returns the entire field as a
    `ProtocolStringList`. If the field is not set, returns an empty list.

The compiler will generate the following methods only in the message's builder:

-   `Builder setFoos(int index, String value)`: Sets the value of the element at
    the given zero-based index.
-   `Builder addFoos(String value)`: Appends a new element to the field with the
    given value.
-   `Builder addAllFoos(Iterable<? extends String> value)`: Appends all elements
    in the given `Iterable` to the field.
-   `Builder clearFoos()`: Removes all elements from the field. After calling
    this, `getFoosCount()` will return zero.

For other simple field types, the corresponding Java type is chosen according to
the
[scalar value types table](/programming-guides/proto2#scalar).
For message and enum types, the type is the message or enum class.

#### Repeated Embedded Message Fields {#repeated-embedded}

For message types, `setFoos()` and `addFoos()` also accept an instance of the
message's builder type as the parameter. This is just a shortcut which is
equivalent to calling `.build()` on the builder and passing the result to the
method. There is also an additional generated method:

-   `Builder addFoos(int index, Field value)`: Inserts a new element at the
    given zero-based index. Shifts the element currently at that position (if
    any) and any subsequent elements to the right (adds one to their indices).

In addition, the compiler generates the following additional accessor methods in
both the message class and its builder for message types, allowing you to access
the relevant sub-builders:

-   `FooOrBuilder getFoosOrBuilder(int index)`: Returns the builder for the
    specified element, if it already exists, or throws
    `IndexOutOfBoundsException` if not. If this is called from a message class,
    it will always return a message (or throw an exception) rather than a
    builder. Calling this method on builders will not create a sub-builder for
    the field.
-   `List<FooOrBuilder> getFoosOrBuilderList()`: Returns the entire field as an
    unmodifiable list of builders (if available) or messages if not. If this is
    called from a message class, it will always return an immutable list of
    messages rather than an unmodifiable list of builders.

The compiler will generate the following methods only in the message's builder:

-   `Builder getFoosBuilder(int index)`: Returns the builder for the element at
    the specified index, or throws `IndexOutOfBoundsException` if the index is
    out of bounds.
-   `Builder addFoosBuilder(int index)`: Inserts and returns a builder for a
    default message instance of the repeated message at the specified index. The
    existing entries are shifted to higher indices to make room for the inserted
    builder.
-   `Builder addFoosBuilder()`: Appends and returns a builder for a default
    message instance of the repeated message.
-   `Builder removeFoos(int index)`: Removes the element at the given zero-based
    index.
-   `List<Builder> getFoosBuilderList()`: Returns the entire field as an
    unmodifiable list of builders.

#### Repeated Enum Fields (proto3 only) {#repeated-enum-proto3}

The compiler will generate the following additional methods in both the message
class and its builder:

-   `int getFoosValue(int index)`: Returns the integer value of the enum at the
    specified index.
-   `List<java.lang.Integer> getFoosValueList()`: Returns the entire field as a
    list of Integers.

The compiler will generate the following additional method only in the message's
builder:

-   `Builder setFoosValue(int index, int value)`: Sets the integer value of the
    enum at the specified index.

#### Name Conflicts {#conflicts}

If another non-repeated field has a name that conflicts with one of the repeated
field's generated methods, then both field names will have their protobuf field
number appended to the end.

For these field definitions:

```proto
int32 foos_count = 1;
repeated string foos = 2;
```

The compiler will first rename them to the following:

```proto
int32 foos_count_1 = 1;
repeated string foos_2 = 2;
```

The accessor methods will subsequently be generated as described above.

<a id="oneof"></a>

### Oneof Fields {#oneof-fields}

For this oneof field definition:

```proto
oneof choice {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```

All the fields in the `choice` oneof will use a single private field for their
value. In addition, the protocol buffer compiler will generate a Java enum type
for the oneof case, as follows:

```java
public enum ChoiceCase
        implements com.google.protobuf.Internal.EnumLite {
      FOO_INT(4),
      FOO_STRING(9),
      ...
      CHOICE_NOT_SET(0);
      ...
    };
```

The values of this enum type have the following special methods:

-   `int getNumber()`: Returns the object's numeric value as defined in the
    .proto file.
-   `static ChoiceCase forNumber(int value)`: Returns the enum object
    corresponding to the given numeric value (or `null` for other numeric
    values).

The compiler will generate the following accessor methods in both the message
class and its builder:

-   `boolean hasFooInt()`: Returns `true` if the oneof case is `FOO`.
-   `int getFooInt()`: Returns the current value of `foo` if the oneof case is
    `FOO`. Otherwise, returns the default value of this field.
-   `ChoiceCase getChoiceCase()`: Returns the enum indicating which field is
    set. Returns `CHOICE_NOT_SET` if none of them is set.

The compiler will generate the following methods only in the message's builder:

-   `Builder setFooInt(int value)`: Sets `Foo` to this value and sets the oneof
    case to `FOO`. After calling this, `hasFooInt()` will return `true`,
    `getFooInt()` will return `value` and `getChoiceCase()` will return `FOO`.
-   `Builder clearFooInt()`:
    -   Nothing will be changed if the oneof case is not `FOO`.
    -   If the oneof case is `FOO`, sets `Foo` to null and the oneof case to
        `FOO_NOT_SET`. After calling this, `hasFooInt()` will return `false`,
        `getFooInt()` will return the default value and `getChoiceCase()` will
        return `FOO_NOT_SET`.
-   `Builder.clearChoice()`: Resets the value for `choice`, returning the
    builder.

For other simple field types, the corresponding Java type is chosen according to
the
[scalar value types table](/programming-guides/proto2#scalar).
For message and enum types, the value type is replaced with the message or enum
class.

### Map Fields {#map-fields}

For this map field definition:

```proto
map<int32, int32> weight = 1;
```

The compiler will generate the following accessor methods in both the message
class and its builder:

-   `Map<Integer, Integer> getWeightMap();`: Returns an unmodifiable `Map`.
-   `int getWeightOrDefault(int key, int default);`: Returns the value for key
    or the default value if not present.
-   `int getWeightOrThrow(int key);`: Returns the value for key or throws
    IllegalArgumentException if not present.
-   `boolean containsWeight(int key);`: Indicates if the key is present in this
    field.
-   `int getWeightCount();`: Returns the number of elements in the map.

The compiler will generate the following methods only in the message's builder:

-   `Builder putWeight(int key, int value);`: Add the weight to this field.
-   `Builder putAllWeight(Map<Integer, Integer> value);`: Adds all entries in
    the given map to this field.
-   `Builder removeWeight(int key);`: Removes the weight from this field.
-   `Builder clearWeight();`: Removes all weights from this field.
-   `@Deprecated Map<Integer, Integer> getMutableWeight();`: Returns a mutable
    `Map`. Note that multiple calls to this method may return different map
    instances. The returned map reference may be invalidated by any subsequent
    method calls to the Builder.

#### Message Value Map Fields {#message-value-map-fields}

For maps with message types as values, the compiler will generate an additional
method in the message's builder:

-   `Foo.Builder putFooBuilderIfAbsent(int key);`: Ensures that `key` is present
    in the mapping, and inserts a new `Foo.Builder` if one does not already
    exist. Changes to the returned `Foo.Builder` will be reflected in the final
    message.

## Any {#any-fields}

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

```java
class Any {
  // Packs the given message into an Any using the default type URL
  // prefix “type.googleapis.com”.
  public static Any pack(Message message);
  // Packs the given message into an Any using the given type URL
  // prefix.
  public static Any pack(Message message,
                         String typeUrlPrefix);

  // Checks whether this Any message’s payload is the given type.
  public <T extends Message> boolean is(class<T> clazz);

  // Unpacks Any into the given message type. Throws exception if
  // the type doesn’t match or parsing the payload has failed.
  public <T extends Message> T unpack(class<T> clazz)
      throws InvalidProtocolBufferException;
}
```

## Enumerations {#enum}

Given an enum definition like:

```proto
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

The protocol buffer compiler will generate a Java enum type called `Foo` with
the same set of values. If you are using proto3, it also adds the special value
`UNRECOGNIZED` to the enum type. The values of the generated enum type have the
following special methods:

-   `int getNumber()`: Returns the object's numeric value as defined in the
    `.proto` file.
-   `EnumValueDescriptor getValueDescriptor()`: Returns the value's descriptor,
    which contains information about the value's name, number, and type.
-   `EnumDescriptor getDescriptorForType()`: Returns the enum type's descriptor,
    which contains e.g. information about each defined value.

Additionally, the `Foo` enum type contains the following static methods:

-   `static Foo forNumber(int value)`: Returns the enum object corresponding to
    the given numeric value. Returns null when there is no corresponding enum
    object.
-   `static Foo valueOf(int value)`: Returns the enum object corresponding to
    the given numeric value. This method is deprecated in favor of
    `forNumber(int value)` and will be removed in an upcoming release.
-   `static Foo valueOf(EnumValueDescriptor descriptor)`: Returns the enum
    object corresponding to the given value descriptor. May be faster than
    `valueOf(int)`. In proto3 returns `UNRECOGNIZED` if passed an unknown value
    descriptor.
-   `EnumDescriptor getDescriptor()`: Returns the enum type's descriptor, which
    contains e.g. information about each defined value. (This differs from
    `getDescriptorForType()` only in that it is a static method.)

An integer constant is also generated with the suffix \_VALUE for each enum
value.

Note that the `.proto` language allows multiple enum symbols to have the same
numeric value. Symbols with the same numeric value are synonyms. For example:

```proto
enum Foo {
  BAR = 0;
  BAZ = 0;
}
```

In this case, `BAZ` is a synonym for `BAR`. In Java, `BAZ` will be defined as a
static final field like so:

```java
static final Foo BAZ = BAR;
```

Thus, `BAR` and `BAZ` compare equal, and `BAZ` should never appear in switch
statements. The compiler always chooses the first symbol defined with a given
numeric value to be the "canonical" version of that symbol; all subsequent
symbols with the same number are just aliases.

An enum can be defined nested within a message type. The compiler generates the
Java enum definition nested within that message type's class.

**Caution: when generating Java code, the maximum number of values in a protobuf
enum may be surprisingly low**&mdash;in the worst case, the maximum is slightly
over 1,700 values. This limit is due to per-method size limits for Java
bytecode, and it varies across Java implementations, different versions of the
protobuf suite, and any options set on the enum in the `.proto` file.

## Extensions (proto2 only) {#extension}

Given a message with an extension range:

```proto
message Foo {
  extensions 100 to 199;
}
```

The protocol buffer compiler will make `Foo` extend
`GeneratedMessage.ExtendableMessage` instead of the usual `GeneratedMessage`.
Similarly, `Foo`'s builder will extend `GeneratedMessage.ExtendableBuilder`. You
should never refer to these base types by name (`GeneratedMessage` is considered
an implementation detail). However, these superclasses define a number of
additional methods that you can use to manipulate extensions.

In particular `Foo` and `Foo.Builder` will inherit the methods `hasExtension()`,
`getExtension()`, and `getExtensionCount()`. Additionally, `Foo.Builder` will
inherit methods `setExtension()` and `clearExtension()`. Each of these methods
takes, as its first parameter, an extension identifier (described below), which
identifies an extension field. The remaining parameters and the return value are
exactly the same as those for the corresponding accessor methods that would be
generated for a normal (non-extension) field of the same type as the extension
identifier.

Given an extension definition:

```proto
extend Foo {
  optional int32 bar = 123;
}
```

The protocol buffer compiler generates an "extension identifier" called `bar`,
which you can use with `Foo`'s extension accessors to access this extension,
like so:

```java
Foo foo =
  Foo.newBuilder()
     .setExtension(bar, 1)
     .build();
assert foo.hasExtension(bar);
assert foo.getExtension(bar) == 1;
```

(The exact implementation of extension identifiers is complicated and involves
magical use of generics&mdash;however, you don't need to worry about how
extension identifiers work to use them.)

Note that `bar` would be declared as a static field of the wrapper class for the
`.proto` file, as described above; we have omitted the wrapper class name in the
example.

Extensions can be declared inside the scope of another type to prefix their
generated symbol names. For example, a common pattern is to extend a message by
a field *inside* the declaration of the field's type:

```proto
message Baz {
  extend Foo {
    optional Baz foo_ext = 124;
  }
}
```

In this case, an extension with identifier `foo_ext` and type `Baz` is declared
inside the declaration of `Baz`, and referring to `foo_ext` requires the
addition of a `Baz.` prefix:

```java
Baz baz = createMyBaz();
Foo foo =
  Foo.newBuilder()
     .setExtension(Baz.fooExt, baz)
     .build();
assert foo.hasExtension(Baz.fooExt);
assert foo.getExtension(Baz.fooExt) == baz;
```

When parsing a message that might have extensions, you must provide an
[`ExtensionRegistry`](/reference/java/api-docs/com/google/protobuf/ExtensionRegistry.html)
in which you have registered any extensions that you want to be able to parse.
Otherwise, those extensions will be treated like unknown fields and the methods
observing extensions will behave as if they don't exist.

```java
ExtensionRegistry registry = ExtensionRegistry.newInstance();
registry.add(Baz.fooExt);
Foo foo = Foo.parseFrom(input, registry);
assert foo.hasExtension(Baz.fooExt);
```

```java
ExtensionRegistry registry = ExtensionRegistry.newInstance();
Foo foo = Foo.parseFrom(input, registry);
assert foo.hasExtension(Baz.fooExt) == false;
```

## Services {#service}

If the `.proto` file contains the following line:

```proto
option java_generic_services = true;
```

Then the protocol buffer compiler will generate code based on the service
definitions found in the file as described in this section. However, the
generated code may be undesirable as it is not tied to any particular RPC
system, and thus requires more levels of indirection than code tailored to one
system. If you do NOT want this code to be generated, add this line to the file:

```proto
option java_generic_services = false;
```

If neither of the above lines are given, the option defaults to `false`, as
generic services are deprecated. (Note that prior to 2.4.0, the option defaults
to `true`)

RPC systems based on `.proto`-language service definitions should provide
[plugins](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)
to generate code appropriate for the system. These plugins are likely to require
that abstract services are disabled, so that they can generate their own classes
of the same names. Plugins are new in version 2.3.0 (January 2010).

The remainder of this section describes what the protocol buffer compiler
generates when abstract services are enabled.

### Interface {#interface}

Given a service definition:

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

The protocol buffer compiler will generate an abstract class `Foo` to represent
this service. `Foo` will have an abstract method for each method defined in the
service definition. In this case, the method `Bar` is defined as:

```java
abstract void bar(RpcController controller, FooRequest request,
                  RpcCallback<FooResponse> done);
```

The parameters are equivalent to the parameters of `Service.CallMethod()`,
except that the `method` argument is implied and `request` and `done` specify
their exact type.

`Foo` subclasses the `Service` interface. The protocol buffer compiler
automatically generates implementations of the methods of `Service` as follows:

-   `getDescriptorForType`: Returns the service's `ServiceDescriptor`.
-   `callMethod`: Determines which method is being called based on the provided
    method descriptor and calls it directly, down-casting the request message
    and callback to the correct types.
-   `getRequestPrototype` and `getResponsePrototype`: Returns the default
    instance of the request or response of the correct type for the given
    method.

The following static method is also generated:

-   `static ServiceDescriptor getServiceDescriptor()`: Returns the type's
    descriptor, which contains information about what methods this service has
    and what their input and output types are.

`Foo` will also contain a nested interface `Foo.Interface`. This is a pure
interface that again contains methods corresponding to each method in your
service definition. However, this interface does not extend the `Service`
interface. This is a problem because RPC server implementations are usually
written to use abstract `Service` objects, not your particular service. To solve
this problem, if you have an object `impl` implementing `Foo.Interface`, you can
call `Foo.newReflectiveService(impl)` to construct an instance of `Foo` that
simply delegates to `impl`, and implements `Service`.

To recap, when implementing your own service, you have two options:

-   Subclass `Foo` and implement its methods as appropriate, then hand instances
    of your subclass directly to the RPC server implementation. This is usually
    easiest, but some consider it less "pure".
-   Implement `Foo.Interface` and use `Foo.newReflectiveService(Foo.Interface)`
    to construct a `Service` wrapping it, then pass the wrapper to your RPC
    implementation.

### Stub {#stub}

The protocol buffer compiler also generates a "stub" implementation of every
service interface, which is used by clients wishing to send requests to servers
implementing the service. For the `Foo` service (above), the stub implementation
`Foo.Stub` will be defined as a nested class.

`Foo.Stub` is a subclass of `Foo` which also implements the following methods:

-   `Foo.Stub(RpcChannel channel)`: Constructs a new stub which sends requests
    on the given channel.
-   `RpcChannel getChannel()`: Returns this stub's channel, as passed to the
    constructor.

The stub additionally implements each of the service's methods as a wrapper
around the channel. Calling one of the methods simply calls
`channel.callMethod()`.

The Protocol Buffer library does not include an RPC implementation. However, it
includes all of the tools you need to hook up a generated service class to any
arbitrary RPC implementation of your choice. You need only provide
implementations of `RpcChannel` and `RpcController`.

### Blocking Interfaces {#blocking}

The RPC classes described above all have non-blocking semantics: when you call a
method, you provide a callback object which will be invoked once the method
completes. Often it is easier (though possibly less scalable) to write code
using blocking semantics, where the method simply doesn't return until it is
done. To accommodate this, the protocol buffer compiler also generates blocking
versions of your service class. `Foo.BlockingInterface` is equivalent to
`Foo.Interface` except that each method simply returns the result rather than
call a callback. So, for example, `bar` is defined as:

```java
abstract FooResponse bar(RpcController controller, FooRequest request)
                         throws ServiceException;
```

Analogous to non-blocking services,
`Foo.newReflectiveBlockingService(Foo.BlockingInterface)` returns a
`BlockingService` wrapping some `Foo.BlockingInterface`. Finally,
`Foo.BlockingStub` returns a stub implementation of `Foo.BlockingInterface` that
sends requests to a particular `BlockingRpcChannel`.

## Plugin Insertion Points {#plugins}

[Code generator plugins](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)
that want to extend the output of the Java code generator may insert code of the
following types using the given insertion point names.

-   `outer_class_scope`: Member declarations that belong in the file's wrapper
    class.
-   `class_scope:TYPENAME`: Member declarations that belong in a message class.
    `TYPENAME` is the full proto name, e.g. `package.MessageType`.
-   `builder_scope:TYPENAME`: Member declarations that belong in a message's
    builder class. `TYPENAME` is the full proto name, e.g.
    `package.MessageType`.
-   `enum_scope:TYPENAME`: Member declarations that belong in an enum class.
    `TYPENAME` is the full proto enum name, e.g. `package.EnumType`.
-   `message_implements:TYPENAME`: Class implementation declarations for a
    message class. `TYPENAME` is the full proto name, e.g.
    `package.MessageType`.
-   `builder_implements:TYPENAME`: Class implementation declarations for a
    builder class. `TYPENAME` is the full proto name, e.g.
    `package.MessageType`.

Generated code cannot contain import statements, as these are prone to conflict
with type names defined within the generated code itself. Instead, when
referring to an external class, you must always use its fully-qualified name.

The logic for determining output file names in the Java code generator is fairly
complicated. You should probably look at the `protoc` source code, particularly
`java_headers.cc`, to make sure you have covered all cases.

Do not generate code which relies on private class members declared by the
standard code generator, as these implementation details may change in future
versions of Protocol Buffers.

## Utility Classes {#utility-classes}

Protocol buffer provides
[utility classes](/reference/java/api-docs/com/google/protobuf/util/package-summary.html)
for message comparison, JSON conversion and working with
[well-known types (predefined protocol buffer messages for common use-cases).](/reference/protobuf/google.protobuf)
