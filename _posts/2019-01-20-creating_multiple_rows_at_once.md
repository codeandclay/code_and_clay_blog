---
layout: post
title: "Rails: How can I create multiple rows at once?"
---

I needed to seed my database with close to 20,000 rows. It took a _long_ time.

The problem was that I was creating the rows one at a time. However, `#create` can take an array which makes it possible to create multiple rows at once.

```ruby
Champion.create([
                  { name: 'Viktor', school: 'Durmstrang' },
                  { name: 'Fleur',  school: 'Beauxbatons' }
                ])
```
