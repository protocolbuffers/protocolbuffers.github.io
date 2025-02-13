+++
title = "Protocol Buffer Compiler Installation"
weight = 15
description = "How to install the protocol buffer compiler."
type = "docs"
no_list = "true"
linkTitle = "Protoc Installation"
+++

The protocol buffer compiler, `protoc`, is used to compile `.proto` files, which
contain service and message definitions. Choose one of the methods given below
to install `protoc`.

### Install Pre-compiled Binaries (Any OS) {#binary-install}

To install the latest release of the protocol compiler from pre-compiled
binaries, follow these instructions:

1.  From https://github.com/google/protobuf/releases, manually download the zip
    file corresponding to your operating system and computer architecture
    (`protoc-<version>-<os>-<arch>.zip`), or fetch the file using commands such
    as the following:

    ```sh
    PB_REL="https://github.com/protocolbuffers/protobuf/releases"
    curl -LO $PB_REL/download/v< param protoc-version >/protoc-< param protoc-version >-linux-x86_64.zip
    ```

2.  Unzip the file under `$HOME/.local` or a directory of your choice. For
    example:

    ```sh
    unzip protoc-< param protoc-version >-linux-x86_64.zip -d $HOME/.local
    ```

3.  Update your environment's path variable to include the path to the `protoc`
    executable. For example:

    ```sh
    export PATH="$PATH:$HOME/.local/bin"
    ```

### Install Using a Package Manager {#package-manager}

{{% alert title="Warning" color="warning" %}} Run
`protoc --version` to check the version of `protoc` after using a package
manager for installation to ensure that it is sufficiently recent. The versions
of `protoc` installed by some package managers can be quite dated. See the
[Version Support page](https://protobuf.dev/support/version-support) to compare
the output of the version check to the minor version number of the supported
version of the language(s) you are
using.{{% /alert %}}

You can install the protocol compiler, `protoc`, with a package manager under
Linux, macOS, or Windows using the following commands.

-   Linux, using `apt` or `apt-get`, for example:

    ```sh
    apt install -y protobuf-compiler
    protoc --version  # Ensure compiler version is 3+
    ```

-   MacOS, using [Homebrew](https://brew.sh):

    ```sh
    brew install protobuf
    protoc --version  # Ensure compiler version is 3+
    ```

-   Windows, using
    [Winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/)

    ```sh
    > winget install protobuf
    > protoc --version # Ensure compiler version is 3+
    ```

### Other Installation Options {#other}

If you'd like to build the protocol compiler from sources, or access older
versions of the pre-compiled binaries, see
[Download Protocol Buffers](https://protobuf.dev/downloads).
