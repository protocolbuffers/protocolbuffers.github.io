+++
title = "Migration Guide"
weight = 920
description = "A list of the breaking changes made to versions of the libraries, and how to update your code to accommodate the changes."
aliases = "/programming-guides/migration/"
type = "docs"
+++

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
migrate to [CMake](http://cmake.org) or
[Bazel](http://bazel.build). We have some
[dedicated instructions](https://github.com/protocolbuffers/protobuf/blob/main/cmake/README.md)
for setting up protobuf with CMake.

### Abseil Dependency {#abseil}

Source of changes:
[PR #10416](https://github.com/protocolbuffers/protobuf/pull/10416)

With v22.0, we've taken on an explicit dependency on
[Abseil](https://github.com/abseil/abseil-cpp). This allowed us to
remove most of our
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
    allow for [heterogeneous lookup](https://abseil.io/tips/144).
    This should be mostly invisible to users, but may cause some subtle behavior
    changes related to table ordering.

*   **logging** - Abseil's
    [logging library](https://abseil.io/docs/cpp/guides/logging) is
    very similar to our old logging code, with just a slightly different
    spelling (for example, `ABSL_CHECK` instead of `GOOGLE_CHECK`). The biggest
    difference is that it doesn't support exceptions, and will now always crash
    when `FATAL` assertions fail. (Previously we had a `PROTOBUF_USE_EXCEPTIONS`
    flag to switch to exceptions.) Since these only occur when serious issues
    are encountered, we feel unconditional crashing is a suitable response.

    Source of logging changes: [PR #11623](https://github.com/protocolbuffers/protobuf/pull/11623)

*   **Build dependency** - A new build dependency can always cause breakages for
    downstream users. We require
    [Abseil LTS 20230117](https://github.com/abseil/abseil-cpp/releases/tag/20230117.rc1)
    or later to build.

    *   For Bazel builds, Abseil will be automatically downloaded and built at a
        pinned LTS release when
        [`protobuf_deps`](https://github.com/protocolbuffers/protobuf/blob/main/protobuf_deps.bzl)
        is run from your `WORKSPACE`. This should be transparent, but if you
        depend on an older version of Abseil, you'll need to upgrade your
        dependency.

    *   For CMake builds, we will first look for an existing Abseil installation
        pulled in by the top-level CMake configuration (see
        [instructions](https://github.com/abseil/abseil-cpp/blob/master/CMake/README.md#traditional-cmake-set-up)).
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
[`google::protobuf::util::TimeUtil::GetCurrentTime()`](/reference/cpp/api-docs/google.protobuf.util.time_util.md#TimeUtil).

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
#ifdef GetCurrent
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
[keywords](https://en.cppreference.com/w/cpp/keyword) in C++
generated protobuf code. As with other reserved keywords, if you use them for
any fields, enums, or messages, we will add an underscore suffix to make them
valid C++. For example, a `concept` field will generate a `concept_()` getter.
In the scenario where you have existing protos that use these keywords, you'll
need to update the C++ code that references them to add the appropriate
underscores.

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

For v22.0 we’ve started cleaning up the `Map` API to make it more consistent
with Abseil and STL. Notably, we’ve replaced the `MapPair` class with an alias
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
