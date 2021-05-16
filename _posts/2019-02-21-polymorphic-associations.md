---
layout: post
title: Polymorphic associations
---

A model can associate with other models by way of a polymorphic association. This can be useful when the associated models are not of the same class but share a common interface, or when a model shares the same type of relationship with multiple other models.

For example, three models `Post`, `Project`, and `Issue` may have many comments.

Instead of creating a different comment model for each of these three models (`PostComment`, `ProjectComment`…) it would be much more straightforward to have a single `Comment` model and establish a relationship to a *subject*.

```ruby
create_table :comments do |t|
  t.text :body
  t.references :subject, polymorhic: true
```
The `subject` could be a `Post`, `Project`, `Issue` or any other model we wish to attach a comment to.

The polymorphic option, instructs the database to store the associated object's type. The above migration creates a `:subject_type` column.

The relationships are established in the models like so:

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
