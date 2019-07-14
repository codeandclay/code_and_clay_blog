---
title: Nested attributes
layout: post
---

Sometimes it's desirable to update one model's attributes through an associated parent. Say, I might provide a form that asks a user for their name and email address but the user's name is stored in the `users` table and their `email` address is recorded in another.

Rails provides the `accepts_nested_attributes_for` macro.

```ruby
class User < ApplicationRecord
    has_one :contact_information, dependent: :destroy, required: true
    accepts_nested_attributes_for :contact_information
```

This allows me to update a user's contact information like so:

```ruby
user.update_attributes( contact_information_attributes: {
    phone_number: '123',
    email_address: 'hey@you.com
})
```

Instead of the headache of multiple forms and multiple controllers, I can now *nest* the contact information fields within the user form.

```erb
<%= bootstrap_form_for(@user) do |f| %>
  <%= f.text_field :name, required: true, autocomplete: "name" %>
  # ...
  <%= f.fields_for :contact_information do |ff| %>
      <%= ff.text_field :email_address, %>
      # ...
```

Finally, I need to specify the associated model's attributes as allowable parameters in the parent resource's controller:

```ruby
def strong_params
    params.require(:user).permit(
      :name,
      contact_information_attributes: %i[
        email_address
      ]
    )
end
```
