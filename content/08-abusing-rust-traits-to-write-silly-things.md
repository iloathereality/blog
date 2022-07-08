+++
title = "(Ab)using Rust traits to write silly things"
date = 2022-06-29
description = "Explaining how to make a function that is callable with and without parentheses"

[taxonomies] 
tags = ["rust"]
+++

Recently I've found some cursed Rust code and decided to make a little joke/question [on twitter].
In the tweet I've presented some unusual code and asked "how could it compile?".
The solution was found quite fast by [@Veykril] (kudos to them!) and in this post I want to explain it in detail.

[on twitter]: https://twitter.com/maybewaffle/status/1541910667812274183
[@Veykril]: https://twitter.com/Veykril

<!-- more -->

# The joke/question

So in the tweet I've presented the following code:

```rust
fn main() {
    for x in lib::iter::<u32> {
        let _: u32 = x; // assert that `x` has type u32
    }
    
    for y in lib::iter::<String>() {
        let _: String = y;
    }
    
    for z in lib::iter { // infer type
        let _: &'static str = z;
    }
    
    for k in lib::iter() {
        let _: f64 = k;
    }
}
```

That compiles with **stable compiler**, if you have a right `lib`!
The interesting bit here is of course that function `lib::iter` can be called with or without parentheses. 
This is normally not possible, so what is going on?

First of all, some low hanging fruit: `for` loops in Rust desugar to something like this:

```rust
// Original
for x in i { f(x); }
```
```rust
// Desugaring
{
    let mut iter = IntoIterator::into_iter(i);
    while let Some(x) = iter.next() { f(x); }
    // while let itself desugars to loop+match,
    // but that's not the point
}
```

So before iteration starts [`into_iter`] is called.
Which allows you to pass any type implementing [`IntoIterator`], not just `Iterator`s.
That's just to say that we need `iter` and `iter()` somehow evaluate to type(s) that implement `IntoIterator`.

[`into_iter`]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html#tymethod.into_iter
[`IntoIterator`]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html

As a side note: on nightly there is a trait [`IntoFuture`] that is just like `IntoIterator`, but for futures and is used for `.await` desugaring.
You can do all the same stuff with it, so async functions with optional `()` are possible too:

```rust
#![feature(into_future)]
async fn _f() {
    let _: String = lib2::fut::<String>.await;
    let _: [u8; 2] = lib2::fut::<[u8; 2]>().await;
    let _: &str = lib2::fut.await;
    let _: u128 = lib2::fut().await;
}
```

BTW I think it [should be stabilized] soon, so keep an eye on the tracking issue (or don't (I'm just very excited for this feature)).

[`IntoFuture`]: https://doc.rust-lang.org/std/future/trait.IntoFuture.html
[should be stabilized]: https://github.com/rust-lang/rust/issues/67644#issuecomment-1163424241

But, this all still leaves us with a question: how can functions be called without parenthesis?

# Tempting idea that doesn't work

{% callout() %}
The title of this section is concerning, but why can't we just make a `const`+`impl IntoIterator for fn()`?
{% end %}

So the idea is to write something like this:

```rust
const iter: fn() -> Iter = || Iter;
impl IntoIterator for fn() -> Iter { /* not important */ }

struct Iter;
impl Iterator for Iter {}
```

Then `for _ in iter {}` would work because of the `IntoIterator` impl
and `for _ in iter() {}` would work because `iter`'s type is a [function pointer] that returns a type that implements `Iterator`.

[function pointer]: https://doc.rust-lang.org/stable/std/primitive.fn.html

But... this doesn't work for the following two reasons:

