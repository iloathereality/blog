+++
title = "What does `&mut &[T]` mean?"
date = 2021-11-02
description = "References can be quite confusing, especially with different mutabilities. Indeed, only a third of people in my poll said that they understand what `&mut &[T]` means!"

[taxonomies] 
tags = ["rust"]
+++

References can be quite confusing, especially with different mutabilities. Indeed, only a third of people in [my poll] said that they understand what `&mut &[T]` means!

[my poll]: https://t.me/ihatereality/1776

<!-- more -->

# Ground rules

In Rust, to change a value you need to have some sort of unique access to it, for example, a `&mut T` (a unique reference to `T`). Shared references (`&T`) on the other hand do not provide unique access on their own.

Since shared references don't grant unique access, a shared reference to a unique reference `&&mut T` is equivalent to `&&T` in terms of mutating the value of type `T`. So, if you have a shared reference in a type, you can't mutate anything "after" it. 

# Slightly easier question: what does `&mut &T` mean?

So, if a shared reference doesn't allow you to mutate anything "after" it, `&mut &T` doesn't allow you to mutate the value of type `T` and thus is the same as `&&T` or `&T`? The former is true — you can't mutate `T` — but the latter is not true. While you can't mutate the `T`, you can mutate the reference `&T`:

```rust
let value = 1;
let mut shared: &u32 = &value;

println!("{r:p}: {r} (value = {v})", r = shared, v = value);
// Prints <addr>: 1 (value = 1)

let unique: &mut &u32 = &mut shared;
*unique = &17;

println!("{r:p}: {r} (value = {v})", r = shared, v = value);
// Prints <different addr>: 17 (value = 1)
```

[(playground)](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=ff4796eeff3f3bf0f26e2174454755ef)

# How is a slice different?

With `&mut &[T]` you can still change the reference, making it point to another slice. But `&[T]` is essentially a fat pointer, which means that it's not only a pointer but also a length of the slice.

Since you can mutate the reference, and the reference stores length as it's part, you can change it too:

```rust
let mut slice: &[u8] = &[0, 1, 2, 3, 4];
let unique: &mut &[u8] = &mut slice;

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
[(playground)](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=87d5eed5915faab0a22c26ae87ced091)

One real-world example of `&mut &[T]` may be the `io::Read` [implementation] for `&[u8]`:

```rust
use std::io::Read;

// We'll be reading *from* this slice
let mut data: &[u8] = &[0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
// And *into* this
let mut buf = [0; 3];

while let Ok(1..) = Read::read(&mut data, &mut buf) {
    println!("({r:p}, {len}): {r:?}", r = data, len = data.len());
    // This will print:
    // (<addr>, 7): [3, 4, 5, 6, 7, 8, 9]
    // (<addr+3>, 4): [6, 7, 8, 9]
    // (<addr+6>, 1): [9]
    // (<addr+7>, 0): []   
    
    // In reality you'd also examine the `buf` contents here
}
```
[(playground)](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=07c6f60ad445f95b69f143427a0df7a7)

[implementation]: https://doc.rust-lang.org/std/io/trait.Read.html#impl-Read-2

---

Now you know what `&mut &[T]` means! I hope that was helpful. Bye.
