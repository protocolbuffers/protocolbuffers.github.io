+++
title = "UTF-8 Validation in Java"
weight = 650
linkTitle = "UTF-8 Validation in Java"
description = "Describes the UTF-8 string behavior in Java protobuf across all syntaxes, editions, and option/feature settings."
type = "docs"
+++

.

Proto2 has both string and bytes, and from the beginning it was documented
that strings must always contain UTF-8 encoded text.
Enforcement of this requirement varies from language to language (details later
in this topic), which means that in practice, many string fields contain
non-UTF-8 data despite what the spec says.

For proto3 it was decided that all parsers must validate UTF-8. This
decision came almost a year after proto3 had debuted in
google3, so this change required
a project to set
`enforce_utf8=false` on all existing string fields before flipping the default
to `enforce_utf8=true`. It was a goal of the LSC to burn down all uses of
`enforce_utf8=false` and ultimately remove the
option.

Here is a summary of UTF-8 validation behavior for Java Protobuf across syntaxes
and editions. This behavior is consistent for both standard string fields and
string-typed extensions:

Syntax / Edition        | `features.utf8_validation`  | `features.(pb.java).utf8_validation`    | `java_string_check_utf8`   | `enforce_utf8`            | Java UTF-8 Validation Behavior
:---------------------- | :-------------------------- | :-------------------------------------- | :------------------------- | :------------------------ | :-----------------------------
**proto2**              | *Disallowed*                | *Disallowed*                            | unset or `false` (default) | *Ignored*                 | **No validation** (proto2 default)
**proto2**              | *Disallowed*                | *Disallowed*                            | `true`                     | *Ignored*                 | **Validation enabled***
**proto3**              | *Disallowed*                | *Disallowed*                            | *Ignored*                  | unset or `true` (default) | **Validation enabled** (proto3 default)
**proto3**              | *Disallowed*                | *Disallowed*                            | *Ignored*                  | `false`                   | **No validation**
**Edition 2023 / 2024** | unset or `VERIFY` (default) | unset, `DEFAULT` (default), or `VERIFY` | *Disallowed*               | *Disallowed*              | **Validation enabled**** (editions default)
**Edition 2023 / 2024** | `NONE`                      | unset or `DEFAULT` (default)            | *Disallowed*               | *Disallowed*              | **No validation**
**Edition 2023 / 2024** | `NONE`                      | `VERIFY`                                | *Disallowed*               | *Disallowed*              | **Validation enabled** (validation in Java only)

\* In Java Lite, `proto2` string-typed extensions are **not** validated when
`java_string_check_utf8=true`.

\** In Java Lite, under edition 2023 and 2024, string-typed extensions are
**not** validated under any combination of options. This is an oversight that
should be corrected in a future
edition.

### Unvalidated Behavior

JavaLite and Java have the same API for string fields, but use different object
representations:

*   **String-Only:** JavaLite always stores strings as `java.lang.String`. This
    means that the field cannot preserve invalid UTF-8. If invalid UTF-8 is
    parsed or assigned via `setFoo(ByteString)`, it immediately undergoes a
    lossy conversion to replacement characters.
*   **Dual Mode:** Java "full" supports a dual representation, where the
    `java.lang.Object` field can be either `java.lang.String` or
    `java.lang.ByteString`. This means that invalid UTF-8 can be preserved when
    we parse proto2, and `ByteString getFooBytes()` will return whatever was
    parsed off the wire.

### Legacy Options

-   **`java_string_check_utf8`** is a **file-level option** applicable only to
    `proto2`. This option can be set in a proto3 file, but it will have no
    effect. Java Lite does **not** validate string-typed extensions.
-   **`enforce_utf8`** is a **field-level option** applicable only to `proto3`.
    Setting it to `false` is the only way to disable validation in proto3, but
    it is deprecated and subject to removal (and requires being on an
    allowlist). This option can be set in a proto2 file, but it will have no
    effect.
-   Both legacy options are **disallowed under Editions**, where you must use
    the features framework instead.

### Editions Features

-   **`features.utf8_validation`** is a file- or field-level feature to control
    UTF-8 validation for **all** languages.
-   **`features.(pb.java).utf8_validation`** is a file- or field-level feature
    to control UTF-8 validation for **Java Only**. The default is to use
    `features.utf8_validation` for Java. This feature should be phased out in
    favor of `features.utf8_validation`.
-   Both features are **disallowed under proto2 and proto3** (as are all
    editions features).
