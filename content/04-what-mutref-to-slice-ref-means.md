+++
title = "What does `&mut &[T]` mean?"
date = 2021-11-02
description = "References can be quite confusing, especially with different mutabilities. Indeed, only a third of people in my poll said that they understand what `&mut &[T]` means!"

[taxonomies] 
tags = ["rust"]
+++

References can be quite confusing, especially with different mutabilities. Indeed, only a third of people in [my poll](https://t.me/ihatereality/1776) said that they understand what `&mut &[T]` means!

<!-- more -->

# Ground rules

In Rust, to change a value you need to have some sort of unique access to it, for example, a `&mut T` (a unique reference to `T`). Shared references (`&T`) on the other hand do not provide unique access on their own.

Since shared references don't grant unique access, a shared reference to a unique reference `&&mut T` is equivalent to `&&T` in terms of mutating the value of type `T`. So, if you have a shared reference in a type, you can't mutate anything "after" it. 

# Slightly easier question: what does `&mut &T` mean?

So, if a shared reference doesn't allow you to mutate anything "after" it, `&mut &T` doesn't allow you to mutate the value of type `T` and thus is the same as `&&T` or `&T`? The former is true — you can't mutate `T` — but the latter is not true. While you can't mutate the `T`, you can mutate the reference `&T`:

```rust
let value = 1;
let mut shared = &value;

println!("{r:p}: {r}", r = shared);
// Prints <addr>: 1

let unique = &mut shared;
*unique = &17;
println!("{r:p}: {r}", r = shared);
// Prints <different addr>: 17
```

# How is a slice different?

With `&mut &[T]` you can still change the reference, making it point to another slice. But `&[T]` is essentially a fat pointer, which means that it's not only a pointer but also a length of the slice.

Since you can mutate the reference, and the reference stores length as it's part, you can change it too:

```rust
let mut slice: &[u8] = &[0, 1, 2, 3, 4];
let unique = &mut slice;

// Since we want to hold unique reference, 
// we can only access the slice through it
println!("({r:p}, {len}): {r:?}", r = *unique, len = unique.len());
// Prints (<addr>, 5): [0, 1, 2, 3, 4]

// Change only the length
*unique = &unique[..4];
println!("({r:p}, {len}): {r:?}", r = *unique, len = unique.len());
// Prints (<addr>, 4): [0, 1, 2, 3]

// Change both the pointer and the length
*unique = &unique[1..];
println!("({r:p}, {len}): {r:?}", r = *unique, len = unique.len());
// Prints (<addr+1>, 3): [1, 2, 3]

// Change only the pointer
*unique = &[17, 17, 42];
println!("({r:p}, {len}): {r:?}", r = *unique, len = unique.len());
// Prints (<different addr>, 3): [17, 17, 42]
```
[(playground)](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=d94fde72d3f8ef51290de2d0c7b11c55)

One real-world example of `&mut &[T]` may be the `io::Read` [implementation](https://doc.rust-lang.org/std/io/trait.Read.html#impl-Read-2) for `&[u8]`:

```rust
use std::io::Read;

// We'll be reading *from* this slice
let mut data: &[_] = &[0, 1, 2, 3, 4, 5, 6, 7, 8];
// And *into* this
let mut buf = [0; 4];

// This implicitly creates a &mut &[u8] via auto-ref
while let Ok(1..) = data.read(&mut buf) {
    println!("({r:p}, {len}): {r:?}", r = data, len = data.len());
    // This will print:
    // (<addr>, 9): [0, 1, 2, 3, 4, 5, 6, 7, 8]
    // (<addr+4>, 5): [4, 5, 6, 7, 8]
    // (<addr+8>, 1): [8]
    // (<addr+9>, 0): []
    
    // In reality you'd also examine the `buf` contents here
}
```
[(playground)](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=c9514a733eba51b8591715f87fce79c7)

---

Now you know what `&mut &[T]` means! I hope that was helpful. Bye.
