---
layout: post
title: Running the console in the test environment helped me debug a test
---

I wrote a test and I couldn't figure out why it was failing. I'd tried the code in the console and proved it should work.

```ruby
test "a profile can have a external links" do
  user = User.create(name: "test user")
  user.profile = Profile.create
  10.times do
    user.profile.external_links << ExternalLink.create(url: "https://www.example.com/test")
  end
  assert_equal 10, user.profile.external_links.count
end
```

An hour later, I remembered it's possible to set the environment under which the console is run and I immediately discovered `user` was not saving to the database because it was not unique. A row in the fixtures already had that name.
