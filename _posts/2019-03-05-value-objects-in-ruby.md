---
Layout: post
Title: Value objects in Ruby
---

**(This post is also published on [dev.to](https://dev.to/updated_tos/value-objects-in-ruby-13p))**

## What is a value object?

>A small simple object, like money or a date range, whose equality isn't based on identity.
[Martin Fowler](https://martinfowler.com/eaaCatalog/valueObject.html)

Objects in Ruby are usually considered to be *entity objects*. Two objects may have matching attribute values but we do not consider them equal because they are distinct objects.

In this example `a` and `c` are not equal:

```ruby
class Panserbjorn
  def initialize(name)
    @name = name
  end  
end

a = Panserbjorn.new('Iorek')
b = Panserbjorn.new('Iofur')
c = Panserbjorn.new('Iorek')

a == c #=> => false

# Three distinct objects:
a.object_id #=> 70165973839880
b.object_id #=> 70165971554200
c.object_id #=> 70165971965460
```

*Value objects* on the other hand, are compared by value. Two different value objects are considered equal when their attribute values match.

`Symbol`, `String`, `Integer` and `Range` are examples of value objects in Ruby.

Here, `a` and `c` are considered equal despite being distinct objects:

```ruby
a = 'Iorek'
b = 'Iofur'
c = 'Iorek'

a == b #=> false
a == c #=> true

# Three distinct objects:
a.object_id #=> 70300461022500
b.object_id #=> 70300453210700
c.object_id #=> 70300461053840
```

## How can I create a value object?

Say I want a class to represent the days of the week and I also want instances of that class to be considered equal if they represent the same day. A *Sunday* object should equal another *Sunday* object. A *Monday* object should equal another *Monday* object, etc…

I might begin with the following class:

```ruby
class DayOfWeek
  DAYS = {
    1 => 'Sunday',
    2 => 'Monday',
    3 => 'Tuesday',
    4 => 'Wednesday',
    5 => 'Thursday',
    6 => 'Friday',
    7 => 'Saturday'
  }.freeze

  def initialize(day)
    raise ArgumentError, 'Day outside range' unless (1..7).cover?(day)

    @day = day
  end

  def to_i
    day
  end

  def to_s
    DAYS[day]
  end

  private

  attr_accessor :day
end
```

Now, I am going to instantiate three objects to represent the days of the week on which I eat pizza, pay the milk man, and put out the recycling for collection:

```ruby
pizza_day = DayOfWeek.new(5)
milk_money_day = DayOfWeek.new(2)
recycling_collection_day = DayOfWeek.new(5)
```

I know that I eat pizza for dinner the same day that I put out the recycling. I consider these objects to represent the same thing: Thursday. They should be equivalent. But they're not:

```ruby
pizza_day == recycling_collection_day #=> false
```

That's because they're not yet value objects. `#==` compares the identities of the objects.

I should override `#==`. I will use pry to find out where the method comes from so we can see how it derives its current behaviour.

```ruby
pizza_day.method(:==).owner #=> BasicObject
```

`DayOfWeek` inherits `#==` from `BasicObject`.

The page for `BasicObject#==` [states](https://ruby-doc.org/core-2.5.0/BasicObject.html#method-i-3D-3D):

> == returns true only if obj and other are the same object. Typically, this method is overridden in descendant classes to provide class-specific meaning.

Aha! The *class specific meaning* in this case is I want to compare its instances by value.

I know that these objects expose an integer. It makes sense to compare against that but I don't want to compare a day with an actual integer. Thursday should not be equivalent to the number *5*.

I also know that a `DayOfWeek` exposes a string as well. It follows that any equivalent days would return matching string and integer values:

```ruby
class DayOfWeek
  # ...

  def ==(other)
    to_i == other.to_i &&
    to_s == other.to_s
  end

  alias eql? ==

  # ...
end
```

I have aliased `#eql?` to `#==`. The `BasicObject` [documentation](https://ruby-doc.org/core-2.5.0/BasicObject.html#method-i-3D-3D) explains:

> For objects of class Object, eql? is synonymous with ==. Subclasses normally continue this tradition by aliasing eql? to their overridden ==

Bingo! We have value objects. `pizza_day` and `recycling_collection_day` are considered equivalent:

```ruby
pizza_day == recycling_collection_day #=> true
```

I could override other comparison methods, `#<==>`. `<=`, `<`, `==`, `>=`, `>` and `between?` as it makes sense to say that `Monday` is less than `Tuesday` or `Friday` is greater than `Thursday` but I have decided that's not needed for now.

There is however, one more important step that I need to implement. These objects are equivalent, so when used as a hash key I would expect them to point to the same bucket.

The `Hash` [documentation](https://ruby-doc.org/core-2.6.0.preview2/Hash.html#class-Hash-label-Hash+Keys) suggests:

>Two objects refer to the same hash key when their hash value is identical and the two objects are eql? to each other.

>A user-defined class may be used as a hash key if the hash and eql? methods are overridden to provide meaningful behavior. By default, separate instances refer to separate hash keys.

Following that advice, I need to change the default behaviour of `#hash`. I already know that integers in Ruby are value objects. I can see that equivalent integers always return the same `#hash`. 

```ruby
a = 1
b = 1

a.object_id #=> 3
b.object_id #=> 3
1.object_id #=> 3

1.hash == 2.hash #=> false
[a,b,1].map(&:hash).uniq.count #=> 1
101.hash == (100 + 1).hash #=> true
```

The same goes for strings:

```ruby
a = 'Svalbard'
b = 'Svalbard'

# Note the different object ids:
a.object_id #=> 70253833847520
b.object_id #=> 70253847146940
'Svalbard'.object_id #=> 70253847210020

# The hash values of equivalent strings match:
'Svalbard'.hash == 'Bolvanger'.hash #=> false
[a,b,"Svalbard"].map(&:hash).uniq.count #=> 1
'Svalbard'.hash == ('Sval' + 'bard').hash #=> true
```

I will generate the hash using its string and integer properties.

```ruby
  def ==(other)
    to_i == other.to_i
  end

  alias eql? ==

  def hash
    to_i.hash ^ to_s.hash
  end
```

Per the [example](https://ruby-doc.org/core-2.5.3/Hash.html#class-Hash-label-Hash+Keys) in the documentation, I've used the XOR operator (`^`) to derive the new hash value.

Now that I have overridden `#hash`, I can see that equivalent `DayOfWeek` instances point to the same bucket:

```ruby
day_1 = DayOfWeek.new(1)
day_2 = DayOfWeek.new(1)

day_1 == day_2 #=> true

notes = {} #=> {}
notes[day_1] = 'Rest'
notes[day_2] = 'Party'

notes.length #=> 1
notes #=> {#<DayOfWeek:0x00007fa193e44170 @day=1>=>"Party"}
```

## Structs

If I want multiple value objects, I might have to override `#hash` and `#==` for each class.

I could decide to use [structs](https://ruby-doc.org/core-2.5.0/Struct.html) instead.

>A Struct is a convenient way to bundle a number of attributes together, using accessor methods, without having to write an explicit class.

Structs are value objects by default. Of course we now have an idea of how this works. The [documentation](https://ruby-doc.org/core-2.5.0/Struct.html#public-instance-method-details) explains:

>Equality—Returns true if other has the same struct subclass and has equal member values (according to Object#==).

Just as we thought!

```ruby
DayOfWeek = Struct.new(:day) do
  DAYS = {
    1 => 'Sunday',
    2 => 'Monday',
    3 => 'Tuesday',
    4 => 'Wednesday',
    5 => 'Thursday',
    6 => 'Friday',
    7 => 'Saturday'
  }.freeze

  def to_s
    DAYS[day]
  end

  def to_i
    day
  end
end

a = DayOfWeek.new(1)
b = DayOfWeek.new(2)
c = DayOfWeek.new(1)

a == c #=> true
a == b #=> false
```

## Summary

We now know the difference between an entity object and a value object. We have learned that we need to override both `#hash` and `#==` if our value objects are to be used as hash keys. And, we have learned that structs provide value object behaviour straight out of the box.
