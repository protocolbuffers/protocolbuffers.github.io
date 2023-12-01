

+++
title = "ruby_generator.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/ruby/ruby_generator.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::ruby</a></code></p><p>Generates Ruby code for a given .proto file. </p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr><tr><td><div><code><a href="#Generator">Generator</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;"><a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> implementation for generated Ruby protocol buffer classes. </div></td></tr></table><h2 id="Generator">class Generator: public <a href="google.protobuf.compiler.code_generator#CodeGenerator">CodeGenerator</a></h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/ruby/ruby_generator.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::ruby</a></code></p><p><a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> implementation for generated Ruby protocol buffer classes. </p><p>If you create your own protocol compiler binary and you want it to support Ruby output, you can do so by registering an instance of this <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> with the <a href='google.protobuf.compiler.command_line_interface#CommandLineInterface'>CommandLineInterface</a> in your main() function. </p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr></table>
