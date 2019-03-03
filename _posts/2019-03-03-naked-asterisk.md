---
layout: post
title: The naked asterisk
---

Andrew Berls writes about the naked asterisk in this [post](http://andrewberls.com/blog/post/naked-asterisk-parameters-in-ruby). I too stumbled across it whilst looking through Rails source.

Andrew says:

>So what's the deal with the unnamed asterisk? Well, this is still the same splat operator at work, the difference being that it does not name the argument array. This may not seem useful, but it actually comes into play when you're not referring to the arguments inside the function, and instead calling `super`. To experiment, I put together this quick snippet:

```ruby
class Bar
  def save(*args)
    puts "Args passed to superclass Bar#save: #{args.inspect}"
  end
end

class Foo < Bar
  def save(*)
    puts "Entered Foo#save"
    super
  end
end

Foo.new.save("one", "two")
```

>which will print out the following:

```ruby
Entered Foo#save
Args passed to superclass Bar#save: ["one", "two"]
```

>The globbed arguments are automatically passed up to the superclass method. Some might be of the opinion that it's better to be explicit and define methods with `def method(*args)`, but in any case it's a neat trick worth knowing!
