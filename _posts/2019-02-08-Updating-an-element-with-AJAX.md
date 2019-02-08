---
layout: post
title: Updating an element with AJAX and UJS
---

UJS makes AJAX super easy.

Here, I want to update an element in response to a `POST` request.

`remote: true` ensures that the form – which `button_to` generates – is submited by AJAX.

```ruby
def base_tag_link(symbol: '&#65293;', body:, params: {}, css_class: 'badge')
    button_to "#{sanitize(symbol)} #{body}", send(request_view.to_sym, params), class: css_class, remote: true
end
```
My controller action is simple.
```ruby
def create
  respond_to do |format|
    format.js
  end
end
```

All I need to do is then create `create.js.erb` file in the corresponding view directory.

```ruby
document.querySelector('.selected-tags')
        .innerHTML = "<%= escape_javascript(render 'shared/selected_tags') %>"
```

The partial is rendered to a string and the JS updates `.selected-tags` with it as its content.
