

+++
title = "Insertion Points and Generic Services Regretted and Deprecated"
weight = 92
description = "Covers why the Protobuf team regrets insertion points and generic services, and what to do instead"
type = "docs"
+++

Very early on in Protobuf's history (\~2008-2010), Protobuf added two different
mechanisms which allow code outside of Protobuf to graft itself onto Protobuf's
generated code:

*   **Generic services**, present since the first public release, are the
    abstract, RPC-system-agnostic `Service`, `Stub`, `RpcChannel` and
    `RpcController` classes that were supported in the C++, Java and Python
    generators
*   **Insertion points**, added in 2010, are comment markers that the generators
    leave in their output so that a *second* protoc plugin can splice arbitrary
    text into the middle of a generated file, including into the body of a
    generated message class.

The two are linked: insertion points were added in the same release that
deprecated generic services in favor of RPC-specific plugins, and insertion
points were one mechanism by which a separate plugin could put its stubs back
into the same generated file that the generic services had occupied.

Generic services have been marked deprecated since 2010, and insertion points
have always been labeled 'experimental', though their continued existence for
over 15 years may have incorrectly signaled their ongoing stability.

Both are design decisions the Protobuf team regrets:

*   Generic services have been formally deprecated since 2010, and no supported
    language after 2010 added them. We're working towards finally turning down
    this behavior in a future breaking release.
*   Insertion points have always been marked 'EXPERIMENTAL' and still work in
    the generators that originally shipped them, but we consider the mechanism a
    mistake. New language implementations do not emit them, we do not recommend
    that new plugins use them, and we know that routine private implementation
    detail changes to our generated code may break users of them.

Our recommendation for both is the same, and it is to follow the model that gRPC
uses: write a standalone plugin that emits its own files, with an API designed
for your system, and compile them in a build target of their own that depends on
the Protobuf generated code which is in a separate file.

## The Shared Mistake {#shared-mistake}

The two features look different but fail for the same reason: both make code
that the Protobuf team does not own depend on the *inside* of Protobuf's
generated code instead of composing with its public surface.

As [We Prefer Opaque APIs](/design-decisions/opaque-apis)
explains, Protobuf is depended on by nearly everything, so every observable
detail of generated code is depended on by someone. The only thing that keeps
Protobuf evolvable is being deliberate about which details are API and which are
implementation. Generated code is co-versioned with the runtime and is *not*
API: we restructure it freely to make parsing faster, messages smaller, and new
features possible (see [Cross-Version Runtime
Guarantee](/support/cross-version-runtime-guarantee)).

Insertion points hand third parties a way to depend on the text of that
generated code. Generic services bake an RPC abstraction into the message code
and the core runtime. Either way, the next improvement to generated code has to
negotiate with code the Protobuf team never saw.

We know what the alternative costs you. Insertion points gave you one import and
methods that appear to live on the message itself; generic services gave you an
RPC-agnostic interface without writing a plugin. Giving those up means a second
import, a second build target, and, unless you are using an existing RPC system
such as gRPC, a plugin of your own. That cost is small, explicit, and paid once
by the plugin author. The cost of grafting onto the generated code instead is
large, hidden, and paid by everyone downstream of the message library, every
time the generated code changes.

## Insertion Points {#insertion-points}

### What They Are {#insertion-points-what}

The C++, Java, Python and Objective-C generators write comment markers into
their output. A C++ header looks roughly like this:

```cpp
// @@protoc_insertion_point(includes)

namespace foo {

class Bar final : public ::google::protobuf::Message {
 public:
  // ...
  // @@protoc_insertion_point(class_scope:foo.Bar)
};

inline int32_t Bar::id() const {
  // @@protoc_insertion_point(field_get:foo.Bar.id)
  return _internal_id();
}

// @@protoc_insertion_point(namespace_scope)
}  // namespace foo

// @@protoc_insertion_point(global_scope)
```

A plugin that runs later in the same `protoc` invocation can return a
`CodeGeneratorResponse.File` whose `insertion_point` names one of these markers.
`protoc` then splices the plugin's text in immediately above the marker line,
inheriting its indentation. The plugin never sees the file it is editing; it
only knows the name of a location in it.

