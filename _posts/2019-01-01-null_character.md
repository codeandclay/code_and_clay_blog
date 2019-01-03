---
layout: post
title: Unicode null
---

Unicode has a [character that represents null](https://en.wikipedia.org/wiki/Null_character).

It is used to represent the end of a string but could also be used when a placeholder character is required. Perhaps as padding, differentiating from `' '`.

In Ruby, `0.chr` evaluates to the Null character.

There is a can of worms called [Null Byte Injection](http://projects.webappsec.org/w/page/13246949/Null%20Byte%20Injection) associated with this character.
