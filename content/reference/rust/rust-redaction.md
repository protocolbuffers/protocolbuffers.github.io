+++
title = "Redaction in Rust"
weight = 785
linkTitle = "Redaction in Rust"
description = "Describes redaction in Rust."
type = "docs"
toc_hide = "true"
+++

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'esrauch' reviewed: '2024-07-26' }
*-->

Use the standard `fmt::Debug` ("`{:?}`" in format strings) on Protobuf messages
for human-readable strings for logging, error messages, exceptions, and similar
use cases. The output of this debug info is not intended to be machine-readable
(unlike `TextFormat` and `JSON` which are
[not be used for debug output](/programming-guides/dos-donts#text-format-interchange)).

Using `fmt::Debug` enables redaction of some sensitive fields. See
go/proto-stringify-redaction for specifics about this redaction.

Note that under upb kernel this redaction is not yet implemented, but is
expected to be added. We expect almost all users in Google3 to use the default
cpp kernel; reach out to protobuf-team@ if you intend to use upb kernel.
