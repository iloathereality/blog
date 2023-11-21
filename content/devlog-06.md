+++
title = "Waffle Devlog 6"
date = 2023-11-21
description = "devlog for September of 2023"

[taxonomies] 
tags = ["devlog"]
+++

This is a devlog for 2023-09-01—2023-09-30, or, in other words, the month of September. This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Triage bot

Rust has a github bot account that helps with management of issues and pull requests ([@rustbot](https://github.com/rustbot), [rustc dev guide page](https://rustc-dev-guide.rust-lang.org/rustbot.html), [docs](https://forge.rust-lang.org/triagebot/index.html)). And it's pretty cool actually. Here are some of my favorite features in no particular order:

- `triagebot claim` which allows to assign oneself to an issue, without being a member of the org
- `triagebot label` which allows to add or remove configured labels to issues and PRs
- The bot can autolabel PRs based on changed files
- `triagebot ready`/`triagebot author` that allow you to switch status labels (`S-waiting-on-review`, `S-waiting-on-author`)

The last one is especially handy. Having status labels makes the responsibility clear — "who needs to do work here right now?" becomes a trivial question. Moreover having explicit comments for "this is ready for review" and "I reviewed enough here, do changes" is also nice.

But the other features (including unmentioned here) are also cool, they all either empower contributors (but not members) to do stuff or lift some tedious managing work off of maintainers. Both of those are very appreciated by me.

The nice thing is, triage bot is open source! You can just host your own instance and configure it for your project! This is exactly what I did ([teloxide/teloxide/941](https://github.com/teloxide/teloxide/pull/941)).

{{ 
  image(
      img="teloxidebot_added_tags.png", 
      alt="GitHub event: teloxidebot added C-core, C-main and S-waiting-on-review tags", 
      style="border-radius: 8px; width: 80%;"
      quality=100
  )
}}

Granted, triagebot is currently hardcoded in a lot of ways to the Rust's use cases and the bot, but it's still nice to have a bot like this. (and I'm also trying to un-hardcode it in PRs like [rust-lang/triagebot/1734](https://github.com/rust-lang/triagebot/pull/1734)).

# Meetings

Rust's [language team](https://www.rust-lang.org/governance/teams/lang) has weekly triage and design meetings, which are open. I don't exactly remember how, but in September I remembered/rediscovered that those are indeed open and decided to try attending them. Luckily in the first meeting I attended one of the discussed topics was a PR I nominated for the discussion, so I was able to provide useful context for it. That was quite a positive experience and so I'm now attending the meetings regularly, trying to help T-lang where I can :)

{% callout() %}

As a reminder, this was a few months ago, so details might be slightly wrong here, due to the nature of human memory.

{% end %}

- T-lang Triage meeting 2023-09-12 ([minutes](https://hackmd.io/@rust-lang-team/BkstjGCA3))
- T-lang Triage meeting 2023-09-19 ([minutes](https://hackmd.io/@rust-lang-team/r1Fq-wDyT))
- T-lang Design meeting 2023-09-20: Add f16 and f128 float types RFC read ([minutes](https://hackmd.io/@rust-lang-team/rkSQNidkp))
- T-lang Design meeting 2023-09-27: MaybeDangling RFC read ([minutes](https://hackmd.io/@rust-lang-team/BJ0yJkfe6))

# Other stuff

- [rust-lang/compiler-team/670](https://github.com/rust-lang/compiler-team/issues/670) — Allow overriding default codegen backend on a per-target basis
    - (This is a Major (Change|Compiler) Proposal)
    - This is something we needed at my `{dayjob}` and that we wanted to upstream :)
- [teloxide/teloxide/938](https://github.com/teloxide/teloxide/pull/938) — Improve graceful shutdown
    - This is hilarious, because the graceful shutdown code was written in a very dumb way (I'm allowed to say that, I'm the author), we were wasting **a lot** of time for nothing
    - No really, this was a known issue for years (I think?) and it actually was (relatively) easily fixable all that time
    - (we need to cut a release that includes this PR, ugh...)
- [rust-lang/rust/115547](https://github.com/rust-lang/rust/pull/115547) — Simplify `core::hint::spin_loop`
- [rust-lang/rust/116205](https://github.com/rust-lang/rust/pull/116205) — Stabilize `[const_]pointer_byte_offsets`
    - Hell yeah!
    - `<*const _>::byte_offset` & co coming to stable December 28!
- [rust-lang/rust/116254](https://github.com/rust-lang/rust/pull/116254) — Assorted improvements for `rustc_middle::mir::traversal`
- [rust-lang/rust-analyzer/15609](https://github.com/rust-lang/rust-analyzer/pull/15609) — internal: Remove most of the duplication from `Semantics{,Impl}` via deref
- [rust-lang/futures-rs/2774](https://github.com/rust-lang/futures-rs/pull/2774) — Remove an outdated comment from docs
- [teloxide/teloxide/943](https://github.com/teloxide/teloxide/pull/943) — Update nightly Rust
- [teloxide/teloxide/940](https://github.com/teloxide/teloxide/issues/940) — Add missing `{Update,Message}::filter_*` functions (issue)
    - This is a call for contributions, that is moreover still not implemented/claimed, you can work on it!
- [teloxide/teloxide/939](https://github.com/rust-lang/rust/issues/115939) — Implement support for TBA 6.5 (issue)
- [rust-lang/rust/115939](https://github.com/rust-lang/rust/issues/115939) — Tracking Issue for `cmp::minmax{_by,_by_key}` (issue)
    - Have you ever wanted to sort 2 elements? Now there is a function for that!
    - Seriously though, I think those are nice functions and this actually comes up in real code
    - Also yeah, the implementation PR predates my devlogs (by a week), but it only got merged in September

# Discussions answered

- [teloxide/teloxide/926](https://github.com/teloxide/teloxide/discussions/926) — `teloxide::update_listeners::polling` > Failed to get webhook info
- [teloxide/teloxide/930](https://github.com/teloxide/teloxide/discussions/930) — How to check if user sending message is privileged (the owner or an administrator of the chat)?

# Reviews

- [rust-lang/rust/114845](https://github.com/rust-lang/rust/pull/114845) — Add alignment to the NPO guarantee
- [rust-lang/rust/115345](https://github.com/rust-lang/rust/pull/115345) — MCP661: Move wasm32-wasi-preview1-threads target to Tier 2
- [rust-lang/rust/115624](https://github.com/rust-lang/rust/pull/115624) Print the path of a return-position impl trait in trait when `return_type_notation` is enabled
- [rust-lang/rust/113807](https://github.com/rust-lang/rust/pull/113807) — Tests crash from inappropriate use of common linkage
- [rust-lang/rust/115765](https://github.com/rust-lang/rust/pull/115765) — Add source type for invalid bool casts
- [rust-lang/rust/115810](https://github.com/rust-lang/rust/pull/115810) — Folding comments
- [rust-lang/rust/115860](https://github.com/rust-lang/rust/pull/115860) Enable varargs support for AAPCS calling convention
- [rust-lang/rust/112380](https://github.com/rust-lang/rust/pull/112380) — Add allow-by-default lint for unit bindings
- [rust-lang/rust/115542](https://github.com/rust-lang/rust/pull/115542) — Simplify/Optimize FileEncoder
- [rust-lang/rust/113955](https://github.com/rust-lang/rust/pull/113955) — Pretty-print argument-position impl trait to name it
- [rust-lang/rust/116062](https://github.com/rust-lang/rust/pull/116062) — Change `start` to `#[start]` in some diagnostics
- [rust-lang/rust/116052](https://github.com/rust-lang/rust/pull/116052) — Add a way to decouple the implementation and the declaration of a `TyCtxt` method
- [rust-lang/rust/116067](https://github.com/rust-lang/rust/pull/116067) — Open the FileEncoder file for reading and writing
- [rust-lang/rust/116103](https://github.com/rust-lang/rust/pull/116103) — test: reproduce the issue with the cstr parsing inside a the proc macro
- [rust-lang/rust/116198](https://github.com/rust-lang/rust/pull/116198) — Add more diagnostic items for clippy
- [rust-lang/rust/116239](https://github.com/rust-lang/rust/pull/116239) Only visit reachable nodes in `SsaLocals`
- [rust-lang/rfcs/3336](https://github.com/rust-lang/rfcs/pull/3336) — `MaybeDangling`
- [teloxide/teloxide/923](https://github.com/teloxide/teloxide/pull/923) — Fix hide attr bug work as meta name value str
- [teloxide/teloxide/936](https://github.com/teloxide/teloxide/pull/936) — Fix the type of `photo_size`,`photo_width` and `photo_height` in the `send_invoice` method
- [teloxide/teloxide/861](https://github.com/teloxide/teloxide/pull/861) — Support setting the help message of commands with `/// ...`
- [teloxide/teloxide/917](https://github.com/teloxide/teloxide/pull/917) — Add `MessageToCopyNotFound` to `teloxide::errors::ApiError`
- [teloxide/teloxide/894](https://github.com/teloxide/teloxide/pull/894) — add fast exit with double ctrlc
- [teloxide/teloxide/934](https://github.com/teloxide/teloxide/pull/934) — Activate tokio-tls when using redis-stroage
- [teloxide/teloxide/932](https://github.com/teloxide/teloxide/pull/932) — Implemend `command_separator` attr to split command and args
- [teloxide/teloxide/946](https://github.com/teloxide/teloxide/pull/946) — Move common fields from `MessageCommon` to `Message`

# Summary

This actually makes me feel better, seeing that I actually did stuff (because it felt like I didn't (and also the previous devlog...)).
