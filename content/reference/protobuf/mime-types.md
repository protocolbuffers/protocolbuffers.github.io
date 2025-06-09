+++
title = "Protocol Buffer MIME Types"
weight = 830
linkTitle = "MIME Types"
description = "Standard MIME types for Protobuf Serializations."
type = "docs"
+++

All Protobuf documents should have the MIME type `application` and the subtype
`protobuf`, with the suffix `+json` for
JSON
encodings according to
[the standard](https://datatracker.ietf.org/doc/draft-murray-dispatch-mime-protobuf/),
followed by the following parameters:

-   `encoding` should be set only to `binary`,
    or `json`, denoting those respective
    formats.
    +   With subtype `protobuf+json`, `encoding` has the default `json` and
        cannot be set to `binary`. With subtype `protobuf` (without `+json`),
        `encoding` has the default `binary` and cannot be set to
        `json`.
    +   Use `+json` for JSON even in HTTP responses that use parser
        breakers as a CORB mitigation.
-   Set `charset` to `utf-8` for all JSONor Text Format encodings, and never set
    it for binary encodings.
    +   If `charset` is unspecified it is assumed to be UTF-8. It is preferable
        to always specify a `charset` as that may prevent certain attack vectors
        when protos are used in HTTP responses.
-   Protobuf reserves the `version` parameter for potential future versioning of
    our wire formats. Do not set it until a wire format is versioned.

So the standard MIME types for common protobuf encodings are:

-   `application/protobuf` for serialized binary protos.
-   `application/protobuf+json; charset=utf-8` for JSON format protos.

Services that read Protobuf should also handle `application/json`, which may be
used to encode JSON format protos.

Parsers must fail if MIME parameters (`encoding`, `charset`, or `version`) have
unknown or illegal values.

When binary protos are transacted over HTTP, Protobuf strongly recommends
Base64-encoding them and setting `X-Content-Type-Options: nosniff` to prevent
XSS, as it is possible for a Protobuf to parse as active content.

It is acceptable to pass additional parameters to these MIME types if desired,
such as a type URL which indicates the content schema; but the MIME type
parameters ***must not*** include encoding options.
