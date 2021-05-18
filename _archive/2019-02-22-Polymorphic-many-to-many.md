---
layout: post
title: Polymorphic many to many
tags: archive
---
I found this to be a bit of a brain twister at first but it's really quite simple.

I have three models, `Comment`, `Project` and `Badge`. Users can react to a comment or project by attaching a badge. The interaction is similar to 'liking' a tweet or post except that it's a bit more open ended. I have in mind that users will be able to choose from a list of emojis: thumbs up, smiley face, rocket, confetti, etc… Badges are unique and can be attached to multiple comments/projects and individual comments/projects multiple times.

I'm not sure I have the semantics right but I've chosen to name my join table 'reactions'. A comment or project would be a reaction target.

It took me some time to realise where the polymorphism would happen. I initially thought that I should declare a badge's relation to the join table as polymorphic. Of course, a badge will always point to the join table so its type is never in question. However, objects that `reaction_target` points to can differ in type. The polymorphic relationship is between the join table and the `reaction_target`.

```ruby
class CreateReactions < ActiveRecord::Migration[5.2]
  def change
    create_table :reactions do |t|
      t.references :reaction_target, polymorphic: true
      t.references :badge
    end
  end
end
```

```ruby
class Reaction < ApplicationRecord
  belongs_to :badge
  belongs_to :reaction_target, polymorphic: true
end
```

```ruby
class Comment < ApplicationRecord
#...
  has_many :reactions, as: :reaction_target
  has_many :badges, through: :reactions
end
```

## Update

This works fine from the pov of the `reaction_target` but I cannot reference the polymorphic associations from the pov of a badge:

```ruby
> Badge.first_or_create(name: "test").reaction_targets
NameError: uninitialized constant Badge::ReactionTarget
```