Each generator exposes its own ad hoc set of markers: C++ has `includes`,
`namespace_scope`, `global_scope`, `class_scope:T` and a marker inside the body
of every generated accessor; Java has `outer_class_scope`, `class_scope:T`,
`builder_scope:T`, `enum_scope:T`, and even `message_implements:T` and
`builder_implements:T`, which let a plugin add arbitrary interfaces to the
`implements` clause of a generated class; Python has `imports`, `module_scope`
and `class_scope:T`.

The plugin-side API for all of this, `GeneratorContext::OpenForInsert()`, has
carried the comment "WARNING: This feature is currently EXPERIMENTAL and is
subject to change" since it was introduced in 2010. It was never promoted out of
that status.

### Why They Are Too Invasive {#insertion-points-why}

**Inserted code sees implementation details and nothing stops it from using
them.** Text spliced into `class_scope` is compiled as a member of the generated
class. It can see private fields, internal helpers, the base class, the macros
that happen to be defined, the headers that happen to be included, and the order
in which everything was declared. Our documentation has always said "do not
generate code which relies on private class members declared by the standard
code generator", but the mechanism has no way to enforce that, and the inserter
has no way to declare what it actually depends on. In practice, Hyrum's Law
applies: whatever is visible gets used.

**They inevitably break as the runtime evolves.** Protobuf generated code is not
frozen; it is a moving implementation detail of the runtime it ships with. Over
the years we have moved C++ field storage into an internal `Impl_` struct, split
rarely-set fields out of the main object, changed how presence bits are tracked,
introduced `_internal_` accessors, added arena support, changed the default
headers and macros, and we will keep doing this. Every such change is a
potential compile error inside someone else's inserted text. Worse, the inserted
text can keep compiling while silently violating a new invariant: code that
writes a field directly without setting its hasbit, or that reads a submessage
field that is now lazily parsed, is wrong in a way that no compiler will report.
These failures surface in another team's plugin, in another repository, at an
unpredictable time, and the Protobuf team cannot see them coming because the
dependency is on a *position in a text file*, not on a declared API.

**They fuse unrelated code into one artifact and one build target.** Inserted
code lives in the same `.pb.h`, `Foo.java` or `_pb2.py` as the messages, so it
lives in the same library. Everyone who depends on the messages now transitively
depends on whatever the inserting plugin needed, typically an entire RPC
runtime, and the file can only be produced by running every contributing plugin
in a single `protoc` invocation, in the right order. This is how a `.proto` that
merely *defines* a service ends up forcing a message-only client to link an RPC
stack, and how message libraries end up failing to build on platforms where that
RPC stack does not exist.

### Every Major User Has Moved Away {#insertion-points-history}

The only significant use of insertion points has been RPC systems inserting
service stubs into message files, and every one of them has since converged on
standalone files:

*   **gRPC C++** has always generated separate `.grpc.pb.h` / `.grpc.pb.cc`
    files, and its plugin refuses to run if `cc_generic_services` is enabled.
