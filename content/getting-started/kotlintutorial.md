+++
title = "Protocol Buffer Basics: Kotlin"
weight = 260
linkTitle = "Kotlin"
description = "A basic Kotlin programmers introduction to working with protocol buffers."
type = "docs"
+++

This tutorial provides a basic Kotlin programmer's introduction to working with
protocol buffers, using the
[proto3](/programming-guides/proto3) version of the
protocol buffers language. By walking through creating a simple example
application, it shows you how to

-   Define message formats in a `.proto` file.
-   Use the protocol buffer compiler.
-   Use the Kotlin protocol buffer API to write and read messages.

This isn't a comprehensive guide to using protocol buffers in Kotlin. For more
detailed reference information, see the
[Protocol Buffer Language Guide](/programming-guides/proto3),
the [Kotlin API Reference](/reference/kotlin/api-docs),
the
[Kotlin Generated Code Guide](/reference/kotlin/kotlin-generated),
and the
[Encoding Reference](/programming-guides/encoding).

## The Problem Domain {#problem-domain}

The example we're going to use is a very simple "address book" application that
can read and write people's contact details to and from a file. Each person in
the address book has a name, an ID, an email address, and a contact phone
number.

How do you serialize and retrieve structured data like this? There are a few
ways to solve this problem:

-   Use kotlinx.serialization. This does not work very well if you need to share
    data with applications written in C++ or Python. kotlinx.serialization has a
    [protobuf mode](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/formats.md#protobuf-experimental),
    but this does not offer the full features of protocol buffers.
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
data file, encoded using protocol buffers. The command `add_person_kotlin` adds
a new entry to the data file. The command `list_people_kotlin` parses the data
file and prints the data to the console.

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

2.  Now run the compiler, specifying the source directory (where your
    application's source code lives -- the current directory is used if you
    don't provide a value), the destination directory (where you want the
    generated code to go; often the same as `$SRC_DIR`), and the path to your
    `.proto`. In this case, you would invoke:

    ```shell
    protoc -I=$SRC_DIR --java_out=$DST_DIR --kotlin_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Because you want Kotlin code, you use the `--kotlin_out` option -- similar
    options are provided for other supported languages.

Note that if you want to generate Kotlin code you must use both `--java_out` and
`--kotlin_out`. This generates a `com/example/tutorial/protos/` subdirectory in
your specified Java destination directory, containing a few generated `.java`
files and a `com/example/tutorial/protos/` subdirectory in your specified Kotlin
destination directory, containing a few generated `.kt` files.

## The Protocol Buffer API {#protobuf-api}

The protocol buffer compiler for Kotlin generates Kotlin APIs that add to the
existing APIs generated for protocol buffers for Java. This ensures that
codebases written in a mix of Java and Kotlin can interact with the same
protocol buffer message objects without any special handling or conversion.

Protocol buffers for other Kotlin compilation targets, such as JavaScript and
native, are not currently supported.

Compiling `addressbook.proto` gives you the following APIs in Java:

-   The `AddressBook` class
    -   which, from Kotlin, has the `peopleList : List<Person>` property
-   The `Person` class
    -   which, from Kotlin, has `name`, `id`, `email`, and `phonesList`
        properties
    -   the `Person.PhoneNumber` nested class with `number` and `type`
        properties
    -   the `Person.PhoneType` nested enum

but also generates the following Kotlin APIs:

-   The `addressBook { ... }` and `person { ... }` factory methods
-   A `PersonKt` object, with a `phoneNumber { ... }` factory method

You can read more about the details of exactly what's generated in the
[Kotlin Generated Code guide](/reference/kotlin/kotlin-generated).

## Writing a Message {#writing-a-message}

Now let's try using your protocol buffer classes. The first thing you want your
address book application to be able to do is write personal details to your
address book file. To do this, you need to create and populate instances of your
protocol buffer classes and then write them to an output stream.

Here is a program which reads an `AddressBook` from a file, adds one new
`Person` to it based on user input, and writes the new `AddressBook` back out to
the file again. The parts which directly call or reference code generated by the
protocol compiler are highlighted.

```kotlin
import com.example.tutorial.Person
import com.example.tutorial.AddressBook
import com.example.tutorial.person
import com.example.tutorial.addressBook
import com.example.tutorial.PersonKt.phoneNumber
import java.util.Scanner

// This function fills in a Person message based on user input.
fun promptPerson(): Person = person {
  print("Enter person ID: ")
  id = readLine().toInt()

  print("Enter name: ")
  name = readLine()

  print("Enter email address (blank for none): ")
  val email = readLine()
  if (email.isNotEmpty()) {
    this.email = email
  }

  while (true) {
    print("Enter a phone number (or leave blank to finish): ")
    val number = readLine()
    if (number.isEmpty()) break

    print("Is this a mobile, home, or work phone? ")
    val type = when (readLine()) {
      "mobile" -> Person.PhoneType.PHONE_TYPE_MOBILE
      "home" -> Person.PhoneType.PHONE_TYPE_HOME
      "work" -> Person.PhoneType.PHONE_TYPE_WORK
      else -> {
        println("Unknown phone type.  Using home.")
        Person.PhoneType.PHONE_TYPE_HOME
      }
    }
    phones += phoneNumber {
      this.number = number
      this.type = type
    }
  }
}

// Reads the entire address book from a file, adds one person based
// on user input, then writes it back out to the same file.
fun main(args: List) {
  if (arguments.size != 1) {
    println("Usage: add_person ADDRESS_BOOK_FILE")
    exitProcess(-1)
  }
  val path = Path(arguments.single())
  val initialAddressBook = if (!path.exists()) {
    println("File not found. Creating new file.")
    addressBook {}
  } else {
    path.inputStream().use {
      AddressBook.newBuilder().mergeFrom(it).build()
    }
  }
  path.outputStream().use {
    initialAddressBook.copy { peopleList += promptPerson() }.writeTo(it)
  }
}
```

## Reading a Message {#reading-a-message}

Of course, an address book wouldn't be much use if you couldn't get any
information out of it! This example reads the file created by the above example
and prints all the information in it.

```kotlin
import com.example.tutorial.Person
import com.example.tutorial.AddressBook

// Iterates though all people in the AddressBook and prints info about them.
fun print(addressBook: AddressBook) {
  for (person in addressBook.peopleList) {
    println("Person ID: ${person.id}")
    println("  Name: ${person.name}")
    if (person.hasEmail()) {
      println("  Email address: ${person.email}")
    }
    for (phoneNumber in person.phonesList) {
      val modifier = when (phoneNumber.type) {
        Person.PhoneType.PHONE_TYPE_MOBILE -> "Mobile"
        Person.PhoneType.PHONE_TYPE_HOME -> "Home"
        Person.PhoneType.PHONE_TYPE_WORK -> "Work"
        else -> "Unknown"
      }
      println("  $modifier phone #: ${phoneNumber.number}")
    }
  }
}

fun main(args: List) {
  if (arguments.size != 1) {
    println("Usage: list_person ADDRESS_BOOK_FILE")
    exitProcess(-1)
  }
  Path(arguments.single()).inputStream().use {
    print(AddressBook.newBuilder().mergeFrom(it).build())
  }
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
