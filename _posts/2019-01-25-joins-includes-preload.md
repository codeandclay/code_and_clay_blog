---
layout: post
title: Joins, Includes, Preload
---

I am trying to understand the differences between `joins` and `includes`.

From what I have learned, the main difference between the two methods is that `joins` lazy loads and `includes` eager loads a table.

But I see there's also a difference in the SQL they generate. Notably, from what I can tell, `joins` uses `INNER JOIN` but `includes` uses `OUTER JOIN`.

I have two models, `Tourist` and `Country`, related to each other with a many to many relationship via a join table named `visits`.

    class Tourist < ApplicationRecord
      has_many :visits
      has_many :countries, through: :visits
    end

    class Country < ApplicationRecord
      has_many :visits
      has_many :tourists, through: :visits
    end

    class Visit < ApplicationRecord
      belongs_to :country
      belongs_to :tourist
    end

I see that by querying using `joins` that one of the tourists has three countries.

    > Tourist.joins(:countries).where(countries: { name: country }).last.countries.map(&:name)
    => ["Narnia", "UK", "Middle Earth"]

The generated SQL:

    SELECT  "tourists".* FROM "tourists"
    INNER JOIN "visits" ON "visits"."tourist_id" = "tourists"."id"
    INNER JOIN "countries" ON "countries"."id" = "visits"."country_id"
    WHERE "countries"."name" = ?
    ORDER BY "tourists"."id" DESC LIMIT ?
    [["name", "UK"], ["LIMIT", 1]]

    SELECT "countries".* FROM "countries"
    INNER JOIN "visits" ON "countries"."id" = "visits"."country_id"
    WHERE "visits"."tourist_id" = ?  [["tourist_id", 62]]

However, the result is different when `includes` is used.

    > Tourist.includes(:countries).where(countries: { name: country }).last.countries.map(&:name)
    => ["UK"]

This time, `LEFT OUTER JOIN` is used instead of `INNER`:

      SELECT DISTINCT "tourists"."id" FROM "tourists"
      LEFT OUTER JOIN "visits" ON "visits"."tourist_id" = "tourists"."id"
      LEFT OUTER JOIN "countries" ON "countries"."id" = "visits"."country_id"
      WHERE "countries"."name" = ? ORDER BY "tourists"."id" DESC LIMIT ?
      [["name", "UK"], ["LIMIT", 1]]

      SELECT "tourists"."id"
      AS t0_r0, "tourists"."name"
      AS t0_r1, "countries"."id"
      AS t1_r0, "countries"."name"
      AS t1_r1, "countries"."created_at"
      AS t1_r2, "countries"."updated_at"
      AS t1_r3 FROM "tourists"
      LEFT OUTER JOIN "visits" ON "visits"."tourist_id" = "tourists"."id"
      LEFT OUTER JOIN "countries" ON "countries"."id" = "visits"."country_id"
      WHERE "countries"."name" = ? AND "tourists"."id" = ?
      ORDER BY "tourists"."id" DESC
      [["name", "UK"], ["id", 62]]

So it appears that there is more of a difference between `joins` and `includes` than lazy/eager loading.

I assume that if there was a way to force `includes` to use `INNER` then I would get all the expected countries. 

At the bottom of p171 (5.6.9) of [The Rails 5 Way](https://blog.arkency.com/2013/12/rails4-preloading/) there's this description of `includes`'s inner workings.

>If possible, `includes` used `LEFT OUTER JOIN` to grab all the data it needs in one query. When that happens, it delegates to `eager_load`. Otherwise, it will use at least two separate queries and delegate to `preload`.

>If you know you want one approach versus the other, you can ensure you get it by using `eager_load` or `preload` directly with the same syntax.

So, I have learned that if I want to load my data up front, I need to use `preload`:

    > Tourist.preload(:countries).where(countries: { name: country })
    => ["Narnia", "UK", "Middle Earth"]

    SELECT "visits".* FROM "visits"
    WHERE "visits"."tourist_id" = ?
    [["tourist_id", 62]]

    SELECT "countries".* FROM "countries"
    WHERE "countries"."id" IN (?, ?, ?)
    [["id", 15], ["id", 14], ["id", 16]]


## Further Reading

- [ActiveRecord: When does includes do a join, and when does it do a second query?](https://www.foraker.com/blog/active-record-includes-query-logic)
(Note to self: `Tourist.joins(:visits).merge(Visit.where(country_id: [15, 14, 16]))`)
- [3 ways to do eager loading (preloading) in Rails 3 & 4 & 5](https://blog.arkency.com/2013/12/rails4-preloading/)
- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html#joins)
