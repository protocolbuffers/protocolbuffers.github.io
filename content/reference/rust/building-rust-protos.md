+++
title = "Building Rust Protos"
weight = 783
linkTitle = "Building Rust Protos"
description = "Describes using Blaze to build Rust protos."
type = "docs"
toc_hide = "true"
+++

The process of building a Rust library for a Protobuf definition is similar to
other programming languages:

1.  Use the language-agnostic `proto_library` rule:

    ```build
    proto_library(
        name = "person_proto",
        srcs = ["person.proto"],
    )
    ```

2.  Create a Rust library:

    ```build {highlight="lines:1,8-11"}
    load("//third_party/protobuf/rust:defs.bzl", "rust_proto_library")

    proto_library(
        name = "person_proto",
        srcs = ["person.proto"],
    )

    rust_proto_library(
        name = "person_rust_proto",
        deps = [":person_proto"],
    )
    ```

3.  Use the library by including it in a Rust binary:

    ```build {highlight="lines:1,14-20"}
    load("//third_party/bazel_rules/rules_rust/rust:defs.bzl", "rust_binary")
    load("//third_party/protobuf/rust:defs.bzl", "rust_proto_library")

    proto_library(
        name = "person_proto",
        srcs = ["person.proto"],
    )

    rust_proto_library(
        name = "person_rust_proto",
        deps = [":person_proto"],
    )

    rust_binary(
        name = "greet",
        srcs = ["greet.rs"],
        deps = [
            ":person_rust_proto",
        ],
    )
    ```

See google3/devtools/rust/examples/protobuf/ for the full example.

**Note:** Don't use `rust_upb_proto_library` or `rust_cc_proto_library`
directly. `rust_proto_library` checks the global build flag to choose the
appropriate backend for you. See go/switching-rust-proto-library-backends if you
want to learn more.
