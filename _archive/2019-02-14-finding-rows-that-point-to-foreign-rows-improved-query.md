---
layout: post
title: An improved query
---

I needed a way to look up rows in one table that pointed towards all the specified rows in another. I posted about my journey towards a solution [here]({% post_url 2019-02-04-finding-rows-that-point-to-all-specified-rows-in-another-table %}).

That solution isn't ideal as the number of database hits is equal to the size of the array.

I now have an improved query:

```ruby
joins(:labels)
  .where(labels: { name: labels })
  .having("COUNT(labels.id) = #{labels.size}")
  .group('issues.id')
```

A row matches if the joining label rows have a name that is contained in the argument array (`labels`). The returned rows must point to all the specified labels so I make sure my result only contains those that point to at least as many labels as names in the array. Lastly, a `GROUP BY` clause is required after `HAVING`.

There was one small problem that I couldn't use `#count` on the returned object. But since I'd only used `#count` in the tests, I decided I should replace those occurrences with `#length`.

The resulting query hits the database twice.
