---
layout: post
title: Be Careful Assigning to has_one Relations
---

via [Andy Croll](https://andycroll.com/ruby/be-careful-assigning-to-has-one-relations/)

>Most of the time, when building relationships between models, you typically use has_many and belongs_to. There are some circumstances where a has_one relationship is more appropriate.
>
>However, the behaviour of has_one has some quirks that make it a little trickier to deal with.
>
>When you assign a new instance of an associated model to its has_one model the existing instance is removed from the association and causes a permanent change to be written to the database. This happens whether the new model is valid or not.

(…)

```ruby
class Biography < ApplicationRecord
  belongs_to :person, optional: true

  validates :text, presence: true
end

class Person < ApplicationRecord
  has_one :biography
end

person = Person.create(name: "Andy Croll")
=> #<Person id: 1, name: "Andy Croll" ...>
person.create_biography(text: "Writes emails")
=> #<Biography id: 1, person_id: 1, text: "Writes emails" ...>
person.biography = Biography.new("Looks handsome")
=> #<Biography id: 2, person_id: 1, text: "Looks handsome" ...>
person.reload.biography
=> #<Biography id: 2, person_id: 1, text: "Looks handsome" ...>
Biography.count
=> 2
Biography.first
=> #<Biography id: 1, person_id: nil, text: "Looks handsome" ...>
```
