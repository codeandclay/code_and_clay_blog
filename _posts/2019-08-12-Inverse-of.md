---
title: Use inverse_of when creating associations with non-standard naming
layout: post
---

I have two models 'Customer' and 'GreenPencil'.

A customer can have many green pencils. But, as far as the customer is concerned, these are just *pencils*. It makes sense for me to call the relationship that.

I first establish that a green pencil belongs to a customer.

```ruby
class GreenPencil < ApplicationRecord
  belongs_to :customer, required: true
  #...
```

Now, when I establish the other side of the relationship (remember, each `GreenPencil` has a foreign key pointing to its `Customer`) I can do this:

```ruby
class Customer < ApplicationRecord
  has_many :pencils,
           class_name: 'GreenPencil',
           inverse_of: :customer

```

I'm telling Rails that a `Customer` has many `pencils`, the associated class is `GreenPencil` and that this relationship is the other side of `GreenPencil`'s relationship with a `customer` that I established at the start.

Rails usually sets `inverse_of` on all associations itself. However, because the class name differs from the association name, we need to specifiy it explicitly.

Rails will infer the inverse relationship if we set `foreign_key` but I think `inverse_of` is clearer to the reader.
