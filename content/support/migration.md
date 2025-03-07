+++
title = "Migration Guide"
weight = 920
description = "A list of the breaking changes made to versions of the libraries, and how to update your code to accommodate the changes."
aliases = "/programming-guides/migration/"
type = "docs"
+++

## Changes in v30.0 {#v30}

The following is a list of the breaking changes made to versions of the
libraries, and how to update your code to accommodate the changes.

This covers breaking changes announced in
[News Announcements for v30.x](/news/v30) and
[Release Notes for v30.0](https://github.com/protocolbuffers/protobuf/releases/tag/v30.0).

### Replaced CMake Submodules with Fetched Deps

Previously, our default CMake behavior was to use Git submodules to grab pinned
dependencies. Specifying `-Dprotobuf_ABSL_PROVIDER=package` would flip our CMake
configs to look for local installations of Abseil (with similar options for
jsoncpp and gtest). These options no longer exist, and the default behavior is
to first look for installations of all our dependencies, falling back to
fetching pinned versions from GitHub if needed.

To prevent any fallback fetching (similar to the old `package` behavior), you
can call CMake with:

```cmake
cmake . -Dprotobuf_LOCAL_DEPENDENCIES_ONLY=ON
```

To *always* fetch dependencies from a fixed version (similar to the old default
behavior), you can call CMake with:

```cmake
cmake . -Dprotobuf_FORCE_FETCH_DEPENDENCIES=ON
```

### string_view return type

Return types are now `absl::string_view` for the following descriptor APIs,
which opens up memory savings:

*   `MessageLite::GetTypeName`
*   `UnknownField::length_delimited`
*   Descriptor API name functions, such as `FieldDescriptor::full_name`

We expect future breaking releases to continue migrating additional APIs to
`absl::string_view`.

In most cases, you should try to update types to use `absl::string_view` where
safe, or explicitly copy to the original type where needed. You may need to
update callers as well if this is returned in a function.

If the string returned by the affected API methods is being used as:

<table>
  <tr>
    <th>Type</th>
    <th>Migration</th>
  </tr>
  <tr>
    <td>
      <p><code>std::string</code></p>
    </td>
    <td>
      <p>Explicitly convert to <code>std::string</code> to preserve the existing behavior.</p>
      <p>Or, switch to the more performant <code>absl::string_view</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p><code>const std::string&</code></p>
    </td>
    <td>
      <p>Migrate to <code>absl::string_view</code></p>
      <p>If infeasible (such as due to large number of dependencies), copying to a <code>std::string</code> might be easier.</p>
    </td>
  </tr>
  <tr>
    <td>
      <p><code>const std::string*</code></p>
      <p><code>const char*</code></p>
    </td>
    <td>
      <p>If nullable, migrate to <code>std::optional&lt;absl::string_view&gt;</code>.</p>
      <p>Otherwise, migrate to <code>absl::string_view</code>.</p>
      <p>Be careful when calling <code>data()</code> since <code>absl::string_view</code> isn't guaranteed to be null-terminated.
    </td>
  </tr>
</table>

For common containers and other APIs, you may be able to migrate to variants
compatible with `absl::string_view`. Below are some common examples.

<table>
  <tr>
    <th>Category</th>
    <th>Pre-Migration</th>
    <th>Migration</th>
  </tr>
  <tr>
    <td>Insertion into <code>std::vector&lt;std::string&gt;</code></td>
    <td>
      <p><code>push_back()</code></p>
      <p><code>push_front()</code></p>
      <p><code>push()</code></p>
    </td>
    <td>
      <p><code>emplace_back()</code></p>
      <p><code>emplace_front()</code></p>
      <p><code>emplace()</code></p>
    </td>
  </tr>
  <tr>
    <td>Insertion for map or sets</td>
    <td>
      <p><code>set.insert(key)</code></p>
      <p><code>map.insert({key, value})</code></p>
      <p><code>map.insert({key,<br/>  {value_params...}})</code></p>
    </td>
    <td>
      <p><code>set.emplace(key)</code></p>
      <p><code>map.emplace(key, value)</code></p>
      <p><code>map.try_emplace(key,<br/>  value_params...)</code></p>
    </td>
  </tr>
  <tr>
    <td>Lookup for map or sets</td>
    <td>
      <p><code>find()</code></p>
      <p><code>count()</code></p>
      <p><code>contains()</code></p>
    </td>
    <td>
      <p>Migrate to Abseil containers.</p>
      <p>Or, define a transparent comparator.</p>
      <p><code>std::set&lt;std::string, std::less&lt;&gt;&gt;<br/>
      std::map&lt;std::string, T, std::less&lt;&gt;&gt;</code></p>
      <p>See <a href="https://abseil.io/tips/144">https://abseil.io/tips/144</a></p>
    </td>
  </tr>
  <tr>
    <td>String Concatenation</td>
    <td>
      <p><code>operator+</code></p>
      <p><code>operator+=</code></p>
    </td>
    <td>
      <p><code>absl::StrCat()</code></p>
      <p><code>absl::StrAppend()</code></p>
      <p>These are recommended for performance reasons anyways. See <a href="https://abseil.io/tips/3">https://abseil.io/tips/3</a>.</p>
   </td>
  </tr>
</table>

See also [https://abseil.io/tips/1](https://abseil.io/tips/1) for general tips
around using `absl::string_view`.

### Poison MSVC + Bazel

Bazel users on Windows should switch to using clang-cl by adding the following
to their project, like in this
[example](https://github.com/protocolbuffers/protobuf/commit/117e7bbe74ac7c7faa9b6f44c1b22de366302854#diff-48bcd3965c4a015a8f61ad82b225790209baef37363ee0478519536a620a85e5).

**.bazelrc**

```bazel
common --enable_platform_specific_config build:windows
--extra_toolchains=@local_config_cc//:cc-toolchain-x64_windows-clang-cl
--extra_execution_platforms=//:x64_windows-clang-cl
```

**MODULE.bazel**

```bazel
bazel_dep(name = "platforms", version = "0.0.10")
bazel_dep(name = "rules_cc", version = "0.0.17")

# For clang-cl configuration
cc_configure = use_extension("@rules_cc//cc:extensions.bzl", "cc_configure_extension")
use_repo(cc_configure, "local_config_cc")
```

**WORKSPACE**

```bazel
load("//:protobuf_deps.bzl", "PROTOBUF_MAVEN_ARTIFACTS", "protobuf_deps")

protobuf_deps()

load("@rules_cc//cc:repositories.bzl", "rules_cc_dependencies", "rules_cc_toolchains")

rules_cc_dependencies()

rules_cc_toolchains()
```

**BUILD**

For users that need compatibility with Bazel 8 only.

```bazel
platform(
    name = "x64_windows-clang-cl",
    constraint_values = [
        "@platforms//cpu:x86_64",
        "@platforms//os:windows",
        "@bazel_tools//tools/cpp:clang-cl",
    ],
)
```

For users that need compatibility with Bazel 7 and 8.

```bazel
platform(
    name = "x64_windows-clang-cl",
    constraint_values = [
        "@platforms//cpu:x86_64",
        "@platforms//os:windows",
        # See https://github.com/bazelbuild/rules_cc/issues/330.
        "@rules_cc//cc/private/toolchain:clang-cl",
    ],
)
```

Users can also temporarily silence the error by setting the opt-out flag
`--define=protobuf_allow_msvc=true` until the next breaking release.

Alternatively, users that wish to continue using MSVC may switch to using CMake.
This can be done with
[Visual Studio](https://learn.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=msvc-170),
or by supplying the CMake command-line an MSVC generator. For example:

```cmake
cmake -G "Visual Studio 17 2022" -A Win64 .
```

### ctype Removed from FieldDescriptor Options {#ctype-removed}

We stopped exposing the `ctype` from `FieldDescriptor` options. You can use the
`FieldDescriptor::cpp_string_type()` API, added in the
[v28 release](https://github.com/protocolbuffers/protobuf/releases/tag/v28.0),
in its place.

### Modified Debug APIs to Redact Sensitive Fields {#debug-redaction}

The Protobuf C++ debug APIs (including Protobuf AbslStringify,
`proto2::ShortFormat`, `proto2::Utf8Format`, `Message::DebugString`,
`Message::ShortDebugString`, `Message::Utf8DebugString`) changed to redact
sensitive fields annotated by `debug_redact`; the outputs of these APIs contain
a per-process randomized prefix, and are no longer parseable by Protobuf
TextFormat Parsers. Users should adopt the new redacted debug format for most
cases requiring a human-readable output (such as logging), or consider switching
to binary format for serialization and deserialization. Users who need the old
deserializable format can use `TextFormat.printer().printToString(proto)`, but
this does not redact sensitive fields and so should be used with caution.

Read more about this in the
[news article released December 4, 2024](/news/2024-12-04.md).

### Removed Deprecated APIs {#remove-deprecated}

We removed the following public runtime APIs, which have been marked deprecated
(such as `ABSL_DEPRECATED`) for at least one minor or major release and that are
obsolete or replaced.

**API:**
[`Arena::CreateMessage`](https://github.com/protocolbuffers/protobuf/blob/f4b57b98b08aec8bc52e6a7b762edb7a1b9e8883/src/google/protobuf/arena.h#L179)

**Replacement:**
[`Arena::Create`](https://github.com/protocolbuffers/protobuf/blob/f4b57b98b08aec8bc52e6a7b762edb7a1b9e8883/src/google/protobuf/arena.h#L191)

**API:**
[`Arena::GetArena`](https://github.com/protocolbuffers/protobuf/blob/237332ef92daf83a53e76decd6ac43c3fcee782b/src/google/protobuf/arena.h#L346)

**Replacement:** `value->GetArena()`

**API:**
[`RepeatedPtrField::ClearedCount`](https://github.com/protocolbuffers/protobuf/blame/f4b57b98b08aec8bc52e6a7b762edb7a1b9e8883/src/google/protobuf/repeated_ptr_field.h#L1157)

**Replacement:** Migrate to Arenas
([migration guide](https://protobuf.dev/support/migration/#cleared-elements)).

**API:**
[`JsonOptions`](https://github.com/protocolbuffers/protobuf/blob/f4b57b98b08aec8bc52e6a7b762edb7a1b9e8883/src/google/protobuf/util/json_util.h#L22)

**Replacement:** `JsonPrintOptions`

### Dropped C++14 Support {#drop-cpp-14}

This release dropped C++ 14 as the minimum supported version and raised it to
17, as per the
[Foundational C++ Support matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md).

Users should upgrade to C++17.

### Introduced ASAN Poisoning After Clearing Oneof Messages on Arena

This change added a hardening check that affects C++ protobufs using Arenas.
Oneof messages allocated on the protobuf arena are now cleared in debug and
poisoned in ASAN mode. After calling clear, future attempts to use the memory
region will cause a crash in ASAN as a use-after-free error.

This implementation requires C++17.

### Dropped our C++ CocoaPods release

We dropped our C++ CocoaPods release, which has been broken since v4.x.x. C++
users should use our
[GitHub release](https://github.com/protocolbuffers/protobuf/releases) directly
instead.

### Changes in Python {#python}

Python bumped its major version from 5.29.x to 6.30.x.

#### Dropped Python 3.8 Support

The minimum supported Python version is 3.9. Users should upgrade.

#### Removed bazel/system_python.bzl Alias {#python-remove-alias}

We removed the legacy `bazel/system_python.bzl` alias.

Remove direct references to `system_python.bzl` in favor of using
`protobuf_deps.bzl` instead. Use `python/dist/system_python.bzl` where it was
moved
[in v5.27.0](https://github.com/protocolbuffers/protobuf/commit/d7f032ad1596ceeabd45ca1354516c39b97b2551)
if you need a direct reference.

#### Field Setter Validation Changes {#python-setter-validation}

Python's and upb's field setters now validate closed enums under edition 2023.
Closed enum fields updated with invalid values generate errors.

#### Removed Deprecated py_proto_library Macro

The deprecated internal `py_proto_library` Bazel macro in `protobuf.bzl` was
removed. It was replaced by the official `py_proto_library` which was moved to
protobuf in `bazel/py_proto_library` in v29.x. This implementation was
previously available in `rules_python` prior to v29.x.

#### Remove Deprecated APIs {#python-remove-apis}

We removed the following public runtime APIs, which had been marked deprecated
for at least one minor or major release.

##### Reflection Methods

**APIs:**
[`reflection.ParseMessage`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/reflection.py#L40),
[`reflection.MakeClass`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/reflection.py#L66)

**Replacement:** `message_factory.GetMessageClass()`

##### RPC Service Interfaces

**APIs:**
[`service.RpcException`](https://github.com/protocolbuffers/protobuf/blob/21c545c8c5cec0b052dc7715b778f0353d37d02c/python/google/protobuf/service.py#L23),
[`service.Service`](https://github.com/protocolbuffers/protobuf/blob/21c545c8c5cec0b052dc7715b778f0353d37d02c/python/google/protobuf/service.py#L28),
[`service.RpcController`](https://github.com/protocolbuffers/protobuf/blob/21c545c8c5cec0b052dc7715b778f0353d37d02c/python/google/protobuf/service.py#L97),
and
[`service.RpcChannel`](https://github.com/protocolbuffers/protobuf/blob/21c545c8c5cec0b052dc7715b778f0353d37d02c/python/google/protobuf/service.py#L180)

**Replacement:** Starting with version 2.3.0, RPC implementations should not try
to build on these, but should instead provide code generator plugins which
generate code specific to the particular RPC implementation.

##### MessageFactory and SymbolDatabase Methods

**APIs:**
[`MessageFactory.GetPrototype`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/message_factory.py#L145),
[`MessageFactory.CreatePrototype`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/message_factory.py#L165),
[`MessageFactory.GetMessages`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/message_factory.py#L185),
[`SymbolDatabase.GetPrototype`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/symbol_database.py#L54),
[`SymbolDatabase.CreatePrototype`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/symbol_database.py#L60),
and
[`SymbolDatabase.GetMessages`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/symbol_database.py#L66)

**Replacement:** `message_factory.GetMessageClass()` and
`message_factory.GetMessageClassesForFiles()`.

##### GetDebugString

**APIs:**
[`GetDebugString`](https://github.com/protocolbuffers/protobuf/blob/7f395af40e86e00b892e812beb67a03564884756/python/google/protobuf/pyext/descriptor.cc#L1510)

**Replacement:**

No replacement. It's only in Python C++ which is no longer released. It is not
supported in pure Python or UPB.

#### Python setdefault Behavior Change for Map Fields

`setdefault` is similar to `dict` for `ScalarMap`, except that both key and
value must be set. `setdefault` is rejected for `MessageMaps`.

#### Python Nested Message Class \_\_qualname\_\_ Contains the Outer Message Name

Python nested message class `__qualname__` now contains the outer message name.
Previously, `__qualname__` had the same result with `__name__` for nested
message, in that the outer message name was not included.

For example:

```python
message Foo {
  message Bar {
    bool bool_field = 1;
  }
}
nested = test_pb2.Foo.Bar()
self.assertEqual('Bar', nested.__class__.__name__)
self.assertEqual('Foo.Bar', nested.__class__.__qualname__) # It was 'Bar' before
```

### Changes in Objective-C {#objc}

**This is the first breaking release for Objective-C**.

Objective-C bumped its major version from 3.x.x to 4.30.x.

#### Overhauled Unknown Field Handling APIs Deprecating Most of the Existing APIs {#objc-field-handling}

We deprecated `GPBUnknownFieldSet` and replaced it with `GPBUnknownFields`. The
new type preserves the ordering of unknown fields from the original input or API
calls, to ensure any semantic meaning to the ordering is maintained when a
message is written back out.

As part of this, the `GPBUnknownField` type also has APIs changes, with almost
all of the existing APIs deprecated and new ones added.

Deprecated property APIs:

*   `varintList`
*   `fixed32List`
*   `fixed64List`
*   `lengthDelimitedList`
*   `groupList`

Deprecated modification APIs:

*   `addVarint:`
*   `addFixed32:`
*   `addFixed64:`
*   `addLengthDelimited:`
*   `addGroup:`

Deprecated initializer `initWithNumber:`.

New property APIs:

*   `type`
*   `varint`
*   `fixed32`
*   `fixed64`
*   `lengthDelimited`
*   `group`

This type models a single field number in its value, rather than *grouping* all
the values for a given field number. The APIs for creating new fields are the
`add*` APIs on the `GPBUnknownFields` class.

We also deprecated `-[GPBMessage unknownFields]`. In its place, there are new
APIs to extract and update the unknown fields of the message.

#### Removed Deprecated APIs {#objc-remove-apis}

We removed the following public runtime APIs, which had been marked deprecated
for at least one minor or major release.

##### GPBFileDescriptor

**API:**
[-[`GPBFileDescriptor` syntax]](https://github.com/protocolbuffers/protobuf/blob/44bd65b2d3c0470d91a23cc14df5ffb1ab0af7cd/objectivec/GPBDescriptor.h#L118-L119)

**Replacement:** Obsolete.

##### GPBMessage mergeFrom:extensionRegistry

**API:**
[-[`GPBMessage mergeFrom:extensionRegistry:`]](https://github.com/protocolbuffers/protobuf/blob/44bd65b2d3c0470d91a23cc14df5ffb1ab0af7cd/objectivec/GPBMessage.h#L258-L261)

**Replacement:**
[-[`GPBMessage mergeFrom:extensionRegistry:error:`]](https://github.com/protocolbuffers/protobuf/blob/44bd65b2d3c0470d91a23cc14df5ffb1ab0af7cd/objectivec/GPBMessage.h#L275-L277)

##### GPBDuration timeIntervalSince1970

**API:**
[-[`GPBDuration timeIntervalSince1970`]](https://github.com/protocolbuffers/protobuf/blob/29fca8a64b62491fb0a2ce61878e70eda88dde98/objectivec/GPBWellKnownTypes.h#L95-L100)

**Replacement:**
[-[`GPBDuration timeInterval`]](https://github.com/protocolbuffers/protobuf/blob/29fca8a64b62491fb0a2ce61878e70eda88dde98/objectivec/GPBWellKnownTypes.h#L74-L89)

##### GPBTextFormatForUnknownFieldSet

**API:**
[`GPBTextFormatForUnknownFieldSet()`](https://github.com/protocolbuffers/protobuf/blob/29fca8a64b62491fb0a2ce61878e70eda88dde98/objectivec/GPBUtilities.h#L32-L43)

**Replacement:** Obsolete - Use
[`GPBTextFormatForMessage()`](https://github.com/protocolbuffers/protobuf/blob/29fca8a64b62491fb0a2ce61878e70eda88dde98/objectivec/GPBUtilities.h#L20-L30),
which includes any unknown fields.

##### GPBUnknownFieldSet

**API:**
[`GPBUnknownFieldSet`](https://github.com/protocolbuffers/protobuf/blob/224573d66a0cc958c76cb43d8b2eb3aa7cdb89f2/objectivec/GPBUnknownFieldSet.h)

**Replacement:**
[`GPBUnknownFields`](https://github.com/protocolbuffers/protobuf/blob/224573d66a0cc958c76cb43d8b2eb3aa7cdb89f2/objectivec/GPBUnknownFields.h)

##### GPBMessage unknownFields

**API:**
[`GPBMessage unknownFields` property](https://github.com/protocolbuffers/protobuf/blob/224573d66a0cc958c76cb43d8b2eb3aa7cdb89f2/objectivec/GPBMessage.h#L73-L76)

**Replacement:**
[-[`GPBUnknownFields initFromMessage:`]](https://github.com/protocolbuffers/protobuf/blob/224573d66a0cc958c76cb43d8b2eb3aa7cdb89f2/objectivec/GPBUnknownFields.h#L30-L38),
[-[`GPBMessage mergeUnknownFields:extensionRegistry:error:`]](https://github.com/protocolbuffers/protobuf/blob/f26bdff7cc0bb7e8ed88253ba16f81614a26cf16/objectivec/GPBMessage.h#L506-L528),
and
[-[`GPBMessage clearUnknownFields`]](https://github.com/protocolbuffers/protobuf/blob/224573d66a0cc958c76cb43d8b2eb3aa7cdb89f2/objectivec/GPBMessage.h#L497-L504C9)

#### Removed Deprecated Runtime APIs for Old Gencode {#objc-remove-apis-gencode}

This release removed deprecated runtime methods that supported the Objective-C
gencode from before the 3.22.x release. The library also issues a log message at
runtime when old generated code is starting up.

**API:** [`+[GPBFileDescriptor
allocDescriptorForClass:file:fields:fieldCount:storageSize:flags:]`](https://github.com/protocolbuffers/protobuf/blob/1b44ce0feef45a78ba99d09bd2e5ff5052a6541b/objectivec/GPBDescriptor_PackagePrivate.h#L213-L220)

**Replacement:** Regenerate with a current version of the library.

**API:** [`+[GPBFileDescriptor
allocDescriptorForClass:rootClass:file:fields:fieldCount:storageSize:flags:]`](https://github.com/protocolbuffers/protobuf/blob/1b44ce0feef45a78ba99d09bd2e5ff5052a6541b/objectivec/GPBDescriptor_PackagePrivate.h#L221-L229)

**Replacement:** Regenerate with a current version of the library.

**API:** [`+[GPBEnumDescriptor
allocDescriptorForName:valueNames:values:count:enumVerifier:]`](https://github.com/protocolbuffers/protobuf/blob/1b44ce0feef45a78ba99d09bd2e5ff5052a6541b/objectivec/GPBDescriptor_PackagePrivate.h#L285-L291)

**Replacement:** Regenerate with a current version of the library.

**API:** [`+[GPBEnumDescriptor
allocDescriptorForName:valueNames:values:count:enumVerifier:extraTextFormatInfo:]`](https://github.com/protocolbuffers/protobuf/blob/1b44ce0feef45a78ba99d09bd2e5ff5052a6541b/objectivec/GPBDescriptor_PackagePrivate.h#L292-L299)

**Replacement:** Regenerate with a current version of the library.

**API:**
[`-[GPBExtensionDescriptor initWithExtensionDescription:]`](https://github.com/protocolbuffers/protobuf/blob/1b44ce0feef45a78ba99d09bd2e5ff5052a6541b/objectivec/GPBDescriptor_PackagePrivate.h#L317-L320)

**Replacement:** Regenerate with a current version of the library.

### Poison Pill Warnings {#poison}

We updated poison pills to emit warnings for old gencode + new runtime
combinations that work under the new rolling upgrade policy, but will break in
the *next* major bump. For example, Python 4.x.x gencode should work against
5.x.x runtime but warn of upcoming breakage against 6.x.x runtime.

### Changes to UTF-8 Enforcement in C# and Ruby {#utf-8-enforcement}

We included a fix to make UTF-8 enforcement consistent across languages. Users
with bad non-UTF8 data in string fields may see surfaced UTF-8 enforcement
errors earlier.

### Ruby and PHP Errors in JSON Parsing

We fixed non-conformance in JSON parsing of strings in numeric fields per the
[JSON spec](https://protobuf.dev/programming-guides/json/).

This fix is not accompanied by a major version bump, but Ruby and PHP now raise
errors for non-numeric strings (such as `""`, `"12abc"`, `"abc"`) in numeric
fields. v29.x includes a warning for these error cases.

## Compiler Changes in v22.0 {#compiler-22}

### JSON Field Name Conflicts {#json-field-names}

Source of changes: [PR #11349](https://github.com/protocolbuffers/protobuf/pull/11349), [PR #10750](https://github.com/protocolbuffers/protobuf/pull/10750)

We've made some subtle changes in how we handle field name conflicts with
respect to JSON mappings. In proto3, we've partially loosened the restrictions
and only give errors when field names produce case-sensitive JSON mappings
(camel case of the original name). We now also check the `json_name` option, and
give errors for case-sensitive conflicts. In proto2, we've tightened
restrictions a bit and will give errors if two `json_name` specifications
conflict. If implicit JSON mappings (camel case) have conflicts, we will give
warnings in proto2.

We've provided a temporary message/enum option for restoring the legacy
behavior. If renaming the conflicting fields isn't an option you can take
immediately, set the `deprecated_legacy_json_field_conflicts` option on the
specific message/enum. This option will be removed in a future release, but
gives you more time to migrate.

## C++ API Changes in v22.0 {#cpp-22}

4.22.0 has breaking changes for C++ runtime and protoc, as
[announced in August](/news/2022-08-03#cpp-changes).

### Autotools Turndown {#autotools}

Source of changes:
[PR #10132](https://github.com/protocolbuffers/protobuf/pull/10132)

In v22.0, we removed all Autotools support from the protobuf compiler and the
C++ runtime. If you're using Autotools to build either of these, you must
migrate to [CMake](http://cmake.org) or [Bazel](http://bazel.build). We have
some
[dedicated instructions](https://github.com/protocolbuffers/protobuf/blob/main/cmake/README.md)
for setting up protobuf with CMake.

### Abseil Dependency {#abseil}

Source of changes:
[PR #10416](https://github.com/protocolbuffers/protobuf/pull/10416)

With v22.0, we've taken on an explicit dependency on
[Abseil](https://github.com/abseil/abseil-cpp). This allowed us to remove most
of our
[stubs](https://github.com/protocolbuffers/protobuf/tree/21.x/src/google/protobuf/stubs),
which were branched from old internal code that later became Abseil. There are a
number of subtle behavior changes, but most should be transparent to users. Some
notable changes include:

*   **string_view** - `absl::string_view` has replaced `const std::string&` in
    many of our APIs. This is most-commonly used for input arguments, where
    there should be no noticeable change for users. In a few cases (such as
    virtual method arguments or return types) users may need to make an explicit
    change to use the new signature.

*   **tables** - Instead of STL sets/maps, we now use Abseil's `flat_hash_map`,
    `flat_hash_set`, `btree_map`, and `btree_set`. These are more efficient and
    allow for [heterogeneous lookup](https://abseil.io/tips/144). This should be
    mostly invisible to users, but may cause some subtle behavior changes
    related to table ordering.

*   **logging** - Abseil's
    [logging library](https://abseil.io/docs/cpp/guides/logging) is very similar
    to our old logging code, with just a slightly different spelling (for
    example, `ABSL_CHECK` instead of `GOOGLE_CHECK`). The biggest difference is
    that it doesn't support exceptions, and will now always crash when `FATAL`
    assertions fail. (Previously we had a `PROTOBUF_USE_EXCEPTIONS` flag to
    switch to exceptions.) Since these only occur when serious issues are
    encountered, we feel unconditional crashing is a suitable response.

    Source of logging changes: [PR #11623](https://github.com/protocolbuffers/protobuf/pull/11623)

*   **Build dependency** - A new build dependency can always cause breakages for
    downstream users. We require
    [Abseil LTS 20230125](https://github.com/abseil/abseil-cpp/releases/tag/20230125.0)
    or later to build.

    *   For Bazel builds, Abseil will be automatically downloaded and built at a
        pinned LTS release when
        [`protobuf_deps`](https://github.com/protocolbuffers/protobuf/blob/main/protobuf_deps.bzl)
        is run from your `WORKSPACE`. This should be transparent, but if you
        depend on an older version of Abseil, you'll need to upgrade your
        dependency.

    *   For CMake builds, we will first look for an existing Abseil installation
        pulled in by the top-level CMake configuration (see
        [instructions](https://github.com/abseil/abseil-cpp/blob/master/CMake/README#traditional-cmake-set-up)).
        Otherwise, if `protobuf_ABSL_PROVIDER` is set to `module` (its default)
        we will attempt to build and link Abseil from our git
        [submodule](https://github.com/protocolbuffers/protobuf/tree/main/third_party).
        If `protobuf_ABSL_PROVIDER` is set to `package`, we will look for a
        pre-installed system version of Abseil.

### Changes in GetCurrentTime Method {#getcurrenttime}

On Windows, `GetCurrentTime()` is the name of a macro provided by the system.
Prior to v22.x, Protobuf incorrectly removed the macro definition for
`GetCurrentTime()`. That made the macro unusable for Windows developers after
including `<protobuf/util/time_util.h>`. Starting with v22.x, Protobuf preserves
the macro definition. This may break customer code relying on the previous
behavior, such as if they use the expression
[`google::protobuf::util::TimeUtil::GetCurrentTime()`](/reference/cpp/api-docs/google.protobuf.util.time_util#TimeUtil).

To migrate your app to the new behavior, change your code to do one of the
following:

*   if the `GetCurrent` macro is defined, explicitly undefine the
    `GetCurrentTime` macro
*   prevent the macro expansion by using
    `(google::protobuf::util::TimeUtil::GetCurrentTime)()` or a similar
    expression

#### Example: Undefining the macro

Use this approach if you don't use the macro from Windows.

Before:

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

After:

```cpp
#include <google/protobuf/util/time_util.h>
#ifdef GetCurrentTime
#undef GetCurrentTime
#endif

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

**Example 2: Preventing macro expansion**

Before:

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

After:

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = (google::protobuf::util::TimeUtil::GetCurrentTime)();
}
```

## C++20 Support {#cpp20}

Source of changes: [PR #10796](https://github.com/protocolbuffers/protobuf/pull/10796)

To support C++20, we've reserved the new
[keywords](https://en.cppreference.com/w/cpp/keyword) in C++ generated protobuf
code. As with other reserved keywords, if you use them for any fields, enums, or
messages, we will add an underscore suffix to make them valid C++. For example,
a `concept` field will generate a `concept_()` getter. In the scenario where you
have existing protos that use these keywords, you'll need to update the C++ code
that references them to add the appropriate underscores.

### Final Classes {#final-classes}

Source of changes: [PR #11604](https://github.com/protocolbuffers/protobuf/pull/11604)

As part of a larger effort to harden assumptions made in the Protobuf library,
we've marked some classes `final` that were never intended to be inherited from.
There are no known use cases for inheriting from these, and doing so would
likely cause problems. There is no mitigation if your code is inheriting from
these classes, but if you think you have some valid reason for the inheritance
you're using, you can
[open an issue](https://github.com/protocolbuffers/protobuf/issues).

### Container Static Assertions {#container-static-assertions}

Source of changes: [PR #11550](https://github.com/protocolbuffers/protobuf/pull/11550)

As part of a larger effort to harden assumptions made in the Protobuf library,
we've added static assertions to the `Map`, `RepeatedField`, and
`RepeatedPtrField` containers. These ensure that you're using these containers
with only expected types, as
[covered in our documentation](https://protobuf.dev/programming-guides/proto/#maps).
If you hit these static assertions, you should migrate your code to use Abseil
or STL containers. `std::vector` is a good drop-in replacement for repeated
field containers, and `std::unordered_map` or `absl::flat_hash_map` for `Map`
(the former gives similar pointer stability, while the latter is more
efficient).

### Cleared Element Deprecation {#cleared-elements}

Source of changes: [PR #11588](https://github.com/protocolbuffers/protobuf/pull/11588), [PR #11639](https://github.com/protocolbuffers/protobuf/pull/11639)

The `RepeatedPtrField` API around "cleared fields" has been deprecated, and will
be fully removed in a later breaking release. This was originally added as an
optimization for reusing elements after they've been cleared, but ended up not
working well. If you're using this API, you should consider migrating to arenas
for better memory reuse.

### UnsafeArena Deprecation {#unsafe-arena}

Source of changes: [PR #10325](https://github.com/protocolbuffers/protobuf/pull/10325)

As part of a larger effort to remove arena-unsafe APIs, we've hidden
`RepeatedField::UnsafeArenaSwap`. This is the only one we've removed so far, but
in later releases we will continue to remove them and provide helpers to handle
efficient borrowing patterns between arenas. Within a single arena (or the
stack/heap), `Swap` is just as efficient as `UnsafeArenaSwap`. The benefit is
that it won't cause invalid memory operations if you accidentally call it across
different arenas.

### Map Pair Upgrades {#map-pairs}

Source of changes: [PR #11625](https://github.com/protocolbuffers/protobuf/pull/11625)

For v22.0 we've started cleaning up the `Map` API to make it more consistent
with Abseil and STL. Notably, we've replaced the `MapPair` class with an alias
to `std::pair`. This should be transparent for most users, but if you were using
the class directly you may need to update your code.

### New JSON Parser {:#json-parser}

Source of changes: [PR #10729](https://github.com/protocolbuffers/protobuf/pull/10729)

We have rewritten the C++ JSON parser this release. It should be mostly a hidden
change, but inevitably some undocumented quirks my have changed; test
accordingly. Parsing documents that are not valid RFC-8219 JSON (such as those
that are missing quotes or using non-standard bools) is deprecated and will be
removed in a future release. The serialization order of fields is now guaranteed
to match the field number order, where before it was less deterministic.

As part of this migration, all of the files under
[util/internal](https://github.com/protocolbuffers/protobuf/tree/21.x/src/google/protobuf/util/internal)
have been deleted. These were used in the old parser, and were never intended to
be used externally.

### `Arena::Init` {#arena-init}

Source of changes: [PR #10623](https://github.com/protocolbuffers/protobuf/pull/10623)

The `Init` method in `Arena` was code that didn't do anything, and has now been
removed. If you were calling this method, you likely meant to call the `Arena`
constructor directly with a set of `ArenaOptions`. You should either delete the
call or migrate to that constructor.

### ErrorCollector Migration {#error-collector}

Source of changes: [PR #11555](https://github.com/protocolbuffers/protobuf/pull/11555)

As part of our Abseil migration, we're moving from `const std::string&` to
`absl::string_view`. For our three error collector classes, this can't be done
without breaking existing code. For v22.0, we've decided to release both
variants, and rename the methods from `AddError` and `AddWarning` to
`RecordError` and `RecordWarning`. The old signature has been marked deprecated,
and will be slightly less efficient (due to string copies), but will otherwise
still work. You should migrate these to the new version, as the `Add*` methods
will be removed in a later breaking release.
