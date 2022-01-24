+++
title = "if-else chains considered harmful"
date = 2022-01-24
description = "As always, I have some strong opinions on things I probably shouldn't have opinions on. This time I want to tell you why I think if-else chains are syntactically harmful."

[taxonomies] 
tags = ["rust"]
+++

As always, I have some strong opinions on things I probably shouldn't have opinions on. This time I want to tell you why I think `if`-`else` chains are syntactically harmful.

<!-- more -->

I think that the syntax for if-else chains that are used in most imperative languages, including Rust, is bad.

```rust
if something.something() {
    do_something()
} else if let Some(x) = get_x() {
    use_x(x);
} else if mutliline {
    a();
    b();
    c();
} else {
    scream("aaaaaaaaaaaaaaaaaaaaaaa")?;
}
```

It's hard to read:

- conditions are mixed with `if`/`else`/`let` keyword-noise
    
    {% callout() %}
    `[else] if let` in Rust allows pattern-matching behaviour in `if` statements
    {% end %}
    
- conditions are indented more than expressions
- conditions are indented inconsistently between the first branch (`if`) and the following ones (`else if`)
- one-liners take a lot of lines
- etc.

{% callout() %}
Singular `if`s are fine I guess, it's the `else if` chains that make all of this particularly bad
{% end %}

I think that a much better way to model the same thing is [`when` from Kotlin](https://kotlinlang.org/docs/control-flow.html#when-expression) or pattern guards/[multi-way-if](https://wiki.haskell.org/Case#MultiWayIf) from Haskell.

<details><summary>Kotlin and Haskell examples</summary>
<p>
    
```kotlin
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is odd")
}
```

```haskell
case () of _
    | condition1 -> expr1
    | condition2 -> expr2
    | condition3 -> expr3
    | otherwise  -> default

if | condition1 -> expr1
    | condition2 -> expr2
    | condition3 -> expr3
    | otherwise  -> default
```


</p>
</details>

In Rust this can be simulated via `match` on unit with pattern guards ([Boxy style](https://twitter.com/EllenNyan0214/status/1356792709391015937)):

```rust
match () {
    _ if something.something() => do_something(),
    _ if let Some(x) = get_x() => use_x(x),
    _ if multiline => {
        a();
        b();
        c();
    }
    _ => scream("aaaaaaaaaaaaaaaaaaaaaaa")?,
}
```

But this is still messy, `if let` guards are [unstable](https://github.com/rust-lang/rust/issues/51114) and the repeated `if`s are still there.

This is why I created [`kiam`](https://docs.rs/kiam/0.1.0/kiam/index.html) (#shamelessplug?) --- a crate that provides a simple macro that makes these chains a lot nicer:

```rust
when! {
    something.something() => do_something(),
    let Some(x) = get_x() => use_x(x),
    multiline => {
        a();
        b();
        c();
    }
    _ => scream("aaaaaaaaaaaaaaaaaaaaaaa")?,
}
```

{% callout() %}
I've explained `kiam` with memes on [twitter dot com](https://twitter.com/maybewaffle/status/1359100586948505603?s=20)
{% end %}

{% callout() %}
One of the only downsides of `when!` is IDE and tooling (`rustfmt`) support (or rather the absence of it). I wish this was part of the language or std...
{% end %}

Soooo yeah, there isn't much to add here. If-else chains are bad for humans, macros are bad for IDEs, I have a cool crate.

Bye.