*   **gRPC Python** originally inserted its services into `_pb2.py` at the
    `module_scope` insertion point. The generator now writes a separate
    `_pb2_grpc.py` by default, and the inserted variant is marked for removal
    ("THESE ELEMENTS WILL BE DEPRECATED. Please use the generated
    `*_pb2_grpc.py` files instead.").
*   **gRPC Go** was once generated by a `plugins=grpc` option inside
    `protoc-gen-go`, emitting services into the same `.pb.go` as the messages.
    The option was removed in the APIv2 rewrite of Go Protobuf; gRPC now ships
    its own `protoc-gen-go-grpc`, which writes `_grpc.pb.go`.

### What to Do Instead {#insertion-points-instead}

Write a standalone plugin that:

*   emits its own files (`foo.grpc.pb.h`, `FooGrpc.java`, `foo_pb2_grpc.py`,
    `foo_grpc.pb.go`, and so on), and never names an `insertion_point`;
*   depends on messages only through their public generated API and through
    reflection, descriptors and custom options; and
*   is driven by its own build rule (a `*_grpc_library` or equivalent) that
    depends on the message library, rather than being hidden inside the message
    build.

If you need per-message behavior that *looks* like a method on the message, use
the idioms the language already has for extending a type you do not own: free
functions or traits in C++ and Rust, static utility classes or extension
functions in Java and Kotlin, wrapper types, and so on. If you need per-message
*data*, put it in a custom option and read it through the descriptor at build
time or at run time.

This applies to small insertions too. A `class_scope` helper that has nothing to
do with RPC does not drag an RPC runtime into the message library, but it is
still compiled as a member of the generated class, still sees every private
detail of it, and still breaks when those details change; the first two problems
in [Why They Are Too Invasive](#insertion-points-why) do not depend on the size
of the insertion. Interfaces added through `message_implements:T`,
`builder_implements:T` or `interface_extends:T` have no one-to-one replacement,
and that is deliberate: use an adapter or wrapper type, or a registry keyed by
descriptor, rather than changing the type hierarchy of a class you do not own.

## Generic Services {#generic-services}

### What They Are {#generic-services-what}

Generic services date from the original 2008 design and were built on the idea
that RPC systems could plug in underneath by implementing `RpcChannel` and
`RpcController`. The intent was that Protobuf would define the shape of a
service once, and every RPC implementation would fit itself to that shape.

### Why They Were an Insufficient Generalization {#generic-services-why}

The `service.h` header describes the controller as "a 'least common denominator'
set of features which we expect all implementations to support". The least
common denominator turned out to be too small to hold a real RPC system:

*   **Unary only.** `CallMethod(method, controller, request, response, done)`
    takes exactly one request message and produces exactly one response message.
    Client streaming, server streaming and bidirectional streaming, first-class
    features of both gRPC and Stubby, cannot be expressed at all.
*   **One hard-coded calling convention.** The interface is asynchronous with a
    `Closure*` completion callback (a Google-specific type exposed in a public
    interface). It does not fit blocking calls, futures, completion queues,
    coroutines, Kotlin `suspend` functions or reactive streams. Supporting just
    one additional convention, blocking calls in Java, required a second
    parallel set of generated types: `BlockingInterface`, `BlockingStub`,
    `BlockingRpcChannel`, `BlockingService` and `ServiceException`.
*   **No room for RPC semantics.** `RpcController` offers `Failed()`,
    `ErrorText()`, `StartCancel()`, `SetFailed(string)`, `IsCanceled()` and
    `NotifyOnCancel()`, and nothing else: no deadlines, no metadata or headers,
    no structured status codes (errors are a string), no compression, no
    authentication context, no interceptors, no flow control. Any real system
    has to downcast the controller to its own subtype to get at these, at which
    point the "generic" layer is nothing but, in the words of our own
    documentation, "more levels of indirection than code tailored to one
    system".
*   **Requires full reflection.** Dispatch goes through `MethodDescriptor` and
    `Message` rather than `MessageLite`, so generic services cannot be used with
    the lite runtimes on mobile and embedded platforms, which are precisely the
    platforms where a portable RPC story would have mattered most. `protoc`
    enforces this directly: "Files with `optimize_for = LITE_RUNTIME` cannot
    define services unless you set both options `cc_generic_services` and
    `java_generic_services` to false."
*   **They tie the message library to an RPC abstraction.** The service code is
    emitted into the same files as the messages, and `service.h` lives in the
    core runtime. This is the same coupling problem as insertion points, with
    the same build-time consequences.

The result is that very few RPC systems were built on generic services. gRPC
generates its own stubs in every language and its C++ plugin fails outright if
`cc_generic_services` is on. The original design goal, "define the shape once
and let RPC systems fit themselves to it", was inverted in practice: the generic
shape fit very few users, and the RPC systems had to be designed around escaping
it.

### What to Do Instead {#generic-services-instead}

Follow the gRPC model. Keep `service` definitions in your `.proto` files: they
remain the authoritative schema of the API, they are available through
reflection and descriptors, and they can carry custom options. But the *code*
for a service should come from the RPC system's own protoc plugin, into its own
files, with an API designed for that system's calling conventions, streaming
model, status codes, deadlines and metadata.

Protobuf's role is to be the interface definition language and the message
layer. The RPC layer belongs to the RPC system.

## Guidance for New Languages and New Plugins {#guidance}

*   New Protobuf language implementations do not emit insertion points and do
    not generate generic services. This is not an oversight to be fixed later.
*   New protoc plugins generate standalone files. A plugin that needs to be
    "inside" Protobuf's generated code is a sign that the design should be
    revisited, not a reason to add an insertion point.
*   Changes to the structure of generated code that break code inserted at an
    insertion point are not considered breaking changes to Protobuf.
