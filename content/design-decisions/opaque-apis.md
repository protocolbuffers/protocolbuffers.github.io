

+++
title = "We Prefer Opaque APIs"
weight = 90
description = "Covers why Protobuf generated code hides a message's memory representation behind accessors"
type = "docs"
+++

Generated Protobuf code can present a message in one of two broad styles:

*   **Open struct APIs** make a message's fields public members of a plain
    struct. User code reads them, writes them, and takes their address directly:
    `msg.foo = 42`, `*msg.foo`, `&msg.foo`.
*   **Opaque APIs** keep the in-memory representation private and expose the
    message only through generated methods: getters, setters, hazzers, clearers
    and, in some languages, builders.

The Protobuf team favors opaque APIs in every language. Open structs look
simpler, and in some languages they look more idiomatic, but they promote a
message's physical memory layout into public API, which causes long term
problems.

## Opaqueness Is Leverage {#leverage}

Protobuf is unusual in two ways, and both point in the same direction.

First, Protobuf is depended on by nearly everything, so *every* observable
behavior of generated code is depended on by someone, somewhere. This is
[Hyrum's Law](https://www.hyrumslaw.com/) at full strength: in practice,
anything we expose we can never change.

Second, Protobuf sits underneath serialization, RPC, and storage for entire
fleets of services, so even small improvements to parsing or to the in-memory
representation have an unusually large aggregate impact.

Put together, too many details exposed in the generated API tends to be the
single largest constraint on our ability to make Protobuf faster and smaller. We
have found that more opaqueness gives a high amount of leverage by comparison:
it lets us be more deliberate about exactly which behaviors are guaranteed, and
it leaves room to change the implementation underneath.

## Examples of What Opaqueness Buys {#what-it-buys}

Each of the following is impossible, or badly compromised, when field storage is
public API.

### Cached State {#cachedstate}

Due to the binary wire format using size-prefixes, the typical serialize
implementation must do two passes: one to compute encoded lengths and then one
that uses those lengths to write.

With Opaque APIs, you can maintain a cached encoded length internally, and
invalidate it when modifications occur. This unlocks significantly speeding up
serializations.

With Open Structs you cannot: without setters there is no way to 'know' if state
has been modified which may have modified the encoded length, which means you
must redo the sizing work on every serialization.

### Lazy Decoding {#lazy}

With Opaque APIs, decoding can be deferred past parse time. Lazy submessage
fields, lazy extensions, lazy UTF-8 validation, and similar techniques all work
by keeping the raw bytes and doing the work on first access, inside the getter.
A service that reads a few top-level routing fields and forwards the rest never
pays for the subtree at all.

With Open Structs you cannot: a plain field read is not a function call, so
there is no interception point, and the parser has no choice but to eagerly
resolve everything as it materializes the message tree.

### Unboxed Values and Dense Hasbits {#hasbits}

Tracking [explicit
presence](/programming-guides/field_presence) requires
storing one bit of information per field. An opaque API stores it as exactly
that: one bit in a dense `hasbits` bitvector next to the inline field value.

An open struct in a language without zero-cost value optionality cannot do this,
because the presence bit is not reachable from the field expression itself. The
typical representation ends up being a pointer per field — `*int32`, `*bool`,
`java.lang.Integer` — which costs a machine word in the parent struct, a
separate allocation, an extra indirection on every read, and garbage collector
pressure, all to carry one bit. Even in languages like Rust with Option<T>, the
Open API precludes maintaining a dense hasbit bitmask over all of the fields,
and instead spreads out the option bit throughout the struct (including
typically with a lot of wasting padding per field).

Dense hasbits are not just cheaper to store; they make whole-message operations
cheaper, because `Clear()`, `ByteSizeLong()` and `MergeFrom()` can test a mask
of many fields at once and skip them without ever loading the field data. For
more on this, see [Implicit
Presence](/design-decisions/implicit-presence#performance).

### Freedom to Change the Memory Layout {#layout}

When callers cannot depend on field offsets and cannot hold a pointer into a
message, the runtime can rearrange the message freely:

*   **Layout by access pattern.** Frequently accessed fields can be placed
    together in the first cache line, and rarely set fields can be moved out
    into an overflow allocation that only exists when one of them is set. These
    are workload-specific decisions, which means they are best driven by
    profiles rather than fixed at code-generation time.
*   **Arena and custom allocation.** Message internals can be allocated out of
    contiguous arenas or slabs, which is only safe if no caller is holding a raw
    pointer to a field whose lifetime we are about to change.
*   **Compact representations for strings and bytes.** Inline small buffers,
    reference-counted or shared slices, and views into the parse buffer are all
    available when the accessor mediates every read.

None of these are one-time wins. The point is that they remain *available*: a
future representation change ships as a new runtime, not as a migration of every
caller.

## Case Study: GoProto {#go}

Go is the canonical example of a language that started with an open struct API
and moved to an opaque one.

Historically, GoProto generated public structs in which explicit-presence
scalars were pointers (`*int32`, `*string`) and proto3 implicit-presence scalars
were bare values. Open structs were considered a requirement for a high-quality
Go implementation at the time proto3 was designed.

In practice the opposite proved true: the Go maintainers found that open structs
had boxed them in. Fields could not be reordered, lazy decoding could not be
implemented, and presence could not be unboxed.

Callers could also depend on the generated struct itself, to the point that
adding a single unused zero-sized field to generated code broke thousands of
tests ,
which meant that even changes that were pure wins were prohibitively expensive
to roll out.

The Go protobuf developers, who argued strongly that they could not create a
high-quality, efficient implementation without open struct, reversed this
decision and introducing accessor methods. They realized that using open structs
boxed them in, restricting what optimizations they are able to implement. With
open structs, they can't reorder fields or implement lazy decoding. If they had
avoided open structs, they would have more flexibility now with how to optimize.

The result was the Go Opaque API, in which struct fields are unexported and
access goes through `Get*`, `Set*`, `Has*`, `Clear*` and builders. It is the
default for [Edition 2024](/editions/overview) and newer,
and the published results line up with the reasoning above: presence for
elementary fields moved from a pointer per field to a bit per field, allocation
counts in the reference benchmarks dropped by roughly 46% and 58%, and the lazy
decoding benchmark saves over 50% of the work and over 87% of allocations. For
details, see [Go Protobuf: The new Opaque
API](https://go.dev/blog/protobuf-opaque), [Go Opaque API
FAQ](/reference/go/opaque-faq) and [Go Opaque API
Migration](/reference/go/opaque-migration).

Note that moving to the Opaque API is a migration, not a deprecation: the Open
Struct API continues to be supported.

Go was not the only such experiment. `javanano`, an open struct Java generator
for Android, was also considered important when proto3 was designed — its
generated messages were mutable objects with public fields and no builders. It
was likewise abandoned in favor of the encapsulated `javalite` runtime. Two
independent attempts at open structs, in two very different languages, both
ended the same way.

## How Opaque Is Opaque Enough? {#how-far}

More opaqueness is always more implementation freedom, but it is not free: past
some point it costs fluency in the target language's ordinary idioms. There is a
real trade-off here, and we make it deliberately, per language, based on what
capability is actually unlocked.

Protobuf Rust is the current frontier of that trade-off. Idiomatic Rust would
have submessage accessors return `&Foo` and `&mut Foo`, and string accessors
return `&str`. Instead, Rust Protobuf returns view and mut *proxy* types,
`FooView<'_>` and `FooMut<'_>`, because native references would impose two
language-level invariants we cannot satisfy:

*   **A `&Foo` requires a real Rust `Foo` to exist in memory at that address.**
    A core goal of Protobuf Rust is adding Rust to binaries that already use C++
    Protobuf at zero cost, sharing messages across the FFI boundary as plain
    pointers rather than serializing across it. Under that design a parsed
    message's children live in the C++ heap and have no corresponding Rust
    object to borrow; handing out `&Foo` would mean eagerly materializing a
    mirror Rust struct for every submessage. It would also fix the memory
    representation, which is exactly what we want to keep free: Protobuf Rust
    supports several kernels with different layouts behind one API.
*   **A `&mut Foo` can be passed to `std::mem::swap`.** A bytewise swap of two
    submessages belonging to different parents cannot fix up arena pointers, and
    more generally it removes our ability to maintain any invariant between a
    parent message and its children. Note that this example may be fixed in
    upcoming changes to Rust, but it is still the case today that there is no
    way to offer `&mut` which is ineligible for `swap`.

Proxy types cost Rust callers some familiarity, and we know it. We took that
cost because it is what makes zero-copy C++/Rust interoperability possible at
all; no smaller amount of opaqueness would have done it. See [Rust Proto Design
Decisions](/reference/rust/rust-design-decisions#view-mut-proxy-types)
for the further discussion.

## What Opaqueness Is Not {#what-it-is-not}

Opaque APIs are about hiding *representation*, not about hiding data or adding
ceremony. In particular:

*   Accessors are not an invitation to add behavior. A getter returns the field
    value, or the field's default if it is unset, see [No Nullable
    Setters/Getters](/design-decisions/nullable-getters-setters).
*   Opaqueness is not immutability. Whether messages are mutable is a separate,
    per-language decision.
*   Opaqueness is not a reason to skip reflection. Every runtime offers Protobuf
    reflection; what an opaque API removes is the temptation to reach for
    *language* reflection over fields instead.
