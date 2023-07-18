+++
title = "csharp_names.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/csharp/csharp_names.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::csharp</a></code></p><p>Provides a mechanism for mapping a descriptor to the fully-qualified name of the corresponding C# class. </p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">File Members</h3><div style="font-style: italic; font-weight: normal;">These definitions are not part of any class.</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetFileNamespace"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetFileNamespace</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#GetFileNamespace.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetClassName</b>(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#GetClassName.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetReflectionClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetReflectionClassName</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#GetReflectionClassName.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetOutputFile"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetOutputFile</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor, const std::string file_extension, const bool generate_directories, const std::string base_namespace, std::string * error)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Generates output file name for given file descriptor.  <a href="#GetOutputFile.details">more...</a></div></td></tr></table> <hr><h3 id="GetFileNamespace.details"><code>std::string csharp::GetFileNamespace(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>
<p>Returns: </p>

<pre>The namespace to use for given file descriptor.</pre>
</div> <hr><h3 id="GetClassName.details"><code>std::string csharp::GetClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>The fully-qualified C# class name.</pre>
</div> <hr><h3 id="GetReflectionClassName.details"><code>std::string csharp::GetReflectionClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>The fully-qualified name of the C# class that provides
access to the file descriptor. Proto compiler generates
such class for each .proto file processed.</pre>
</div> <hr><h3 id="GetOutputFile.details"><code>std::string csharp::GetOutputFile(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string file_extension,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const bool generate_directories,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string base_namespace,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::string * error)</code></h3><div style="margin-left: 16px"><p>Generates output file name for given file descriptor. </p><p>If generate_directories is true, the output file will be put under directory corresponding to file's namespace. base_namespace can be used to strip some of the top level directories. E.g. for file with namespace "Bar.Foo" and base_namespace="Bar", the resulting file will be put under directory "Foo" (and not "Bar/Foo").</p>
<p>Requires: </p>
<pre>descriptor != NULL
error != NULL</pre>

<p>Returns: </p>

<pre>The file name to use as output file for given file descriptor. In case
of failure, this function will return empty string and error parameter
will contain the error message.</pre>

</div>
