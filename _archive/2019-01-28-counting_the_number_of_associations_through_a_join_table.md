---
layout: post
title: Counting rows with the most many to many associations
tags: archive
---

I need the labels associated with the most issues. I can use a following to get a hash of label names with issue counts.

`Label.joins(:issues).group(:name).order('count_id DESC').count('id')`

```
SELECT COUNT("labels"."id") AS count_id,
"labels"."name" AS labels_name FROM "labels"
INNER JOIN "tags" ON "tags"."label_id" = "labels"."id"
INNER JOIN "issues" ON "issues"."id" = "tags"."issue_id"
GROUP BY "labels"."name" ORDER BY count_id DESC
```

But I don't fully understand how it works.

Working backwards:

`ActiveRecord::Calculations#count`
<https://api.rubyonrails.org/classes/ActiveRecord/Calculations.html>

> If count is used with Relation#group, it returns a Hash whose keys represent the aggregated column, and the values are the respective amounts:

```ruby
Person.group(:city).count
# => { 'Rome' => 5, 'Paris' => 3 }
```

`order('count_id DESC')` is confusing me. Where does `count_id` come from? There isn't a column named `count_id`.

```ruby
Label.joins(:issues).group(:name).column_names
#=> ["id", "name", "created_at", "updated_at"]
```

`order(*args)` <https://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html>

```ruby
User.order('name DESC')
# SELECT "users".* FROM "users" ORDER BY name DESC
```

I'm still none the wiser about `count_id`. I assume there's some SQL knowledge required.

I've found some SQL examples [here](https://www.dofactory.com/sql/group-by). I think it's basically the same as `ORDER BY COUNT(Id)`:

```
SELECT COUNT(Id), Country
FROM Customer
GROUP BY Country
ORDER BY COUNT(Id) DESC
```
