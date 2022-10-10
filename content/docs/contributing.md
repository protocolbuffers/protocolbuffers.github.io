---
title: "Contributing"
weight: 1030
toc_hide: false
linkTitle: "Contributing"
no_list: "true"
type: docs
---

If you're interested in contributing to the protocol buffers project, read about
the following scenarios.

## Can I Add Support for a New Language to Protocol Buffers? {#contribute-language}

Yes! In fact, the protocol buffer compiler is designed such that it's easy to
write your own compiler. Check out the `CommandLineInterface` class, which is
available as part of the `libprotoc` library.

We encourage you to create code generators and runtime libraries for new
languages. You should start your own, independent project for this&mdash;this
way, you will have the freedom to manage your project as you see fit, and will
not be held back by our release process. Join the
[protocol buffers discussion group](http://groups.google.com/group/protobuf) and
let us know about your project; we will be happy to link to it and help you out
with design issues.

## Can I Contribute Patches to Protocol Buffers? {#patches}

Yes! Join the
[protocol buffers discussion group](http://groups.google.com/group/protobuf) and
talk to us about it.

## Can I Add New Features to Protocol Buffers? {#new-features}

Maybe. We always like suggestions, but we're very cautious about adding things.
One thing we've learned over the years is that lots of people have interesting
ideas for new features. Most of these features are very useful in specific
cases, but if we accepted all of them, protocol buffers would become a bloated,
confusing mess. So, we have to be very picky. When evaluating new features, we
look for additions that are very widely useful or very simple&mdash;or hopefully
both. We regularly turn down feature additions from Google employees. We even
regularly turn down feature additions from our own team members.

That said, we'd still like to hear what you have in mind. Join the
[protocol buffers discussion group](http://groups.google.com/group/protobuf) and
let us know. We might be able to help you find a way to do what you want without
changing the underlying library. Or, maybe we'll decide that your feature is so
useful or so simple that it should be added.
