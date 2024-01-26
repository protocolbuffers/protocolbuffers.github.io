+++
title = "Protocol Buffer Basics: Dart"
weight = 230
linkTitle = "Dart"
description = "A basic Dart programmers introduction to working with protocol buffers."
type = "docs"
+++

This tutorial provides a basic Dart programmer's introduction to working with
protocol buffers, using the
[proto3](/programming-guides/proto3) version of the
protocol buffers language. By walking through creating a simple example
application, it shows you how to

-   Define message formats in a `.proto` file.
-   Use the protocol buffer compiler.
-   Use the Dart protocol buffer API to write and read messages.

This isn't a comprehensive guide to using protocol buffers in Dart . For more
detailed reference information, see the
[Protocol Buffer Language Guide](/programming-guides/proto3),
the
[Dart Language Tour,](https://www.dartlang.org/guides/language/language-tour)
the [Dart API Reference](https://pub.dartlang.org/documentation/protobuf), the
[Dart Generated Code Guide](/reference/dart/dart-generated),
and the
[Encoding Reference](/programming-guides/encoding).

## The Problem Domain {#problem-domain}

The example we're going to use is a very simple "address book" application that
can read and write people's contact details to and from a file. Each person in
the address book has a name, an ID, an email address, and a contact phone
number.

How do you serialize and retrieve structured data like this? There are a few
ways to solve this problem:

-   You can invent an ad-hoc way to encode the data items into a single
    string -- such as encoding 4 ints as "12:3:-23:67". This is a simple and
    flexible approach, although it does require writing one-off encoding and
    parsing code, and the parsing imposes a small run-time cost. This works best
    for encoding very simple data.
-   Serialize the data to XML. This approach can be very attractive since XML is
    (sort of) human readable and there are binding libraries for lots of
    languages. This can be a good choice if you want to share data with other
    applications/projects. However, XML is notoriously space intensive, and
    encoding/decoding it can impose a huge performance penalty on applications.
    Also, navigating an XML DOM tree is considerably more complicated than
    navigating simple fields in a class normally would be.

Protocol buffers are the flexible, efficient, automated solution to solve
exactly this problem. With protocol buffers, you write a `.proto` description of
the data structure you wish to store. From that, the protocol buffer compiler
creates a class that implements automatic encoding and parsing of the protocol
buffer data with an efficient binary format. The generated class provides
getters and setters for the fields that make up a protocol buffer and takes care
of the details of reading and writing the protocol buffer as a unit.
Importantly, the protocol buffer format supports the idea of extending the
format over time in such a way that the code can still read data encoded with
the old format.

## Where to Find the Example Code {#example-code}

Our example is a set of command-line applications for managing an address book
data file, encoded using protocol buffers. The command `dart add_person.dart`
adds a new entry to the data file. The command `dart list_people.dart` parses
the data file and prints the data to the console.

You can find the complete example in the
[examples directory](https://github.com/protocolbuffers/protobuf/tree/master/examples)
of the GitHub repository.

## Defining Your Protocol Format {#protocol-format}

To create your address book application, you'll need to start with a `.proto`
file. The definitions in a `.proto` file are simple: you add a *message* for
each data structure you want to serialize, then specify a name and a type for
each field in the message. In our example, the `.proto` file that defines the
messages is
[`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto).

The `.proto` file starts with a package declaration, which helps to prevent
naming conflicts between different projects.

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

Next, you have your message definitions. A message is just an aggregate
containing a set of typed fields. Many standard simple data types are available
as field types, including `bool`, `int32`, `float`, `double`, and `string`. You
can also add further structure to your messages by using other message types as
field types.

```proto
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```

In the above example, the `Person` message contains `PhoneNumber` messages,
while the `AddressBook` message contains `Person` messages. You can even define
message types nested inside other messages -- as you can see, the `PhoneNumber`
type is defined inside `Person`. You can also define `enum` types if you want
one of your fields to have one of a predefined list of values -- here you want
to specify that a phone number can be one of `PHONE_TYPE_MOBILE`,
`PHONE_TYPE_HOME`, or `PHONE_TYPE_WORK`.

The " = 1", " = 2" markers on each element identify the unique "tag" that field
uses in the binary encoding. Tag numbers 1-15 require one less byte to encode
than higher numbers, so as an optimization you can decide to use those tags for
the commonly used or repeated elements, leaving tags 16 and higher for
less-commonly used optional elements. Each element in a repeated field requires
re-encoding the tag number, so repeated fields are particularly good candidates
for this optimization.

If a field value isn't set, a
[default value](/programming-guides/proto3#default) is
used: zero for numeric types, the empty string for strings, false for bools. For
embedded messages, the default value is always the "default instance" or
"prototype" of the message, which has none of its fields set. Calling the
accessor to get the value of a field which has not been explicitly set always
returns that field's default value.

If a field is `repeated`, the field may be repeated any number of times
(including zero). The order of the repeated values will be preserved in the
protocol buffer. Think of repeated fields as dynamically sized arrays.

You'll find a complete guide to writing `.proto` files -- including all the
possible field types -- in the
[Protocol Buffer Language Guide](/programming-guides/proto3).
Don't go looking for facilities similar to class inheritance, though -- protocol
buffers don't do that.

## Compiling Your Protocol Buffers {#compiling-protocol-buffers}

Now that you have a `.proto`, the next thing you need to do is generate the
classes you'll need to read and write `AddressBook` (and hence `Person` and
`PhoneNumber`) messages. To do this, you need to run the protocol buffer
compiler `protoc` on your `.proto`:

1.  If you haven't installed the compiler,
    [download the package](/downloads) and follow the
    instructions in the README.

2.  Install the Dart Protocol Buffer plugin as described in
    [its README](https://github.com/google/protobuf.dart/tree/master/protoc_plugin#how-to-build).
    The executable `bin/protoc-gen-dart` must be in your `PATH` for the protocol
    buffer `protoc` to find it.

3.  Now run the compiler, specifying the source directory (where your
    application's source code lives -- the current directory is used if you
    don't provide a value), the destination directory (where you want the
    generated code to go; often the same as `$SRC_DIR`), and the path to your
    `.proto`. In this case, you would invoke:

    ```shell
    protoc -I=$SRC_DIR --dart_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Because you want Dart code, you use the `--dart_out` option -- similar
    options are provided for other supported languages.

This generates `addressbook.pb.dart` in your specified destination directory.

## The Protocol Buffer API {#protobuf-api}

Generating `addressbook.pb.dart` gives you the following useful types:

-   An `AddressBook` class with a `List<Person> get people` getter.
-   A `Person` class with accessor methods for `name`, `id`, `email` and
    `phones`.
-   A `Person_PhoneNumber` class, with accessor methods for `number` and `type`.
-   A `Person_PhoneType` class with static fields for each value in the
    `Person.PhoneType` enum.

You can read more about the details of exactly what's generated in the
[Dart Generated Code guide](/reference/dart/dart-generated).

## Writing a Message {#writing-a-message}

Now let's try using your protocol buffer classes. The first thing you want your
address book application to be able to do is write personal details to your
address book file. To do this, you need to create and populate instances of your
protocol buffer classes and then write them to an output stream.

Here is a program which reads an `AddressBook` from a file, adds one new
`Person` to it based on user input, and writes the new `AddressBook` back out to
the file again. The parts which directly call or reference code generated by the
protocol compiler are highlighted.

```dart
import 'dart:io';

import 'dart_tutorial/addressbook.pb.dart';

// This function fills in a Person message based on user input.
Person promptForAddress() {
  Person person = Person();

  print('Enter person ID: ');
  String input = stdin.readLineSync();
  person.id = int.parse(input);

  print('Enter name');
  person.name = stdin.readLineSync();

  print('Enter email address (blank for none) : ');
  String email = stdin.readLineSync();
  if (email.isNotEmpty) {
    person.email = email;
  }

  while (true) {
    print('Enter a phone number (or leave blank to finish): ');
    String number = stdin.readLineSync();
    if (number.isEmpty) break;

    Person_PhoneNumber phoneNumber = Person_PhoneNumber();

    phoneNumber.number = number;
    print('Is this a mobile, home, or work phone? ');

    String type = stdin.readLineSync();
    switch (type) {
      case 'mobile':
        phoneNumber.type = Person_PhoneType.PHONE_TYPE_MOBILE;
        break;
      case 'home':
        phoneNumber.type = Person_PhoneType.PHONE_TYPE_HOME;
        break;
      case 'work':
        phoneNumber.type = Person_PhoneType.PHONE_TYPE_WORK;
        break;
      default:
        print('Unknown phone type.  Using default.');
    }
    person.phones.add(phoneNumber);
  }

  return person;
}

// Reads the entire address book from a file, adds one person based
// on user input, then writes it back out to the same file.
main(List arguments) {
  if (arguments.length != 1) {
    print('Usage: add_person ADDRESS_BOOK_FILE');
    exit(-1);
  }

  File file = File(arguments.first);
  AddressBook addressBook;
  if (!file.existsSync()) {
    print('File not found. Creating new file.');
    addressBook = AddressBook();
  } else {
    addressBook = AddressBook.fromBuffer(file.readAsBytesSync());
  }
  addressBook.people.add(promptForAddress());
  file.writeAsBytes(addressBook.writeToBuffer());
}
```

## Reading a Message {#reading-a-message}

Of course, an address book wouldn't be much use if you couldn't get any
information out of it! This example reads the file created by the above example
and prints all the information in it.

```dart
import 'dart:io';

import 'dart_tutorial/addressbook.pb.dart';
import 'dart_tutorial/addressbook.pbenum.dart';

// Iterates though all people in the AddressBook and prints info about them.
void printAddressBook(AddressBook addressBook) {
  for (Person person in addressBook.people) {
    print('Person ID: ${ person.id}');
    print('  Name: ${ person.name}');
    if (person.hasEmail()) {
      print('  E-mail address:${ person.email}');
    }

    for (Person_PhoneNumber phoneNumber in person.phones) {
      switch (phoneNumber.type) {
        case Person_PhoneType.PHONE_TYPE_MOBILE:
          print('   Mobile phone #: ');
          break;
        case Person_PhoneType.PHONE_TYPE_HOME:
          print('   Home phone #: ');
          break;
        case Person_PhoneType.PHONE_TYPE_WORK:
          print('   Work phone #: ');
          break;
        default:
          print('   Unknown phone #: ');
          break;
      }
      print(phoneNumber.number);
    }
  }
}

// Reads the entire address book from a file and prints all
// the information inside.
main(List arguments) {
  if (arguments.length != 1) {
    print('Usage: list_person ADDRESS_BOOK_FILE');
    exit(-1);
  }

  // Read the existing address book.
  File file = new File(arguments.first);
 AddressBook addressBook = new AddressBook.fromBuffer(file.readAsBytesSync());
  printAddressBook(addressBook);
}
```

## Extending a Protocol Buffer {#extending-a-protobuf}

Sooner or later after you release the code that uses your protocol buffer, you
will undoubtedly want to "improve" the protocol buffer's definition. If you want
your new buffers to be backwards-compatible, and your old buffers to be
forward-compatible -- and you almost certainly do want this -- then there are
some rules you need to follow. In the new version of the protocol buffer:

-   you *must not* change the tag numbers of any existing fields.
-   you *may* delete fields.
-   you *may* add new fields but you must use fresh tag numbers (i.e. tag
    numbers that were never used in this protocol buffer, not even by deleted
    fields).

(There are
[some exceptions](/programming-guides/proto3#updating) to
these rules, but they are rarely used.)

If you follow these rules, old code will happily read new messages and simply
ignore any new fields. To the old code, singular fields that were deleted will
simply have their default value, and deleted repeated fields will be empty. New
code will also transparently read old messages.

However, keep in mind that new fields will not be present in old messages, so
you will need to do something reasonable with the default value. A type-specific
[default value](/programming-guides/proto3#default) is
used: for strings, the default value is the empty string. For booleans, the
default value is false. For numeric types, the default value is zero.
