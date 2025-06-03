+++
title = "No Nullable Setters/Getters Support"
weight = 89
description = "Covers why Protobuf doesn't support nullable setters and getters"
type = "docs"
aliases = "/programming-guides/nullable-getters-setters/"
+++

We have heard feedback that some folks would like protobuf to support nullable
getters/setters in their null-friendly language of choice (particularly Kotlin,
C#, and Rust). While this does seem to be a helpful feature for folks using
those languages, the design choice has tradeoffs which have led to the Protobuf
team choosing not to implement them.

Explicit presence is not a concept that directly maps to the traditional notion
of nullability. It is subtle, but explicit presence philosophy is closer to "the
fields are not nullable, but you can detect if the field was explicitly assigned
a value or not. Normal access will see some default value if it is not assigned,
but you can check if the field was actively written to or not, when needed."

The biggest reason not to have nullable fields is the intended behavior of
default values specified in a `.proto` file. By design, calling a getter on an
unset field will return the default value of that field.

**Note:** C# does treat *message* fields as nullable. This inconsistency with
other languages stems from the lack of immutable messages, which makes it
impossible to create shared immutable default instances. Because message fields
can't have defaults, there's no functional problem with this.

As an example, consider this `.proto` file:

```proto
message Msg { optional Child child = 1; }
message Child { optional Grandchild grandchild = 1; }
message Grandchild { optional int32 foo = 1 [default = 72]; }
```

and corresponding Kotlin getters:

```kotlin
// With our API where getters are always non-nullable:
msg.child.grandchild.foo == 72

// With nullable submessages the ?. operator fails to get the default value:
msg?.child?.grandchild?.foo == null

// Or verbosely duplicating the default value at the usage site:
(msg?.child?.grandchild?.foo ?: 72)
```

and corresponding Rust getters:

```rust
// With our API:
msg.child().grandchild().foo()   // == 72

// Where every getter is an Option<T>, verbose and no default observed
msg.child().map(|c| c.grandchild()).map(|gc| gc.foo()) // == Option::None

// For the rare situations where code may want to observe both the presence and
// value of a field, the _opt() accessor which returns a custom Optional type
// can also be used here (the Optional type is similar to Option except can also
// be aware of the default value):
msg.child().grandchild().foo_opt() // Optional::Unset(72)
```

If a nullable getter existed, it would necessarily ignore the user-specified
defaults (to return null instead) which would lead to surprising and
inconsistent behavior. If users of nullable getters want to access the default
value of the field, they would have to write their own custom handling to use
the default if null is returned, which removes the supposed benefit of
cleaner/easier code with null getters.

Similarly, we do not provide nullable setters as the behavior would be
unintuitive. Performing a set and then get would not always give the same value
back, and calling a set would only sometimes affect the has-bit for the field.

Note that message-typed fields are always explicit presence fields (with
hazzers). Proto3 defaults to scalar fields having implicit presence (without
hazzers) unless they are explicitly marked `optional`, while Proto2 does not
support implicit presence. With
[Editions](/editions/features#field_presence), explicit
presence is the default behavior unless an implicit presence feature is used.
With the forward expectation that almost all fields will have explicit presence,
the ergonomic concerns that come with nullable getters are expected to be more
of a concern than they may have been for Proto3 users.

Due to these issues, nullable setters/getters would radically change the way
default values can be used. While we understand the possible utility, we have
decided it’s not worth the inconsistencies and difficulty it introduces.
