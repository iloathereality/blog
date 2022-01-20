+++
title = "Comments are weak"
date = 2022-01-30
description = "I like writing comments. But at the same time comments kind of suck. In this post, I want to outline some ways we could improve this."

[taxonomies] 
tags = []
+++

I like writing comments. But at the same time comments kind of suck. In this post, I want to outline some ways we could improve this. 

<!-- more -->

I like writing long comments that have references to code and conversations, that explain complex things, that have diagrams, etc, etc.
But comments are just text. Just raw, unparsed, unformatted text.

And this is **bad**!

Comments are just a very internal form of documentation, and documentation is usually written in some markup language (like markdown, in the case of Rust).
We render it, we allow it to link other parts of code. So why don’t we have anything like that for comments? 
Is it because comments predate documentation and markup languages, and we just continue to treat them like we always treated them?

{% callout() %}
tip: in Rust, you can use paths in inline links in documentation to mention items, like `[this function](crate::path::to::fun)`. 
more on this in the [`rustdoc` documentation](https://doc.rust-lang.org/rustdoc/linking-to-items-by-name.html).
{% end %}

One of the things I like about newer versions of JetBrains IDEs is that they can render documentation:

{{ 
  image(
      img="idea_rendered_documentation.png", 
      alt="Screenshot of a function with documentation. The documentation reads “JetBrains IDEs can render documentation and that’s quite cool. Ex: fun(), bold, italic, etc”, the “JetBrains IDEs” and “fun()” are links (highlighted in blue), “bold” and “italic” are bold and italic. The function is called fun.", 
      style="border-radius: 8px; width: 80%;"
      quality=100
  )
}}

{{ 
  image(
      img="idea_unrendered_documentation.png", 
      alt="Screenshot of the same function and its documentation, but documentation is not rendered and is written in markdown.", 
      style="border-radius: 8px; width: 80%;"
      quality=100
  )
}}

I think that we should treat comments similarly — allow some kind of formatting inside them and then render them in IDEs.
I especially want to be able to use inline links, because commit hashes, message ids and such often cause links to be very long.

I wish comments were treated better than we treat them right now.
