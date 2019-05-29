---
layout: post
title: Understanding how to implement a feature that enables users to follow each other
---
Many sites enable users to follow and be followed by other users. It's a common feature. Dev.to implements it. Twitter implements it.

Michael Hartl demonstrates an implementation in [Chapter 14](https://www.railstutorial.org/book) of [The Ruby on Rails Tutorial](https://www.railstutorial.org/book). I didn't really understand what was going on at first. It wasn't until I'd gained a better understanding of ActiveRecord that I fully comprehended what was going on.

Here, I will explain my understanding of how it works.

Most of my following example is identical to Michael Hartl's implementation.

## What am I trying to implement?

Any user can follow any other user. At the same time, any user can be followed by any other user. The relationship is not reciprocal. That is, if one user follows another user, it is not necessary that the other follows back.

Essentially though, whether or not a user *follows* or is *followed*, I am describing two sides of the same relationship – one user points to the other.

- I will call a user that follows another user a *follower*.
- I will call a user that is followed by another user a *followee*.
- A user can have many followers. Likewise, a user can follow many followees.
- I will describe the relationship between a follower and followee as a *followship*.

## Followships

I now have three concepts: followers, followees and followships.

Followships sounds like the name of a table to me. This table will have two columns. One will point to a follower. The other will point to a followee. Each is referenced by its `id`.

![Followships table](/assets/images/followships_table.png)

Knowing this, I can generate a model named `Followship`. I will add an `id` column for followers and followees.

(`Followships` is a special kind of table called a *join table*. A join table is used to establish many to many relationships – where models can *have many* of each other.)

I'll add an index for both `follower_id` and `followee_id` since I will want to look up followships by both followers and followees. Also, I'll place a `unique` constraint on the rows because it doesn't make sense that a user follows another more than once.

```ruby
class CreateFollowships < ActiveRecord::Migration[5.2]
  def change
    create_table :followships do |t|
      t.integer :follower_id
      t.integer :followee_id

      t.timestamps
    end

    add_index :followships, :follower_id
    add_index :followships, :followee_id
    add_index :followships, [:follower_id, :followee_id], unique: true
  end
end
```

Before establishing the relationship in the model, there's a couple of things to be aware of.

Firstly, an association name does not have to share the name of the model it points to. Let's say I have the classes `User` and `Project`. A user can have many projects:

```ruby
class User < ApplicationRecord
  has_many :projects
end
```

Each `Project` belongs to a single `User`. I could establish the relationship like this:

```ruby
class Project < ApplicationRecord
  belongs_to :user
end
```

But I would like to refer to the user as the *owner* of the project.

So how about this?

```ruby
class Project < ApplicationRecord
  # This will not work!
  belongs_to :owner
end
```

Using the above, Rails believes that I want to reference a model name `Owner`. It infers the class name of the associated object from the name I've provided.

So I need to be more explicit.

```ruby
class Project < ApplicationRecord
  belongs_to :owner, class_name: 'User'
end
```

Now that I know this technique, I can create `belongs_to` associations for a `followship`'s `follower` and `followee`. Despite their names, both columns point to `User`.

```ruby
class Followship < ApplicationRecord
  belongs_to :follower, class_name: 'User'
  belongs_to :followee, class_name: 'User'
```

Note: because both of these are `belongs_to` associations, it is the responsibility of `Followship` to record the foreign ids of the rows to which it belongs.

From the [Rails guides](https://guides.rubyonrails.org/association_basics.html#belongs-to-association-reference):

> ## 4.1 belongs_to Association Reference
The belongs_to association creates a one-to-one match with another model. In database terms, this association says that this class contains the foreign key. If the other class contains the foreign key, then you should use has_one instead.

One final step here. I want to ensure that both the `follower_id` and `followee_id` is present. There cannot be a relationship if one is without the other.

The class ends up looking like this:

```ruby
class Followship < ApplicationRecord
  belongs_to :follower, class_name: 'User'
  belongs_to :followee, class_name: 'User'

  validates :follower_id, presence: true
  validates :followee_id, presence: true
end
```

## Catching breath

So what have I done so far?

- I have created a model named `Followship`.
- Each row in `followships` points to a `follower_id` and a `followee_id`.
- I've indexed by both columns and ensured each pairing is unique.
- I've made sure both ids are present in every row.
- I've established that `Followship` belongs to a `follower` and `followee`. (In doing so, I've had to tell Rails that in both cases I'm actually pointing to `User`.)

## Connecting User to the Followship

From the point of view of `User`, followships come in two flavours.

Depending on which way you look at it, a `followship` can either be a relationship in which the user is a `follower` of another or it can be a relationship in which the user is a `followee` of the other.

In the first case, I want to establish that the user has many relationships in which it follows another user. I want to tell Rails that it should reference the `user` by `follower_id` in the `followships` table.

I want the `User` model to declare to Rails:

"I can have many `followships` where I am *following* another user.

"I will refer to these `followships` as `follower_followships`.

"In this case, I will refer to myself as a `follower`.

"Store my `id` in `followship`'s' `follower_id` field."

```ruby
# app/models/user.rb
# ...
# A follower is a user than follows another user.
has_many :follower_followships,
  class_name: "Followship",
  foreign_key: "follower_id",
  dependent: :destroy
```

(Like the `project`/`owner` example above, I have given the association a name that is different from the class it points to. I also know that because the other class is on the `belonging` side of the relationship, it is the other class that stores the `id`. In this case, the `user id` is stored as the foreign key `follower_id`.

Notice that I have also made the relationship dependent on the user existing. If the user is destroyed, so too are any rows in `followships` that correspond to it.)

All good but I haven't yet pointed to another user. So far, `User` points only to the join table.

## Through

Join tables are called such because they join one table to another. They are the glue in a `has_many`/`has_many` relationship. (They provide the `belongs_to` association that stores the foreign keys). Relationships are made *through* the join table.

I know that each row has a `follower` and `followee` column. I know that if my user follows another, its `id` will be stored in the `follower_id` column. I also know that in each row my user is recorded as a `follower`, there will be the corresponding `followee_id`.

Now I want the `User` model to declare:

"I can have many *followees*.

"I can follow many users. Look up my *follower_followships*. If you see any, the corresponding *followee_id* in those rows is the id of the *followee* I am following."

Like so:

![Follower followships](/assets/images/follower_followships.png)

I declare the `has_many` association with `followees` *through* the previously established `follower_followships` association:

```ruby
# app/models/user.rb
# To see the users that the user follows, we reference them through the join
# table.
has_many :followees, through: :follower_followships
```
So far, the `User` looks like this:

```ruby
class User < ApplicationRecord
  # A follower is a user than follows another user.
  has_many :follower_followships,
    class_name: "Followship",
    foreign_key: "follower_id",
    dependent: :destroy

  # We reference the users that the user follows through the join
  # table.
  has_many :followees, through: :follower_followships
```


Now, I need to do the reverse. I need to establish the other flavour of `followship` where the user is the `followee`. A user can be a `follower` but when other users point to it, it is also a `followee`.

```ruby
# A followee is a user that is followed by another user.
has_many :followee_followships,
  class_name: "Followship",
  foreign_key: "followee_id",
  dependent: :destroy
```

And I can look up `followers` through the join table:

```ruby
# To see the users that follow a user, we reference them through the join
# table.
has_many :followers, through: :followee_followships
```

And now, `followees` can look up `followers` through the join table.

![Followee followships](/assets/images/followee_followships.png)

## A couple of convenience methods

I can now access the users which a user follows by calling `self.followees`.

Knowing that, I can create two methods that allow me to follow and unfollow users conveniently.

```ruby
def follow(user)
  followees << user
end

def unfollow(followed_user)
  followees.delete followed_user
end
```

The final class looks like this:

```ruby
class User < ApplicationRecord
  # A follower is a user than follows another user.
  has_many :follower_followships,
    class_name: "Followship",
    foreign_key: "follower_id",
    dependent: :destroy

  # To see the users that the user follows, we reference them through the join
  # table.
  has_many :followees, through: :follower_followships

  # A followee is a user that is followed by another user.
  has_many :followee_followships,
    class_name: "Followship",
    foreign_key: "followee_id",
    dependent: :destroy

  # To see the users that follow a user, we reference them through the join
  # table.
  has_many :followers, through: :followee_followships

  def follow(user)
    followees << user
  end

  def unfollow(followed_user)
    followees.delete followed_user
  end
end
```

## In the wild

`Dev.to`'s [implementation](https://github.com/thepracticaldev/dev.to/blob/master/app/models/follow.rb) is more complicated as it deals with polymorphic associations. However, something similar can be seen in [mentor_relationship.rb](https://github.com/thepracticaldev/dev.to/blob/master/app/models/mentor_relationship.rb) and in the relevant part of [User.rb](https://github.com/thepracticaldev/dev.to/blob/master/app/models/user.rb):

```ruby
has_many :mentor_relationships_as_mentee,
       class_name: "MentorRelationship", foreign_key: "mentee_id", inverse_of: :mentee
has_many :mentor_relationships_as_mentor,
       class_name: "MentorRelationship", foreign_key: "mentor_id", inverse_of: :mentor
has_many :mentors,
       through: :mentor_relationships_as_mentee,
       source: :mentor
has_many :mentees,
       through: :mentor_relationships_as_mentor,
       source: :mentee
```

## Wrapping up

### What have I done?

- I've created a model called `Followship`.
- I've established the `Followship` model *belongs* to a `follower` and `followee`.
- The corresponding table `followships` is a join table. Users will relate to each other as `followers` and `followees` through this table.
- Each row in the `followships` table stores a `follower_id` and a `followee_id`.
- I've established the a `User` can have many `follower_followships` and many `followee_followships`.
- A `follower_followship` (a `followship` in which the user does the *following*) is where the user `id` is stored under `followship`'s `follower_id` column.
- A `followee_followhsip` (a `followship` in which the user does is *followed*) is where the user's `id` is stored as a foreign key under `followship`'s `followee_id` column.
- A user can have many `followees` through `follower_followships`.
- A user can have many `followers` through `followee_followships`.
- I've added a couple of convenience methods to follow and unfollow users.

### What did I need to understand to get here?
- A model that *belongs to* another model is responsible for storing the other model's id as a foreign key.
- I can establish many to many relationships *through* a join table.
- I can give a relation a name that is different from the name of the class it refers to.

