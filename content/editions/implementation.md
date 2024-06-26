+++
title = "Implementing Editions Support"
weight = 43
description = "Instructions for implementing Editions support in runtimes and plugins."
type = "docs"
+++

This topic explains how to implement editions in new runtimes and generators.

## Overview {#overview}

### Edition 2023 {#edition-2023}

The first edition released is Edition 2023, which is designed to unify proto2
and proto3 syntax. The features we’ve added to cover the difference in behaviors
are detailed in
[Feature Settings for Editions](/editions/features).

### Feature Definition {#feature-definition}

In addition to *supporting* editions and the global features we've defined, you
may want to define your own features to leverage the infrastructure. This will
allow you to define arbitrary features that can be used by your generators and
runtimes to gate new behaviors. The first step is to claim an extension number
for the `FeatureSet` message in descriptor.proto above 9999. You can send a
pull-request to us in GitHub, and it will be included in our next release (see,
for example, [#15439](https://github.com/protocolbuffers/protobuf/pull/15439)).

Once you have your extension number, you can create your features proto (similar
to
[cpp_features.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/cpp_features.proto)).
These will typically look something like:

```proto
edition = "2023";

package foo;

import "google/protobuf/descriptor.proto";

extend google.protobuf.FeatureSet {
  MyFeatures features = <extension #>;
}

message MyFeatures {
  enum FeatureValue {
    FEATURE_VALUE_UNKNOWN = 0;
    VALUE1 = 1;
    VALUE2 = 2;
  }

  FeatureValue feature_value = 1 [
    targets = TARGET_TYPE_FIELD,
    targets = TARGET_TYPE_FILE,
    feature_support = {
      edition_introduced: EDITION_2023,
      edition_deprecated: EDITION_2024,
      deprecation_warning: "Feature will be removed in 2025",
      edition_removed: EDITION_2025,
    },
    edition_defaults = { edition: EDITION_LEGACY, value: "VALUE1" },
    edition_defaults = { edition: EDITION_2024, value: "VALUE2" }
  ];
}
```

Here we’ve defined a new enum feature `foo.feature_value` (currently only
boolean and enum types are supported). In addition to defining the values it can
take, you also need to specify how it can be used:

*   **Targets** - specifies the type of proto descriptors this feature can be
    attached to. This controls where users can explicitly specify the feature.
    Every type must be explicitly listed.
*   **Feature support** - specifies the lifetime of this feature relative to
    edition. You must specify the edition it was introduced in, and it will not
    be allowed before then. You can optionally deprecate or remove the feature
    in later editions.
*   **Edition defaults** - specifies any changes to the default value of the
    feature. This must cover every supported edition, but you can leave out any
    edition where the default didn’t change. Note that `EDITION_PROTO2` and
    `EDITION_PROTO3` can be specified here to provide defaults for the “legacy”
    editions (see [Legacy Editions](#legacy_editions)).

#### What is a Feature? {#what-feature}

Features are designed to provide a mechanism for ratcheting down bad behavior
over time, on edition boundaries. While the timeline for actually removing a
feature may be years (or decades) in the future, the desired goal of any feature
should be eventual removal. When a bad behavior is identified, you can introduce
a new feature that guards the fix. In the next edition (or possibly later), you
would flip the default value while still allowing users to retain their old
behavior when upgrading. At some point in the future you would mark the feature
deprecated, which would trigger a custom warning to any users overriding it. In
a later edition, you would then mark it removed, preventing users from
overriding it anymore (but the default value would still apply). Until support
for that last edition is dropped in a breaking release, the feature will remain
usable for protos stuck on older editions, giving them time to migrate.

Flags that control optional behaviors you have no intention of removing are
better implemented as
[custom options](/programming-guides/proto2/#customoptions).
This is related to the reason we’ve restricted features to be either boolean or
enum types. Any behavior controlled by a (relatively) unbounded number of values
probably isn’t a good fit for the editions framework, since it’s not realistic
to eventually turn down that many different behaviors.

One caveat to this is behaviors related to wire boundaries. Using
language-specific features to control serialization or parsing behavior can be
dangerous, since any other language could be on the other side. Wire-format
changes should always be controlled by global features in `descriptor.proto`,
which can be respected by every runtime uniformly.

### Generators {#generators}

Generators written in C++ get a lot for free, because they use the C++ runtime.
They don’t need to handle [Feature Resolution](#feature_resolution) themselves,
and if they need any feature extensions they can register them in
`GetFeatureExtensions` in their CodeGenerator. They can generally use
`GetResolvedSourceFeatures` for access to resolved features for a descriptor in
codegen and `GetUnresolvedSourceFeatures` for access to their own unresolved
features.

Plugins written in the same language as the runtime they generate code for may
need some custom bootstrapping for their feature definitions.

#### Explicit Support {#explicit-support}

Generators must specify exactly which editions they support. This allows you to
safely add support for an edition after it’s been released, on your own
schedule. Protoc will reject any editions protos sent to generators that don’t
include `FEATURE_SUPPORTS_EDITIONS` in the `supported_features` field of their
`CodeGeneratorResponse`. Additionally, we have `minimum_edition` and
`maximum_edition` fields for specifying your precise support window. Once you’ve
defined all of the code and feature changes for a new edition, you can bump
`maximum_edition` to advertise this support.

#### Codegen Tests {#codegen-tests}

We have a set of codegen tests that can be used to lock down that Edition 2023
produces no unexpected functional changes. These have been very useful in
languages like C++ and Java, where a significant amount of the functionality is
in gencode. On the other hand, in languages like Python, where the gencode is
basically just a collection of serialized descriptors, these are not quite as
useful.

This infrastructure is not reusable yet, but is planned to be in a future
release. At that point you will be able to use them to verify that migrating to
editions doesn’t have any unexpected codegen changes.

### Runtimes {#runtimes}

Runtimes without reflection or dynamic messages should not need to do anything
to implement Editions. All of that logic should be handled by the code
generator.

Languages *with* reflection but *without* dynamic messages need resolved
features, but may optionally choose to handle it in their generator only. This
can be done by passing both resolved and unresolved feature sets to the runtime
during codegen. This avoids re-implementing
[Feature Resolution](#feature_resolution) in the runtime with the main downside
being efficiency, since it will create a unique feature set for every
descriptor.

Languages with dynamic messages must fully implement Editions, because they need
to be able to build descriptors at runtime.

#### Syntax Reflection {#syntax_reflection}

The first step in implementing Editions in a runtime with reflection is to
remove all direct checks of the `syntax` keyword. All of these should be moved
to finer-grained feature helpers, which can continue to use `syntax` if
necessary.

The following feature helpers should be implemented on descriptors, with
language-appropriate naming:

*   `FieldDescriptor::has_presence` - Whether or not a field has explicit
    presence
    *   Repeated fields *never* have presence
    *   Message, extension, and oneof fields *always* have explicit presence
    *   Everything else has presence iff `field_presence` is not `IMPLICIT`
*   `FieldDescriptor::is_required` - Whether or not a field is required
*   `FieldDescriptor::requires_utf8_validation` - Whether or not a field should
    be checked for utf8 validity
*   `FieldDescriptor::is_packed` - Whether or not a repeated field has packed
    encoding
*   `FieldDescriptor::is_delimited` - Whether or not a message field has
    delimited encoding
*   `EnumDescriptor::is_closed` - Whether or not a field is closed

**Note:** In most languages, the message encoding feature is still currently
signaled by `TYPE_GROUP` and required fields still have `LABEL_REQUIRED` set.
This is not ideal, and was done to make downstream migrations easier.
Eventually, these should be migrated to the appropriate helpers and
`TYPE_MESSAGE/LABEL_OPTIONAL`.

Downstream users should migrate to these new helpers instead of using syntax
directly. The following class of existing descriptor APIs should ideally be
deprecated and eventually removed, since they leak syntax information:

*   `FileDescriptor` syntax
*   Proto3 optional APIs
    *   `FieldDescriptor::has_optional_keyword`
    *   `OneofDescriptor::is_synthetic`
    *   `Descriptor::*real_oneof*` - should be renamed to just “oneof” and the
        existing “oneof” helpers should be removed, since they leak information
        about synthetic oneofs (which don’t exist in editions).
*   Group type
    *   The `TYPE_GROUP` enum value should be removed, replaced with the
        `is_delimited` helper.
*   Required label
    *   The `LABEL_REQUIRED` enum value should be removed, replaced with the
        `is_required` helper.

There are many classes of user code where these checks exist but *aren’t*
hostile to editions. For example, code that needs to handle proto3 `optional`
specially because of its synthetic oneof implementation won’t be hostile to
editions as long as the polarity is something like `syntax == "proto3"` (rather
than checking `syntax != "proto2"`).

If it’s not possible to remove these APIs entirely, they should be deprecated
and discouraged.

#### Feature Visibility {#visibility}

As discussed in
[editions-feature-visibility](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/editions-feature-visibility.md),
feature protos should remain an internal detail of any Protobuf implementation.
The *behaviors* they control should be exposed via descriptor methods, but the
protos themselves should not. Notably, this means that any options that are
exposed to the users need to have their `features` fields stripped out.

The one case where we permit features to leak out is when serializing
descriptors. The resulting descriptor protos should be a faithful representation
of the original proto files, and should contain *unresolved features* inside of
the options.

#### Legacy Editions {#legacy_editions}

As discussed more in
[legacy-syntax-editions](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/legacy-syntax-editions.md),
a great way to get early coverage of your editions implementation is to unify
proto2, proto3, and editions. This effectively migrates proto2 and proto3 to
editions under the hood, and makes all of the helpers implemented in
[Syntax Reflection](#syntax_reflection) use features exclusively (instead of
branching on syntax). This can be done by inserting a *feature inference* phase
into [Feature Resolution](#feature_resolution), where various aspects of the
proto file can inform what features are appropriate. These features can then be
merged into the parent’s features to get the resolved feature set.

While we provide reasonable defaults for proto2/proto3 already, for edition 2023
the following additional inferences are required:

*   required - we infer `LEGACY_REQUIRED` presence when a field has
    `LABEL_REQUIRED`
*   groups - we infer `DELIMITED` message encoding when a field has `TYPE_GROUP`
*   packed - we infer `PACKED` encoding when the `packed` option is true
*   expanded - we infer `EXPANDED` encoding when a proto3 field has `packed`
    explicitly set to false

#### Conformance Tests {#conformance-tests}

Editions-specific conformance tests have been added, but need to be opted-in to.
A `--maximum_edition 2023` flag can be passed to the runner to enable these. You
will need to configure your testee binary to handle the following new message
types:

*   `protobuf_test_messages.editions.proto2.TestAllTypesProto2` - Identical to
    the old proto2 message, but transformed to edition 2023
*   `protobuf_test_messages.editions.proto3.TestAllTypesProto3` - Identical to
    the old proto3 message, but transformed to edition 2023
*   `protobuf_test_messages.editions.TestAllTypesEdition2023` - Used to cover
    edition-2023-specific test cases

### Feature Resolution {#feature-resolution}

Editions use lexical scoping to define features, meaning that any non-C++ code
that needs to implement editions support will need to reimplement our *feature
resolution* algorithm. However, the bulk of the work is handled by protoc
itself, which can be configured to output an intermediate `FeatureSetDefaults`
message. This message contains a “compilation” of a set of feature definition
files, laying out the default feature values in every edition.

For example, the feature definition above would compile to the following
defaults between proto2 and edition 2025 (in text-format notation):

```
defaults {
  edition: EDITION_PROTO2
  overridable_features { [foo.features] {} }
  fixed_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_PROTO3
  overridable_features { [foo.features] {} }
  fixed_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_2023
  overridable_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_2024
  overridable_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE2 }
  }
}
defaults {
  edition: EDITION_2025
  overridable_features {
    // Global feature defaults…
  }
  fixed_features { [foo.features] { feature_value: VALUE2 } }
}
minimum_edition: EDITION_PROTO2
maximum_edition: EDITION_2025

```

Global feature defaults are left out for compactness, but they would also be
present. This object contains an ordered list of every edition with a unique set
of defaults (some editions may end up not being present) within the specified
range. Each set of defaults is split into *overridable* and *fixed* features.
The former are supported features for the edition that can be freely overridden
by users. The fixed features are those which haven’t yet been introduced or have
been removed, and can’t be overridden by users.

We provide a Bazel rule for compiling these intermediate objects:

```
load("@com_google_protobuf//editions:defaults.bzl", "compile_edition_defaults")

compile_edition_defaults(
    name = "my_defaults",
    srcs = ["//some/path:lang_features_proto"],
    maximum_edition = "PROTO2",
    minimum_edition = "2024",
)
```

The output `FeatureSetDefaults` can be embedded into a raw string literal in
whatever language you need to do feature resolution in. We also provide an
`embed_edition_defaults` macro to do this:

```
embed_edition_defaults(
    name = "embed_my_defaults",
    defaults = ":my_defaults",
    output = "my_defaults.h",
    placeholder = "DEFAULTS_DATA",
    template = "my_defaults.h.template",
)
```

Alternatively, you can invoke protoc directly (outside of Bazel) to generate
this data:

```
protoc --edition_defaults_out=defaults.binpb --edition_defaults_minimum=PROTO2 --edition_defaults_maximum=2023 <feature files...>
```

Once the defaults message is hooked up and parsed by your code, feature
resolution for a file descriptor at a given edition follows a simple algorithm:

1.  Validate that the edition is in the appropriate range [`minimum_edition`,
    `maximum_edition`]
2.  Binary-search the ordered `defaults` field for the highest entry less than
    or equal to the edition
3.  Merge `overridable_features` into `fixed_features` from the selected
    defaults
4.  Merge any explicit features set on the descriptor (the `features` field in
    the file options)

From there, you can recursively resolve features for all other descriptors:

1.  Initialize to the parent descriptor’s feature set
2.  Merge any explicit features set on the descriptor (the `features` field in
    the options)

For determining the “parent” descriptor, you can reference our
[C++ implementation](https://github.com/protocolbuffers/protobuf/blob/27.x/src/google/protobuf/descriptor.cc#L1129).
This is straightforward in most cases, but extensions are a bit surprising
because their parent is the enclosing scope rather than the extendee. Oneofs
also need to be considered as the parent of their fields.

#### Conformance Tests {#conformance-tests}

In a future release, we plan to add conformance tests to verify feature
resolution cross-language. Until then, our regular
[conformance tests](#conformance_tests) do give partial coverage, and our
[example inheritance unit tests](https://github.com/protocolbuffers/protobuf/blob/27.x/python/google/protobuf/internal/descriptor_test.py#L1386)
can be ported to provide more comprehensive coverage.

### Examples {#examples}

Below are some real examples of how we implemented editions support in our
runtimes and plugins.

#### Java {#java}

*   [#14138](https://github.com/protocolbuffers/protobuf/pull/14138) - Bootstrap
    compiler with C++ gencode for Java features proto
*   [#14377](https://github.com/protocolbuffers/protobuf/pull/14377) - Use
    features in Java, Kotlin, and Java Lite code generators, including codegen
    tests
*   [#15210](https://github.com/protocolbuffers/protobuf/pull/15210) - Use
    features in Java full runtimes covering Java features bootstrap, feature
    resolution, and legacy editions, along with unit-tests and conformance
    testing

#### Pure Python {#python}

*   [#14546](https://github.com/protocolbuffers/protobuf/pull/14546) - Setup
    codegen tests in advance
*   [#14547](https://github.com/protocolbuffers/protobuf/pull/14547) - Fully
    implements editions in one shot, along with unit-tests and conformance
    testing

#### 𝛍pb {#upb}

*   [#14638](https://github.com/protocolbuffers/protobuf/pull/14638) - First
    pass at editions implementation covering feature resolution and legacy
    editions
*   [#14667](https://github.com/protocolbuffers/protobuf/pull/14667) - Added
    more complete handling of field label/type, support for upb’s code
    generator, and some tests
*   [#14678](https://github.com/protocolbuffers/protobuf/pull/14678) - Hooks up
    upb to the Python runtime, with more unit tests and conformance tests

#### Ruby {#ruby}

*   [#16132](https://github.com/protocolbuffers/protobuf/pull/16132) - Hook up
    upb/Java to all four Ruby runtimes for full editions support
