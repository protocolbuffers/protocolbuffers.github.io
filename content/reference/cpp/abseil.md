+++
title = "Asbseil Support"
weight = 530
linkTitle = "Asbseil Support"
description = "The C++ implementation of Protocol Buffers has an explicit dependency on Abseil."
type = "docs"
+++

In [version 22.x](/news/v22#abseil-dep), C++ protobuf
added an explicit dependency on Abseil.

## Bazel Support {#bazel}

If you are using Bazel, to determine the version of Abseil that your protobuf
version supports, you can use the `bazel mod` command:

```shell
$ bazel mod deps abseil-cpp --enable_bzlmod
<root> (protobuf@30.0-dev)
└───abseil-cpp@20240722.0
    ├───bazel_skylib@1.7.1
    ├───googletest@1.15.2
    └───platforms@0.0.10
```

`bazel mod graph` produces the full output:

```shell
$ bazel mod graph --enable_bzlmod
<root> (protobuf@30.0-dev)
├───abseil-cpp@20240722.0
│   ├───bazel_skylib@1.7.1 (*)
│   ├───googletest@1.15.2 (*)
│   └───platforms@0.0.10 (*)
├───bazel_features@1.18.0
│   └───bazel_skylib@1.7.1 (*)
├───bazel_skylib@1.7.1
│   ├───platforms@0.0.10 (*)
│   └───rules_license@1.0.0 (*)
├───googletest@1.15.2
│   ├───abseil-cpp@20240722.0 (*)
│   ├───platforms@0.0.10 (*)
│   └───re2@2024-07-02
...
```

## CMake Support {#cmake}

Our CMake support is best-effort compared to Bazel. To check for support, try
the following steps:

1.  Run the `cmake .` command.
2.  Open `_deps/absl-src/CMakeLists.txt`.

Look for the following line:

```
project(absl LANGUAGES CXX VERSION 20240722)
set(ABSL_SOVERSION "2407.0.0")
include(CTest)
```
