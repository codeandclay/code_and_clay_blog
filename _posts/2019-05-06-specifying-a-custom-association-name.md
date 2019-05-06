---
title: Giving an association a custom name
layout: post
---

Sometimes, it may be desirable to reference an association with a name that differs from the one generated from the table name.

I have a model named `Steps`. Each `step` can have many `steps`. Each `step` belongs to one other parent `step`.

```ruby
class Step < ApplicationRecord
  belongs_to :step
  has_many :steps
end
```

```ruby
a = Step.create
b = Step.create
c = Step.create

a.steps >> b
a.steps >> c
```

I could get the parent step of `b` like so:

```ruby
b.step
# => #<Step:0x00007fdf24b916e8
```

I don't think that's intuitive enough though. Since I'm asking for the parent step, it should be more like:

```ruby
b.parent
# => #<Step:0x00007fdf24b916e8
```

Setting up the custom name is simple. I can supply the name of my choosing. Then, all I need to do is specify the class name and foreign id in the options.

```ruby
class Step < ApplicationRecord
  belongs_to :parent, class_name: 'Step', foreign_key: :step_id
  has_many :steps
end
```
