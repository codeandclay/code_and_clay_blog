---
layout: post
title: Polymorphic associations
---

A model can associate with other models implicitly by way of a polymorphic association. For instance, three models 'Post', 'Project', and 'Issue' may have many comments. Each model could be said to be the `subject`.

```ruby
create_table :comments do |t|
  t.text :body
  t.references :subject, polymorhic: true
```

The polymorphic option, instructs the database to store the associated object's type. Here, it would create a `:subject_type` column.

The relationships are established like so:

```ruby
class Comment < ApplicationRecord
  belongs_to :subject, polymorphic: true
end

class Post < ApplicationRecord
  has_many :comments, as: subject
end

class Project < ApplicationRecord
  has_many :comments, as: subject
end

class Issue < ApplicationRecord
  has_many :comments, as: subject
end
```
