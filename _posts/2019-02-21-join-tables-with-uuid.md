---
layout: post
title: UUID as column type in create_join_table
---

via [BigBinary](https://blog.bigbinary.com/2016/06/16/rails-5-create-join-table-with-uuid.html)

>Rails 5 has started supporting UUID as a column type for primary key, so create_join_table should also support UUID as a column type instead of only integers.

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration[5.0]
  def change
    create_join_table(:customers, :products, column_options: {type: :uuid})
  end
end
````
