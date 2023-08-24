+++
title = "Waffle Devlog 4"
date = 2023-08-24
description = "..."

[taxonomies] 
tags = ["devlog"]
+++

The devlog is back, now as a ~~bi-weekly~~ hopefully regular thing. This is a devlog for 2023-07-17—2023-08-06. This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Refactor vtable encoding and optimize it for the case of multiple marker traits 

{% callout() %}

Sooo, originally there was meant to be an explanation for [this](https://github.com/rust-lang/rust/pull/113856) pull request. However, explaining the PR requires quite a lot of knowledge about trait objects/vtables, which not that many people have. The explanation including trait objects/vtables basics was taking too much time to write, so it'll have to wait for another day. When the article about vtables will be ready (soon™), a link will be added here.

{% end %}

The TL;DR is: the layout of vtables was suboptimal in some specific, yet relatively common, cases. I refactored the surrounding code and optimized the layout, rainbows and unicorns.

In a similar vain I'm also not sure I can properly explain [#114007](https://github.com/rust-lang/rust/issues/114007), but TL;DR is that Rust vtables currently include methods that can't be called. I have a fix ([#114260](https://github.com/rust-lang/rust/pull/114260)), but it turns out to be non-trivial for *reasons*.

# Rust Foundation Fellowship

This year I again received Fellowship grant from Rust Foundation, you can read more in their blog post: [Announcing the Rust Foundation’s 2023 Fellows](https://foundation.rust-lang.org/news/announcing-the-rust-foundation-s-2023-fellows/).

It's a shame Rust Foundation did not have enough funding to pay fellowships for even the number of people who had them last year. I know a lot of people working hard on Rust, who deserve a grant more than anyone else.

# Explicit Tail Calls™ experiment update

I didn't do much on tail calls in this time frame, but a lot has happened in between devlogs (I couldn't make myself write them :( ), so this deserves a mention.

There is a `T-lang` approved **experiment** at implementing Explicit Tail Calls™ in Rust, based on draft Rust RFC [#3407](https://github.com/rust-lang/rfcs/pull/3407). The experiment is lead by myself, by which I mean "I'm implementing explicit tail calls in rust lol".

The tracking issue for the experiment is [#112788](https://github.com/rust-lang/rust/issues/112788), you can find all the PRs opened so far there. Current status/roadmap is:

- Parse explicit tail call expressions — **done**
- Expose llvm's `setTailCallKind` function to rustc — **done**
- Add tail calls to HIR (high-level intermediate representation) — **done**
- Add tail calls to THIR (typed high-level intermediate representation) — **done**
- Write documentation for `become` keyword — *PR opened*
- Support tail calls in MIR (middle-level intermediate representation) — *PR opened*
    - This is the level that, in a way, describes semantics, as such this is the first step that is actually "useful", but at the same time it's the first step that is actually hard
    - CTFE and MIRI work on MIR, so this PR should allow using tail calls in `const` functions and when using MIRI
    - There is quite a bit of work I need to do here — reviewers have found quite a few things I need to change
- Check tail calls — most changes are done, but a PR is not opened yet
    - This will add checks like "signatures of caller and callee must match exactly", and similar ones that allow us to *guarantee* tail calls without loosing something else
- Properly tell LLVM about tail calls — most changes are done, but this is blocked on MIR support

# Other stuff

- [rust-lang/rust/113832](https://github.com/rust-lang/rust/pull/113832) — Add `#[track_caller]` to lint related diagnostic functions
- [teloxide/teloxide/906](https://github.com/teloxide/teloxide/pull/906) — Make `mentioned_users` somewhat less terrible
    - This refactors code so that it uses `Either<Either<I0, I1>, Either<I2, I3>>` instead of `Either<I0, Either<I1, Either<I2, I3>`
    - Interestingly this likely does not matter (or is even a pessimization), as `rustc` can squash all enums' discriminants into the same either way ([play](https://play.rust-lang.org/?version=nightly&mode=debug&edition=2021&gist=bdeaf31f41cbe026340099ac773a61c7))
- [teloxide/teloxide/905](https://github.com/teloxide/teloxide/pull/905/files) — Assorted user/chat id additions
- [rust-lang/rust/114258](https://github.com/rust-lang/rust/pull/114258) — Simplify `Span::can_be_used_for_suggestions` a little tiny bit
- [rust-lang/rust/112655](https://github.com/rust-lang/rust/pull/112655) — Mark `map_or` as `#[must_use]`

# Reviews

- [rust-lang/rust/112953](https://github.com/rust-lang/rust/pull/112953) — Support interpolated block for `try` and `async`
- [rust-lang/rust/113993](https://github.com/rust-lang/rust/pull/113993) — Optimize format usage (specifically this removes the reference `&format!(..)` for cases where we convert to `String` later anyway)
- [rust-lang/rust/114052](https://github.com/rust-lang/rust/pull/114052) — Suggest `{Option,Result}::as_ref()` instead of `cloned()` in some cases
    - This diagnostic improvement got reverted in [114931](https://github.com/rust-lang/rust/pull/114931) because it made invalid suggestions :(
- [rust-lang/rust/114150](https://github.com/rust-lang/rust/pull/114150) — Refactor + improve diagnostics for `&mut T`/`T` mismatch inside `Option`/`Result`
    - Interestingly this was a followup to the previous PR, but did not get reverted
- [rust-lang/rust/114066](https://github.com/rust-lang/rust/pull/114066), [rust-lang/rust/114068](https://github.com/rust-lang/rust/pull/114068), [rust-lang/rust/114074](https://github.com/rust-lang/rust/pull/114074), [rust-lang/rust/114075](https://github.com/rust-lang/rust/pull/114075) — a few PRs that use newer `format!("{x}")` syntax instead of the older `format!("{}", x)` (this was automated using `clippy`)
- [rust-lang/rust/114123](https://github.com/rust-lang/rust/pull/114123) — Turns out opaque types can have hidden types registered during mir validation
- [rust-lang/rust/114152](https://github.com/rust-lang/rust/pull/114152) — \[rustc]\[data_structures] Simplify binary_search_slice
    - (rustc hand-rolls its own binary search impl to guarantee position in case there are multiple elements with the same value)
- [rust-lang/rust/114201](https://github.com/rust-lang/rust/pull/114201) — Allow explicit `#[repr(Rust)]`
    - `#[repr(Rust)]` is often used in documentation/blog post/discussions/etc to name the default unspecified layout of types
    - *However*, you currently can't explicitly write `#[repr(Rust)]`, only omit `repr` and use the default
    - This PR fixes that! Thanks, Centri :3
- [teloxide/teloxide/864](https://github.com/teloxide/teloxide/pull/864) — Telegram struct serializing similar to original (skip empty/defaults)
- [teloxide/teloxide/915](https://github.com/teloxide/teloxide/pull/915) (and its previous versions...) — Chat member update example
- [rust-lang/rust/114256](https://github.com/rust-lang/rust/pull/114256) — Fix invalid suggestion for mismatched types in closure arguments

# Summary

I haven't done much in this period (note that this devlog describes more than a week), still adopting to working on a real work™ and contributing to open source in free time/a set part of the work time. Additionally I'm having some issues in real life™ and troubles with the brain™, so yeah... A lot of stuff happening, which makes it harder to spend time on Rust. Hopefully with some changes to my routine, sleep schedule & so on, I'll be able to do more stuff, spending less energy, idk.

Aaaaanyway, till next devlog, bye
