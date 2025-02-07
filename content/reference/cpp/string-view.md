+++
title = "String View APIs"
weight = 510
linkTitle = "Generated Code Guide"
description = "Describes exactly what C++ code the protocol buffer compiler generates for any given protocol definition. "
type = "docs"
+++

C++ string field APIs that use `std::string` significantly constrain the
internal protobuf implementation and its evolution. For example,
`mutable_string_field()` returns `std::string*` that forces us to use
`std::string` to store the field. This complicates its interaction on arenas and
we have to maintain arena donation states to track whether string payload
allocation is from arena or heap.

Long-term, we would like to migrate all of our runtime and generated APIs to
accept `string_view` as inputs and return them from accessors. This document
describes the state of the migration as of our 30.x release.

## String Field Accessors {#string-type}

As part of edition 2023, the
[`string_type`](/editions/features#string_type) feature
was released with a `VIEW` option to allow for the incremental migration to
generated `string_view` APIs. Using this feature will affect the
[C++ Generated Code](/reference/cpp/cpp-generated) of
`string` and `bytes` fields.

### Interaction with ctype {#ctype}

In edition 2023, you can still specify `ctype` at the field level, while you can
specify `string_type` at either the file or field level. It is not allowed to
specify both on the same field. If `string_type` is set at the file-level,
`ctype` specified on fields will take precedent.

Except for the `VIEW` option, all possible values of `string_type` have a
corresponding `ctype` value that is spelled the same and gives the same
behavior. For example, both enums have a `CORD` value.

In edition 2024 and beyond, it will no longer be possible to specify `ctype`.

### Generated Singular Fields {#singular-view}

For either of these field definitions in edition 2023:

```proto
bytes foo = 1 [features.(pb.cpp).string_type=VIEW];
string foo = 1 [features.(pb.cpp).string_type=VIEW];
```

The compiler will generate the following accessor methods:

-   `::absl::string_view foo() const`: Returns the current value of the field.
    If the field is not set, returns the default value.
-   `void clear_foo()`: Clears the value of the field. After calling this,
    `foo()` will return the default value.
-   `bool has_foo()`: Returns `true` if the field is set.
-   `void set_foo(::absl::string_view value)`: Sets the value of the field.
    After calling this, `has_foo()` will return `true` and `foo()` will return a
    copy of `value`.
-   `void set_foo(const string& value)`: Sets the value of the field. After
    calling this, `has_foo()` will return `true` and `foo()` will return a copy
    of `value`.
-   `void set_foo(string&& value)`: Sets the value of the field, moving from the
    passed string. After calling this, `has_foo()` will return `true` and
    `foo()` will return `value`.
-   `void set_foo(const char* value)`: Sets the value of the field using a
    C-style null-terminated string. After calling this, `has_foo()` will return
    `true` and `foo()` will return a copy of `value`.

### Generated Repeated Fields {#repeated-view}

For either of these field definitions:

```proto
repeated string foo = 1 [features.(pb.cpp).string_type=VIEW];
repeated bytes foo = 1 [features.(pb.cpp).string_type=VIEW];
```

The compiler will generate the following accessor methods:

-   `int foo_size() const`: Returns the number of elements currently in the
    field.
-   `::absl::string_view foo(int index) const`: Returns the element at the given
    zero-based index. Calling this method with index outside of `[0,
    foo_size()-1]` yields undefined behavior.
-   `void set_foo(int index, ::absl::string_view value)`: Sets the value of the
    element at the given zero-based index.
-   `void set_foo(int index, const string& value)`: Sets the value of the
    element at the given zero-based index.
-   `void set_foo(int index, string&& value)`: Sets the value of the element at
    the given zero-based index, moving from the passed string.
-   `void set_foo(int index, const char* value)`: Sets the value of the element
    at the given zero-based index using a C-style null-terminated string.
-   `void add_foo(::absl::string_view value)`: Appends a new element to the end
    of the field with the given value.
-   `void add_foo(const string& value)`: Appends a new element to the end of the
    field with the given value.
-   `void add_foo(string&& value)`: Appends a new element to the end of the
    field, moving from the passed string.
-   `void add_foo(const char* value)`: Appends a new element to the end of the
    field using a C-style null-terminated string.
-   `void clear_foo()`: Removes all elements from the field. After calling this,
    `foo_size()` will return zero.
-   `const RepeatedPtrField<string>& foo() const`: Returns the underlying
    [`RepeatedPtrField`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField)
    that stores the field's elements. This container class provides STL-like
    iterators and other methods.
-   `RepeatedPtrField<string>* mutable_foo()`: Returns a pointer to the
    underlying mutable `RepeatedPtrField` that stores the field's elements. This
    container class provides STL-like iterators and other methods.

### Generated Oneof Fields {#oneof-view}

For any of these [oneof](#oneof) field definitions:

```proto
oneof example_name {
    string foo = 1 [features.(pb.cpp).string_type=VIEW];
    ...
}
oneof example_name {
    bytes foo = 1 [features.(pb.cpp).string_type=VIEW];
    ...
}
```

The compiler will generate the following accessor methods:

-   `bool has_foo() const`: Returns `true` if the oneof case is `kFoo`.
-   `::absl::string_view foo() const`: Returns the current value of the field if
    the oneof case is `kFoo`. Otherwise, returns the default value.
-   `void set_foo(::absl::string_view value)`:
    -   If any other oneof field in the same oneof is set, calls
        `clear_example_name()`.
    -   Sets the value of this field and sets the oneof case to `kFoo`.
    -   `has_foo()` will return `true`, `foo()` will return a copy of `value`
        and `example_name_case()` will return `kFoo`.
-   `void set_foo(const string& value)`: Like the first `set_foo()`, but copies
    from a `const string` reference.
-   `void set_foo(string&& value)`: Like the first `set_foo()`, but moving from
    the passed string.
-   `void set_foo(const char* value)`: Like the first `set_foo()`, but copies
    from a C-style null-terminated string.
-   `void clear_foo()`:
    -   If the oneof case is not `kFoo`, nothing will be changed.
    -   If the oneof case is `kFoo`, frees the field and clears the oneof case.
        `has_foo()` will return `false`, `foo()` will return the default value,
        and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.

## Enumeration Name Helper {#enum-name}

Beginning in edition 2024, a new feature `enum_name_uses_string_view` is
introduced and defaults to true. Unless disabled, for an enum like:

```proto
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

The protocol buffer compiler, in addition to the `Foo` enum, will generate the
following new function in addition to the standard
[generated code](/reference/cpp/cpp-generated#enum):

-   `::absl::string_view Foo_Name(int value)`: Returns the name for given
    numeric value. Returns an empty string if no such value exists. If multiple
    values have this number, the first one defined is returned. In the above
    example, `Foo_Name(5)` would return `VALUE_B`.

This can be reverted back to the old behavior by adding a feature override like:

```proto
enum Foo {
  option features.(pb.cpp).enum_name_uses_string_view = false;

  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

In which case the name helper will switch back to `const string& Foo_Name(int
value)`.
