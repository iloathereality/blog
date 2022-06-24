+++
title = "Why we can’t have nice things: min/max"
date = 2022-06-24
description = "In this post I’ll talk about Rust’s BTreeSet and tell a sad story about its methods"

[taxonomies] 
tags = ["rust"]
+++

In this post I’ll talk about Rust’s [`BTreeSet`] and tell a sad story about its methods.

[`BTreeSet`]: https://doc.rust-lang.org/std/collections/btree_set/struct.BTreeSet.html

<!-- more -->

# The nice things (that we can’t have)

So, [`BTreeSet`] is an ordered collection that has an element with the maximal value (if it isn’t empty) and an element with the minimal value (again, if not empty).
And there are methods that return references to these elements.
In a similar fashion to other Rust collections (that have their own “extremes").
These methods have signature `&Self -> Option<&T>`, that is, they take a reference to the set and return a reference to the element with the minimal/maximal value (if the set is not empty).

{% callout() %}
So far so good, right? And how are these methods named?
{% end %}

The methods are named `first` and `last`. 

{% callout() %}
And which of them is the minimal and which of them is the maximal?
{% end %}

Well... that’s “the problem” of this blog post, I don’t remember!
*checks docs*.
Ok, so [`first`] is the minimum and [`last`] is the maximum, thus the order is ascending.
I would probably accept this, if it was consistent through the whole `std`... but no, of course there is [`BinaryHeap`] that works the other way around and now everything is even more confusing...

[`first`]: https://doc.rust-lang.org/std/collections/btree_set/struct.BTreeSet.html#method.first
[`last`]: https://doc.rust-lang.org/std/collections/btree_set/struct.BTreeSet.html#method.last
[`BinaryHeap`]: https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html

{% callout() %}
But wait, the methods are unstable, so can’t we just... rename them to `min` & `max`?
{% end %}

Turns out we can’t!
And for a completely stupid reason.
I’ll explain everything in a moment, but first (pun unintended),

# A short detour to all the “extremes” methods

In Rust standard library there are quite a few collections that have some notion of order and they all have these getter methods for the “extremes”.
The names, for the most part, make sense, but they are quite inconsistent. Here is the table

| Type(s) | The min-ish | The max-ish | Has &mut versions? | Has inherent order |
| --- | --- | --- | --- | --- |
| [T] slices and Vec | first | last | Yes ✅ | No ❌ |
| BTreeSet and BTreeMap | first | last | No ❌ | Yes ✅ |
| VecDeque and LinkedList | front | back | Yes ✅ | No ❌ |
| BinaryHeap | - | peek* | Yes ✅ | Yes ✅ |

Notes:

1. For collections that do not have inherent order, min-ish and max-ish methods were chosen on the basis of the sorted state — in the sorted slice `<[T]>::first` returns the minimum element, so it’s categorized as “min-ish”. Note that when the collection is not sorted min-ish methods may return not the smallest element and max-ish methods mey return not the largest element.
2. `BTreeSet` and `BTreeMap` don’t have `&mut` versions because changing an element would change its order
    - `BinaryHeap` sidesteps this issue by having `peek_mut` return `Option<PeekMut<’_, T>>` instead of `Option<&mut T>` and fixing the order in drop
    - Really, the `BinaryHeap` is the weirdo here...
3. `BTreeSet` and `BTreeMap` have some additional methods like `pop_`, `_key_value`, `_entry`, see the [tracking issue][btreeemapti]
4. It’s nice that all min-ish methods use 5 letters and all the max-ish methods use 4 letters, I hadn't noticed it until I wrote this
5. `Vec` actually has its `first`/`last` methods via `Deref<Target = [T]>`
6. This list is too long

[btreeemapti]: https://github.com/rust-lang/rust/issues/62924

So, `BTreeSet`/`BTreeMap` method names were chosen to be consistent with slice.
But still, why can’t we have `BTreeSet::{min, max}`?

# The reason why we can’t have nice things

So, me being me, I’ve tried to rename the `BTreeSet` methods to `min`/`max` and... failed.
Here is the error (not the actual error I’ve gotten while renaming, that was more than ~~2~~ 5 months ago, but it’s a good enough reenactment for today):

```
error[E0061]: this function takes 1 argument but 0 arguments were supplied
   --> src/lib.rs:11:20
    |
11  |     assert_eq!(set.min(), Some(&-1));
    |                    ^^^- supplied 0 arguments
    |                    |
    |                    expected 1 argument

error[E0308]: mismatched types
  --> src/lib.rs:11:5
   |
11 |     assert_eq!(set.min(), Some(&-1));
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected struct `BTreeSet`, found enum `Option`
   |
   = note: expected struct `BTreeSet<i32>`
                found enum `Option<&{integer}>`
   = note: this error originates in the macro `assert_eq` (in Nightly builds, run with -Z macro-backtrace for more info)
```

Why does the compiler expect `min` to have an argument, why does it think it returns `BTreeSet<i32>` when it’s clearly declared as `pub fn min(&self) -> Option<&T>`, what is going on?

After some digging I’ve figured out that `set.min()` is resolved by the compiler to [`Ord::min`], not `BTreeSet::min`.
But don’t inherent methods take precedence over trait methods?

[`Ord::min`]: https://doc.rust-lang.org/std/cmp/trait.Ord.html#method.min

I couldn’t find the place from where I’ve taken the fact that “inherent methods” take precedence, but it is indeed [the case]:

[the case]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=8d7c8bb69bd64b613329468261985f8b

```rust
struct A;
impl A  { fn f(&self) -> &'static str { "inherent" } }
trait T { fn f(&self) {} }
impl T for A {}

// This works
assert_eq!(A.f(), "inherent");
```

So in this case inherent method takes precedence, so what’s the difference with the `BTreeSet::min` & `Ord::min`?
The secret is in their signatures:

```rust
// trait Ord
fn min(self, other: Self) -> Self where Self: Sized { ... }

// impl BTreeSet
fn min(&self) -> Option<&T> { ... }
```

Do you see the problem?
`BTreeSet::min` takes self by reference (`&self`) while `Ord::min` takes self by value (`self`).
And methods with receivers that are “less distant from the original type” always “win” by the autoref rules.
It’s been the behavior since forever and it’s [documented]:

[documented]: https://doc.rust-lang.org/reference/expressions/method-call-expr.html

> When looking up a method call, the receiver may be automatically dereferenced or borrowed in order to call a method.
This requires a more complex lookup process than for other functions, since there may be a number of possible methods to call.
The following procedure is used:
> 
> 
> The first step is to build a list of candidate receiver types.
> Obtain these by repeatedly [dereferencing](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-dereference-operator) the receiver expression's type, adding each type encountered to the list, then finally attempting an [unsized coercion](https://doc.rust-lang.org/reference/type-coercions.html#unsized-coercions) at the end, and adding the result type if that is successful.
> Then, for each candidate `T`, add `&T` and `&mut T` to the list immediately after `T`.
> 
> For instance, if the receiver has type `Box<[i32;2]>`, then the candidate types will be `Box<[i32;2]>`, `&Box<[i32;2]>`, `&mut Box<[i32;2]>`, `[i32; 2]` (by dereferencing), `&[i32; 2]`, `&mut [i32; 2]`, `[i32]` (by unsized coercion), `&[i32]`, and finally `&mut [i32]`.
> 
> Then, for each candidate type `T`, search for a [visible](https://doc.rust-lang.org/reference/visibility-and-privacy.html) method with a receiver of that type in the following places:
> 
> 1. `T`'s inherent methods (methods implemented directly on `T`).
> 2. Any of the methods provided by a [visible](https://doc.rust-lang.org/reference/visibility-and-privacy.html) trait implemented by `T`.
> If `T` is a type parameter, methods provided by trait bounds on `T` are looked up first.
> Then all remaining methods in scope are looked up.

For the case of `BTreeSet`, `.min()` would result in the following list: `BTreeSet<T>` (hit! it’s `Ord::min`), `&BTreeSet<T>` (`BTreeSet::min`, but it’s too late), `&mut BTreeSet<T>` (`BTreeSet` doesn’t implement `Deref` or `Unsize`).

So here we are, `min` doesn’t work as a name of a method that takes self by ref on a type that implements `Ord`, `get_min` and `minumum` were [rejected] by the libs-api team, we are stuck with the confusing `first`/`last`.

[rejected]: https://github.com/rust-lang/rust/pull/93709#issuecomment-1098126012

And the saddest part?
I don’t even think `Ord::{min, max}` are good methods.
They don’t make much sense as methods (I think `min(a, b)` is nicer and more symmetric than `a.min(b)`) and they have too-generic names (as seen by this whole issue with `BTreeSet`) for methods of the trait that is in the prelude and is implemented for a large amount of types.

But, it’s too late to change anything.

That was the reason why we can’t have nice things, bye.
