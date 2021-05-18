---
layout: post
title: Dependent destroy when another table uses a different foreign_id
tags: archive
---

I have two models `Comment` and `User`. A user can have many comments and a comment has one user -- referred to as its author.

Here are my migrations:

```ruby
class CreateUsers < ActiveRecord::Migration[5.2]
  def change
    create_table :users, id: :uuid, default: 'gen_random_uuid()' do |t|
      t.string :username, null: false, index: { unique: true }
      t.timestamps
    end
  end
end
```

```ruby
class CreateComments < ActiveRecord::Migration[5.2]
  def change
    create_table :comments do |t|
      t.text :body, null: false, default: ""
      t.references :author, index: true, foreign_key: { to_table: :users }, type: :uuid
      t.timestamps
    end
  end
end
```

And my models:

```ruby
class Comment < ApplicationRecord
  belongs_to :author, class_name: "User"
end

class User < ApplicationRecord
  has_many :comments, dependent: :destroy
end
```

I can create a new comment as follows:

```ruby
#> author = User.create(username: 'Test')
#> comment = Comment.create(author: author)

#> Comment.create(author: author)
#> Comment.count
#=> 1
```

My intention is that destroying the author will also remove the comment from the db. However, I'm running into difficulty.

```ruby
#> author.destroy
```

…results in an error:

```
ActiveRecord::StatementInvalid: PG::UndefinedColumn: ERROR:  column
comments.user_id does not exist LINE 1: SELECT "comments".* FROM
...
```

The database is looking for a `user_id` column in the comments table when I know it doesn't exist.

The solution is that I need to specifiy the column name when using the `destroy` action:

```ruby
has_many :comments, dependent: :destroy, foreign_key: :author_id
```

It irks me that `users` has to know what the other table calls it. I don't yet know if that's a limitation of ActiveRecord/SQL or if there's a more OOP approach. I expect it's the former.
