

+++
title = "type_resolver_util.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/util/type_resolver_util.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>Defines utilities for the TypeResolver. </p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">File Members</h3><div style="font-style: italic; font-weight: normal;">These definitions are not part of any class.</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code><a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> *</code></td><td style="border-left-width: 0px"id="NewTypeResolverForDescriptorPool"><div style="padding-left: 16px; text-indent: -16px"><code><b>NewTypeResolverForDescriptorPool</b>(const std::string &amp; url_prefix, const <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> * pool)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Creates a <a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> that serves type information in the given descriptor pool.  <a href="#NewTypeResolverForDescriptorPool.details">more...</a></div></td></tr></table> <hr><h3 id="NewTypeResolverForDescriptorPool.details"><code><a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * util::NewTypeResolverForDescriptorPool(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; url_prefix,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> * pool)</code></h3><div style="margin-left: 16px"><p>Creates a <a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> that serves type information in the given descriptor pool. </p><p>Caller takes ownership of the returned <a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a>. </p>

</div>
