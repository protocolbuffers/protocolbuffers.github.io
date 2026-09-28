

+++
title = "Implicit Presence"
weight = 91
description = "Covers why the Protobuf team regrets implicit field presence"
type = "docs"
+++

Implicit presence — the proto3 behavior in which a singular scalar field has no
hazzer and the value `0`, `false` or `""` is indistinguishable from "never set"
— is a design decision the Protobuf team regrets.

We do not recommend using implicit presence for new fields.

*   In proto3, prefer to tag your fields `optional`. This is a weak guidance
    though; if you are still using the Go Open API you may have good reason to
    choose to prefer implicit presence fields.
*   Explicit presence is the silent default in [Protobuf
    Editions](/editions/overview) going forward.

For the operational details of how presence behaves in each dialect, see the
[Field Presence](/programming-guides/field_presence)
application note. This page is about *why*.

## What Implicit Presence Is {#what}

[Field presence](/programming-guides/field_presence) is
whether a message tracks that a field was set, separately from what it was set
to.

*   Under **explicit presence**, the message stores a presence bit alongside the
    value and generates a `has_foo()` hazzer. Setting `foo = 0` makes the field
    present and writes it to the wire; clearing it makes it absent.
*   Under **implicit presence**, the message stores only the value, and no
    hazzer is generated. Setting `foo = 0` is indistinguishable from clearing
    `foo` or never touching it, and `0` is never written to the wire.

The choice for explicit or implicit presence applies only exists for singular
numeric, `bool`, `string`, `bytes` and `enum` fields. Message fields and `oneof`
members are always explicit presence, and repeated fields and maps are always
implicit presence.

### proto3 {#proto3}

In proto3, implicit presence is the default for those field types, and the
`optional` keyword opts back in to explicit presence:

```proto
syntax = "proto3";

message UserAccount {
  // Implicit presence (the proto3 default).
  // No has_login_count() is generated, login_count = 0 is indistinguishable
  // from unset, and 0 is never serialized.
  int32 login_count = 1;

  // Explicit presence (recommended).
  // has_display_name() is generated, and display_name = "" is serialized.
  optional string display_name = 2;
}
```

### Editions {#editions}

