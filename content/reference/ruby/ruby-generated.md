+++
title = "Ruby Generated Code Guide"
weight = 780
linkTitle = "Generated Code Guide"
description = "Describes the API of message objects that the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

You should
read the language guides for
[proto2](/programming-guides/proto2) or
[proto3](/programming-guides/proto3) before reading this
document.

The protocol compiler for Ruby emits Ruby source files that use a DSL to define
the message schema. However the DSL is still subject to change. In this guide we
only describe the API of the generated messages, and not the DSL.

## Compiler Invocation {#invocation}

The protocol buffer compiler produces Ruby output when invoked with the
`--ruby_out=` command-line flag. The parameter to the `--ruby_out=` option is
the directory where you want the compiler to write your Ruby output. The
compiler creates a `.rb` file for each `.proto` file input. The names of the
output files are computed by taking the name of the `.proto` file and making two
changes:

-   The extension (`.proto`) is replaced with `_pb.rb`.
-   The proto path (specified with the `--proto_path=` or `-I` command-line
    flag) is replaced with the output path (specified with the `--ruby_out=`
    flag).

So, for example, let's say you invoke the compiler as follows:

```shell
protoc --proto_path=src --ruby_out=build/gen src/foo.proto src/bar/baz.proto
```

The compiler will read the files `src/foo.proto` and `src/bar/baz.proto` and
produce two output files: `build/gen/foo_pb.rb` and `build/gen/bar/baz_pb.rb`.
The compiler will automatically create the directory `build/gen/bar` if
necessary, but it will *not* create `build` or `build/gen`; they must already
exist.

## Packages {#package}

The package name defined in the `.proto` file is used to generate a module
structure for the generated messages. Given a file like:

```proto
package foo_bar.baz;

message MyMessage {}
```

The protocol compiler generates an output message with the name
`FooBar::Baz::MyMessage`.

However, if the `.proto` file contains the `ruby_package` option, like this:

```proto
option ruby_package = "Foo::Bar";
```

then the generated output will give precedence to the `ruby_package` option
instead and generate `Foo::Bar::MyMessage`.

## Messages {#message}

Given a simple message declaration:

```proto
message Foo {}
```

The protocol buffer compiler generates a class called `Foo`. The generated class
derives from the Ruby `Object` class (protos have no common base class). Unlike
C++ and Java, Ruby generated code is unaffected by the `optimize_for` option in
the `.proto` file; in effect, all Ruby code is optimized for code size.

You should *not* create your own `Foo` subclasses. Generated classes are not
designed for subclassing and may lead to \"fragile base class\" problems.

Ruby message classes define accessors for each field, and also provide the
following standard methods:

-   `Message#dup`, `Message#clone`: Performs a shallow copy of this message and
    returns the new copy.
-   `Message#==`: Performs a deep equality comparison between two messages.
-   `Message#hash`: Computes a shallow hash of the message's value.
-   `Message#to_hash`, `Message#to_h`: Converts the object to a ruby `Hash`
    object. Only the top-level message is converted.
-   `Message#inspect`: Returns a human-readable string representing this
    message.
-   `Message#[]`, `Message#[]=`: Gets or sets a field by string name. In the
    future this will probably also be used to get/set extensions.

The message classes also define the following methods as static. (In general we
prefer static methods, since regular methods can conflict with field names you
defined in your .proto file.)

-   `Message.decode(str)`: Decodes a binary protobuf for this message and
    returns it in a new instance.
-   `Message.encode(proto)`: Serializes a message object of this class to a
    binary string.
-   `Message.decode_json(str)`: Decodes a JSON text string for this message and
    returns it in a new instance.
-   `Message.encode_json(proto)`: Serializes a message object of this class to a
    JSON text string.
-   `Message.descriptor`: Returns the `Google::Protobuf::Descriptor` object for
    this message.

When you create a message, you can conveniently initialize fields in the
constructor. Here is an example of constructing and using a message:

