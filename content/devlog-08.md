+++
title = "Waffle Devlog 8"
date = 2023-11-30
description = "devlog for October of 2023"

[taxonomies] 
tags = ["devlog"]
+++

This is a devlog for 2023-11-01—2023-11-30, or, in other words, the month of November. This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Forbidden function casts RFC

I've written a [small RFC](https://github.com/rust-lang/rfcs/pull/3526)!

To get the full context you should read the RFC, but tl;dr, Rust allows the following:

```rust
fn f() {}
let b = f as u8; // get lower byte of function pointer address??
```

This is not only very cursed, but is also error prone — sometimes it's easy to accidentally write `f as u8` instead of `f() as u8`. My RFC proposes to forbid those kinds of casts, replacing them with more explicit and less error prone options.

{% callout() %}
Why was that ever allowed??
{% end %}

Old Rust is wild sometimes...

# Meetings

Somehow I haven't missed a single T-lang meeting this month!

{% callout() %}
Next step — attend T-compiler meetings (the team you are actually related to)?
{% end %}

...

Anyway, here is the list of meeting:

- T-lang Triage meeting 2023-11-01 ([minutes](https://hackmd.io/@rust-lang-team/HJ8yvyxXa))
- T-lang Planning meeting 2023-11-01 ([minutes](https://hackmd.io/@rust-lang-team/BycoyWlma))
    - Admittedly I didn't do anything there. Note to self: there is no reason to come to planning meetings if you are not T-lang.
- T-lang Triage meeting 2023-11-08 ([minutes](https://hackmd.io/@rust-lang-team/HkkNflY7p))
- T-lang Design meeting 2023-11-08: Minimal TAIT stabilization proposal from T-types (Mini-TAIT) ([minutes](https://hackmd.io/@rust-lang-team/SyJFnSFma))
- T-lang Triage meeting 2023-11-15 ([minutes](https://hackmd.io/@rust-lang-team/BkIjk9Z4a))
- T-lang Design meeting 2023-11-15: 2024 Edition Review ([minutes](https://hackmd.io/@rust-lang-team/BJwSHtM4T))
- T-lang Triage meeting 2023-11-22 ([minutes](https://hackmd.io/@rust-lang-team/S1jBJ6q4p))
- T-lang Triage meeting 2023-11-29 ([minutes](https://hackmd.io/@rust-lang-team/rJu4t_EH6))
- T-lang Design meeting 2023-11-29 “Weak” type aliases in Rust 2024 ([minutes](https://hackmd.io/@rust-lang-team/ByiKGRQrp))
    - The meeting itself was a bit ???, I feel like a lot of the time everyone was just a bit (or maybe a bit more than a bit) lost and didn't know what to do. But in the end we've reached some understanding of the next steps, so it wasn't for nothing. Very excited for type aliases to finally check bounds!


{% callout() %}
As a reminder: those meetings are open to everyone.
{% end %}

# Reviews

- [rust-lang/team/1116](https://github.com/rust-lang/team/pull/1116) — Update `fuchsia.toml` to reflect team changes
- [teloxide/teloxide/971](https://github.com/teloxide/teloxide/pull/971) — Change poll question length
- [teloxide/teloxide/973](https://github.com/teloxide/teloxide/pull/973) — Change setChatTitle title length
- [rust-lang/rust/116988](https://github.com/rust-lang/rust/pull/116988) — document that the null pointer has the 0 address
- [rust-lang/rust/117683](https://github.com/rust-lang/rust/pull/117683) — When encountering struct fn call literal with private fields, suggest all builders
- [rust-lang/rust/118138](https://github.com/rust-lang/rust/pull/118138) — Fixes error count display is different when there's only one error left
    - This PR is merge conflicts as a service
- [rust-lang/rust/118147](https://github.com/rust-lang/rust/pull/118147) — Fix some unnecessary casts
- [rust-lang/rust/118133](https://github.com/rust-lang/rust/pull/118133) — Stabilize RFC3324 dyn upcasting coercion
    - Trait upcasting! Stable! Soon! (2024-02-08)
- [rust-lang/rust/117301](https://github.com/rust-lang/rust/pull/117301) — Call `FileEncoder::finish` in rmeta encoding
- [rust-lang/rust/118358](https://github.com/rust-lang/rust/pull/118358) — make const tests independent of std debug assertions
- [rust-lang/rust/117200](https://github.com/rust-lang/rust/pull/117200) — Don't add redundant help for object safety violations
- [rust-lang/rust/118381](https://github.com/rust-lang/rust/pull/118381) — rustc_span: Use correct edit distance start length for suggestions
    - Unicode never sleeps >:)
- [rust-lang/rust/118397](https://github.com/rust-lang/rust/pull/118397) — Fix comments for unsigned non-zero `checked_add`, `saturating_add`
- [rust-lang/rust/118456](https://github.com/rust-lang/rust/pull/118456) — rustc_span: Remove unused symbols
- [rust-lang/rust/118297](https://github.com/rust-lang/rust/pull/118297) — Merge `unused_tuple_struct_fields` into `dead_code`
    - Got assigned to it right after the T-lang meeting that approved the idea, fun

# Issues

- [teloxide/teloxide/963](https://github.com/teloxide/teloxide/issues/963) — Add `cargo-semver-checks` to CI checks (call for contribution!)
- [rust-lang/rust/117691](https://github.com/rust-lang/rust/issues/117691) — Tracking Issue for convenience methods on `NonNull`
- [matklad/pale-fire/55](https://github.com/matklad/pale-fire/issues/55) — `static mut` is not highlighted as unsafe
- [rust-lang/compiler-team/693](https://github.com/rust-lang/compiler-team/issues/693) — De-llvm some integer intrinsics that on the Rust side always use `u32`

# Code

- [teloxide/teloxide/961](https://github.com/teloxide/teloxide/pull/961) — Support more methods in `DefaultParseMode`
- [teloxide/teloxide/962](https://github.com/teloxide/teloxide/pull/962) — Don't auto-add `C-main` label to all PRs
- [rust-lang/rust/117697](https://github.com/rust-lang/rust/pull/117697) — Non null convenience ops
- [rust-lang/rust/117824](https://github.com/rust-lang/rust/pull/117824) — Stabilize `ptr::{from_ref, from_mut}`
- [rust-lang/rust/118313](https://github.com/rust-lang/rust/pull/118313) — Improve some comments for non-zero ops
- [rust-lang/rust/118314](https://github.com/rust-lang/rust/pull/118314) — Rename `{collections=>alloc}{tests,benches}`
- [rust-lang/rust/118315](https://github.com/rust-lang/rust/pull/118315) — Use `usize::repeat_u8` instead of implementing `repeat_byte` in `memchr.rs`
- [rust-lang/rust/118321](https://github.com/rust-lang/rust/pull/118321) — rustdoc: Remove space from fake-variadic fn ptr impls
- [rust-lang/rust/118326](https://github.com/rust-lang/rust/pull/118326) — Add `NonZero*::count_ones`

# Summary

The backlog is finally cleared, my devlogs are at the present time!

It's somewhat sad I didn't have time/energy to do significant code contributions. I'm trying to change my schedule to do less context switching and have more *consecutive* time working on rustc, so hopefully I'll be back on track soon. (no promises tho)
