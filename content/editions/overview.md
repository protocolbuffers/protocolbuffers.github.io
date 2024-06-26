+++
title = "Protobuf Editions Overview"
linkTitle = "Overview"
weight = 42
description = "An overview of the Protobuf Editions functionality."
type = "docs"
+++

Protobuf Editions replace the proto2 and proto3 designations that we have used
for Protocol Buffers. Instead of adding `syntax = "proto2"` or `syntax =
"proto3"` at the top of proto definition files, you use an edition number, such
as `edition = "2023"`, to specify the default behaviors your file will have.
Editions enable the language to evolve incrementally over time.

Instead of the hardcoded behaviors that older versions have had, editions
represent a collection of [features](/editions/features)
with a default value (behavior) per feature. Features are options on a file,
message, field, enum, and so on, that specify the behavior of protoc, the code
generators, and protobuf runtimes. You can explicitly override a behavior at
those different levels (file, message, field, ...) when your needs don't match
the default behavior for the edition you've selected. You can also override your
overrides. The [section later in this topic on lexical scoping](#scoping) goes
into more detail on that.

*The latest released edition is 2023.*

## Lifecycle of a Feature {#lifecycles}

Editions provide the fundamental increments for the lifecycle of a feature.
Features have an expected lifecycle: introducing
it, changing its default behavior, deprecating it, and then removing it. For
example:

1.  Edition 2031 creates `feature.amazing_new_feature` with a default value of
    `false`. This value maintains the same behavior as all earlier editions.
    That is, it defaults to no impact.

2.  Developers update their .proto files to `edition = "2031"`.

3.  A later edition, such as edition 2033, switches the default of
    `feature.amazing_new_feature` from `false` to `true`. This is the desired
    behavior for all protos, and the reason that the protobuf team created the
    feature.

    Using the Prototiller tool to migrate earlier versions of proto files to
    edition 2033 adds explicit `feature.amazing_new_feature = false` entries as
    needed to continue to retain the previous behavior. Developers remove these
    newly-added settings when they want the new behavior to apply to their
    .proto files.

<!-- mdformat off (preserve single lines/no wrapping) -->

4.  At some point, `feature.amazing_new_feature` is marked deprecated in an edition and removed in a later one.

    When a feature is removed, the code generators for that behavior and the
    runtime libraries that support it may also be removed. The timelines will be
    generous, though. Following the example in the earlier steps of the
    lifecycle, the deprecation might happen in edition 2034 but not be removed
    until edition 2036, roughly two years later. Removing a feature will always
    initiate a major version bump.

<!-- mdformat on -->

Because of this lifecycle, any `.proto` file that does not use deprecated
features has a no-op upgrade from one edition to the next.
You will have the full
window of the Google migration plus the deprecation window to upgrade your code.

The preceding lifecycle example used boolean values for the features, but
features may also use enums. For example, `features.field_presence` has values
`LEGACY_REQUIRED`, `EXPLICIT`, and `IMPLICIT.`

## Migrating to Protobuf Editions {#migrating}

Editions won't break existing binaries and don't change a message's binary,
text, or JSON serialization format. The first edition is as minimally disruptive
as possible. The first edition establishes the baseline and combines proto2 and
proto3 definitions into a new single definition format.

When the subsequent editions are released, default behaviors for features may
change. You can have Prototiller do a no-op transformation of your .proto file
or you can choose to accept some or all of the new behaviors. Editions are
planned to be released roughly once a year.

### Proto2 to Editions {#proto2-migration}

This section shows a proto2 file, and what it might look like after running the
Prototiller tool to change the definition files to use Protobuf Editions syntax.

<section class="tabs">

#### Proto2 syntax {.new-tab}

```proto
// proto2 file
syntax = "proto2";

package com.example;

message Player {
  // in proto2, optional fields have explicit presence
  optional string name = 1;
  // proto2 still supports the problematic "required" field rule
  required int32 id = 2;
  // in proto2 this is not packed by default
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto2 enums are closed
  optional Handed handed = 4;

  reserved "gender";
}
```

#### Editions syntax {.new-tab}

```proto
// Edition version of proto2 file
edition = "2023";

package com.example;

message Player {
  // fields have explicit presence, so no explicit setting needed
  string name = 1;
  // to match the proto2 behavior, LEGACY_REQUIRED is set at the field level
  int32 id = 2 [features.field_presence = LEGACY_REQUIRED];
  // to match the proto2 behavior, EXPANDED is set at the field level
  repeated int32 scores = 3 [features.repeated_field_encoding = EXPANDED];

  enum Handed {
    // this overrides the default edition 2023 behavior, which is OPEN
    option features.enum_type = CLOSED;
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;

  reserved gender;
}
```

</section>

### Proto3 to Editions {#proto3-migration}

This section shows a proto3 file, and what it might look like after running the
Prototiller tool to change the definition files to use Protobuf Editions syntax.

<section class="tabs">

#### Proto3 syntax {.new-tab}

```proto
// proto3 file
syntax = "proto3";

package com.example;

message Player {
  // in proto3, optional fields have explicit presence
  optional string name = 1;
  // in proto3 no specified field rule defaults to implicit presence
  int32 id = 2;
  // in proto3 this is packed by default
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto3 enums are open
  optional Handed handed = 4;

  reserved "gender";
}
```

#### Editions syntax {.new-tab}

```proto
// Editions version of proto3 file
edition = "2023";

package com.example;

message Player {
  // fields have explicit presence, so no explicit setting needed
  string name = 1;
  // to match the proto3 behavior, IMPLICIT is set at the field level
  int32 id = 2 [features.field_presence = IMPLICIT];
  // PACKED is the default state, and is provided just for illustration
  repeated int32 scores = 3 [features.repeated_field_encoding = PACKED];

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;

  reserved gender;
}
```

</section>

<a name="inheritance"></a>

### Lexical Scoping {#scoping}

Editions syntax supports lexical scoping, with a per-feature list of allowed
targets. For example, in the first edition, features can be specified at only
the file level or the lowest level of granularity. The implementation of lexical
scoping enables you to set the default behavior for a feature across an entire
file, and then override that behavior at the message, field, enum, enum value,
oneof, service, or method level. Settings made at a higher level (file, message)
apply when no setting is made within the same scope (field, enum value). Any
features not explicitly set conform to the behavior defined in the edition
version used for the .proto file.

The following code sample shows some features being set at the file, field, and
enum level. The settings are in the highlighted lines:

```proto {highlight="lines:3,7,16"}
edition = "2023";

option features.enum_type = CLOSED;

message Person {
  string name = 1;
  int32 id = 2 [features.field_presence = IMPLICIT];

  enum Pay_Type {
    PAY_TYPE_UNSPECIFIED = 1;
    PAY_TYPE_SALARY = 2;
    PAY_TYPE_HOURLY = 3;
  }

  enum Employment {
    option features.enum_type = OPEN;
    EMPLOYMENT_UNSPECIFIED = 0;
    EMPLOYMENT_FULLTIME = 1;
    EMPLOYMENT_PARTTIME = 2;
  }
  Employment employment = 4;
}
```

In the preceding example, the presence feature is set to `IMPLICIT`; it would
default to `EXPLICIT` if it wasn't set. The `Pay_Type` `enum` will be `CLOSED`,
as it applies the file-level setting. The `Employment` `enum`, though, will be
`OPEN`, as it is set within the enum.

### Prototiller {#prototiller}

We provide both a migration guide and migration tooling that ease the migration
to and between editions. The tool, called Prototiller, will enable you to:

*   convert proto2 and proto3 definition files to the new editions syntax, at
    scale
*   migrate files from one edition to another
*   manipulate proto files in other ways

### Backward Compatibility {#compatibility}

We are building Protobuf Editions to be as minimally disruptive as possible. For
example, you can import proto2 and proto3 definitions into editions-based
definition files, and vice versa:

```proto
// file myproject/foo.proto
syntax = "proto2";

enum Employment {
  EMPLOYMENT_UNSPECIFIED = 0;
  EMPLOYMENT_FULLTIME = 1;
  EMPLOYMENT_PARTTIME = 2;
}
```

```proto
// file myproject/edition.proto
edition = "2023";

import "myproject/foo.proto";
```

While the generated code changes when you move from proto2 or proto3 to
editions, the wire format does not. You'll still be able to access proto2 and
proto3 data files or file streams using your editions-syntax proto definitions.