```ruby
message = MyMessage.new(:int_field => 1,
                        :string_field => "String",
                        :repeated_int_field => [1, 2, 3, 4],
                        :submessage_field => SubMessage.new(:foo => 42))
serialized = MyMessage.encode(message)

message2 = MyMessage.decode(serialized)
raise unless message2.int_field == 1
```

### Nested Types

A message can be declared inside another message. For example:

```proto
message Foo {
  message Bar { }
}
```

In this case, the `Bar` class is declared as a class inside of `Foo`, so you can
refer to it as `Foo::Bar`.

## Fields

For each field in a message type, there are accessor methods to set and get the
field. So given a field `foo` you can write:

```ruby
message.foo = get_value()
print message.foo
```

Whenever you set a field, the value is type-checked against the declared type of
that field. If the value is of the wrong type (or out of range), an exception
will be raised.

### Singular Fields

For singular primitive fields (numbers, strings, and boolean), the value you
assign to the field should be of the correct type and must be in the appropriate
range:

-   **Number types**: the value should be a `Fixnum`, `Bignum`, or `Float`. The
    value you assign must be exactly representable in the target type. So
    assigning `1.0` to an int32 field is ok, but assigning `1.2` is not.
-   **Boolean fields**: the value must be `true` or `false`. No other values
    will implicitly convert to true/false.
-   **Bytes fields**: the assigned value must be a `String` object. The protobuf
    library will duplicate the string, convert it to ASCII-8BIT encoding, and
    freeze it.
-   **String fields**: the assigned value must be a `String` object. The
    protobuf library will duplicate the string, convert it to UTF-8 encoding,
    and freeze it.

No automatic `#to_s`, `#to_i`, etc. calls will happen to perform automatic
conversion. You should convert values yourself first, if necessary.

#### Checking Presence

When using `optional` fields, field presence is checked by calling a generated
`has_...?` method. Setting any value&mdash;even the default value&mdash;marks
the field as present. Fields can be cleared by calling a different generated
`clear_...` method. For example, for a message `MyMessage` with an int32 field
`foo`:

```ruby
m = MyMessage.new
raise unless !m.has_foo?
m.foo = 0
raise unless m.has_foo?
m.clear_foo
raise unless !m.has_foo?
```

### Singular Message Fields {#embedded_message}

For submessages, unset fields will return `nil`, so you can always tell if the
message was explicitly set or not. To clear a submessage field, set its value
explicitly to `nil`.

```ruby
if message.submessage_field.nil?
  puts "Submessage field is unset."
else
  message.submessage_field = nil
  puts "Cleared submessage field."
end
```

In addition to comparing and assigning `nil`, generated messages have `has_...`
and `clear_...` methods, which behave the same as for basic types:

```ruby
if message.has_submessage_field?
  raise unless message.submessage_field == nil
  puts "Submessage field is unset."
else
  raise unless message.submessage_field != nil
  message.clear_submessage_field
  raise unless message.submessage_field == nil
  puts "Cleared submessage field."
end
```

When you assign a submessage, it must be a generated message object of the
correct type.

It is possible to create message cycles when you assign submessages. For
example:

```proto
// foo.proto
message RecursiveMessage {
  RecursiveMessage submessage = 1;
}

# test.rb

require 'foo'

message = RecursiveSubmessage.new
message.submessage = message
```

If you try to serialize this, the library will detect the cycle and fail to
serialize.

### Repeated Fields

Repeated fields are represented using a custom class
`Google::Protobuf::RepeatedField`. This class acts like a Ruby `Array` and mixes
in `Enumerable`. Unlike a regular Ruby array, `RepeatedField` is constructed
with a specific type and expects all of the array members to have the correct
type. The types and ranges are checked just like message fields.

```ruby
int_repeatedfield = Google::Protobuf::RepeatedField.new(:int32, [1, 2, 3])

raise unless !int_repeatedfield.empty?

# Raises TypeError.
int_repeatedfield[2] = "not an int32"

# Raises RangeError
int_repeatedfield[2] = 2**33

message.int32_repeated_field = int_repeatedfield

# This isn't allowed; the regular Ruby array doesn't enforce types like we need.
message.int32_repeated_field = [1, 2, 3, 4]

# This is fine, since the elements are copied into the type-safe array.
message.int32_repeated_field += [1, 2, 3, 4]

# The elements can be cleared without reassigning.
int_repeatedfield.clear
raise unless int_repeatedfield.empty?
```

