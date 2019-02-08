---
layout: post
title: Prevent controllers from loading all helper modules
---
I have written some helper methods intended for use in all views. However, some views require specific behaviour. I can override helper methods in the controller specific helper modules. But first, I need to configure Rails not to load all helper modules for all controllers.

In `config/application.rb`, I need to set:

```ruby
# Prevent controller from loading all the helpers
config.action_controller.include_all_helpers = false
```

This way I define generic helpers in `application_helper` and override any methods I need to in a controller's corresponding helper module.
