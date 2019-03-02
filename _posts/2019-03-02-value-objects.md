---
layout: post
title: Value objects
---
>A small simple object, like money or a date range, whose equality isn't based on identity.
[Martin Fowler](https://martinfowler.com/eaaCatalog/valueObject.html)

`Symbol`, `String`, `Integer` and `Range` are example of value objects.

[Value Objects in Ruby on Rails](https://revs.runtime-revolution.com/value-objects-in-ruby-on-rails-9df64bc8db34)

## Notes to [this presentation](https://www.youtube.com/watch?v=iTjurFO1l5Q).

March is not 3.
3 is not March.
March is March.

March is a range of days, 1st – 31st.

Examples of value objects:

- Date, Time
- Weight, Height
- Duration
- Temperature
- Address
- Money

A value object has:

- Identity based on state
  - a dollar is a dollar
  - Nov 5 is Nov 5
  - 98.6F is 98.6F
  - 10 Downing Street is 10 Downing Street
- Equality based on type and state
  - 98.6F != $98.60

They are immutable -- or should be treated as such -- and typically do not have setters.

### Scalar values example

```ruby
class Patient
  attr_accessor :height, :weight
end

patient.height #=> 65
patient.weight #=> 90
```

What do these values represent? What are their units?

```ruby
class Height
  def initialize(inches)
    @inches = inches
  end

  def inches
    @inches
  end

  def centimetres
    inches * 2.54
  end
end

class Weight
  def initialize(pounds)
    @pounds = pounds
  end

  def pounds
    @pounds
  end

  def kilograms
    pounds / 2.2
  end
end

class Patient
  attr_accessor :height_inches, :weight_pounds

  def height
    Height.new(@height_inches)
  end

  def weight
    Weight.new(@weight_pounds)
  end
end

patient.weight.pounds #=> 100
patient.weight.kilograms #=> 45.4545454545

patient.height.inches #=> 65
patient.height.centimetres #=> 165.1
```

### Equality

```ruby
a = Weight.new(100)
b = Weight.new(110)
c = Weight.new(100)

a == b #=> false
a == c #=> false
```

`a` and `c` are not equal because they are different objects.

```ruby
class Weight
  # ...

  def hash
    pounds.hash
  end

  def eql?(other)
    self.class == other.class &&
      self.pounds == other.pounds
  end
  alias :== eql?
end
```

We need to override `#hash` when we override `#==`.

The weight class now looks like:

```ruby
  attr_reader :pounds

  def initialize(pounds)
    @pounds = pounds
  end

  def kilograms
    pounds / 2.2
  end

  def hash
    pounds.hash
  end

  def eql?(other)
    self.class == other.class &&
      self.pounds == other.pounds
  end
  alias :== eql?
end
```

### Refactor to a struct

```ruby
Weight = Struct.new(:pounds) do
  def kilograms
    pounts/2.2
  end
end

a = Weight.new(100)
b = Weight.new(110)
c = Weight.new(100)

a == b #=> false
a == c #=> true
```

But structs are mutable.

### Value object gem

You can use `value_object` gem.

```ruby
class Timespan < ValueObject::Base
  has_fields :start_time, :end_time

  def duration
    Duration.new(end_time - start_time)
  end

  def contains?(time)
    (start_time..end_time).cover?(time)
  end

  def overlays?(other)
    contains?(other.start_time) || contains?(other.end_time)
  end
end
```
