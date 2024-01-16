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
[SemVer](https://semver.org) conventions; in the version "3.21.7," we
say that "3" is the major version, "21" is the minor version, and "7" is the
micro or patch number.

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

Protobuf does not officially have a release cadence; however, we strive to
release updates quarterly, on a best-effort basis. Our support windows are
defined by our
[library breaking change policy](https://opensource.google/documentation/policies/library-breaking-change).

## Support Duration {#duration}

The most recent release is always supported. Support for earlier minor versions
ends when a new minor version under the same major version is released. Support
for earlier major versions ends four quarters beyond the quarter that the
breaking release is introduced. For example, when Python 4.21.0 was released in
May of 2022, that set the end of public support of Python 3.20.1 at the
[end of 2023 Q2](#python).

The following sections provide a guide to the support for each language.

## C++ {#cpp}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.21.x</td>
    <td>25 May 2022</td>
    <td>31 Mar 2024</td>
  </tr>
  <tr>
    <td class="gray">4.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD, projected to be<br/>31 Mar 2025</td>
  </tr>
</table>

This table graphically shows support durations.

C++ will target making major version bumps annually in Q1 of each year.

<table>
  <tr>
    <th>protoc</th>
    <th>C++</th>
    <th>22Q2</th>
    <th>22Q3</th>
    <th>22Q4</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
  </tr>
  <tr>
    <td class="gray">21.x</td>
    <td class="gray">3.21.x</td>
    <td title=22Q2 class="blue">IR</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1 class="green">PS</td>
    <td title=23Q2 class="green">PS</td>
    <td title=23Q3 class="green">PS</td>
    <td title=23Q4 class="green">PS</td>
    <td title=24Q1 class="red">SE</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">4.22.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">4.23.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=22Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">4.24.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">4.25.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="13">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">5.26.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">5.27.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

### C++ Tooling, Platform, and Library Support {#cpp-tooling}

Protobuf is committed to following the tooling, platform, and library support
policy described in
[Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
For specific versions supported, see
[Foundational C++ Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md).

## C&#35; {#csharp}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD</td>
  </tr>
</table>

This table graphically shows support durations.

<table>
  <tr>
    <th>protoc</th>
    <th>C#</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

### C&#35; Platform and Library Support {#csharp-support}

Protobuf is committed to following the platform and library support policy
described in
[.NET Support Policy](https://opensource.google/documentation/policies/dotnet-support).

## Java {#java}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.19.x</td>
    <td class="gray">20 Oct 2021</td>
    <td class="gray"><s>25 Mar 2022</s></td>
  </tr>
  <tr>
    <td class="gray">3.20.x</td>
    <td class="gray">25 Mar 2022</td>
    <td class="gray"><s>25 May 2022</s></td>
  </tr>
  <tr>
    <td class="gray">3.21.x</td>
    <td class="gray">25 May 2022</td>
    <td class="gray"><s>16 Feb 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD, projected to be<br/>31 Mar 2025</td>
  </tr>
</table>

This table graphically shows support durations.

Java will target making major version bumps annually in Q1 of each year.

<table>
  <tr>
    <th>protoc</th>
    <th>Java</th>
    <th>22Q2</th>
    <th>22Q3</th>
    <th>22Q4</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
  </tr>
  <tr>
    <td class="gray">21.x</td>
    <td class="gray">3.21.x</td>
    <td title=22Q2 class="blue">IR</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4>
    <td title=24Q1>
    <td title=24Q2>
    <td title=24Q3>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
    <td title=24Q3 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">4.26.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">4.27.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3></td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

## Objective-C {#objc}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD</td>
  </tr>
</table>

This table graphically shows support durations.

<table>
  <tr>
    <th>protoc</th>
    <th>ObjC</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

## PHP {#php}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD, projected to be<br/>31 Mar 2025</td>
  </tr>
</table>

This table graphically shows support durations.

<table>
  <tr>
    <th>protoc</th>
    <th>PHP</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">4.26.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">4.27.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

### PHP Platform and Library Support {#php-support}

Protobuf is committed to following the platform and library support policy
described in
[PHP Support Policy](https://cloud.google.com/php/getting-started/supported-php-versions).
For specific versions supported, see
[Foundational PHP Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-php-support-matrix.md).

## Python {#python}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.20.x</td>
    <td class="gray">25 Mar 2022</td>
    <td class="gray"><s>30 Jun 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.21.x</td>
    <td class="gray">25 May 2022</td>
    <td class="gray"><s>16 Feb 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">4.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD, projected to be<br/>31 Mar 2025</td>
  </tr>
</table>

This table graphically shows support durations.

<table>
  <tr>
    <th>protoc</th>
    <th>Python</th>
    <th>22Q3</th>
    <th>22Q4</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
  </tr>
  <tr>
    <td class="gray">20.x</td>
    <td class="gray">3.20.x</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1 class="green">PS</td>
    <td title=23Q2 class="red">SE</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">21.x</td>
    <td class="gray">4.21.x</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">4.22.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">4.23.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">4.24.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">4.25.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">5.26.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">5.27.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

### Python Platform and Library Support {#python-support}

Protobuf is committed to following the platform and library support policy
described in
[Python Support Policy](https://cloud.google.com/python/docs/supported-python-versions).
For specific versions supported, see
[Foundational Python Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-python-support-matrix.md).

## Ruby {#ruby}

This table provides specific dates for support duration.

<table>
  <tr>
    <th>Branch</th>
    <th>Initial Release</th>
    <th>Public Support Until</th>
  </tr>
  <tr>
    <td class="gray">3.21.x</td>
    <td class="gray">25 May 2022</td>
    <td class="gray"><s>16 Feb 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">16 Feb 2023</td>
    <td class="gray"><s>8 May 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">8 May 2023</td>
    <td class="gray"><s>8 Aug 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">8 Aug 2023</td>
    <td class="gray"><s>1 Nov 2023</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>1 Nov 2023</td>
    <td>TBD, projected to be<br/>31 Mar 2025</td>
  </tr>
</table>

This table graphically shows support durations.

<table>
  <tr>
    <th>protoc</th>
    <th>Ruby</th>
    <th>22Q2</th>
    <th>22Q3</th>
    <th>22Q4</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
  </tr>
  <tr>
    <td class="gray">21.x</td>
    <td class="gray">3.21.x</td>
    <td title=22Q2 class="blue">IR</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="10">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">4.26.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">4.27.x</td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>Legend</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      Initial release (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      Public support (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      Support ends (SE)
    </td>
  </tr>
</table>

### Ruby Platform and Library Support {#ruby-support}

Protobuf is committed to following the platform and library support policy
described in
[Ruby Support Policy](https://cloud.google.com/ruby/getting-started/supported-ruby-versions).
For specific versions supported, see
[Foundational Ruby Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-ruby-support-matrix.md).
