---
layout: post
title: 'Override `#hash` when overriding `#eql?`'
---

In my notes [here]({{ site.baseurl }}{% link _posts/2019-03-02-value-objects.md %}), I've written that `#hash` must be overridden when overriding `#eql?`. I was unsure why this was the case.

## What is `#hash`?

I see in pry that all objects respond to `#hash`:

```
> 1.hash
=> 3748939910403886956
> 'a'.hash
=> 1677925148165319732
> [1,2,3].hash
=> -2230614089907012012
> Time.now.hash
=> -2249667312364590389
```

Equal objects return the same value:

```
> 1 == 1
=> true
> 1.hash
=> 3748939910403886956
> 1.hash
=> 3748939910403886956
```

Hash comes from the `Kernel` module:

```
> 1.method(:hash).owner
=> Kernel
```

>The Kernel module is included by class Object, so its methods are available in every Ruby object.

It's basically a bunch of helper methods made available to every class.

`Kernel#hash` seems to be undocumented though. It's not in in the docs for the `Kernel` [module](https://ruby-doc.org/core-2.6.1/Kernel.html#method-i-Hash). Though, that page does point towards the `Object` class [page](https://ruby-doc.org/core-2.6.1/Object.html) for information on `Kernel` instance methods. But there's nothing on it there either.

The explanation to why `#hash` needs to be overridden when `#eql` is changed is in the [`Hash` docs](https://ruby-doc.org/core-2.5.3/Hash.html#class-Hash-label-Hash+Keys).

>Two objects refer to the same hash key when their hash value is identical and the two objects are eql? to each other.

Two objects *can* have the same hash value but be unequal however this would be detriment to the speed of the hash.

So when keying by multiple objects, so long as their `hash` and `eql?` values match, they will point to the same bucket.
