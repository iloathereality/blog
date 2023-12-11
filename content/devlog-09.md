+++
title = "Waffle Devlog 9"
date = 2023-12-11
description = "devlog for 2023-12-01—2023-12-11"

[taxonomies] 
tags = ["devlog"]
+++

This is a devlog for 2023-12-01—2023-12-11. This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Meetings

- T-lang Triage meeting 2023-12-06 ([minutes](https://hackmd.io/Jtaa3G0SSsuSpZt897l2iQ))
- T-lang Planning meeting 2023-12-06 ([minutes](https://hackmd.io/jNCHJnQrQGiXwVPAmmdYCg))
    - This was a fun one, in the end we (me and a few people who did not need to leave sooner) stayed in the meeting for I think 1.5 hours more, because we wanted to hummer down a topic @\_@

{% callout() %}
As a reminder: those meetings are open to everyone. (although not necessarily accessible)
{% end %}

# Reviews

- [teloxide/teloxide/981](https://github.com/teloxide/teloxide/pull/981) — Add `#[must_use]` to bot adaptors
- [rust-lang/rust/118504](https://github.com/rust-lang/rust/pull/118504) — Enforce `must_use` on associated types and RPITITs that have a must-use trait in bounds
- [rust-lang/rust/118756](https://github.com/rust-lang/rust/pull/118756) — use bold magenta instead of bold white for highlighting
    - this some real rustc development right here
- [rust-lang/rust/118534](https://github.com/rust-lang/rust/pull/118534) — codegen: panic when trying to compute size/align of extern type

# Code

- [rust-lang/rust/118705](https://github.com/rust-lang/rust/pull/118705) — Change `rustc_codegen_ssa`'s `atomic_cmpxchg` interface to return a pair of values
- [rust-lang/rust-analyzer/16039](https://github.com/rust-lang/rust-analyzer/pull/16039) — fix: Don't emit "missing items" diagnostic for negative impls

# Summary

Small devlog, not much to talk about. Mainly because this one is aligned to end on a week end, but the last one was aligned on a month end, so instead of 2 weeks, you get a bit more than 1.
