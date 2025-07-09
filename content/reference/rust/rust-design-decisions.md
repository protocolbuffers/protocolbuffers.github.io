+++
title = "Rust Proto Design Decisions"
weight = 785
linkTitle = "Design Decisions"
description = "Explains some of the design choices that the Rust Proto implementation makes."
type = "docs"
+++

As with any library, Rust Protobuf is designed considering the needs of both
Google's first-party usage of Rust as well that of external users. Choosing a
path in that design space means that some choices made will not be optimal for
some users in some cases, even if it is the right choice for the implementation
overall.

This page covers some of the larger design decisions that the Rust Protobuf
implementation makes and the considerations which led to those decisions.

## Designed to Be ‘Backed’ by Other Protobuf Implementations, Including C++ Protobuf {#backed-by-cpp}

Protobuf Rust is not a pure Rust implementation of protobuf, but a safe Rust API
implemented on top of existing protobuf implementations, or as we call these
implementations: kernels.

The biggest factor that goes into this decision was to enable zero-cost of
adding Rust to a preexisting binary which already uses non-Rust Protobuf. By
enabling the implementation to be ABI-compatible with the C++ Protobuf generated
code, it is possible to share Protobuf messages across the language boundary
(FFI) as plain pointers, avoiding the need to serialize in one language, pass
the byte array across the boundary, and deserialize in the other language. This
also reduces binary size for these use cases by avoiding having redundant schema
information embedded in the binary for the same messages for each language.

Protobuf Rust currently supports three kernels:

*   C++ kernel - the generated code is backed by C++ Protocol Buffers (the
    "full" implementation, typically used for servers). This kernel offers
    in-memory interoperability with C++ code that uses the C++ runtime. This is
    the default for servers within Google.
*   C++ Lite kernel - the generated code is backed by C++ Lite Protocol Buffers
    (typically used for mobile). This kernel offers in-memory interoperability
    with C++ code that uses the C++ Lite runtime. This is the default for
    for mobile apps within Google.