Editions removed the `optional` keyword and replaced it with the
[`field_presence`
feature](/editions/features#field_presence), which
defaults to `EXPLICIT`. Implicit presence is now the behavior you must
specifically set:

```proto
edition = "2023";

message UserAccount {
  // Explicit presence, no keyword required.
  string display_name = 1;

  // Implicit presence requires an explicit opt-in.
  int32 login_count = 2 [features.field_presence = IMPLICIT];
}
```

> **NOTE:** Implicit presence does not exist in proto2. Every singular proto2
> field has explicit presence.

## Why it was Introduced {#why-introduced}

Implicit presence was not introduced for its semantics. It was introduced to
enable [open struct APIs](/design-decisions/opaque-apis).

In a language without zero-cost value optionality, a public struct field cannot
carry a presence bit; the only way to represent "unset" is to box the field into
a pointer. The proto3 premise was that if the data model simply deleted the
distinction between "unset" and "set to the default value," generated code could
be plain structs of unboxed public fields, with no accessors and no `hasbits` —
and that this would be a good enough deal to be worth the semantic compromise.

The original proto3 design doc put it this way:

> Removal of field presence logic for primitive value fields, removal of
> required fields, and removal of default values. This makes Proto3
> significantly easier to implement with open struct representations, as in
> languages like Android Java, Objective-C, or Go.

The design rationale rested on two claims: that open structs made protobuf
meaningfully easier to implement, and that presence tracking was pure overhead
for most fields, both of which turned out to not be true.

## The Open Struct Premise Did Not Hold {#open-structs}

Google realized that open structs were overly constraining on implementations,
as explained in [Opaque
APIs](/design-decisions/opaque-apis).

So the API shape that implicit presence was designed to unlock is no longer
considered a good design, while the semantics it traded away are still present.

## It Is Not Faster {#performance}

The most persistent belief about implicit presence is that not maintaining
hasbits would mean less work. This has turned out not to hold in practice.

Explicit presence packs presence into dense 32-bit words at the front of the
message. That density is the point — it lets whole-message operations make
decisions about many fields with a single load:

*   **Batch skipping.** In `Clear()`, `ByteSizeLong()` and `MergeFrom()`, the
    C++ generator gates a group of fields behind a single mask test
    (`BatchCheckHasBit(cached_has_bits, 0x000000ffU)`), and can gate whole
    regions of a wide message behind a test over several has-words. Absent
    fields are skipped without their memory ever being touched, which matters
    because real messages are sparse and cold fields are exactly the ones you do
    not want to pull into cache.
*   **Branchless sizing.** For a group of fixed-width fields, `ByteSizeLong()`
    computes the contribution of the entire group as `popcount(mask &
    cached_has_bits) * size`, with no branches and without loading a single
    field value. The generator restricts this optimization to fields that
    `has_presence()`, because it is only valid when the bit is authoritative.

Implicit presence forfeits all of this. Presence is tied to the value, so
deciding whether a field participates requires loading the field and comparing
it against zero — once per field, every time, across the whole struct.

More recently, we have begun having hasbits even for implicit presence fields
anyway: these are hints a set bit means "probably present," so accessors must
still compare the value, but the batch skip they enable has proven to be a
beneficial optimization.

Once you implement this optimization, implicit presence fields actually are
strictly *more* work per field than explicit presence: an explicit-presence
field tests one bit and acts but an implicit-presence field tests the hint bit
*and then* loads and compares the value.

## The Semantics Are 'Bad' {#semantics}

Implicit presence does not remove the concept of presence. It makes zero and the
empty string into magic values and ties presence to them. The distinction
resurfaces anywhere the system needs to talk about what was *set* rather than
what the value *is*.

### Merging {#merging}

`MergeFrom` copies fields that are present in the source over the destination.
Under implicit presence, a field set to zero is not present, so it does not
merge — which means a merge can produce a value that existed in neither input.

The canonical demonstration is a well-known type. `google.protobuf.Timestamp`
was introduced with proto3, so both of its fields have implicit presence:

```proto
syntax = "proto3";

package google.protobuf;

message Timestamp {
  int64 seconds = 1;
  int32 nanos = 2;
}
```

Take three messages containing a `Timestamp`:

*   `A = {timestamp: {seconds: 100, nanos: 50}}` — 100.000000050s
*   `B = {timestamp: {seconds: 200, nanos: 1}}` — 200.000000001s
*   `C = {timestamp: {seconds: 200, nanos: 0}}` — 200.000000000s

Submessages merge recursively, field by field:

*   `A.MergeFrom(B)` gives `{seconds: 200, nanos: 1}`. Both of `B`'s fields are
    non-zero, so both are present, so both overwrite.
*   `A.MergeFrom(C)` gives **`{seconds: 200, nanos: 50}`**. `C`'s `nanos: 0` is
    indistinguishable from unset, so it does not overwrite, and `A`'s old
    `nanos: 50` survives.

Merging the instant 200.000000000s onto 100.000000050s produces 200.000000050s:
a timestamp that appears in neither operand, off by 50 nanoseconds, with no
error anywhere. Had `Timestamp` been defined with explicit presence, `C` would
carry a set bit for `nanos` and the merge would be correct.

Nothing about this is specific to timestamps. Any message whose meaning depends
on a combination of fields — a coordinate, a range, a version triple, a ratio —
has the same hazard, and the hazard is worst precisely where zero is a perfectly
ordinary value.

### MergeFrom(msg) != MergeFrom(bytes)

Protobuf intends to have `x.MergeFrom(Parse(bytes))` semantically be the same as
`x.MergeFrom(bytes)` as an important first class semantic.

One of the rare edge cases where this is violated is implicit presence: if the
bytes did contain a zero value, the `Parse(bytes)` will effectively discard it
and so the subsequent merge operation will not propagate it. By contrast, when
merging directly from bytes the zero value can be 'known' and replace a value
that was already set.

### Patch Semantics {#patch}

The same rule breaks partial updates in general. If a field has implicit
presence, a patch cannot express "set this to zero," because a zero in the patch
is indistinguishable from an omission. That gap is what
`google.protobuf.FieldMask` exists to fill: an out-of-band, hand-maintained list
of which fields the caller meant, however `FieldMask` was found to have other
serious design deficiencies.

### JSON {#json}

Native JSON implementations essentially have explicit presence: after a parse
you can check if a given key was present on the wire encoding or not.

In ProtoJSON, with implicit presence there is no hazzer: a zero-valued field
simply disappears from the output, and there is no easy way to 'set' the field
in a way that will signal it should be emitted. The only available remedy was to
add serializer options for including default values, which is exactly the kind
of format-dialect knob we otherwise [work hard to
avoid](/programming-guides/json#json-options). Explicit
presence makes this a non-question: set the field and it is emitted, clear it
and it is not.

## The Escape Hatches Were Worse {#workarounds}

Because real systems do need presence, proto3 users were routed to
`google.protobuf.Int32Value` and friends, to `FieldMask`, and to single-field
`oneof` blocks. Each of these is worse than a hasbit on every axis that matters:

*   **Wrapper types are not wire-compatible with the primitive field they
    replace**, so the decision has to be made correctly before the first release
    and cannot be reversed.
*   **Wrapper types are more expensive**, turning one bit in a bitvector into a
    nested submessage with its own pointer, its own allocation, and its own
    unknown-field set.
*   **`FieldMask` moves presence out of the message** and into a parallel
    structure the application has to keep in sync by hand.
*   **Single-field `oneof` blocks** do carry presence, but they misrepresent the
    schema, they generate a different API in every language, and that API is
    consistently worse than a hazzer.

These were not theoretical costs. The extra allocation per wrapper was a
measurable source of latency in real client libraries, and [the GitHub issue
asking for presence to be readded to
proto3](https://github.com/protocolbuffers/protobuf/issues/1606) became the
most-commented issue in the Protobuf repository.

The `optional` keyword was restored to proto3 as a result: accepted behind
`--experimental_allow_proto3_optional` in v3.12.0, and Editions then made
explicit presence the default outright.
