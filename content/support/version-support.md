+++
title = "Version Support"
weight = 910
description = "A list of the support windows provided for language implementations."
aliases = "/version-support/"
type = "docs"
+++

<link rel="stylesheet" href="/includes/version-tables.css">

Support windows for protoc and the various languages are covered in the tables
later in this topic. Version numbers throughout this topic use
[SemVer](https://semver.org) conventions; in the version "3.21.7," we say that
"3" is the major version, "21" is the minor version, and "7" is the micro or
patch number.

Starting with the v20.x protoc release, we changed our versioning scheme to
enable nimbler updates to language-specific parts of Protocol Buffers. In the
new scheme, each language has its own major version that can be incremented
independently of other languages. The minor and patch versions, however, remain
coupled. This allows us to introduce breaking changes into some languages
without requiring a bump of the major version in languages that do not
experience a breaking change. For example, a single release might include protoc
version 24.0, Java runtime version 4.24.0 and C# runtime version 3.24.0.

The first instance of this new versioning scheme was the 4.21.0 version of the
Python API, which followed the preceding version, 3.20.1. Other language APIs
released at the same time were released as 3.21.0.

## Release Cadence {#cadence}

Protobuf strives to release updates quarterly. We may add a release if there is
an urgent need such as a security fix that requires new APIs. Skipping a release
should be a very rare event.

Major (breaking) releases will be targeted to the Q1 release. We may introduce a
major breaking change at any time if there is an urgent need, but this should be
very rare.

Our support windows are defined by our
[library breaking change policy](https://opensource.google/documentation/policies/library-breaking-change).

Protobuf does *not* consider enforcement of its documented language, tooling,
platform, and library support policies to be a breaking change. For example, a
release may drop support for an EOL language version without bumping major
versions.

## What Changes in a Release? {#changes}

**The binary wire format does not change** even in major version updates. You
will continue to be able to read old binary wire format proto data from newer
versions of Protocol Buffers. Newly generated protobuf bindings serialized to
binary wire format will be parseable by older binaries. This is a fundamental
design principle of Protocol Buffers. Note that JSON and textproto formats *do
not* offer the same stability guarantees.

**The descriptor.proto schema can change.** In minor or patch releases, we may
add new message, fields, enums, enum values, editions, editions
[features](/editions/features) etc. We may also mark
existing elements as deprecated. In a major release, we may remove deprecated
options, enums, enum values, messages, fields, etc.

**The .proto language grammar can change.** In minor or patch releases, we may
add new language constructs and alternative syntax for existing features. We may
also mark certain features as deprecated. This could result in new warnings that
were not previously emitted by `protoc`. In a major release, we may remove
support for obsolete features, syntax, editions in a way that will require
updates to client code.

**Gencode and runtime APIs can change.** In minor or patch releases, changes
will either be purely additive for new functionality or source-compatible
updates. Simply recompiling code should work. In a major release, the gencode or
runtime API can change in incompatible ways that require callsite changes. We
try to minimize these. Changes fixing or otherwise affecting undefined behavior
are not considered breaking, and do not require a major release.

**Operating system, programming language, and tooling version support can
change.** In a minor or patch release, we may add or drop support for particular
versions of an operating system, programming language, or tooling. See
[foundational support matrices](https://github.com/google/oss-policies-info/tree/main)
for our supported languages.

In general:

*   Minor or patch releases should only contain purely additive or
    source-compatible updates per our
    [Cross Version Runtime Guarantee](https://protobuf.dev/support/cross-version-runtime-guarantee/#minor)
*   Major releases may remove functionality, features, or change APIs in ways
    that require updates to callsites.

## Support Duration {#duration}

The most recent release is always supported. Support for earlier minor versions
ends when a new minor version under the same major version is released. Support
for earlier major versions ends four quarters beyond the quarter that the
breaking release is introduced. For example, when Protobuf Python 5.26.0 was
released in Q1 of 2024, that set the end of support of Protobuf Python 4.25.x at
the [end of Q1 2025](#python).

The following sections provide a guide to the support for each language.

## C++ {#cpp}

C++ will target making major version bumps annually in Q1 of each year.

The protoc version can be inferred from the Protobuf C++ minor version number.
Example: Protobuf C++ version 4.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf C++</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>25 May 2022</td>
    <td>31 Mar 2024</td>
  </tr>
  <tr class="maintenance">
    <th>4.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="active">
    <th>5.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2026</td>
  </tr>
  <tr class="future">
    <th>6.x</th>
    <td>Q1 2025</td>
    <td>31 Mar 2027</td>
  </tr>
</table>

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf C++</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>21.x</td>
    <td class="y23q1 maintenance">3.21</td>
    <td class="y23q2 maintenance">3.21</td>
    <td class="y23q3 maintenance">3.21</td>
    <td class="y23q4 maintenance">3.21</td>
    <td class="y24q1 maintenance">3.21</td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
  </tr>
  <tr class="maintenance">
    <th>4.x</th>
    <td>22.x-25.x</td>
    <td class="y23q1 active">4.22</td>
    <td class="y23q2 active">4.23</td>
    <td class="y23q3 active">4.24</td>
    <td class="y23q4 active">4.25</td>
    <td class="y24q1 maintenance">4.25</td>
    <td class="y24q2 maintenance">4.25</td>
    <td class="y24q3 maintenance">4.25</td>
    <td class="y24q4 maintenance">4.25</td>
    <td class="y25q1 maintenance">4.25</td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
  </tr>
  <tr class="active">
    <th>5.x</th>
    <td>26.x-29.x</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">5.26</td>
    <td class="y24q2 active">5.27</td>
    <td class="y24q3 active">5.28</td>
    <td class="y24q4 active">5.29</td>
    <td class="y25q1 maintenance">5.29</td>
    <td class="y25q2 maintenance">5.29</td>
    <td class="y25q3 maintenance">5.29</td>
    <td class="y25q4 maintenance">5.29</td>
  </tr>
  <tr class="future">
    <th>6.x</th>
    <td>30.x-33.x</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1 active">6.30</td>
    <td class="y25q2 active">6.31</td>
    <td class="y25q3 active">6.32</td>
    <td class="y25q4 active">6.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

### C++ Tooling, Platform, and Library Support {#cpp-tooling}

Protobuf is committed to following the tooling, platform, and library support
policy described in
[Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
For specific versions supported, see
[Foundational C++ Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md).

## C&#35; {#csharp}

The protoc version can be inferred from the Protobuf C# minor version number.
Example: Protobuf C# version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf C#</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="active">
    <th>3.x</th>
    <td>16 Feb 2023</td>
    <td>TBD</td>
  </tr>
</table>

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf C#</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="active">
    <th>3.x</th>
    <td>22.x-33.x</td>
    <td class="y23q1 active">3.22</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 active">3.26</td>
    <td class="y24q2 active">3.27</td>
    <td class="y24q3 active">3.28</td>
    <td class="y24q4 active">3.29</td>
    <td class="y25q1 active">3.30</td>
    <td class="y25q2 active">3.31</td>
    <td class="y25q3 active">3.32</td>
    <td class="y25q4 active">3.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

### C&#35; Platform and Library Support {#csharp-support}

Protobuf is committed to following the platform and library support policy
described in
[.NET Support Policy](https://opensource.google/documentation/policies/dotnet-support).
For specific versions supported, see
[Foundational .NET Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-dotnet-support-matrix.md).

## Java {#java}

Java will target making major version bumps annually in Q1 of each year.

The protoc version can be inferred from the Protobuf Java minor version number.
Example: Protobuf Java version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf Java</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2026*</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2027</td>
  </tr>
  <tr class="future">
    <th>5.x</th>
    <td>Q1 2026*</td>
    <td>31 Mar 2028</td>
  </tr>
</table>

{{% alert title="Note" color="note" %}}
The maintenance support window for the Protobuf Java 3.x release will be 24
months rather than the typical 12 months for the final release in a major
version line. Future major version updates (5.x, planned for Q1 2026) will adopt
an improved
["rolling compatibility window"](/support/cross-version-runtime-guarantee/#major)
that should allow a return to 12-month support windows. There will be no major
version bump in Q1 2025.{{% /alert %}}

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Java</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>22.x-25.x</td>
    <td class="y23q1 active">3.22</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 maintenance">3.25</td>
    <td class="y24q2 maintenance">3.25</td>
    <td class="y24q3 maintenance">3.25</td>
    <td class="y24q4 maintenance">3.25</td>
    <td class="y25q1 maintenance">3.25</td>
    <td class="y25q2 maintenance">3.25</td>
    <td class="y25q3 maintenance">3.25</td>
    <td class="y25q4 maintenance">3.25</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>26.x-33.x</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">4.26</td>
    <td class="y24q2 active">4.27</td>
    <td class="y24q3 active">4.28</td>
    <td class="y24q4 active">4.29</td>
    <td class="y25q1 active">4.30</td>
    <td class="y25q2 active">4.31</td>
    <td class="y25q3 active">4.32</td>
    <td class="y25q4 active">4.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

### Java Platform and Library Support {#java-support}

Protobuf is committed to following the platform and library support policy
described in
[Java Support Policy](https://cloud.google.com/java/docs/supported-java-versions).
For specific versions supported, see
[Foundational Java Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-java-support-matrix.md).

On Android, Protobuf supports the minimum SDK version that is supported by
[Google Play services](https://developers.google.com/android/guides/setup) and
is the default in
[Jetpack](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-main/docs/api_guidelines/modules#module-minsdkversion).
If both versions differ, the lower version is supported.

## Objective-C {#objc}

The protoc version can be inferred from the Protobuf Objective-C minor version
number. Example: Protobuf Objective-C version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf Objective-C</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="active">
    <th>3.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2026</td>
  </tr>
  <tr class="future">
    <th>4.x</th>
    <td>Q1 2025</td>
    <td>TBD</td>
  </tr>
</table>

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Objective-C</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="active">
    <th>3.x</th>
    <td>22.x-29.x</td>
    <td class="y23q1 active">3.22</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 active">3.26</td>
    <td class="y24q2 active">3.27</td>
    <td class="y24q3 active">3.28</td>
    <td class="y24q4 active">3.29</td>
    <td class="y25q1 maintenance">3.29</td>
    <td class="y25q2 maintenance">3.29</td>
    <td class="y25q3 maintenance">3.29</td>
    <td class="y25q4 maintenance">3.29</td>
  </tr>
  <tr class="future">
    <th>4.x</th>
    <td>30.x+</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1 active">4.30</td>
    <td class="y25q2 active">4.31</td>
    <td class="y25q3 active">4.32</td>
    <td class="y25q4 active">4.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

## PHP {#php}

The protoc version can be inferred from the Protobuf PHP minor version number.
Example: Protobuf PHP version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf PHP</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>13 Mar 2024</td>
    <td>TBD</td>
  </tr>
</table>

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf PHP</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>22.x-25.x</td>
    <td class="y23q1 active">3.22</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 maintenance">3.25</td>
    <td class="y24q2 maintenance">3.25</td>
    <td class="y24q3 maintenance">3.25</td>
    <td class="y24q4 maintenance">3.25</td>
    <td class="y25q1 maintenance">3.25</td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>26.x+</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">4.26</td>
    <td class="y24q2 active">4.27</td>
    <td class="y24q3 active">4.28</td>
    <td class="y24q4 active">4.29</td>
    <td class="y25q1 active">4.30</td>
    <td class="y25q2 active">4.31</td>
    <td class="y25q3 active">4.32</td>
    <td class="y25q4 active">4.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

### PHP Platform and Library Support {#php-support}

Protobuf is committed to following the platform and library support policy
described in
[PHP Support Policy](https://cloud.google.com/php/getting-started/supported-php-versions).
For specific versions supported, see
[Foundational PHP Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-php-support-matrix.md).

## Python {#python}

The protoc version can be inferred from the Protobuf Python minor version
number. Example: Protobuf Python version 4.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf Python</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="maintenance">
    <th>4.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="active">
    <th>5.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2026</td>
  </tr>
  <tr class="future">
    <th>6.x</th>
    <td>Q1 2025</td>
    <td>TBD</td>
  </tr>
</table>

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Python</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="maintenance">
    <th>4.x</th>
    <td>22.x-25.x</td>
    <td class="y23q1 active">4.22</td>
    <td class="y23q2 active">4.23</td>
    <td class="y23q3 active">4.24</td>
    <td class="y23q4 active">4.25</td>
    <td class="y24q1 maintenance">4.25</td>
    <td class="y24q2 maintenance">4.25</td>
    <td class="y24q3 maintenance">4.25</td>
    <td class="y24q4 maintenance">4.25</td>
    <td class="y25q1 maintenance">4.25</td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
  </tr>
  <tr class="active">
    <th>5.x</th>
    <td>26.x-29.x</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">5.26</td>
    <td class="y24q2 active">5.27</td>
    <td class="y24q3 active">5.28</td>
    <td class="y24q4 active">5.29</td>
    <td class="y25q1 maintenance">5.29</td>
    <td class="y25q2 maintenance">5.29</td>
    <td class="y25q3 maintenance">5.29</td>
    <td class="y25q4 maintenance">5.29</td>
  </tr>
  <tr class="future">
    <th>6.x</th>
    <td>30.x+</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1 active">6.30</td>
    <td class="y25q2 active">6.31</td>
    <td class="y25q3 active">6.32</td>
    <td class="y25q4 active">6.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

### Python Platform and Library Support {#python-support}

Protobuf is committed to following the platform and library support policy
described in
[Python Support Policy](https://cloud.google.com/python/docs/supported-python-versions).
For specific versions supported, see
[Foundational Python Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-python-support-matrix.md).

## Ruby {#ruby}

The protoc version can be inferred from the Protobuf Ruby minor version number.
Example: Protobuf Ruby version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

<table class="version-table">
  <tr>
    <th>Protobuf Ruby</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>13 Mar 2024</td>
    <td>TBD</td>
  </tr>
</table>

**Release support chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Ruby</th>
    <th>protoc</th>
    <th class="y23q1"><span>23Q1</span></th>
    <th class="y23q2"><span>23Q2</span></th>
    <th class="y23q3"><span>23Q3</span></th>
    <th class="y23q4"><span>23Q4</span></th>
    <th class="y24q1"><span>24Q1</span></th>
    <th class="y24q2"><span>24Q2</span></th>
    <th class="y24q3"><span>24Q3</span></th>
    <th class="y24q4"><span>24Q4</span></th>
    <th class="y25q1"><span>25Q1</span></th>
    <th class="y25q2"><span>25Q2</span></th>
    <th class="y25q3"><span>25Q3</span></th>
    <th class="y25q4"><span>25Q4</span></th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>22.x-25.x</td>
    <td class="y23q1 active">3.22</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 maintenance">3.25</td>
    <td class="y24q2 maintenance">3.25</td>
    <td class="y24q3 maintenance">3.25</td>
    <td class="y24q4 maintenance">3.25</td>
    <td class="y25q1 maintenance">3.25</td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>26.x+</td>
    <td class="y23q1"></td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">4.26</td>
    <td class="y24q2 active">4.27</td>
    <td class="y24q3 active">4.28</td>
    <td class="y24q4 active">4.29</td>
    <td class="y25q1 active">4.30</td>
    <td class="y25q2 active">4.31</td>
    <td class="y25q3 active">4.32</td>
    <td class="y25q4 active">4.33</td>
  </tr>
</table>

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

### Ruby Platform and Library Support {#ruby-support}

Protobuf is committed to following the platform and library support policy
described in
[Ruby Support Policy](https://cloud.google.com/ruby/getting-started/supported-ruby-versions).
For specific versions supported, see
[Foundational Ruby Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-ruby-support-matrix.md).

JRuby is not officially supported, but we provide unofficial support for the
latest JRuby version targeting compatibility with our minimum Ruby version or
above on a best-effort basis.
