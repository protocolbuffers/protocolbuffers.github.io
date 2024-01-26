+++
title = "PHP Generated Code Guide"
weight = 730
linkTitle = "Generated Code Guide"
description = "Describes the PHP code that the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

You should read the
[proto3 language guide](/programming-guides/proto3)
before reading this document. Note that the protocol buffer compiler currently
only supports proto3 code generation for PHP.

## Compiler Invocation {#invocation}

The protocol buffer compiler produces PHP output when invoked with the
`--php_out=` command-line flag. The parameter to the `--php_out=` option is the
directory where you want the compiler to write your PHP output. In order to
conform to PSR-4, the compiler creates a sub-directory corresponding to the
package defined in the proto file. In addition, for each message in the proto
file input the compiler creates a separate file in the package's subdirectory.
The names of the output files of messages are composed of three parts:

-   Base directory: The proto path (specified with the `--proto_path=` or `-I`
    command-line flag) is replaced with the output path (specified with the
    `--php_out=` flag).
-   Sub-directory: `.` in the package name are replaced by the operating system
    directory separator. Each package name component is capitalized.
-   File: The message name is appended by `.php`.

So, for example, let's say you invoke the compiler as follows:

```shell
protoc --proto_path=src --php_out=build/gen src/example.proto
```

And `src/example.proto` is defined as:

```proto
package foo.bar;
message MyMessage {}
```

The compiler will read the files `src/foo.proto` and produce the output file:
`build/gen/Foo/Bar/MyMessage.php`. The compiler will automatically create the
directory `build/gen/Foo/Bar` if necessary, but it will *not* create `build` or
`build/gen`; they must already exist.

## Packages {#package}

The package name defined in the `.proto` file is used by default to generate a
module structure for the generated PHP classes. Given a file like:

```proto
package foo.bar;

message MyMessage {}
```

The protocol compiler generates an output class with the name
`Foo\Bar\MyMessage`.

### Namespace Options {#namespace-options}

The compiler supports additional options to define the PHP and metadata
namespace. When defined, these are used to generate the module structure and the
namespace. Given options like:

```proto
package foo.bar;
option php_namespace = "baz\\qux";
option php_metadata_namespace = "Foo";
message MyMessage {}
```

The protocol compiler generates an output class with the name
`baz\qux\MyMessage`. The class will have the namespace `namespace baz\qux`.

The protocol compiler generates a metadata class with the name `Foo\Metadata`.
The class will have the namespace `namespace Foo`.

*The options generated are case-sensitive. By default, the package is converted
to Pascal case.*

## Messages {#message}

Given a simple message declaration:

```proto
message Foo {}
```

The protocol buffer compiler generates a PHP class called `Foo`. This class
inherits from a common base class, `Google\Protobuf\Internal\Message`, which
provides methods for encoding and decoding your message types, as shown in the
following example:

```php
$from = new Foo();
$from->setInt32(1);
$from->setString('a');
$from->getRepeatedInt32()[] = 1;
$from->getMapInt32Int32()[1] = 1;
$data = $from->serializeToString();
try {
  $to->mergeFromString($data);
} catch (Exception $e) {
  // Handle parsing error from invalid data.
  ...
}
```

You should *not* create your own `Foo` subclasses. Generated classes are not
designed for subclassing and may lead to \"fragile base class\" problems.

Nested messages result in a PHP class of the same name prefixed by their
containing message and separated by underscores, as PHP doesn't support nested
classes. So, for example, if you have this in your `.proto`:

```proto
message TestMessage {
  optional int32 a = 1;
  message NestedMessage {...}
}
```

The compiler will generate the following classes:

```php
class TestMessage {
  public a;
}

// PHP doesn’t support nested classes.
class TestMessage_NestedMessage {...}
```

If the message class name is reserved (for example, `Empty`), the prefix `PB` is
prepended to the class name:

```php
class PBEmpty {...}
```

We have also provided the file level option `php_class_prefix`. If this is
specified, it is prepended to all generated message classes.

## Fields

For each field in a message type, there are accessor methods to set and get the
field. So given a field `x` you can write:

```php
$m = new MyMessage();
$m->setX(1);
$val = $m->getX();

$a = 1;
$m->setX($a);
```

Whenever you set a field, the value is type-checked against the declared type of
that field. If the value is of the wrong type (or out of range), an exception
will be raised. By default type conversions (for example, when assigning a value
to a field or adding an element to a repeated field) are permitted to and from
integer, float, and numeric strings. Conversions that are not permitted include
all conversions to/from arrays or objects. Float to integer overflow conversions
are undefined.

You can see the corresponding PHP type for each scalar protocol buffers type in
the
[scalar value types table](/programming-guides/proto3#scalar).

### Singular Message Fields {#embedded_message}

A field with a message type defaults to nil, and is not automatically created
when the field is accessed. Thus you need to explicitly create sub messages, as
in the following:

```php
$m = new MyMessage();
$m->setZ(new SubMessage());
$m->getZ()->setFoo(42);

$m2 = new MyMessage();
$m2->getZ()->setFoo(42);  // FAILS with an exception
```

You can assign any instance to a message field, even if the instance is also
held elsewhere (for example, as a field value on another message).

### Repeated Fields

The protocol buffer compiler generates a special `RepeatedField` for each
repeated field. So, for example, given the following field:

```proto
repeated int32 foo = 1;
```

The generated code lets you do this:

```php
$m->getFoo()[] =1;
$m->setFoo($array);
```

### Map Fields

The protocol buffer compiler generates a `MapField` for each map field. So given
this field:

```proto
map<int32, int32> weight = 1;
```

You can do the following with the generated code:

```php
$m->getWeight()[1] = 1;
```

## Enumerations {#enum}

PHP doesn't have native enums, so instead the protocol buffer compiler generates
a PHP class for each enum type in your `.proto` file, just like for
[messages](#message), with constants defined for each value. So, given this
enum:

```proto
enum TestEnum {
  Default = 0;
  A = 1;
}
```

The compiler generates the following class:

```php
class TestEnum {
  const DEFAULT = 0;
  const A = 1;
}
```

Also as with messages, a nested enum will result in a PHP class of the same name
prefixed by its containing message(s) and separated with underscores, as PHP
does not support nested classes.

```php
class TestMessage_NestedEnum {...}
```

If an enum class or value name is reserved (for example, `Empty`), the prefix
`PB` is prepended to the class or value name:

```php
class PBEmpty {
  const PBECHO = 0;
}
```

We have also provided the file level option `php_class_prefix`. If this is
specified, it is prepended to all generated enum classes.

## Oneof

For a [oneof](/programming-guides/proto3#oneof)s, the
protocol buffer compiler generates the same code as it would for regular
singular fields, but also adds a special accessor method that lets you find out
which oneof field (if any) is set. So, given this message:

```proto
message TestMessage {
  oneof test_oneof {
    int32 oneof_int32 = 1;
    int64 oneof_int64 = 2;
  }
}
```

The compiler generates the following fields and special method:

```php
class TestMessage {
  private oneof_int32;
  private oneof_int64;
  public function getOneofInt32();
  public function setOneofInt32($var);
  public function getOneofInt64();
  public function setOneofInt64($var);
  public function getTestOneof();  // Return field name
}
```

The accessor method's name is based on the oneof's name, and returns an enum
value representing the field in the oneof that is currently set.