The `RepeatedField` type supports all of the same methods as a regular Ruby
`Array`. You can convert it to a regular Ruby Array with `repeated_field.to_a`.

Unlike singular fields, `has_...?` methods are never generated for repeated
fields.

### Map Fields

Map fields are represented using a special class that acts like a Ruby `Hash`
(`Google::Protobuf::Map`). Unlike a regular Ruby hash, `Map` is constructed with
a specific type for the key and value and expects all of the map's keys and
values to have the correct type. The types and ranges are checked just like
message fields and `RepeatedField` elements.

```ruby
int_string_map = Google::Protobuf::Map.new(:int32, :string)

# Returns nil; items is not in the map.
print int_string_map[5]

# Raises TypeError, value should be a string
int_string_map[11] = 200

# Ok.
int_string_map[123] = "abc"

message.int32_string_map_field = int_string_map
```

## Enumerations {#enum}

Since Ruby does not have native enums, we create a module for each enum with
constants to define the values. Given the `.proto` file:

```proto
message Foo {
  enum SomeEnum {
    VALUE_A = 0;
    VALUE_B = 5;
    VALUE_C = 1234;
  }
  optional SomeEnum bar = 1;
}
```

You can refer to enum values like so:

```ruby
print Foo::SomeEnum::VALUE_A  # => 0
message.bar = Foo::SomeEnum::VALUE_A
```

You may assign either a number or a symbol to an enum field. When reading the
value back, it will be a symbol if the enum value is known, or a number if it is
unknown. Since **proto3** uses open enum semantics, any number may be assigned
to an enum field, even if it was not defined in the enum.

```ruby
message.bar = 0
puts message.bar.inspect  # => :VALUE_A
message.bar = :VALUE_B
puts message.bar.inspect  # => :VALUE_B
message.bar = 999
puts message.bar.inspect  # => 999

# Raises: RangeError: Unknown symbol value for enum field.
message.bar = :UNDEFINED_VALUE

# Switching on an enum value is convenient.
case message.bar
when :VALUE_A
  # ...
when :VALUE_B
  # ...
when :VALUE_C
  # ...
else
  # ...
end
```

An enum module also defines the following utility methods:

-   `Enum#lookup(number)`: Looks up the given number and returns its name, or
    `nil` if none was found. If more than one name has this number, returns the
    first that was defined.
-   `Enum#resolve(symbol)`: Returns the number for this enum name, or `nil` if
    none was found.
-   `Enum#descriptor`: Returns the descriptor for this enum.

## Oneof

Given a message with a oneof:

```proto
message Foo {
  oneof test_oneof {
     string name = 1;
     int32 serial_number = 2;
  }
}
```

The Ruby class corresponding to `Foo` will have members called `name` and
`serial_number` with accessor methods just like regular [fields](#fields).
However, unlike regular fields, at most one of the fields in a oneof can be set
at a time, so setting one field will clear the others.

```ruby
message = Foo.new

# Fields have their defaults.
raise unless message.name == ""
raise unless message.serial_number == 0
raise unless message.test_oneof == nil

message.name = "Bender"
raise unless message.name == "Bender"
raise unless message.serial_number == 0
raise unless message.test_oneof == :name

# Setting serial_number clears name.
message.serial_number = 2716057
raise unless message.name == ""
raise unless message.test_oneof == :serial_number

# Setting serial_number to nil clears the oneof.
message.serial_number = nil
raise unless message.test_oneof == nil
```

For proto2 messages, oneof members have individual `has_...?` methods as well:

```ruby
message = Foo.new

raise unless !message.has_test_oneof?
raise unless !message.has_name?
raise unless !message.has_serial_number?
raise unless !message.has_test_oneof?

message.name = "Bender"
raise unless message.has_test_oneof?
raise unless message.has_name?
raise unless !message.has_serial_number?
raise unless !message.has_test_oneof?
```
