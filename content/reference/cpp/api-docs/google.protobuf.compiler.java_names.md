+++
title = "java_names.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/java/java_names.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::java</a></code></p><p>Provides a mechanism for mapping a descriptor to the fully-qualified name of the corresponding Java class. </p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">File Members</h3><div style="font-style: italic; font-weight: normal;">These definitions are not part of any class.</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#ClassName.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#EnumDescriptor'>EnumDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#ClassName.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#ClassName.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#ServiceDescriptor'>ServiceDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#ClassName.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="FileJavaPackage"><div style="padding-left: 16px; text-indent: -16px"><code><b>FileJavaPackage</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#FileJavaPackage.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="CapitalizedFieldName"><div style="padding-left: 16px; text-indent: -16px"><code><b>CapitalizedFieldName</b>(const <a href='google.protobuf.descriptor#FieldDescriptor'>FieldDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Requires:  <a href="#CapitalizedFieldName.details">more...</a></div></td></tr></table> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>
<p>Returns: </p>

<pre>The fully-qualified Java class name.</pre>
</div> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#EnumDescriptor'>EnumDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>The fully-qualified Java class name.</pre>
</div> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>The fully-qualified Java class name.</pre>
</div> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#ServiceDescriptor'>ServiceDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>The fully-qualified Java class name.</pre>
</div> <hr><h3 id="FileJavaPackage.details"><code>std::string java::FileJavaPackage(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>Java package name.</pre>
</div> <hr><h3 id="CapitalizedFieldName.details"><code>std::string java::CapitalizedFieldName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FieldDescriptor'>FieldDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p> Returns: </p>

<pre>Capitalized camel case name field name.</pre>

</div>
