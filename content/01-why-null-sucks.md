+++
title = "Why null sucks, even if it's checked"
date = 2021-08-20
updated = 2021-08-25

[taxonomies] 
tags = ["kotlin", "csharp", "haskell", "rust", "go"]
+++

We all know that `null` is a ["billion-dollar mistake"], that it creates a lot of easy ways to make terrible mistakes. But it's only so bad when it's not checked by anyone and the compiler doesn't force you to check it, right? Well, the title might be a spoiler, but let's find out...

["billion-dollar mistake"]: https://en.wikipedia.org/wiki/Null_pointer#History

<!-- more -->

## Context / What are you talking about?

In this article, I'm specifically talking about the [Kotlin][kt-null-safety] and [C# post v8][csharp-nullable-reference-types] approach to `null`. I'm more familiar with Kotlin than C#, so I'll mostly be talking about it, but their approaches are similar, so that doesn't really matter.

[kt-null-safety]: https://kotlinlang.org/docs/null-safety.html
[csharp-nullable-reference-types]: https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#nullable-reference-types

Basically, you can't assign null to just any reference type in Kotlin (and C# post v8 with certain settings). For example, this is a compilation error:

```kotlin
// compilation error
val a: String = null;

// ok
var b: String = "not null";

// compilation error (again)
b = null;
```

And since you can't assign `null` to non-nullable types, `null` can't screw you! You can't get a null pointer (reference) exception, etc. Cool!

If you really want to assign null to something, you need to explicitly mark the type of the variable as nullable with `T?` syntax:

```kotlin
// ok
val a: String? = null;

// also ok
var b: String? = "not null";
b = null;
```

But when you have a `T?` (nullable) type, you also need to explicitly check for null. E.g. given a class `Mine` with a `cat` field and a variable `mine` of type `Mine?`, you can't access `mine.cat`, it would be a compilation error:

```kotlin
val mine: Mine? = /* ... */
val your: Mine  = /* ... */

// compilation error
val a: Cat = mine.cat

// elvis operator, if `mine` is null,
// then `your` will be used
val b: Cat = (mine ?: your).cat  

// safe call operator, if `mine` is null, 
// then so be `c`        
val c: Cat? = mine?.cat

// “trust me, I'm an engineer” operator, 
// throws an exception if `mine` is null 
val d: Cat = mine!!.cat

if mine != null {
    // Special Kotlin trick: 
    // in this block `mine` actually has type `Mine`.

    // perfectly fine (we've checked)
    val x: Cat = mine.cat
}
```

(The same applies for calling a function with an argument of non-nullable type, you need to either check that something is not null or explicitly assume so)

The "special kotlin trick" is called "smart cast". Basically, if you check that a variable is not null (or has a particular type), then the compiler changes type of the variable in the block, where the check holds. You can read more about it in Kotlin docs: [typecasts][kt-typecasts], [null-safety][kt-null-safety-cast].

[kt-typecasts]: https://kotlinlang.org/docs/typecasts.html#smart-casts
[kt-null-safety-cast]: https://kotlinlang.org/docs/null-safety.html#checking-for-null-in-conditions

And it's all great, don't get me wrong! This behaviour is a lot better than the plain old "every reference can be null, just crash if it is and it'll be fine". However...

## Why it's still meh

In my opinion, this comes down to 2 things: generality and extensibility.

### Generality

In modern languages, we have ways to abstract over types. Usually, the mechanism to do so is called generics. For example, a `HashMap` doesn't care what the key and value types are, it only cares that the key is comparable and hashable. Thus you can have `HashMap<String, i32>`, `HashMap<User, Settings>`, etc while only writing `HashMap` once. 

That's all good and all, but what's the problem? You may ask. Well... Consider this example: the `HashMap` has a method `get` which accepts key and returns the value associated with the key if there is one. But what should it return if there is no value associated with the key? It seems reasonable to return `null` since there is no value. So something like this:

```kotlin
class HashMap<K, V> {
    // If `key` is present in the hash map, 
    // returns the value associated with it. 
    // Otherwise returns `null`.
    fun get(key: K): V? { /* ... */ }
}
```

But what happens if we use `HashMap<_, T?>`? Well, then `V` is `T?` and so `V?` is `T??` which is the same as `T?` because there is only one `null` value. 

The crux of the problem is this: you can't distinguish between "absence of value" (no key in the map) and "presence of absence of value" (value is `null`). And if you need to distinguish them, then you need to call `containsKey` or something which is suboptimal.

This is just a simple example, but in general, you can't use `null` in generic code because `null` is too special and **unique**.

### Extensibility

`null` is not extensible. This mechanism is only useful when you have an optional value: either there is a value, or there isn't. 

It isn't useful when you need to express errors (`null` is used for this anyway, but this is problematic), exclusive or (either you have `A` or `B`, but not both and not neither), etc. 

