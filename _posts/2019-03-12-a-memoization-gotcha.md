---
title: Be careful when memoizing booleans!
layout: post
---
**(This post is also published on [dev.to](https://dev.to/updated_tos/a-memoization-gotcha-14c6))**

It took me an hour or so of frustration to figure out why a method was being called multiple times despite my attempt at memoizing its return value.

## The problem

My problem looked a bit like this.

```ruby
def happy?
  @happy ||= post_complete?
end
```

My intention was that value of `post_complete?` would be stored as `@happy` so that `post_complete?` would be fired only once.

However, that's not guaranteed to happen here. `post_complete?` might be evaluated and its value assigned to `@happy` *every* time I call `happy?`.

Can you see why?

```ruby
  @happy ||= post_complete?
```

## What's going on?

The question mark denotes that `post_complete?` is expected to return a boolean value. But, what if that value is always `false`?

Another way of writing the statement is:

```ruby
@happy || @happy = post_complete?
```

In the above example, I want to know if at least one of the sides is true.

Remember that if the left-hand side of an `||` statement is `false`, then the right-hand side is evaluated. If the left side is truthy, there's no need to evaluate the right side – the statement has already been proven to be true – and so the statement [short circuits](https://blog.revathskumar.com/2013/05/short-circuit-evaluation-in-ruby.html).

If I replace `post_complete?` with boolean values, it's easier to see what is happening.

In this example, `@happy` becomes `true`:

```ruby
def happy?
  @happy || @happy = true
  # @happy == true
end
```

However, in this example, `@happy` becomes `false`:

```ruby
def happy?
  @happy || @happy = false
  # @happy == false
end
```

In the former, `@happy` is falsey the first time the method is called, then `true` on subsequent calls. In that example, the right-hand side is evaluated once only. In the latter, `@happy` is always `false` and so both sides are always evaluated.

When using the `||=` style of memoization, only truthy values will be memoized.

So the problem is that if `post_complete?` returns `false` the first time `happy?` is called, it will be evaluated until it returns `true`.

## A solution

So how do I go about memoizing a false value?

Instead of testing the truthiness of `@happy`, I could check whether or not it has a value assigned to it. If it has, I can return `@happy`. It if hasn't, then I will assign one. I will use `Object#defined?`.

The [documentation](http://ruby-doc.org/docs/keywords/1.9/Object.html#method-i-defined-3F) states:

>defined? expression tests whether or not expression refers to anything recognizable (literal object, local variable that has been initialized, method name visible from the current scope, etc.). The return value is nil if the expression cannot be resolved. Otherwise, the return value provides information about the expression.

>Note that the expression is not executed.

I use it like so:

```ruby
def happy?
  return @happy if defined? @happy
  @happy = false
end
```

Referring back to the documentation, there's one thing I need to be aware of. This isn't the same as checking for `nil` or `false`. It's a bit counterintuitive, but `defined?` doesn't return a boolean. Instead, it returns information about the argument object in the form of a string:

```ruby
> @a, $a, a = 1,2,3
> defined? @a
#=> "instance-variable"
> defined? $a
#=> "global-variable"
> defined? a
#=> "local-variable"
> defined? puts
#=> "method"
```

If I assign `nil` to a variable, what do you think the return value will be when I call `defined?` with that variable?

```ruby
> defined? @b
#=> nil
> @b = nil
#=> nil
> defined? @b
#=> "instance-variable"
```

So, as long as the variable has been assigned with *something* (even `nil`), then `defined?` will be truthy. Only if the variable is uninitialized, it returns `nil`.

Of course, you can guess what happens when we set the variable's value to `false`.

```ruby
> @c = false
#=> false
> defined? @c
=> "instance-variable"
```

## Update: An improved solution

Prompted by [Valentin Baca's](https://dev.to/val_baca/comment/9coa) comment, I've reassessed my original solution. Do I really need to check whether or not the variable is initialised or is checking for `nil` enough?

`@happy.nil?` should suffice as I'm only interested in knowing that the variable is `nil` rather than `false`. (`false` and `nil` are [the only falsey values](http://ruby-doc.org/core-2.1.1/FalseClass.html) in Ruby.)

I think this version is more readable:

```ruby
def happy?
  @happy = post_complete? if @happy.nil?
  @happy
end
```

## Wrapping up

I now know that the `||=` style of memoization utilizes short-circuiting. If the left-hand side variable is `false`, then the right-hand part of the statement will be evaluated. If that's an expensive method call which always returns `false`, then the performance of my program would be impacted. So instead of `||=` I can check if the variable is initialized or I can check if it's `nil`.

And now I'm happy.

```ruby
def happy?
  @happy = post_complete? if @happy.nil?
  @happy
end
```