1. You can't implement a foreign trait (`IntoIterator`) for a foreign type (`fn() -> _`) and standard library doesn't (yet?) implement `IntoIterator` for function pointers.
   (you could patch `std` but then this won't work with the stable compiler)
2. Constants can't have generic parameters! So `iter::<T>` won't work.

So, we need to find something else to (ab)use.

# Hack #1

{% callout() %}
Uh? There are multiple hacks at play here already?
{% end %}

Yes! There is a hack for `iter::<T>` and for `iter::<T>()`, we'll start with the former.

`iter::<T>` looks a lot like a unit structure with a generic parameter. 

{% callout() %}
Maybe it **is** a unit structure with a generic parameter?
{% end %}

If only things were that simple...
In Rust you need to use all generic parameters in the type, or else your code won't compile:

```rust
struct S<T>(u8);
//~^ error: parameter `T` is never used
//~| help: consider removing `T`, referring to it in a field, or using a marker such as `PhantomData`
//~| help: if you intended `T` to be a const parameter, use `const T: usize` instead
```

This is because compiler wants to infer [variance] of all parameters.
Since a unit structure, by definition, doesn't have any fields, you can't use generic parameters in it!

[variance]: https://doc.rust-lang.org/nomicon/subtyping.html

{% callout() %}
Wait, compiler mentioned `PhantomData`, isn't that a unit structure with a generic parameter?...
{% end %}


It is! 

{% callout() %}
So I assume we can't use it because we can't impl `IntoIterator` for it either.
But why can't we copy its definition into our code?
{% end %}

Well... just look at its definition:

```rust
#[lang = "phantom_data"] // <-- *compiler magic*
pub struct PhantomData<T: ?Sized>;
```

{% callout() %}
Oh.
{% end %}

Yeah...

So, there is no way around this, to define a `PhantomData`-like type, we need to do something hacky...
A hack to do this I first saw implemented by dtolnay (not surprising, is it?) in their crate [`ghost`].

[`ghost`]: https://github.com/dtolnay/ghost

The hack basically looks like this:

```rust
mod imp {
    pub enum Void {}

    pub enum Type<T> {
        Type,
        __Phantom(T, Void),
    }

    pub mod reexport_hack {
        pub use super::Type::Type;
    }
}

#[doc(hidden)]
pub use imp::reexport_hack::*;
pub type Type<T> = imp::Type<T>;
```

{% callout() %}
Wha-
{% end %}

It may seem convoluted at first, but it's actually quite simple!

So, let's unpack this item-by-item:

1. `pub enum Void {}` defines a type with no values, also known as uninhabited type. 
   The nice property for us is that values of this type can't be created.
   That is basically a stable replacement for the [`!`] type.
2. `Type<T>` has two variants: unit variant `Type` and a struct variant `__Phantom(T, Void)`.
   The latter uses `T`, solving the "parameter `T` is never used" error
   while simultaneously being impossible to construct because of the `Void` field.
   Since `__Phantom` variant is impossible to create / uninhabited, 
   `Type<_>` effectively has only a single usable variant.
3. `reexport_hack` reexports `Type::Type` (variant `Type` of the type `Type`)
4. `pub use imp::reexport_hack::*;` is a glob reexport that reexports `Type::Type` 
   that was reexported by `reexport_hack`.
   I'm not entirely sure why, but using glob is important.
5. `pub type Type<T> = imp::Type<T>;` basically reexports the `Type` itself.
   It's just rendered in docs in a nicer way than if reexported by `pub use`

[`!`]: https://doc.rust-lang.org/stable/std/primitive.never.html

And now the magic: `Type` now refers both to the type **and** to the variant.
This works because Rust has different namespaces for types and values.
Glob reexport somehow suppresses an error about clashing names that arises when importing directly.
Idk why it's this way :shrug:

Ah, and `let _ = Type::<u8>` works because you can apply generic parameters to variants.
It's the same way as `None::<Fish>` is an expression of type `Option<Fish>`
or `Ok::<_, E>(())` is an expression of type `Result<(), E>`.

{% callout() %}
That's a lot... But I think I've grasped the concept
{% end %}

With this hack, you can define types that are indistinguishable from `PhantomData`!
And this time we can use it to define an `iter<_>` "unit struct":

```rust
pub type iter<T> = imp::iter<T>;
pub use imp::reexport_hack::*;

impl<T> IntoIterator for iter<T> {
    type Item = T;
    type IntoIter = Iter<T>; 
    // capitalized -^
}

struct Iter<T>(...);
impl<T> Iterator for Iter<T> { /* not that important */ }

mod imp { /* basically the same as before */}
```

This already allows us to do cool stuff like this:

```rust
for x in lib::iter::<u32> {
    let _: u32 = x; // assert that `x` has type u32
}

for z in lib::iter { // infer type
    let _: &'static str = z;
}

let iter: lib::iter<()> = lib::iter::<()>;
//     type ---^^^^^^^^        ^^^^^^^^^^--- constant
```

Now to the next hack, that would allow us to call `iter` too instead of using it as a constant!

# Hack #2

{% callout() %}
I want to make a guess of what we'll do!!
{% end %}

Uh ok, go ahead!

{% callout() %}
So I've heard that in Rust all functions and closures have unique types.
And the ability to call these types with `()` is controlled via [`Fn`], [`FnMut`] and [`FnOnce`] traits.
Can we just implement these traits for `iter<T>`?

[`Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
{% end %}

We could! But we can't.
These traits are unstable and I'm in the stable-compiler jail today.

{% callout() %}
Oh... Okay then... Do you have another "impossible to guess if haven't seen before" kind of thing?
{% end %}

Kind of!
We can (ab)use [`Deref`] trait:

[`Deref`]: https://doc.rust-lang.org/std/ops/trait.Deref.html

```rust
impl<T> Deref for iter<T> {
    type Target = fn() -> Iter<T>;
    
    fn deref(&self) -> &Self::Target {
        &((|| Iter([])) as _)
    }
}
```

Normally `Deref` is used for smart pointers like `Box` or `Arc` so that
- You can use the dereference operator on them (`*my_beloved_arc`)
- You can call methods of the inner type (`my_beloved_arc.nice()`)

This makes a lot of sense because smart pointers still just point to values and it's nice to be able to just call methods.

**But!**

There is nothing stopping you from implementing `Deref` for non-smart pointer types (besides, what _is_ a smart pointer?).
And so abnormally `Deref` is used to forward methods.

{% callout() %}
Is this considered a bad practice?
{% end %}

Uh well mmm yemmm aaa phhhh mmm... mh.. yes? But everyone uses it anyway.
It's even used this way in the [compiler itself], so who cares?

[compiler itself]: https://github.com/rust-lang/rust/blob/2953edc7b7a00d14c4ba940ebb46b4e7148a9d71/compiler/rustc_errors/src/diagnostic_builder.rs#L275

Ok, so what was I- Ah, right, and what came as a surprise to me, when you are writing `f()` deref coercions can deref `f` too!
So `f()` can become more like `(&*f)()` or in other words `f.deref()()`.
This means that by implementing deref to a function pointer for our `iter<T>` we can allow to call it!

Full code is on the [playground] if you want to play with it.

[playground]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2015&gist=d2643f25de8d2cc6beb9d009f01d1ba3

That's all I have for today, two hacks that I saw used ["in the wild"] (right, this one is also by dtolnay) and thought that it's quite surprising and fun thing.

["in the wild"]: https://github.com/dtolnay/inventory/blob/e105c33023492a443b53a58cd6ab0230d0434138/src/lib.rs#L261

bye.
