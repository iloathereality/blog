+++
title = "Waffle Devlog 5"
date = 2023-11-20
description = "devlog for 2023-08-07—2023-08-31"

[taxonomies] 
tags = ["devlog"]
+++

This is a devlog for 2023-08-07—2023-08-31. This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

{% callout() %}

Will you really ignore the fact that this devlog is \[*checks date*\] 4 months late?

{% end %}

# Stuff

- [mikaelmello/inquire/171](https://github.com/mikaelmello/inquire/pull/171) — Don't `.reserve()` without a need
- [yuankunzhang/charming/20](https://github.com/yuankunzhang/charming/issues/20) — `ImageRenderer` only renders the first 400 points in a scatter graph (issue)
    - This was very sad to discover, I wanted to use the library :(
    - (I ended up procrastinating so much, the thing I wanted to make is not relevant anymore)
- [rust-lang/rust/114942](https://github.com/rust-lang/rust/issues/114942) — Imperfect vtable layout with an empty super trait comming after non-empty one (issue)
- [rust-lang/rust/116124](https://github.com/rust-lang/rust/pull/116124) — Properly print cstr literals in `proc_macro::Literal::to_string`

# Reviews

- [rust-lang/rfcs/3467](https://github.com/rust-lang/rfcs/pull/3467) — UnsafePinned: allow aliasing of pinned mutable references
- [teloxide/teloxide/913](https://github.com/teloxide/teloxide/pull/913)— Add versions of `.dispatch()` that don't panic on errors
- [teloxide/teloxide/907](https://github.com/teloxide/teloxide/pull/907) — Add an example of working with chat member updates
- [rust-lang/rust/115210](https://github.com/rust-lang/rust/pull/115210) — Make `rustc_on_unimplemented` std-agnostic for `alloc::rc`
- [rust-lang/rust/115258](https://github.com/rust-lang/rust/pull/115258) — (I will not describe this PR, as it is not a genuine contribution...)
- [rust-lang/rust/115260](https://github.com/rust-lang/rust/pull/115260) — Use `preserve_mostcc` for `extern "rust-cold"`
- [rust-lang/rust/115319](https://github.com/rust-lang/rust/pull/115319) don't use `SnapshotVec` in `Graph` implementation, as it looks unused; use `Vec` instead

# Summary

To be quite honest with you, `{reader}`, I do not remember what happened in August. I have almost no idea why I've done so little. I had a bit of a vacation at the start, but still... Maybe I also had hard time adapting to my (at the time) new job?

Either way it doesn't feel good to say "yeah, I didn't do much", but this is the truth, so here we go.
