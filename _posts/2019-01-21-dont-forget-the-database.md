---
layout: post
title: "Don't forget the database!"
---

I have two models `Issue` and `Label`. I want to establish a relationship where issues can have many labels and labels can have many issues.

I have created a join table named tags. (I have chosen the name because I will later use it to establish a similar relationship between issues and a another model.)


```ruby
class CreateTags < ActiveRecord::Migration[5.2]
  def change
    create_table :tags do |t|
      t.integer :issue_id
      t.integer :label_id
      t.timestamps
    end

    add_index :tags, [:issue_id, :label_id]
  end
end
```

I have established the relationships in the models:

```ruby
class Tag < ApplicationRecord
  belongs_to :issue
  belongs_to :label
end

class Issue < ApplicationRecord
  has_many :tags
  has_many :labels, through: :tags
end

class Label < ApplicationRecord
  has_many :tags
  has_many :issues, through: :tags
end
```

I'm not aware that I've missed any steps. However, I am unable to add a label to an issue:

```ruby
> label = Label.create
=> #<Label id: 980190966, name: nil, issue_id: nil, created_at: "2019-01-25 10:18:29", updated_at: "2019-01-25 10:18:29">

> issue = Issue.create
=> #<Issue id: 980190967, assigned: nil, description: nil, created_at: "2019-01-25 10:19:04", updated_at: "2019-01-25 10:19:04">

> issue.labels << label
Traceback (most recent call last):
        1: from (irb):13
ActiveModel::UnknownAttributeError (unknown attribute 'issue_id' for Tag.)
```

I can see in `schema.rb` that there is an `issue_id` column in tags:

```ruby
create_table "tags", force: :cascade do |t|
  t.integer "issue_id"
  t.integer "label_id"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.index ["issue_id", "label_id"], name: "index_tags_on_issue_id_and_label_id"
end
```

So what is preventing me from adding the label to the issue? Have I missed a step? What am I doing wrong?

The problem is not what I think it is. I am running the console in the test environment. Switching to the development environment, I *can* add the label to the issue.

Why is that? Because I ran `rails db:reset` on it five minutes earlier.

Previously I had established the relationships of my two models using HABTM. The database is set up for that. I need to reset the test db.

```
rails db:drop db:create db:migrate RAILS_ENV=test
```

I can now add the label to the issue in the test environment and my existing tests now pass.

I often check `schema.rb` to see what the database looks like. It's a mistake to assume the database looks the same. Had I looked at the database itself, I would have noticed that it was not as I expected it to be.
