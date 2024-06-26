+++
title = "Protocol Buffers Version 3 Language Specification"
weight = 810
linkTitle = "Version 3 Language Specification"
description = "Language specification reference for version 3 of the Protocol Buffers language (proto3)."
type = "docs"
+++

The syntax is specified using
[Extended Backus-Naur Form (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form):

```
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

For more information about using proto3, see the
[language guide](/programming-guides/proto3).

## Lexical Elements {#lexical_elements}

### Letters and Digits {#letters_and_digits}

```
letter = "A" ... "Z" | "a" ... "z"
decimalDigit = "0" ... "9"
octalDigit   = "0" ... "7"
hexDigit     = "0" ... "9" | "A" ... "F" | "a" ... "f"
```

### Identifiers

```
ident = letter { letter | decimalDigit | "_" }
fullIdent = ident { "." ident }
messageName = ident
enumName = ident
fieldName = ident
oneofName = ident
mapName = ident
serviceName = ident
rpcName = ident
messageType = [ "." ] { ident "." } messageName
enumType = [ "." ] { ident "." } enumName
```

### Integer Literals {#integer_literals}

```
intLit     = decimalLit | octalLit | hexLit
decimalLit = [-] ( "1" ... "9" ) { decimalDigit }
octalLit   = [-] "0" { octalDigit }
hexLit     = [-] "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### Floating-point Literals

```
floatLit = [-] ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ] ) | "inf" | "nan"
decimals  = [-] decimalDigit { decimalDigit }
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### Boolean

```
boolLit = "true" | "false"
```

### String Literals {#string_literals}

```
strLit = strLitSingle { strLitSingle }
strLitSingle = ( "'" { charValue } "'" ) |  ( '"' { charValue } '"' )
charValue = hexEscape | octEscape | charEscape | unicodeEscape | unicodeLongEscape | /[^\0\n\\]/
hexEscape = '\' ( "x" | "X" ) hexDigit [ hexDigit ]
octEscape = '\' octalDigit [ octalDigit [ octalDigit ] ]
charEscape = '\' ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | '\' | "'" | '"' )
unicodeEscape = '\' "u" hexDigit hexDigit hexDigit hexDigit
unicodeLongEscape = '\' "U" ( "000" hexDigit hexDigit hexDigit hexDigit hexDigit |
                              "0010" hexDigit hexDigit hexDigit hexDigit
```

### EmptyStatement

```
emptyStatement = ";"
```

### Constant

```
constant = fullIdent | ( [ "-" | "+" ] intLit ) | ( [ "-" | "+" ] floatLit ) |
                strLit | boolLit | MessageValue
```

`MessageValue` is defined in the
[Text Format Language Specification](/reference/protobuf/textformat-spec#fields).

## Syntax

The syntax statement is used to define the protobuf version.

```
syntax = "syntax" "=" ("'" "proto3" "'" | '"' "proto3" '"') ";"
```

Example:

```proto
syntax = "proto3";
```

## Import Statement {#import_statement}

The import statement is used to import another .proto's definitions.

```
import = "import" [ "weak" | "public" ] strLit ";"
```

Example:

```proto
import public "other.proto";
```

## Package

The package specifier can be used to prevent name clashes between protocol
message types.

```
package = "package" fullIdent ";"
```

Example:

```proto
package foo.bar;
```

## Option

Options can be used in proto files, messages, enums and services. An option can
be a protobuf defined option or a custom option. For more information, see
[Options](/programming-guides/proto3#options) in the
language guide.

```
option = "option" optionName  "=" constant ";"
optionName = ( ident | bracedFullIdent ) { "." ( ident | bracedFullIdent ) }
bracedFullIdent = "(" ["."] fullIdent ")"
optionNamePart = { ident | "(" ["."] fullIdent ")" }
```

Example:

```proto
option java_package = "com.example.foo";
```

## Fields

Fields are the basic elements of a protocol buffer message. Fields can be normal
fields, oneof fields, or map fields. A field has a type and field number.

```
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### Normal Field {#normal_field}

Each field has type, name and field number. It may have field options.

```
field = [ "repeated" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

Examples:

```proto
foo.Bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### Oneof and Oneof Field {#oneof_and_oneof_field}

A oneof consists of oneof fields and a oneof name.

```
oneof = "oneof" oneofName "{" { option | oneofField } "}"
oneofField = type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
```

Example:

```proto
oneof foo {
    string name = 4;
    SubMessage sub_message = 9;
}
```

### Map Field {#map_field}

A map field has a key type, value type, name, and field number. The key type can
be any integral or string type.

```
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

Example:

```proto
map<string, Project> projects = 3;
```

## Reserved

Reserved statements declare a range of field numbers or field names that cannot
be used in this message.

```
reserved = "reserved" ( ranges | strFieldNames ) ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

Examples:

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## Top Level Definitions {#top_level_definitions}

### Enum Definition {#enum_definition}

The enum definition consists of a name and an enum body. The enum body can have
options, enum fields, and reserved statements.

```
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement | reserved } "}"
enumField = ident "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

Example:

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 2 [(custom_option) = "hello world"];
}
```

### Message Definition {#message_definition}

A message consists of a message name and a message body. The message body can
have fields, nested enum definitions, nested message definitions, options,
oneofs, map fields, and reserved statements. A message cannot contain two fields
with the same name in the same message schema.

```
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | option | oneof | mapField |
reserved | emptyStatement } "}"
```

Example:

```proto
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    int64 ival = 1;
  }
  map<int32, string> my_map = 2;
}
```

None of the entities declared inside a message may have conflicting names. All
of the following are prohibited:

```
message MyMessage {
  optional string foo = 1;
  message foo {}
}

message MyMessage {
  optional string foo = 1;
  oneof foo {
    string bar = 2;
  }
}

message MyMessage {
  optional string foo = 1;
  enum E {
    foo = 0;
  }
}
```

### Service Definition {#service_definition}

```
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" {option | emptyStatement } "}" ) | ";")
```

Example:

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## Proto File {#proto_file}

```
proto = [syntax] { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | service
```

An example .proto file:

```proto
syntax = "proto3";
import public "other.proto";
option java_package = "com.example.foo";
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2 [(custom_option) = "hello world"];
}
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    int64 ival = 1;
  }
  repeated Inner inner_message = 2;
  EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
}
```
