---
layout: post
title: Finding rows in a table that point to rows in another that match all the specified column values…
---
I have models named `Issue` and `Label`. Each `issue` can have many `labels`. Each `label` can have many `issues`.

I am trying to build a query that will return an issue that contains all of the supplied labels.

For instance, if I supply `['bug', 'fix', 'enhancement']` I want issues that have all three of those three labels at least.

Currently, I have:

```ruby
labels = ['bug', 'fix', 'enhancement']
Issue.joins(:labels).where(labels: { name: labels }).distinct
```

But this is insufficient as it returns issues that have at least one of the label names. I see this is because is because an `IN` operator is generated:

```
WHERE "labels"."name" IN ('bug', 'fix', 'enhancement')
```

And from here on, I'm lost. I imagine I might be able to use `#inject` to build the query from the array…
