//[protobuf-kotlin](/reference/kotlin/api-docs/)/[com.google.protobuf.kotlin](/reference/kotlin/api-docs/protobuf-kotlin/com.google.protobuf.kotlin/)/DslMap

# DslMap

[JVM] class [DslMap]()<[K](), [V](), [P]() :
[DslProxy](../-dsl-proxy/)>constructor(**delegate**:
[Map](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/index.html)<[K](),
[V]()>) :
[Map](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/index.html)<[K](),
[V]()>

A simple wrapper around a
[Map](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/index.html)
with an extra generic parameter that can be used to disambiguate extension
methods.

<p>This class is used by Kotlin protocol buffer extensions, and its constructor
is public only because generated message code is in a different compilation
unit. Others should not use this class directly in any way.

## Constructors

Name | Summary
--- | ---
[DslMap]) | [JVM] fun <[K](), [V]()> [DslMap]()(delegate: [Map](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/index.html)<[K](), [V]()>)

## Functions

Name                                                                                                                                                                                                                        | Summary
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------
<a name="kotlin.collections/Map/containsKey/#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[containsKey](#189495335%2FFunctions%2F-246181541)                                                         | <a name="kotlin.collections/Map/containsKey/#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[JVM] <br>Content <br>open override fun [containsKey](#189495335%2FFunctions%2F-246181541)(key: [K]()): [Boolean](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-boolean/index.html) <br><br><br>
<a name="kotlin.collections/Map/containsValue/#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[containsValue](#-337993863%2FFunctions%2F-246181541)                                                    | <a name="kotlin.collections/Map/containsValue/#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[JVM] <br>Content <br>open override fun [containsValue](#-337993863%2FFunctions%2F-246181541)(value: [V]()): [Boolean](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-boolean/index.html) <br><br><br>
<a name="com.google.protobuf.kotlin/DslMap/equals/#kotlin.Any?/PointingToDeclaration/"></a>[equals](equals)                                                                                                              | <a name="com.google.protobuf.kotlin/DslMap/equals/#kotlin.Any?/PointingToDeclaration/"></a>[JVM] <br>Content <br>open operator override fun [equals](equals)(other: [Any](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/index.html)?): [Boolean](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-boolean/index.html) <br><br><br>
<a name="kotlin.collections/Map/forEach/#java.util.function.BiConsumer[TypeParam(bounds=[kotlin.Any?]),TypeParam(bounds=[kotlin.Any?])]/PointingToDeclaration/"></a>[forEach](#1890068580%2FFunctions%2F-246181541) | <a name="kotlin.collections/Map/forEach/#java.util.function.BiConsumer[TypeParam(bounds=[kotlin.Any?]),TypeParam(bounds=[kotlin.Any?])]/PointingToDeclaration/"></a>[JVM] <br>Content <br>open fun [forEach](#1890068580%2FFunctions%2F-246181541)(p0: [BiConsumer](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiConsumer.html)<in [K](), in [V]()>) <br><br><br>
<a name="kotlin.collections/Map/get/#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[get](#1589144509%2FFunctions%2F-246181541)                                                                        | <a name="kotlin.collections/Map/get/#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[JVM] <br>Content <br>open operator override fun [get](#1589144509%2FFunctions%2F-246181541)(key: [K]()): [V]()? <br><br><br>
<a name="kotlin.collections/Map/getOrDefault/#TypeParam(bounds=[kotlin.Any?])#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[getOrDefault](#1493482850%2FFunctions%2F-246181541)                      | <a name="kotlin.collections/Map/getOrDefault/#TypeParam(bounds=[kotlin.Any?])#TypeParam(bounds=[kotlin.Any?])/PointingToDeclaration/"></a>[JVM] <br>Content <br>open fun [getOrDefault](#1493482850%2FFunctions%2F-246181541)(key: [K](), defaultValue: [V]()): [V]() <br><br><br>
<a name="com.google.protobuf.kotlin/DslMap/hashCode/#/PointingToDeclaration/"></a>[hashCode](hash-code)                                                                                                                  | <a name="com.google.protobuf.kotlin/DslMap/hashCode/#/PointingToDeclaration/"></a>[JVM] <br>Content <br>open override fun [hashCode](hash-code)(): [Int](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-int/index.html) <br><br><br>
<a name="kotlin.collections/Map/isEmpty/#/PointingToDeclaration/"></a>[isEmpty](#-1708477740%2FFunctions%2F-246181541)                                                                                              | <a name="kotlin.collections/Map/isEmpty/#/PointingToDeclaration/"></a>[JVM] <br>Content <br>open override fun [isEmpty](#-1708477740%2FFunctions%2F-246181541)(): [Boolean](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-boolean/index.html) <br><br><br>
<a name="com.google.protobuf.kotlin/DslMap/toString/#/PointingToDeclaration/"></a>[toString](to-string)                                                                                                                  | <a name="com.google.protobuf.kotlin/DslMap/toString/#/PointingToDeclaration/"></a>[JVM] <br>Content <br>open override fun [toString](to-string)(): [String](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/index.html) <br><br><br>

## Properties

Name                                                                                                                                 | Summary
------------------------------------------------------------------------------------------------------------------------------------ | -------
<a name="com.google.protobuf.kotlin/DslMap/entries/#/PointingToDeclaration/"></a>[entries](entries)                               | <a name="com.google.protobuf.kotlin/DslMap/entries/#/PointingToDeclaration/"></a> [JVM] open override val [entries](entries): [Set](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-set/index.html)<[Map.Entry](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/-entry/index.html)<[K](), [V]()>> <br>
<a name="com.google.protobuf.kotlin/DslMap/keys/#/PointingToDeclaration/"></a>[keys](keys)                                        | <a name="com.google.protobuf.kotlin/DslMap/keys/#/PointingToDeclaration/"></a> [JVM] open override val [keys](keys): [Set](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-set/index.html)<[K]()> <br>
<a name="com.google.protobuf.kotlin/DslMap/size/#/PointingToDeclaration/"></a>[size](#-2063973537%2FProperties%2F-246181541) | <a name="com.google.protobuf.kotlin/DslMap/size/#/PointingToDeclaration/"></a> [JVM] open override val [size](#-2063973537%2FProperties%2F-246181541): [Int](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-int/index.html) <br>
<a name="com.google.protobuf.kotlin/DslMap/values/#/PointingToDeclaration/"></a>[values](values)                                  | <a name="com.google.protobuf.kotlin/DslMap/values/#/PointingToDeclaration/"></a> [JVM] open override val [values](values): [Collection](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-collection/index.html)<[V]()> <br>
