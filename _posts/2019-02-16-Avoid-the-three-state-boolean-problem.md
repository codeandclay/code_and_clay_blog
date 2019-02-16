---
layout: post
title: Avoid the three-state boolean problem
---

via [thoughtbot](https://thoughtbot.com/blog/avoid-the-threestate-boolean-problem)

>The NOT NULL constraint means that this migration will fail if we have existing users, because Postgres doesn’t know what to set the column values to other than NULL. We get around this by adding a default value:

```ruby
add_column :users, :admin, :boolean, null: false, default: false
```