It may seem like this isn’t a problem --- we were talking about expressing optional values from the very beginning. But bear with me, and for now just understand that `null` is not extensible and there may be other, more extensible, mechanisms.

## The saviour?

As I said before, there may be a more general and extensible way to handle optional values. And there is! It has a lot of different names --- sum types, enumerations, tagged unions, discriminated unions, etc. They’re all conceptually similar, but I like the "sum types" name the most, so that’s what I’ll call it.

In any case, the concept stays the same --- a type that can represent one out of multiple choices. Here are some well-known examples of definitions of sum types:

```rust
enum Option<T> { None, Some(T) }
enum Either<L, R> { Left(L), Right(R) }
```

```haskell
data Maybe a = Nothing | Just a
data Either a b = Left a | Right b
```

{% callout() %}
I've shown examples in Rust and Haskell because I'm familiar with them but note that sum types exist in [many languages][langs-with-adt]. 

[langs-with-adt]: https://en.wikipedia.org/wiki/Algebraic_data_type#Programming_languages_with_algebraic_data_types
{% end %}


`Option` (`Maybe`) is exactly what we're looking for! It can be either `None` (`Nothing`) or `Some(value)` (`Just value`). So either there is nothing or there is some value, exactly the same as with `null` so far. 

But it actually solves all the `null` problems I've mentioned!

{% callout() %}
Another nice fact about `Option` (`Maybe`) is that it can be defined in a library. Unlike nullable types, there's nothing special about this type. The compiler doesn't need to know about it, it's just a type.
{% end %}

### Generality

`Option<Option<T>>` is a meaningful type, unlike `T??`. In the same example with `HashMap::get` there isn't any problems if it returns `Option<_>`.

```rust
impl<K, V> HashMap<K, V> {
    // If `key` is present in the hash map, 
    // returns the value associated with it. 
    // Otherwise returns `None`.
    fn get(&self, key: &K) -> Option<V> { /* ... */ }
}
```

If the key isn't present in the `HashMap<K, Option<V>>`, then `None` is returned. If the key is present but the associated value is `None`, `Some(None)` is returned. Otherwise, if the key is present and the value isn't `None`, then `Some(Some(value))` is returned. 

The use of sum types gives us 3 distinct kinds of values that can be distinguished from one another, instead of an ambiguous null.

### Extensibility

Sum types can be used for optional values via `Option`-like types. But they are not limited to only this. You can define your own sum types. It's very handy when you need to return errors (See rust [`Result`](https://doc.rust-lang.org/std/result/index.html) for example), define the errors themselves or just in general when you need to hold different kinds (types) of data in one place. 

### Explicitness

There is one thing you may prefer about `null` over sum types: `T` can be [coerced][type-coercion] to `T?`.  That means that you don't need to explicitly wrap values in `Some` (`Just`). 

[type-coercion]: https://en.wikipedia.org/wiki/Type_conversion

This may be handy, because you need to type ~6 characters less if you want to pass `T` to a function that accepts `T?`. However, I don't think that wrapping inconvenience outweighs the benefits of sum types.

```kotlin
fun f(x: Int?): Int? = x?.let { it + 1 }
val res = f(1)
```

```rust
fn f(x: Option<i32>) -> Option<i32> { x.map(|it| it + 1) }
let res = f(Some(1));
//          ^^^^^ ^
```

It would be interesting to see a language with sum types and `T` to `Option<T>` coercion though 👀

## Niche optimization

{% callout() %}
I could only find mentions of this optimization in Rust, but I think it's very neat anyway, so I'll talk about Rust in this paragraph.
{% end %}

There are some types that have unused space in their memory representation --- a *niche*. For example, `bool` is only 1 bit of information, but it uses 1 byte of space, so 7 bits are unused. References (`&T`, `&mut T`) and `NonNull` can never be null, meaning that they have an unused bit pattern --- all zeroes. The same goes with [`NonZero*`]. Enumerations, just as `bool`, can have unused bits/bit patterns.

[`NonZero*`]: https://doc.rust-lang.org/std/num/index.html

What if we could use this for something actually useful? Well, Rust can.

If a sum type has one of its variants being a type with a niche, then it can use it to represent other variants, instead of using a tag. I.e. `Option<&T>` isn't identical to `struct { value: union { Some(&T), None }, tag: u8 }` (pseudo syntax), but instead it's just `union { Some(&T), None }` where the `None` is encoded as `0` (since reference can never be `0` you can distinguish variants without a `tag`). Moreover, this exact optimization of `Option<&T>` is even [guaranteed][rust-option-representation], so it is fully layout compatible with `*const T`. This allows using it in C-ffi, making `None` on the Rust side a `null` on the C side.

[rust-option-representation]: https://doc.rust-lang.org/std/option/index.html#representation

This optimization can be observed using `mem::size_of` function:

