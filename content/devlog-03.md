+++
title = "Waffle Devlog 3"
date = 2023-04-21
description = "This week was something else! There is a lot I've done this week (2023-04-17—2023-04-21)."

[taxonomies] 
tags = ["devlog"]
+++


This week was something else! There is a lot I've done this week (2023-04-17—2023-04-21). This is mostly a log for *myself* so that I remember that I did things, but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Assure everyone that `has_type_flags` is fast

This is funny.
So I've been working on some type-flags-adjacent changes,
in particular I've been looking at `TypeVisitableExt::has_non_region_param`.
`has_non_region_param` calls `has_type_flags` from the same trait,
which "visits" `self` with `HasTypeFlagsVisitor`.

At the first glance this looks very suspicious — a visitor normally recursively visits a structure,
but this is not needed here!
For example `Ty` (a type representing types in the compiler) has cached type flags,
so you don't need to visit it to compute them again!
My first though was "how could such inefficient thing be used in the hot path??",
but of course I've just missed something.

`HasTypeFlagsVisitor` doesn't actually recurse all the way down!
If it visits a given type, it doesn't go to the subtypes, it just uses the cached type flags.
So this is in fact quite efficient and totally fine.

It turns out that I'm not the first one to stumble upon this and be confused, so I've made a [PR](https://github.com/rust-lang/rust/pull/110465) that explains all of that in a comment, so that I'm hopefully the last one to be confused by that.

```rust
number_of_people_who_has_tripped_on_this += 1;
```

# No transmutes of `GenericArg` and `Ty`

This was my "quest" of the week.

It all started from me changing how `Ty` is represented, trying to make a small optimization
(yet to be completed, hopefully those changes will see the light of day next week...).
Everything was good, until the compiler sigsegv'ed while compiling some `core`...

At first I thought that my optimization itself was unsound or at least very broken.
Which it kind of was — I did a little mistake, but it was not causing the sigsegv.
Then I learned from `@oli-obk` that we have some places where we transmute `GenericArg` and `Ty`,
which would be very broken for many reasons, including the fact that I changed the `Ty` representation.

Then I noticed in the compiler output, that it crashed inside
`_RNvMs7_NtCs8SrYHSYT6I_12rustc_middle2tyNtB5_2Ty13from_interned` (`rustc_middle::ty::Ty::from_interned`)
which was **very** suspicious, because `from_interned` is pretty much a trivial constructor,
which should not cause sigsegvs.
And then I noticed
`_RNvMse_NtNtCs8SrYHSYT6I_12rustc_middle2ty5substINtNtB7_4list4ListNtB5_10GenericArgE16try_as_type_list` (`rustc_middle::ty::subst::list::List::GenericArg::try_as_type_list`),
which, as you can guess from the name, transmutes between `GenericArg` and `Ty`,
causing problems because I changed `Ty`'s layout...

{% callout() %}
As a side note: even before waffle's changes, this was *technically* unsound,
because both `GenericArg` and `Ty` are `repr(Rust)`, which means that their layout is unstable. Yeah...
{% end %}

This was originally added as a clever optimization, because, well, we can!
`GenericArg` is a tagged pointer that is basically a compressed version of `Ty | Region | Const`
(the types of possible generic arguments — types, lifetimes and constants).
Since we've chosen `Ty`'s tag to be `0b00`, `GenericArg` and `Ty` has the same layout!
In theory this allows us to store, hash and use types and generic args that are also types interchangeably.

In practice however everything is less nice:

1. The transmutes are scattered across the code and are hard to maintain (as shown by my struggles with the sigsegv)
2. This optimization actually doesn't noticeably change the performance and memory usage of the compiler
3. ... While also blocking other optimizations (as, again, shown by my problems)

With all of that in mind I've made [#110496](https://github.com/rust-lang/rust/pull/110496) which removed the transmutes.
Later I also followed it by [#110599](https://github.com/rust-lang/rust/pull/110599) and [#110622](https://github.com/rust-lang/rust/pull/110622) which removed other artifacts of this optimization.

# `deny(unsafe_op_in_unsafe_fn)` in `rustc_data_structures`

This is the continuation of my improvements of tagged pointers, after their cleanup it made sense to deny `unsafe_op_in_unsafe_fn` lint in the crate.

`unsafe_op_in_unsafe_fn` lints against the following code:
```rust
unsafe fn dangerous() {}

unsafe fn something_unsafe() {
    dangerous()
    //^-- this is allowed, by linted by `unsafe_op_in_unsafe_fn`
}

unsafe fn something_unsafe_fixed() {
    unsafe {
        dangerous()
    }
    //^-- this is the fix for the lint
}
```

The reason for this lint's existence is that `unsafe fn` mainly conveys that it's unsafe to call *that* function, but by default it also allows to use other `unsafe` functions inside it, without `unsafe` blocks. This might be a little foot-gun, at it places more code in `unsafe` context than necessary, potentially leading to missed safety requirements. With the lint and `unsafe {}` inside `unsafe` functions everything is more explicit and it's less easy to make mistakes, which is very important for `unsafe` code.

# Closure suggestions finally merged!

The pull request I've been working on on [streams](https://ihatereality.space/devlog-01/#stream) has been finally [merged](https://github.com/rust-lang/rust/pull/110061), yay!

# Add `impl_tag!` macro to implement `Tag` for tagged pointer easily

As the title suggests, this is a [PR](https://github.com/rust-lang/rust/pull/110615) that continues my [work](https://ihatereality.space/devlog-02/#tagged-pointer-rewrite) on tagged pointers in rustc, by adding a macro to implement the `Tag` trait. As a reminder this is basically how `Tag` looks:
```rust
unsafe trait Tag {
    /// The number of bits required
    const BITS: u32;

    fn into_usize(self) -> usize;

    unsafe fn from_usize(tag: usize) -> Self;
}
```
Implementing it correctly is a little bit tricky, since you need to correctly handle all the `unsafe` invariants, so adding a macro seems very nice. At the moment the macro usage looks something like the following, although I'll probably remove the explicit tags later:
```rust
impl_tag! {
    impl Tag for Type;
    Type::A <=> 0,
    Type::B { v: false } <=> 1,
    Type::B { v: true } <=> 2,
}
```

I hope you agree that it's a lot nicer than manually doing the implementation :)

As a side note, while working on this macro I've discovered [#110613](https://github.com/rust-lang/rust/issues/110613) which is annoying bug/feature of how macros and lints work, which was pretty sad. I won't copy the issue description here, so go look at it if you want.

# Other stuff

- [rust/110461](https://github.com/rust-lang/rust/pull/110461) — Use `Item::expect_*` and `ImplItem::expect_*` more
- [rust-lang/libs-team/212](https://github.com/rust-lang/libs-team/issues/212) — `Option::is_none_or` (API change proposal)
- [rust/110545](https://github.com/rust-lang/rust/pull/110545) — Add `GenericArgKind::as_{type,const,region}` 
- [rust/110539](https://github.com/rust-lang/rust/pull/110539) — Move around `{Idx, IndexVec, IndexSlice}` adjacent code
- [rust/110621](https://github.com/rust-lang/rust/pull/110621) — (More) consistently use "region" terminology in `rustc_middle`
- [rust/110451](https://github.com/rust-lang/rust/pull/110451) — Minor changes to `IndexVec::ensure_contains_elem` & related methods
- [teloxide/discussions/872](https://github.com/teloxide/teloxide/discussions/872) — How to add a `tracing::span` to every incoming message?
    - I answered this :>

# Reviews

- [rust/110394](https://github.com/rust-lang/rust/pull/110394) — Various minor Idx-related tweaks
- [rust/110313](https://github.com/rust-lang/rust/pull/110313) — allow `repr(align = x)` on inherent methods
- [rust/106934](https://github.com/rust-lang/rust/pull/106934) — Add offset_of! macro (RFC 3308)
    - very happy with this one in particular, it was very nice to finally approve it!
- [rust/110438](https://github.com/rust-lang/rust/pull/110438) — Allow `transmute`-ing for tuples with `[u8; <const N: usize>]`
    - this seems like a bad idea and so was basically rejected
- [rust/110473](https://github.com/rust-lang/rust/pull/110473) — Assert that global cache values are stable on insertion
- [rust/108106](https://github.com/rust-lang/rust/pull/108106) — Improve niche placement by trying two strategies and picking the better result
- [rust/110548](https://github.com/rust-lang/rust/pull/110548) — Make `impl Debug for Span` not panic on not having session globals.
- [rust/110540](https://github.com/rust-lang/rust/pull/110540) — Fix wrong comment in rustc_hir/src/hir.rs
- [rust/110596](https://github.com/rust-lang/rust/pull/110596) — Rename `wasm32-wasi` to `wasm32-wasi-preview1`

# Bisection

When a regression is found in the rust compiler it is helpful to run a bisection, which allows you to find which exact PR/commit/version introduced the regression. For rustc bisection is usually done via [`cargo bisect-rustc`](https://github.com/rust-lang/cargo-bisect-rustc), which is pretty cool to look at, it runs a binary search, checking the regression and trying to find the place that changes the compilation result or script result, etc (it has multiple modes).

- [rust/110551](https://github.com/rust-lang/rust/issues/110551) — Codegen regression involving `assume`/`unreachable_unchecked`
    - A codegen regression was found, which I helped to bisect/track (although finding the root cause and fixing was done by `@saethlin`)
    - For some reason I had a lot of problems bisecting this one, but I think I was just unlucky with my choice of upper/lower bounds on the bisection
- [110457](https://github.com/rust-lang/rust/issues/110457) — Incremental recompilation MIR ICE

# Stream

This week I did not make the programming stream, and I probably won't make them in the near future, following my [concerns](https://ihatereality.space/devlog-02/#stream) from the previous devlog

# Summary

It feels like I did **a lot** this week, it feels great!!
Hopefully I won't lose *all* that momentum next week, heh.
See you next time 0/
