# Recommended Way to Downcast `proto2::Message`

go/protobuf-downcast-recommendation

<!--
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'debadribasak' reviewed: '2023-10-10' }
*-->

TL;DR: While downcasting `proto2::Message` to a generated type, use
[`proto2::DynamicCastToGenerated`](https://source.corp.google.com/piper///depot/google3/third_party/protobuf/message.h;rcl=571241355;l=1530)
where `dynamic_cast` is required and use
[`proto2::DownCastToGenerated`](https://source.corp.google.com/piper///depot/google3/third_party/protobuf/message.h;rcl=571241355;l=1577)
for all other types of cast (`static_cast`, C-style cast and
[`down_cast`](https://source.corp.google.com/piper///depot/google3/base/casts.h;rcl=570699316;l=59))

## Problem {#problem}

If there are multiple `message` definitions inside a `.proto` file and only few
of them are referenced, code is still generated and will contribute towards the
final binary size. Code generated for unused Protocol Buffer messages is
stripped in an effort to reduce the size of the final binary.

Implementing
[weak-reflection-based messages](http://go/protobuf-weak-speed-messages)
modifies code generation to define the `default_instance` of every generated
type as a weak symbol, and defines each of them in a separate section. This
makes the linker drop the unused sections from the final binary, resulting in a
reduction in total size.

There can be false positives, such as when one `default_instance` gets dropped
even if it is getting used inside code. This happens in the cases where a
generated type is getting created using reflection but is used outside of
reflection. This can lead to undefined behavior when a base `proto2::Message`
that holds an object of a generated type is downcast to the corresponding
generated type. This will make the type appear in the code, without its
`default_instance` being present. This situation will arise when some built-in
cast operators and functions are used, such as `static_cast`, `dynamic_cast`,
C-style cast, and
[`down_cast`](https://source.corp.google.com/piper///depot/google3/base/casts.h;rcl=570699316;l=59).

### Example of Undefined Behavior {#example-undefined}

Consider this proto definition:

```proto
message MessageA {
  optional int32 value = 1;
}

message MessageB {
  optional int32 value1 = 1;
  optional int32 value2 = 2 [default = 5];
}
```

Which is used in this C++ code:

```cpp {.bad}
// Bad example
proto2_unittest::protobuf_stripping::MessageA messageA;
  messageA.set_value(12345);
  const proto2::DescriptorPool* generated_pool =
      proto2::DescriptorPool::generated_pool();
  if (generated_pool == nullptr) return 1;
  proto2::Message* message =
      proto2::MessageFactory::generated_factory()
          ->GetPrototype(
              proto2::DescriptorPool::generated_pool()->FindMessageTypeByName(
                  "proto2_unittest.protobuf_stripping.MessageB"))
          ->New();
  const proto2::Reflection* reflection = message->GetReflection();
  const proto2::FieldDescriptor* field =
      message->GetDescriptor()->FindFieldByName("value1");
  reflection->SetInt32(message, field, 123);
  const auto* messageB =
      static_cast<proto2_unittest::protobuf_stripping::MessageB*>(message);
  std::cout << messageB->value2();
```

In this example, `MessageA` is getting directly used, so its `default_instance`
would not get dropped. But `MessageB` is getting generated using its descriptor
and the type is never used inside the code. There is no way for the linker to
know about the usage and its `default_instance` will get dropped. The code sets
`value1` using reflection, but `value2` is never set. `value2` has a default
value of 5 which would have been the case if `default_instance` was pinned. But
in this example the output statement where we print the `value2` field will have
some default value (like 0).

## Solution {#solution}

To prevent undefined behavior related to downcasting, use two functions:
[`proto2::DynamicCastToGenerated`](https://source.corp.google.com/piper///depot/google3/third_party/protobuf/message.h;rcl=571241355;l=1530)
and
[`proto2::DownCastToGenerated`](https://source.corp.google.com/piper///depot/google3/third_party/protobuf/message.h;rcl=571241355;l=1577).

These functions perform type-pinning, which ensures that the `default_instance`
of the destination type is not dropped from the final binary.

### Usage Instructions {#usage}

#### `proto2::DynamicCastToGenerated` {#DynamicCastToGenerated}

Use `proto2::DynamicCastToGenerated` only when it is necessary to check whether
the destination type is a derived type, such as a child class of
`proto2::Message`. As it uses `dynamic_cast` in its implementation, it can have
performance impact where there is no necessity of type-checking. Therefore, it
is recommended to use it in the places where `dynamic_cast` should be used.

#### `proto2::DownCastToGenerated` {#DownCastToGenerated}

Use `proto2::DownCastToGenerated` when type-checking is not necessary.
Internally, this function performs `static_cast` with additional type-pinning.
Therefore, it is recommended to use in places where the caller is certain that
the destination type is a derived type. It should be used where `static_cast`,
C-style cast or `down_cast` is being used currently for downcasting
`proto2::Message` to a derived type.

### Examples {#examples}

#### Case 1: Type-checking is not Necessary {#case-1}

```c++ {.bad}
// Bad example
Example* DowncastToExample(proto2::Message* message) {
  return static_cast<Example*>(message);
}
```

```c++ {.good}
// Good example
Example* DowncastToExample(proto2::Message* message) {
  return proto2::DownCastToGenerated<Example>(message);
}
```

#### Case 2: Type-checking is Required {#case-2}

Here the template parameter `T` can be any type. The function will compile only
if it is a type derived from `proto2::Message`

```c++ {.bad}
// Bad example
template<typename T>
T* DowncastToExample(proto2::Message* message) {
  return dynamic_cast<T*>(message);
}
```

```c++ {.good}
// Good example
template<typename T>
T* DowncastToExample(proto2::Message* message) {
  return proto2::DynamicCastToGenerated<T>(message);
}
```