*   upb kernel - the generated code is backed by
    [upb](https://github.com/protocolbuffers/protobuf/tree/main/upb),
    a highly performant and small-binary-size Protobuf library written in C. upb
    is designed to be used as an implementation detail by Protobuf runtimes in
    other languages. This is the default in open source builds where we expect
    static linking with code already using C++ Protobuf to be more rare.

The decision to support multiple non-Rust kernels significantly influences our
public API decisions, including the types used on getters (discussed later in
this document).

### No Pure Rust Kernel {#no-pure-rust}

Given that we designed the API to be implementable by multiple backing
implementations, a natural question is why the only supported kernels are
written in the memory unsafe languages of C and C++ today.

While Rust being a memory-safe language can significantly reduce exposure to
critical security issues, no language is immune to security issues. The Protobuf
implementations that we support as kernels have been scrutinized and fuzzed to
the extent that Google is comfortable using those implementations to perform
unsandboxed parsing of untrusted inputs in our own servers and apps. A
greenfield binary parser written in Rust at this time would be understood to be
much more likely to contain critical vulnerabilities than the preexisting C++
Protobuf parser.

There are legitimate arguments for long-term supporting a pure Rust
implementation, including toolchain difficulties for developers using our
implementation in open source.

It is a reasonable assumption that Google will support a pure Rust
implementation at some later date, but we are not investing in it today and have
no concrete roadmap for it at this time.

## View/Mut Proxy Types {#view-mut-proxy-types}

The Rust Proto API is designed with opaque "Proxy" types. For a .proto file that
defines `message SomeMsg {}`, we generate the Rust types `SomeMsg`,
`SomeMsgView<'_>` and `SomeMsgMut<'_>`. The simple rule of thumb is that we
expect the View and Mut types to stand in for `&SomeMsg` and `&mut SomeMsg` in
all usages by default, while still getting all of the borrow checking/Send/etc.
behavior that you would expect from those types.

### Another Lens to Understand These Types {#another-lens}

To better understand the nuances of these types, it may be useful to think of
these types as follows:

```rust
struct SomeMsg(Box<cpp::SomeMsg>);
struct SomeMsgView<'a>(&'a cpp::SomeMsg);
struct SomeMsgMut<'a>(&'a mut cpp::SomeMsg);
```

Under this lens you can see that:

-   Given a `&SomeMsg` it is possible to get a `SomeMsgView` (similar to how
    given a `&Box<T>` you can get a `&T`)
-   Given a `SomeMsgView` it in *not* possible to get a `&SomeMsg` (similar to
    how given a `&T` you couldn't get a `&Box<T>`).

Just like with the `&Box` example, this means that on function arguments, it is
generally better to default to use `SomeMsgView<'a>` rather than a `&'a
SomeMsg`, as it will allow a superset of callers to use the function.

### Why {#why}

There are two main reasons for this design: to unlock possible optimization
benefits, and as an inherent outcome of the kernel design.

#### Optimization Opportunity Benefit {#optimization}

Protobuf being such a core and widespread technology makes it unusually both
prone to all possible observable behaviors being depended on by someone, as well
as relatively small optimizations having unusually major net impact at scale. We
have found that more opaqueness of types gives unusually high amount of
leverage: they permit us to be more deliberate about exactly what behaviors are
exposed, and give us more room to optimize the implementation.

A `SomeMsgMut<'_>` provides those opportunities where a `&mut SomeMsg` would
not: namely that we can construct them lazily and with an implementation detail
which is not the same as the owned message representation. It also inherently
allows us to control certain behaviors that we couldn't otherwise limit or
control: for example, any `&mut` can be used with `std::mem::swap()`, which is a
behavior that would place strong limits on what invariants you are able to
maintain between a parent and child struct if `&mut SomeChild` is given to
callers.

#### Inherent to Kernel Design {#kernel-design}

The other reason for the proxy types is more of an inherent limitation to our
kernel design; when you have a `&T` there must be a real Rust `T` type in memory
somewhere.

Our C++ kernel design allows you to parse a message which contains nested
messages, and create only a small Rust stack-allocated object to representing
the root message, with all other memory being stored on the C++ Heap. When you
later access a child message, there will be no already-allocated Rust object
which corresponds to that child, and so there's no Rust instance to borrow at
that moment.

By using proxy types, we're able to on-demand create the Rust proxy types that
semantically acting as borrows, without there being any eagerly allocated Rust
memory for those instances ahead of time.

## Non-Std Types {#non-std}

### Simple Types Which May Have a Directly Corresponding Std Type {#corresponding-std}

In some cases the Rust Protobuf API may choose to create our own types where a
corresponding std type exists with the same name, where the current
implementation may even simply wrap the std type, for example
`protobuf::UTF8Error`.

Using these types rather than std types gives us more flexibility in optimizing
the implementation in the future. While our current implementation uses the Rust
std UTF-8 validation today, by creating our own `protobuf::Utf8Error` type it
enables us to change the implementation to use the highly optimized C++
implementation of UTF-8 validation that we use from C++ Protobuf which is faster
than Rust's std UTF-8 validation.

### ProtoString {#proto-string}

Rust's `str` and `std::string::String` types maintain a strict invariant that
they only contain valid UTF-8, but C++ Protobuf and C++'s `std::string` type
generally do not enforce any such guarantee. `string` typed Protobuf fields are
intended to only ever contain valid UTF-8, but the enforcement of this has many
holes where a `string` field may end up containing invalid UTF-8 contents at
runtime.

To deliver on zero-cost message sharing between C++ and Rust while minimizing
costly validations or risk of undefined behavior in Rust, we chose not to using
the `str`/`String` types for `string` field getters, and introduced the types
`ProtoStr` and `ProtoString` instead which are equivalent types except they
could contain invalid UTF-8 in rare situations. Those types let the application
code choose if they wish to perform the validation on-demand to get a `&str`, or
operate on the raw bytes to avoid any validation.

We are aware that vocabulary types like `str` are very important to idiomatic
usage, and intend to keep an eye on if this decision is the right one as usage
details of Rust evolves.