```rust
// `u8`, `bool` and `Option<bool>` all occupy 1 byte.
// `Option::<bool>::None` is encoded as 2. It's fine, since `bool`
// can only ever by 1 or 0.
size_of::<u8>() = 1
size_of::<bool>() = 1
size_of::<Option<bool>>() = 1
unsafe { transmute::<Option<bool>, u8>(None) } = 2
unsafe { transmute::<Option<bool>, u8>(Some(true)) } = 1
unsafe { transmute::<Option<bool>, u8>(Some(false)) } = 0

// `Option<Option<bool>>` behaves similarly, it just needs 2 patterns.
size_of::<Option<Option<bool>>>() = 1
unsafe { transmute::<Option<Option<bool>>, u8>(None) } = 3
unsafe { transmute::<Option<Option<bool>>, u8>(Some(None)) } = 2

// On 64 bit machines references take 8 bytes.
// `Option::<&_>::None` is encoded as a `0`/`null`.
size_of::<&u8>() = 8
size_of::<Option<&u8>>() = 8
unsafe { transmute::<Option<&u8>, *mut u8>(None) } = 0x0000000000000000

// `u8` has no niche, so `Option` needs a tag (in this case: first byte)
size_of::<Option<u8>>() = 2
unsafe { transmute::<Option<u8>, Pair<u8, MaybeUninit<u8>>>(None) } = (0, MaybeUninit<u8>)
unsafe { transmute::<Option<u8>, [u8; 2]>(Some(0xAA)) } = [1, 170]

// You may have expected 9, instead of 16, 
// but in Rust, size is always a multiple of the alignment.
// References are 8-bytes aligned. The nearest 
// multiple of 8 bigger than 8 is 16.
size_of::<Option<Option<&u8>>>() = 16
// It seems like the first word is tag and the second
// is optional reference.
unsafe { transmute::<Option<Option<&u8>>, Pair<usize, MaybeUninit<usize>>>(None) } = (0, MaybeUninit<u8>)
unsafe { transmute::<Option<Option<&u8>>, [usize; 2]>(Some(None)) } = [1, 0]
unsafe { transmute::<Option<Option<&u8>>, [usize; 2]>(Some(Some(&1))) } = [1, 93965956011124]
```

_Edited output of the [test program][rs-play-0]._

[rs-play-0]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=0166b25cd98164e32879456d65a8e9b8

## A sad note: Kotlin

Kotlin *does* support sum types via [sealed classes][kt-sealed-classes]. Here is an example how you could define and use `Option` ([play][kt-play-0]): 

[kt-sealed-classes]: https://kotlinlang.org/docs/sealed-classes.html
[kt-play-0]: https://pl.kotl.in/XZ1fBjith

```kotlin
sealed class Option<T> {
    class None<T>() : Option<T>() {}
    class Some<T>(val value: T) : Option<T>() {}
}

fun main() {
    val a: Option<Int> = Option.Some(12);
    val b = when(a) {
        is Option.None -> "None"

        // `a` is smart-casted to `Some`, so `value` is accessible.
        // IMO pattern matching would be better, but that's ok.
        is Option.Some -> a.value.toString()
    }
    println(b)
}
```

{% callout() %}
In my opinion, Kotlin's support of sum types is extremely hacky, but who am I to say that, right?
{% end %}

Nevertheless, Kotlin doesn't use this for optional values! In fact, it doesn't even have an `Option` class in the standard library. For me, it seems like a big omission. It seems like making `T?` equivalent to `Option<T>` and `Option<N>` (where `N` is not `Option`) be layout compatible with `Java`'s `N` (i.e. `Java`'s `null` being the same as `None<N>`) would be sufficient...

## A sad note: C#

C# doesn't support sum types. To some extent they can be simulated using inheritance from an interface or abstract class, however, such an approach lacks one of the greatest benefits of sum types, namely the exhaustiveness check.

It's especially sad since F# (which is running on the same VM) supports sum types (F# docs call them [*Discriminated Unions*][fsharp-discr-unions]), but C# doesn't.

[fsharp-discr-unions]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions

```f#
type Option<'a> = None | Some of 'a
type Either<'a, 'b> = Left of 'a | Right of 'b
```

## A sad note: Go

Go doesn't support sum types. It's a little sad on its own, but Go also doesn't have exceptions and all reference types are implicitly nullable. This means that if a function wants to return an error, it needs to return a tuple of success and error values. This not only makes checking which is `null` (actually `nil`, but it’s the same thing) pretty annoying, but also leaves the possibility for an invalid state where neither success nor error values are null. 

I think it's inexcusable to have such error-prone design flaws in 2012.

## Conclusion

There is no nice conclusion. The reality is painful as usual. We are stuck with a 56-year old abstraction that has proven itself to be error-prone for a while now. We are stuck with it, even though a better solution exists for longer than I have.

bye.