+++
title = "Waffle Devlog 2"
date = 2023-04-14
description = "Blah. Things I've done this week (2023-04-10—2023-04-14)."

[taxonomies] 
tags = ["devlog"]
+++

Blah. Things I've done this week (2023-04-10—2023-04-14). This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Tagged Pointer Rewrite

This is my main work of this week — rewriting tagged pointer implementation in `rustc` ([rust/110243]).

To be honest, tagged pointers probably deserve an article of their own, with all the bits, tricks and bit tricks
(please do tell me if you think I should write it/you want to read it).

TL;DR is that "tagged pointers" is a technique of packing a pointer with a few (typically 2-3) bits of additional information (tag).

Since it needs to perform bit tricks on the pointer value,
original implementation used `NonZeroUsize` as the inner type of the tagged pointer type.
This technically works, but is generally a bad idea.
You are **really** not supposed to store pointers as integers.

I've rewritten the implementation using the new ["strict provenance" APIs] and a `NonNull` pointer as the inner type, 
doing other code & documentation improvements along the way.
In my opinion the resulting code is quite nice and cute, I like it.

Along the way I also discovered a missed optimization case in LLVM ([llvm/62093])
and tried to fix it on the rust side using scottmcm's suggestion
([rust/110318], although it looks like this is doing more harm than good, this PR is likely to be closed).
I also learned a lot about tagged pointers and some stuff about tools around LLVM.

Overall, proud of this work!

[rust/110243]: https://github.com/rust-lang/rust/pull/110243
["strict provenance" APIs]: https://doc.rust-lang.org/std/ptr/index.html#provenance
[llvm/62093]: https://github.com/llvm/llvm-project/issues/62093
[rust/110318]: https://github.com/rust-lang/rust/pull/110318

# Sharing bytes

This builds on top of the [yeeting owning ref] from the previous devlog.
Originally I stored `owner: Box<dyn Send + Sync>`, which works, but makes it impossible to clone the "`OwnedSlice`".
This is not the biggest deal, given that you can wrap one `OwnedSlice` in another (and still get the same access performance)...
But some `rustc` uses become nicer with the ability to clone/re-slice an `OwnedSlice`,
so in [rust/110145] I try to support that by storing `owner: Arc<dyn Send + Sync>` instead.
The PR still needs some documentation fixes and minor adjustments, but is mostly done.

[yeeting owning ref]: https://blog.ihatereality.space/devlog-01/#yeet-owningref
[rust/110145]: https://github.com/rust-lang/rust/pull/110145

# Small Stuff

A couple of very small PRs that don't really deserve descriptions:

- [rust/110182](https://github.com/rust-lang/rust/pull/110182) — `itertools::Either` instead of own impl
- [rust/110291](https://github.com/rust-lang/rust/pull/110291) — Implement `Copy` for `LocationDetail`

I was also working on relaxing some bounds on `BTree{Set, Map}`,
but this turned out to be more work than I expected, so I ended up postponing that.

# Stream

The stream started out with some doc changes for tagged pointers (I really wanted to finish them!)
and then I continued work on [rust/110061] which I've been working on for the last 3 streams (uhhh).
Tbh the stream was quite uneventful,
although I've finally made the [rust/110061]'s suggestion work as expected
(it now suggests to change all variable uses, not just the first one)!

This makes me think if the "rustc contribution" format is just bad for streams, or at least one that I can't pull-of.
Maybe I should stop doing them, at least for the time being? idk...

[rust/110061]: https://github.com/rust-lang/rust/pull/110061

# CI

Just so you know, this week CI was bullying me.
I did what feels like a million "fix doc", "fix fmt" and similar commits, UGH. Anyway-

# Reviews

Pull requests I've reviewed in this week:

- [rust/110193](https://github.com/rust-lang/rust/pull/110193) — Check for body owner fallibly in error reporting
- [rust/110315](https://github.com/rust-lang/rust/pull/110315) — Add a stable MIR way to get the main function
- [rust/110313](https://github.com/rust-lang/rust/pull/110313) — allow `repr(align = x)` on inherent methods
-  [rust-lang/rfcs/3407](https://github.com/rust-lang/rfcs/pull/3407) — "Explicit Tail Calls" RFC, I continue trying to review & improve it (again, bit fun of explicit tail calls)

# Summary

Although I didn't do that much work, working on tagged pointers was very fun and overall I'm satisfied with this week. "See" you next week!
