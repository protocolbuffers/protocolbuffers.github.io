+++
title = "Version Support"
weight = 910
description = "A list of the support windows provided for language implementations."
aliases = "/version-support/"
type = "docs"
+++

<link rel="stylesheet" href="/includes/version-tables.css">

## Currently Supported Release Versions {#currently-supported}

Language | Active Support | Maintenance Only      | Minimum Gencode
-------- | -------------- | --------------------- | ---------------
protoc   | 34.x           | 29.x, 25.x (for Java) |
C++      | 7.34.x         | 5.29.x                | Exact Match
C#       | 3.34.x         |                       | 3.0.0
Java     | 4.34.x         | 3.25.x                | 3.0.0
PHP      | 5.34.x         |                       | 4.0.0
Python   | 7.34.x         | 5.29.x                | [3.20.0](https://protobuf.dev/support/cross-version-runtime-guarantee#python)
Ruby     | 4.34.x         |                       | 3.0.0

### Minimum Supported Gencode {#min-gencode}

While each language runtime has a minimum supported gencode version, we
recommend regenerating your gencode with every release update. Support for older
generated code exists solely to ensure backwards compatibility for existing
projects, not for new adoption of older gencode versions.

See
[Cross Version Runtime Guarantee](/support/cross-version-runtime-guarantee)
for more information.

## Supported Editions {#supported-editions}

Protobuf release versions are independent of edition "versions" (proto2, proto3,
2023, 2024). All currently supported release versions support all editions.

The current supported editions are:

Edition | Released Version | Date Released
------- | ---------------- | -------------
2024    | 32.0             | 23 May 2025
2023    | 27.0             | 13 Aug 2024
proto3  | 3.0              | 2016
proto2  | 2.0              | 2008

## Numbering Scheme {#numbering}

Version numbers use [SemVer](https://semver.org) conventions. In the version
"3.21.7", "3" is the major version, "21" is the minor version, and "7" is the
patch number.

Protobuf releases use only a `minor.point` format, for example `29.5`.

Each language runtime shares this `minor.point` but uses a language-specific
major version. For example, Protobuf release `29.5` corresponds to Java runtime
`4.29.5` and C# runtime `3.29.5`. We recommend using `protoc` `29.5` with Java
`4.29.5` and C# `3.29.5`.

This scheme unifies release numbers across all languages while decoupling major
version changes. For example, release `30.0` contained breaking changes for
Python but not for Java. Therefore, Python advanced from `5.29.0` to `6.30.0`,
while Java advanced from `4.29.0` to `4.30.0`.

We introduced this versioning scheme in 2022 with release 21. Previously, all
languages used major version 3.

## Release Cadence {#cadence}

Protobuf releases updates quarterly. We target major (breaking) releases for Q1.
Security fixes and other urgent needs will require additional releases.

Our
[library breaking change policy](https://opensource.google/documentation/policies/library-breaking-change)
defines our support windows.

Enforcing documented language, tooling, platform, and library support policies
is *not* a breaking change. For example, we might drop support for an
End-of-Life (EOL) language version without bumping the major version.

## What Changes in a Release? {#changes}

**The binary wire format never changes.** You can read old binary wire format
data with newer Protobuf versions. You can read new binary wire format data with
older Protobuf versions (as long as the .proto syntax is supported). ProtoJSON
format offers these same stability guarantees. TextProto is intended to be used
for configuration use-cases and should not be used as a wire format between two
servers.

**The `descriptor.proto` schema can change.** In minor or patch releases, we
might add or deprecate elements (messages, fields, enums, enum values, editions,
or editions [features](/editions/features)). In releases
with breaking changes (major releases), we might remove deprecated elements.

**The `.proto` language grammar can change with new Editions**. In minor
release, support for a new Edition may be added, which will add language
constructs or deprecate existing behaviors. Adopting a new Edition may require
client code updates. A future release may drop support for an old syntax.
However, we have no current concrete plans to do so.

**Gencode and runtime APIs can change.** Minor and patch releases include purely
additive or source-compatible updates. You can simply recompile your code.
Releases with breaking changes (major releases) introduce incompatible API
changes that require callsite updates. We minimize these changes. Bug fixes for
undefined behavior do not require a major release.

**Operating system, programming language, and tooling support can change.**
Minor or patch releases might add or drop support for specific operating
systems, programming languages, or tools. See the
[foundational support matrices](https://github.com/google/oss-policies-info/tree/main)
for supported languages.

In general:

*   Minor or patch releases include purely additive or source-compatible updates
    based on our
    [Cross Version Runtime Guarantee](https://protobuf.dev/support/cross-version-runtime-guarantee/#minor).
*   Major releases might remove functionality, features, or change APIs in ways
    that require updates to callsites.

## Support Duration {#duration}

We always support the most recent release. Releasing a new minor version
immediately ends support for the previous minor version. Releasing a major
version ends support for the previous major version four quarters later.

For example, Protobuf Python 5.26.0 launched in Q1 2024. Therefore, Protobuf
Python 4.25.x support ended in [Q1 2025](#python).

The following sections detail support timelines for each language. Future plans
appear in *italics* and might change.

**Legend**

<table class="legend">
  <tr class="active">
    <th>Active</th>
    <td>Minor and patch releases with new features, compatible changes, and bug fixes.</td>
  </tr>
  <tr class="maintenance">
    <th>Maintenance</th>
    <td>Patch releases with critical bug and security vulnerability fixes.</td>
  </tr>
  <tr class="end-of-life">
    <th>End of Life</th>
    <td>Release is unsupported. Users should upgrade to a supported release.</td>
  </tr>
  <tr class="future">
    <th>Future</th>
    <td>Projected release. Shown for planning purposes.</td>
  </tr>
</table>

## C++ {#cpp}

**Release Support Dates**

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
  <tr class="end-of-life">
    <th>4.x</th>
    <td>16 Feb 2023</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="maintenance">
    <th>5.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2026</td>
  </tr>
  <tr class="active">
    <th>6.x</th>
    <td>4 Mar 2025</td>
    <td>31 Mar 2027</td>
  </tr>
  <tr class="future">
    <th>7.x</th>
    <td>25 Feb 2026</td>
    <td>TBD</td>
  </tr>
</table>

**Release Support Chart**

<table class="version-chart">
  <tr>
    <th>Protobuf C++</th>
    <th>protoc</th>
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
    <th class="y26q1"><span>26Q1</span></th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>21.x</td>
    <td class="y23q2 maintenance" colspan=3>3.21</td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1"></td>
  </tr>
  <tr class="end-of-life">
    <th>4.x</th>
    <td>22.x-25.x</td>
    <td class="y23q2 active">4.23</td>
    <td class="y23q3 active">4.24</td>
    <td class="y23q4 active">4.25</td>
    <td class="y24q1 maintenance" colspan=4>4.25</td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1"></td>
  </tr>
  <tr class="maintenance">
    <th>5.x</th>
    <td>26.x-29.x</td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">5.26</td>
    <td class="y24q2 active">5.27</td>
    <td class="y24q3 active">5.28</td>
    <td class="y24q4 active">5.29</td>
    <td class="y25q1 maintenance" colspan=4>5.29</td>
    <td class="y26q1"></td>
  </tr>
  <tr class="active">
    <th>6.x</th>
    <td>30.x-33.x</td>
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
    <td class="y26q1 maintenance">6.33</td>
  </tr>
  <tr class="future">
    <th>7.x</th>
    <td>34.x+</td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1 future">7.34</td>
  </tr>
</table>

### C++ Tooling, Platform, and Library Support {#cpp-tooling}

Protobuf follows the
[Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
For supported versions, see the
[Foundational C++ Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md).

## C&#35; {#csharp}

**Release Support Dates**

<table class="version-table">
  <tr>
    <th>Protobuf C#</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="active">
    <th>3.x</th>
    <td>25 May 2022</td>
    <td>TBD</td>
  </tr>
</table>

**Release Support Chart**

<table class="version-chart">
  <tr>
    <th>Protobuf C#</th>
    <th>protoc</th>
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
    <th class="y26q1"><span>26Q1</span></th>
  </tr>
  <tr class="active">
    <th>3.x</th>
    <td>21.x+</td>
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
    <td class="y26q1 future">3.34</td>
  </tr>
</table>

### C&#35; Platform and Library Support {#csharp-support}

Protobuf is committed to following the platform and library support policy
described in
[.NET Support Policy](https://opensource.google/documentation/policies/dotnet-support).
For specific versions supported, see
[Foundational .NET Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-dotnet-support-matrix.md).

## Java {#java}

**Release Support Dates**

<table class="version-table">
  <tr>
    <th>Protobuf Java</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>25 May 2022</td>
    <td>31 Mar 2027*</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2028</td>
  </tr>
  <tr class="future">
    <th>5.x</th>
    <td>Q1 2027*</td>
    <td>31 Mar 2029</td>
  </tr>
</table>

**Release Support Chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Java</th>
    <th>protoc</th>
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
    <th class="y26q1"><span>26Q1</span></th>
  </tr>
  <tr class="maintenance">
    <th>3.x</th>
    <td>21.x-25.x</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 maintenance" colspan=9>3.25</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>26.x+</td>
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
    <td class="y26q1 future">4.34</td>
  </tr>
</table>

{{% alert title="Note" color="note" %}}
The maintenance support window for the Protobuf Java 3.x release line is 36
months instead of the typical 12
months.{{% /alert %}}

### Java Platform and Library Support {#java-support}

Protobuf follows the
[Java Support Policy](https://cloud.google.com/java/docs/supported-java-versions).
For supported versions, see the
[Foundational Java Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-java-support-matrix.md).

On Android, Protobuf supports the minimum SDK version defined by
[Google Play services](https://developers.google.com/android/guides/setup) or
the default in
[Jetpack](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-main/docs/api_guidelines/modules#module-minsdkversion).
We support whichever version is lower.

## PHP {#php}

**Release Support Dates**

<table class="version-table">
  <tr>
    <th>Protobuf PHP</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>25 May 2022</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2026</td>
  </tr>
  <tr class="future">
    <th>5.x</th>
    <td>25 Feb 2026</td>
    <td>TBD</td>
  </tr>
</table>

**Release Support Chart**

<table class="version-chart">
  <tr>
    <th>Protobuf PHP</th>
    <th>protoc</th>
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
    <th class="y26q1"><span>26Q1</span></th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>21.x-25.x</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 maintenance" colspan=4>3.25</td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1"></td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>26.x - 33.x</td>
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
    <td class="y26q1 maintenance">4.33</td>
  </tr>
    <tr class="future">
    <th>5.x</th>
    <td>34.x+</td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1 future">5.34</td>
  </tr>
</table>

### PHP Platform and Library Support {#php-support}

Protobuf follows the
[PHP Support Policy](https://cloud.google.com/php/getting-started/supported-php-versions).
For supported versions, see the
[Foundational PHP Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-php-support-matrix.md).

## Python {#python}

**Release Support Dates**

<table class="version-table">
  <tr>
    <th>Protobuf Python</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="end-of-life">
    <th>4.x</th>
    <td>25 May 2022</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="maintenance">
    <th>5.x</th>
    <td>13 Mar 2024</td>
    <td>31 Mar 2026</td>
  </tr>
  <tr class="active">
    <th>6.x</th>
    <td>4 Mar 2025</td>
    <td>31 Mar 2027</td>
  </tr>
  <tr class="future">
    <th>7.x</th>
    <td>25 Feb 2026</td>
    <td>TBD</td>
  </tr>
</table>

**Release Support Chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Python</th>
    <th>protoc</th>
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
    <th class="y26q1"><span>26Q1</span></th>
  </tr>
  <tr class="end-of-life">
    <th>4.x</th>
    <td>21.x-25.x</td>
    <td class="y23q2 active">4.23</td>
    <td class="y23q3 active">4.24</td>
    <td class="y23q4 active">4.25</td>
    <td class="y24q1 maintenance" colspan=4>4.25</td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1"></td>
  </tr>
  <tr class="maintenance">
    <th>5.x</th>
    <td>26.x-29.x</td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1 active">5.26</td>
    <td class="y24q2 active">5.27</td>
    <td class="y24q3 active">5.28</td>
    <td class="y24q4 active">5.29</td>
    <td class="y25q1 maintenance" colspan=4>5.29</td>
    <td class="y26q1"></td>
  </tr>
  <tr class="active">
    <th>6.x</th>
    <td>30.x-33.x</td>
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
    <td class="y26q1 maintenance">6.33</td>
  </tr>
    <tr class="future">
    <th>7.x</th>
    <td>34.x+</td>
    <td class="y23q2"></td>
    <td class="y23q3"></td>
    <td class="y23q4"></td>
    <td class="y24q1"></td>
    <td class="y24q2"></td>
    <td class="y24q3"></td>
    <td class="y24q4"></td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1 future">7.34</td>
  </tr>
</table>

### Python Platform and Library Support {#python-support}

Protobuf follows the
[Python Support Policy](https://cloud.google.com/python/docs/supported-python-versions).
For supported versions, see the
[Foundational Python Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-python-support-matrix.md).

## Ruby {#ruby}

**Release Support Dates**

<table class="version-table">
  <tr>
    <th>Protobuf Ruby</th>
    <th>Release date</th>
    <th>End of support</th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>25 May 2022</td>
    <td>31 Mar 2025</td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>13 Mar 2024</td>
    <td>TBD</td>
  </tr>
</table>

**Release Support Chart**

<table class="version-chart">
  <tr>
    <th>Protobuf Ruby</th>
    <th>protoc</th>
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
    <th class="y26q1"><span>26Q1</span></th>
  </tr>
  <tr class="end-of-life">
    <th>3.x</th>
    <td>21.x-25.x</td>
    <td class="y23q2 active">3.23</td>
    <td class="y23q3 active">3.24</td>
    <td class="y23q4 active">3.25</td>
    <td class="y24q1 maintenance" colspan=4>3.25</td>
    <td class="y25q1"></td>
    <td class="y25q2"></td>
    <td class="y25q3"></td>
    <td class="y25q4"></td>
    <td class="y26q1"></td>
  </tr>
  <tr class="active">
    <th>4.x</th>
    <td>26.x+</td>
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
    <td class="y26q1 future">4.34</td>
  </tr>
</table>

### Ruby Platform and Library Support {#ruby-support}

Protobuf follows the
[Ruby Support Policy](https://cloud.google.com/ruby/getting-started/supported-ruby-versions).
For supported versions, see the
[Foundational Ruby Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-ruby-support-matrix.md).

We do not officially support JRuby. However, we provide unofficial, best-effort
support for the latest JRuby version. This targets compatibility with our
minimum supported Ruby version.
