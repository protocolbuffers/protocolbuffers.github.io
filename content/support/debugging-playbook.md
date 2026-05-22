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
freshness: { owner: 'sangki' reviewed: '2026-05-18' }
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

## Protostats Flume Pipeline

The Protostats pipeline runs on borg through a Plx workflow defined by files in
google3/net/proto2/tools/internal/protostats and it requires quota to run. The
quota is taken from the `gpi-flex-pool`. Currently, Borg quota is registered for
the `protobuf-stats` user in 3 different cells: `yq`, `viatlrb`, and `vibrura`.
The latter are go/virtual-cells. The cell diversity and the use of virtual cells
are both intended to provide redundancy against stockouts caused by real
resource constraints or the occasional DiRT exercise. If registration needs to
be adjusted, see directions in
google3/net/proto2/tools/internal/protostats/BUILD.
