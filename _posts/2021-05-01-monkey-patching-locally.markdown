---
title: Monkey-patching locally
layout: post
---

In Ruby, classes are open. They can be modified at any time. You can add new methods to existing classes and even re-define methods. Rubyists call this *monkey-patching*.

In the example below, I've added `String#word_count` to return the number of words in a string.

```ruby
class String
  def word_count
    self.split.count
  end
end
```

```ruby
"Don't put your blues where your shoes should be.".word_count # => 9
```

However, this change is global. What happens if someone else defines their own version of `String#word_count` based on different rules? Say, they might want to count the individual parts of a contraction, or ignore digits and any non-word characters.

[Refinements](https://docs.ruby-lang.org/en/3.0.0/doc/syntax/refinements_rdoc.html) allow us to monkey-patch classes and modules locally.

Here, I've created a new module and used `Module#refine` to add `word_count` to `String`.

```ruby
module MyStringUtilities
  refine String do
    def word_count
      self.split.count
    end
  end
end
```

The refinement isn't available in the global scope:

```ruby
"Take my shoes off and throw them in the lake.".word_count # => NoMethodError: undefined method `word_count' for…
```

To activate the refinment, I need to use `using`:

```ruby
using MyStringUtilities

"Take my shoes off and throw them in the lake.".word_count # => 10
"A pseudonym to fool him".word_count # => 5
```

If I activate the refinement within a class or module, the refinement is available until the end of the class or module definition.

Below, the refinement is not available at the top level because it is scoped to `MyClass`.

```ruby
module MyStringUtilities
  refine String do
    def word_count
      self.split.count
    end
  end
end

class MyClass
  using MyStringUtilities

  def self.number_of_words_in(string)
    string.word_count
  end
end

"Out on the wily, windy moors".word_count #=> NoMethodError

MyClass.number_of_words_in("Out on the wily, windy moors") #=> 6
```
