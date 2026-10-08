+++
title = "History"
weight = 1020
description = "A brief history behind the creation of Protobuf."
type = "docs"
+++

Understanding why
Protobuf was created and the decisions that changed it over time can help you to
better use the features of the tool.

## Why Did You Release Protobuf? {#why}

There are several reasons that we released Protobuf.

Protobuf is used by many projects inside Google. We had other projects we wanted
to release as open source that use Protobuf, so to do this, we needed to release
Protobuf first. In fact, bits of the technology had already found their way into
the open; if you dig into the code for Google AppEngine, you might find some of
it.

We wanted to provide public APIs that accept Protobuf as well as XML, both
because it is more efficient and because we convert that XML to Protobuf on our
end, anyway.

We thought that people outside Google might find Protobuf useful. Getting
Protobuf into a form we were happy to release was a fun side project.

## Why Is the First Release Version 2? What Happened to Version 1? {#version-1}

The initial version of Protobuf ("Proto1") was developed starting in early 2001
and evolved over the course of many years, sprouting new features whenever
someone needed them and was willing to do the work to create them. Like anything
created in such a way, it was a bit of a mess. We came to the conclusion that it
would not be feasible to release the code as it was.

Version 2 ("Proto2") was a complete rewrite, though it kept most of the design
and used many of the implementation ideas from Proto1. Some features were added,
some removed. Most importantly, though, the code was cleaned up and did not have
any dependencies on Google libraries that were not yet open-sourced.

## Why the Name "Protocol Buffers"? {#name}

The name originates from the early days of the format, before we had the
Protobuf compiler to generate classes for us. At the time, there was a class
called `ProtocolBuffer` that actually acted as a buffer for an individual
method. Users would add tag/value pairs to this buffer individually by calling
methods like `AddValue(tag, value)`. The raw bytes were stored in a buffer that
could then be written out once the message had been constructed.

Since that time, the "buffers" part of the name has lost its meaning, but it is
still the name we use. Today, people usually use the term "protocol message" to
refer to a message in an abstract sense, "protocol buffer" to refer to a
serialized copy of a message, and "protocol message object" to refer to an
in-memory object representing the parsed message.

## What About Proto3 and Editions? {#proto3}

In 2023, we introduced Protobuf Editions to allow for a more granular control
over features and to move away from the "all or nothing" approach of `proto2`
and `proto3`. With Editions, you can now explicitly specify which features you
want to use in your `.proto` files.

For example, to use the 2023 edition, you would add the following to your
`.proto` file:

```proto
edition = "2023";
```

There will be a transitional period during which `proto2`, `proto3`, and
Editions coexist. For more information on Editions, see the
[Protobuf Editions Overview](/editions/overview).

## Does Google Have Any Patents on Protobuf? {#patents}

Google currently has no issued patents on Protobuf, and we are happy to address
any concerns around Protobuf and patents that people may have.

## How Does Protobuf Differ from ASN.1, COM, CORBA, and Thrift? {#differ}

We think all of these systems have strengths and weaknesses. Google relies on
Protobuf internally and it is a vital component of our success, but that doesn't
mean it is the ideal solution for every problem. You should evaluate each
alternative in the context of your own project.

It is worth noting, though, that several of these technologies define both an
interchange format and an RPC (remote procedure call) protocol. Protobuf is just
an interchange format. It could easily be used for RPC&mdash;and, indeed, it
does have limited support for defining RPC services&mdash;but it is not tied to
any one RPC implementation or protocol.
