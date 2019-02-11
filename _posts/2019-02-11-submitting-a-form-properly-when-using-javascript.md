---
layout: post
title: Submitting a form properly when using javascript
---

I have a search form in a view.

```ruby
<%= form_with url: issues_path, class: 'filter-box' do %>
  <%= text_field_tag(:filter_by, '', id: 'filter-text-field') %>
<% end %>
```

When the form is submitted, the following action is run:

```ruby
class IssuesController < ApplicationController
  ...

  def create
    @labels = Label.by_search_term(params[:filter_by])
    respond_to do |format|
      format.js
    end
  end
end
```

All works well.

However, I want the action to be triggered with every keystroke. To that end, I have added the following javascript.

```html
<script>
  document.getElementById('filter-text-field').addEventListener('keyup', function(){
    document.querySelector('.filter-box').submit()
  })
</script>
```

But when the form is submitted on key up it breaks and I get an error:
```
ActionController::UnknownFormat in IssuesController#create.
```

The problem is that no format is specified in the request when the javascript submits the form.

The solution is to use Rails's own javascript helper `Rails.fire`:

```html
    <script>
      document.getElementById('filter-text-field').addEventListener('keyup', function(){
        Rails.fire(this.form, 'submit')
      })
    </script>
```

It's implemented [here](https://github.com/rails/rails/blob/master/actionview/app/assets/javascrpts/rails-ujs/start.coffee) and [here](https://github.com/rails/rails/blob/master/actionview/app/assets/javascripts/rails-ujs/utils/event.coffee).

