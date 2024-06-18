+++
title = "plugin.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/plugin.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>Front-end for protoc code generator plugins written in C++. </p><p>To implement a protoc plugin in C++, simply write an implementation of <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a>, then create a main() function like: </p>

<pre>int main(int argc, char* argv[]) {
  MyCodeGenerator generator;
  return google::protobuf::compiler::PluginMain(argc, argv, &amp;generator);
}</pre>

<p> You must link your plugin against libprotobuf and libprotoc.</p>

<p>The core part of PluginMain is to invoke the given <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> on a CodeGeneratorRequest to generate a CodeGeneratorResponse. This part is abstracted out and made into function GenerateCode so that it can be reused, for example, to implement a variant of PluginMain that does some preprocessing on the input CodeGeneratorRequest before feeding the request to the given code generator.</p>

<p>To get protoc to use the plugin, do one of the following:</p>

<ul>
  <li>Place the plugin binary somewhere in the PATH and give it the name "protoc-gen-NAME" (replacing "NAME" with the name of your plugin). If you then invoke protoc with the parameter &ndash;NAME_out=OUT_DIR (again, replace "NAME" with your plugin's name), protoc will invoke your plugin to generate the output, which will be placed in OUT_DIR.</li>
  <li>
    <p>Place the plugin binary anywhere, with any name, and pass the &ndash;plugin parameter to protoc to direct it to your plugin like so: </p>
<pre>protoc --plugin=protoc-gen-NAME=path/to/mybinary --NAME_out=OUT_DIR</pre>
    <p> On Windows, make sure to include the .exe suffix: </p>
<pre>protoc --plugin=protoc-gen-NAME=path/to/mybinary.exe --NAME_out=OUT_DIR</pre>
  </li>
</ul>

<table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">File Members</h3><div style="font-style: italic; font-weight: normal;">These definitions are not part of any class.</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>int</code></td><td style="border-left-width: 0px"id="PluginMain"><div style="padding-left: 16px; text-indent: -16px"><code><b>PluginMain</b>(int argc, char * argv, const <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Implements main() for a protoc plugin exposing the given code generator. </div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>bool</code></td><td style="border-left-width: 0px"id="GenerateCode"><div style="padding-left: 16px; text-indent: -16px"><code><b>GenerateCode</b>(const CodeGeneratorRequest &amp; request, const <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> &amp; generator, CodeGeneratorResponse * response, std::string * error_msg)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Generates code using the given code generator.  <a href="#GenerateCode.details">more...</a></div></td></tr></table> <hr><h3 id="GenerateCode.details"><code>bool compiler::GenerateCode(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const CodeGeneratorRequest &amp; request,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> &amp; generator,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CodeGeneratorResponse * response,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::string * error_msg)</code></h3><div style="margin-left: 16px"><p>Generates code using the given code generator. </p><p>Returns true if the code generation is successful. If the code generation fails, error_msg may be populated to describe the failure cause. </p>
</div>
