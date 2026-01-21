+++
title = "Debugging"
weight = 915
description = "Debugging common issues in Protocol Buffers."
type = "docs"
+++

Frequently asked questions and debugging scenarios encountered by protobuf
users.

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'sangki' reviewed: '2025-02-11' }
*-->

## Bazel: Resolving Issues with Prebuilt Protoc {#prebuilt-protoc}

As mentioned in [this news article](/news/2026-01-16),
Bazel 7 and later support builds that use a prebuilt protoc. When everything is
configured correctly, this can save a lot of time building, reduce logging
volume, and eliminate the need for C++ toolchain in non-C++ implementations.

To prevent protoc compilation from source, add the following to your `.bazelrc`
file:

```bazel
common --per_file_copt=external/.*protobuf.*@--PROTOBUF_WAS_NOT_SUPPOSED_TO_BE_BUILT
common --host_per_file_copt=external/.*protobuf.*@--PROTOBUF_WAS_NOT_SUPPOSED_TO_BE_BUILT
common --per_file_copt=external/.*grpc.*@--GRPC_WAS_NOT_SUPPOSED_TO_BE_BUILT
common --host_per_file_copt=external/.*grpc.*@--GRPC_WAS_NOT_SUPPOSED_TO_BE_BUILT
```

When there are hard-coded references to `@com_google_protobuf//:protoc` that are
reachable by the dependency graph, protoc will force it to be built for those
cases, negating this benefit. To resolve this, you'll need to find where the
dependencies are and request that those rulesets be updated.

To find where you depend on protoc, use the following command:

```none
bazel cquery 'somepath("//...", "@com_google_protobuf//:all")'
```

**Note:** Prebuilts are only available for our standard set of platforms. Anyone
on a nonstandard platform will have to build protoc from source.
