+++
title = "Waffle Devlog 1"
date = 2023-04-07
description = "Is this week already over? That fast? In this economy? Anyway-"

[taxonomies] 
tags = ["devlog"]
+++

Is this week already over? That fast? In this economy?
Anyway, things I've done this week (2023-04-03—2023-04-07).
This is mostly a log for *myself* so that I remember that I did things™,
but you may be interested in reading it too, for whatever reason.

<!-- more -->

# Yeet `OwningRef`

Originally I wanted to cleanup documentation of `OwningRef` inside `rustc_data_structures` (filling [#109948]).
But later I was informed that `OwningRef` is actually unsound and is even used unsoundly in the compiler...
So I tried fixing that instead.

{% callout() %}
What even *is* `OwningRef`? 
{% end %}

`OwningRef` is the "solution" to the problem of returning data borrowed from the current function:

```rust
fn f() -> (Vec<u8>, &'??? [u8]) {
    let vec = vec![0, 1, 2, 3, 4];
    (vec, &vec[1..][..3]) 
    //    ^-- error: cannot return value referencing local variable `vec`
}
```

With owning ref you can do something like this:

```rust
fn a() -> OwningRef<Vec<u8>, [u8]> {
    let vec = vec![0, 1, 2, 3, 4];
    let or = OwningRef::new(vec);
    let or = or.map(|v| &v[1..3]);
    or
}
```

Under the hood `OwningRef<Owner, View>` asserts that `Owner`'s `Deref` implementation is "stable"
(meaning it returns a reference with the same address if the owner value does not change)
and stores a pointer alongside with the owner (basically the same `Vec<u8>, &'??? [u8]` tuple). 
Given that `OwningRef` doesn't allow you to access the owner, it looks all good — without access to the owner,
you can't invalidate the pointer, so it's fine to use it.

But there is a nuance... In some cases you *can* invalidate the pointer without touching the owner.

{% callout() %}
How? This sounds weird...
{% end %}

Well... Under the current understanding of Rust semantics, `Box` is a **unique** owning pointer, which means that it can't be aliased. Moving the `Box` asserts this invariant, [invalidating the pointer]:

```rust
let owner = Box::new(1);
let ptr: *const u32 = &*owner;

// move of `owner` (similarly to a move inside `OwningRef::new`)
// this invalidates `ptr`
let owning = (owner, ptr);

// Observe the pointer being invalid (miri reports UB)
unsafe { dbg!(*owning.1) };
```

So, while `Box` has a stable `Deref`, it turns out that this is not sufficient for `OwningRef` soundness.
Moreover it turns out that all rustc's uses of `OwningRef` use `Box` (to erase the owner type).

There are a few possible solutions,
but in the end I decided to implement a different specialized abstraction instead of trying to fix `OwningRef`.

To fix the box issue, I box the owner *again* first, so that the box I'm getting the pointer from, doesn't move and [doesn't invalidate the pointer]:

```rust
let owner = Box::new(1);

let boxed: Box<Box<u32>> = Box::new(owner);

let ptr: *const u32 = &**boxed;

// move of `boxed` doesn't move the owner,
// so it doesn't invalidate the pointer
let owning = (boxed, ptr);

// Observe the pointer being valid (miri does not report UB)
unsafe { dbg!(*owning.1) };
```

This design doesn't even require stable `Deref`!
By using HRTB bound for the function that gets the reference from the owner,
we can pretend that we are giving it a reference that lives as long as the `OwnedSlice`:

```rust
pub fn slice_owned<O, F>(owner: O, slicer: F) -> OwnedSlice
where
    // This is the magic bound,
    // this function must be valid for all lifetimes,
    // so it can only return borrows from the argument,
    // or 'static data (which is also fine)
    F: for<'a> FnOnce(&'a O) -> &'a [u8],

    // `OwnedSlice` has `Box<dyn Send + Sync>` internally
    // to erase types and stay thread-safe
    O: Send + Sync + 'static,
```

This is perfectly valid, unless we access the owner somehow. The result can be seen in this PR: [#109971].

[#109948]: https://github.com/rust-lang/rust/pull/109948
[invalidating the pointer]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=50ba5fb85fbc7f7033665247f8b44087
[doesn't invalidate the pointer]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=a399f56dd98a2ce36a241b631c274d42
[#109971]: https://github.com/rust-lang/rust/pull/109971

# `Sync` in `reference` docs

Turns out we forgot to document that references implement `Sync` if their pointee implements `Sync`. [Oops].

[Oops]: https://github.com/rust-lang/rust/pull/110060

# Reviews

Pull requests I've reviewed in this week:

- [rust/109843] — Improve `transmute` codegen
- [rust/109819] — Use `&IndexSlice` in place of `&IndexVec` (similarly to normal slices/vecs)
- [rust/109913] — Document `IndexVec::from_elem`
- [rust/110013] — Label `non_exhaustive` attribute on privacy errors from non-local items
    - A funny thing happened here, actually. Mark started a perf run (even though this PR only edits error path, so it can't change perf...), I then approved with   `r=me` and Estedan did `@bors r=WaffleLapkin`, forgetting about a bug in bors... Which makes it so PR approved while a try build is building (which is required for perf run) is insta-merged, without running tests. Good thing compiler-errors noticed this and un-approved PR in time, before the insta-merge :')
- [rust/110021] — Fixes for ICEs (Internal Compiler Errors) introduced by the first PR in this list
- [rust-lang/rfcs/3407] — RFC that proposes introducing explicit tail calls to Rust (I'm a big fun of tail calls...), I've written some feedback

[rust/109843]: https://github.com/rust-lang/rust/pull/109843
[rust/109819]: https://github.com/rust-lang/rust/pull/109819
[rust/109913]: https://github.com/rust-lang/rust/pull/109913
[rust/110013]: https://github.com/rust-lang/rust/pull/110013
[rust/110021]: https://github.com/rust-lang/rust/pull/110021
[rust-lang/rfcs/3407]: https://github.com/rust-lang/rfcs/pull/3407

# Stream

I did another stream on Wednesday.
This time it went fine and I was able to finish (well, almost finish) the work from the previous stream,
resulting in a small commit for [#109782] and  almost finishing the PR that adds suggestion to use closure argument instead of capturing.

{% callout() %}
Suggestion to do what?
{% end %}

Basically there is a common pattern, where you have a method, that takes a `&mut self` and then passes it to a closure argument:

```rust
struct S;

impl S {
    fn call(&mut self, f: impl FnOnce(&mut Self)) {
        // change state or something ... 
        f(self);
        // change state or something ... 
    }

    fn get(&self) {}
}
```

It is then very easy to accidentally use the old variable, instead of the argument passed to the closure:

```rust
    let mut v = S;
    v.call(|this: &mut S| v.get());
//  ^-- error: cannot borrow `v` as mutable
//             because it is also borrowed as immutable
```

I was trying to add a suggestion to change `v.get()` to `this.get()`, fixing the error.

{% callout() %}
Huh, okay
{% end %}

I ended up cleaning the code off-stream (and almost deleting everything while rebasing, oops (thanks to `git reflog` for restoring everything back...)) a little and opened [#110061].

[#109782]: https://github.com/rust-lang/rust/pull/109782
[#110061]: https://github.com/rust-lang/rust/pull/110061

# Summary

Overall I'm feeling a lot better this week!
I had a lot of fun with removing `OwningRef` from the compiler (the resulting code is very cute, imo,,,) and looking closely at the tail call proposal.
"See" you next week!
