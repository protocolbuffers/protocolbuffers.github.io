

+++
title = "type_resolver.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/util/type_resolver.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>Defines a TypeResolver for the Any message. </p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr><tr><td><div><code><a href="#TypeResolver">TypeResolver</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Abstract interface for a type resolver. </div></td></tr></table><h2 id="TypeResolver">class TypeResolver</h2><p><code>#include &lt;<a href="#">google/protobuf/util/type_resolver.h</a>&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>Abstract interface for a type resolver. </p><p>Implementations of this interface must be thread-safe. </p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="TypeResolver.TypeResolver"><div style="padding-left: 16px; text-indent: -16px"><code><b>TypeResolver</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual </code></td><td style="border-left-width: 0px"id="TypeResolver.~TypeResolver"><div style="padding-left: 16px; text-indent: -16px"><code><b>~TypeResolver</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual util::Status</code></td><td style="border-left-width: 0px"id="TypeResolver.ResolveMessageType"><div style="padding-left: 16px; text-indent: -16px"><code><b>ResolveMessageType</b>(const std::string &amp; type_url, google::protobuf::Type * message_type)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Resolves a type url for a message type. </div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual util::Status</code></td><td style="border-left-width: 0px"id="TypeResolver.ResolveEnumType"><div style="padding-left: 16px; text-indent: -16px"><code><b>ResolveEnumType</b>(const std::string &amp; type_url, google::protobuf::Enum * enum_type)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Resolves a type url for an enum type. </div></td></tr></table>
