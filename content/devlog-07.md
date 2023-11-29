+++
title = "Waffle Devlog 7"
date = 2023-11-29
description = "devlog for October of 2023"

[taxonomies] 
tags = ["devlog"]
+++

This is a devlog for 2023-10-01—2023-10-31, or, in other words, the month of October. This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# EuroRust

It was in October, huh.This was my second in person conference ever and the first in a pretty long time. And it was **so awesome**. I had the luck to speak to so many cool folks (you know who you are!! <3), some whom I've known for a long time online, some whom I've never known before, some whom I looked up to, but never interacted with... Just. So. Many. Awesome. Folks!

Coincidentally I moved to Europe at the same time which also contributed to the mindblowedness of the experience. This was/is also fun (both in genuine *and* sarcastic ways).

Although I prioritized talked to people and so missed a lot of talks, the ones I've been to were pretty cool. Still need to watch the recordings of the ones I've missed @\_@

Really looking forward to the next EuroRust. Although I should probably look for other Rust events I can go to...

# Meetings

- T-lang triage meeting 2023-10-03 ([minutes](https://hackmd.io/@rust-lang-team/S17WIAKep))
- T-lang planning meeting 2023-10-04 ([minutes](https://hackmd.io/@rust-lang-team/HJmMdMie6))
- T-lang triage meeting 2023-10-18 ([minutes](https://hackmd.io/@rust-lang-team/rJlDv4pb6))
- T-lang design meeting 2023-10-18: consts in patterns ([minutes](https://hackmd.io/mewbxALaR5O2eCQ9cq0pew?view))
    - I'm really hoping we'll be able to "solve" the issue of constants in patterns...
- T-lang triage meeting 2023-10-25 ([minutes](https://hackmd.io/@rust-lang-team/r1qWenSMp))
- T-lang design meeting 2023-10-25: Implementable trait aliases ([minutes](https://hackmd.io/@rust-lang-team/Syx0GQUMT))

{% callout() %}
As a reminder: those meetings are open to everyone.
{% end %}

# Reviews

- [teloxide/teloxide/913](https://github.com/teloxide/teloxide/pull/913) — non-panicking dispatching APIs
- [teloxide/teloxide/946](https://github.com/teloxide/teloxide/pull/946) — Move common fields from `MessageCommon` to `Message`
- [teloxide/teloxide/915](https://github.com/teloxide/teloxide/pull/915) — Chat member update example
- [teloxide/teloxide/877](https://github.com/teloxide/teloxide/pull/877) — Add tracing feature
- [teloxide/teloxide/953](https://github.com/teloxide/teloxide/pull/953) — Fix typos in documentation
- [teloxide/teloxide/954](https://github.com/teloxide/teloxide/pull/954) — Support for the TBA 6.5
- [teloxide/teloxide/959](https://github.com/teloxide/teloxide/pull/959) — Make it usable in WASM
- [rust-lang/rust/116343](https://github.com/rust-lang/rust/pull/116343) — Stop mentioning internal lang items in no_std binary errors
- [rust-lang/rust/116393](https://github.com/rust-lang/rust/pull/116393) — Emit feature gate *warning* for `auto` traits pre-expansion
- [rust-lang/rust/116052](https://github.com/rust-lang/rust/pull/116052) — Add a way to decouple the implementation and the declaration of a TyCtxt method
- [rust-lang/rust/116269](https://github.com/rust-lang/rust/pull/116269) — Bring back generic parameters for indices in rustc_abi and make it compile on stable
- [rust-lang/rust/116198](https://github.com/rust-lang/rust/pull/116198) — Add more diagnostic items for clippy
- [rust-lang/rust/116400](https://github.com/rust-lang/rust/pull/116400) — Detect missing `=>` after match guard during parsing
- [rust-lang/rust/116607](https://github.com/rust-lang/rust/pull/116607) — WIP: `IntoIterator for Box<[T]>` + method dispatch mitigation for editions < 2024
- [rust-lang/rust/116688](https://github.com/rust-lang/rust/pull/116688) — Format all the let-chains in compiler crates
- [rust-lang/rust/107009](https://github.com/rust-lang/rust/pull/107009) — Implement jump threading MIR opt
- [rust-lang/rust/116818](https://github.com/rust-lang/rust/pull/116818) — Stop telling people to submit bugs for internal feature ICEs
- [rust-lang/rust/116829](https://github.com/rust-lang/rust/pull/116829) — Make `#[repr(Rust)]` incompatible with other (non-modifier) representation hints like `C` and `simd`
- [rust-lang/rust/116828](https://github.com/rust-lang/rust/pull/116828) — Begin to abstract `rustc_type_ir` for rust-analyzer
- [rust-lang/rust/117035](https://github.com/rust-lang/rust/pull/117035) — Use MIR slice indexing for `get[unchecked][_mut]` on slices
- [rust-lang/rust/95198](https://github.com/rust-lang/rust/pull/95198) — Add `slice::{split_,}{first,last}_chunk{,_mut}`
- [rust-lang/rust/117200](https://github.com/rust-lang/rust/pull/117200) — Don't add redundant help for object safety violations
- [rust-lang/rust/110837](https://github.com/rust-lang/rust/pull/110837) — Use MIR's `Offset` for pointer `add` too

# Issues

- [tokio-rs/tokio/6048](https://github.com/tokio-rs/tokio/issues/6048) — CI `tests-pass` is likely not working as it should
- [rust-lang/rust-clippy/11610](https://github.com/rust-lang/rust-clippy/issues/11610) — FP with `needless_pass_by_ref_mut`, async function and closures
- [teloxide/teloxide/956](https://github.com/teloxide/teloxide/issues/956) — Figure out how to properly represent `ChatPermissions` #956
- [rust-lang/rust/117186](https://github.com/rust-lang/rust/issues/117186) — Don't show the same help multiple times for object safety violations
    - This is a problem from my PR, but I was too lazy to fix it, so opened an [`E-easy`](https://github.com/rust-lang/rust/labels/E-easy) issue
- [rust-lang/rust-analyzer/15797](https://github.com/rust-lang/rust-analyzer/issues/15797) — Highlight usages of `static mut` as unsafe
    - Turns out this is an issue with the editor theme I'm using, not r-a

# Code

- [teloxide/teloxide/950](https://github.com/teloxide/teloxide/pull/950) — Fix CI
    - Imagine having CI that actually prevents you from merging bad code
- [teloxide/teloxide/951](https://github.com/teloxide/teloxide/pull/951) — Make `ci-pass` job actually fail when some required job failed
    - GitHub actions is somewhat of a joke CI system, given that they don't properly support failing (see the tokio issue above for more info...)
- [rust-lang/rust/116661](https://github.com/rust-lang/rust/pull/116661) — Make "request changes" reviews apply `S-waiting-on-author`
    - It is of note that I made this PR on a phone, while I was at EuroRust
        - It's such a pain to edit stuff on a phone
            - (it was only a config change, so it's fine tho)
- [rust-lang/triagebot/1733](https://github.com/rust-lang/triagebot/pull/1733) — Allow switching prs to waiting on review when author has requested a review from an assignee
    - Little nice addition to the triagebot (see previous devlog for more info about triagebot)
- [rust-lang/rust-forge/704](https://github.com/rust-lang/rust-forge/pull/704) — Add documentation for review request handling feature
- [rust-lang/rust/116776](https://github.com/rust-lang/rust/pull/116776) — Enable `review-requested` feature for rustbot
- [rust-lang/triagebot/1734](https://github.com/rust-lang/triagebot/pull/1734/files) — Allow configuring bot name
- [rust-lang/rust/116399](https://github.com/rust-lang/rust/pull/116399) — Small changes w/ `query::Erase<_>`
- [rust-lang/rust/116401](https://github.com/rust-lang/rust/pull/116401) — Return multiple object-safety violation errors and code improvements to the object-safety check
    - That's the PR that introduced the problem I've opened an issue for and then reviewed the fix for :p
- [rust-lang/rust/116714](https://github.com/rust-lang/rust/pull/116714) — Derive `Ord`, `PartialOrd` and `Hash` for `SocketAddr*`
- [rust-lang/rust/116791](https://github.com/rust-lang/rust/pull/116791) — Allow codegen backends to opt-out of parallel codegen
    - Upstream from my work at my job
    - Sadly this got a lot of merge conflicts that I still haven't figured out how to resolve
- [rust-lang/rust/116793](https://github.com/rust-lang/rust/pull/116793) — Allow targets to override default codegen backend
    - Also an upstream
    - Also is waiting on me figuring out merge conflicts/review questions
- [rust-lang/rust/116762](https://github.com/rust-lang/rust/pull/116762) — Fixup `Atomic*::from_ptr` safety docs

# Summary

Not *that* much for a month for OS contributions, but maybe too much for everything combined for me. Especially the number of reviews, I do not remember doing so many ^^'

I'd say this is a good month, even though it was a hard one.
