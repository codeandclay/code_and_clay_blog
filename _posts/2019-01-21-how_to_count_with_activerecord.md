---
layout: post
title: How to Count With ActiveRecord
---

via [https://work.stevegrossi.com](https://work.stevegrossi.com/2015/04/25/how-to-count-with-activerecord/)

>Counting records returned from the database by ActiveRecord is quite a bit more complicated than you’d expect, but that’s the price of Rails’ magic and the invisible efficiency it gives you behind the scenes. ActiveRecord has its own implementation of Ruby’s size, any?, and empty? methods which behave differently than their array counterparts. Understanding how each works will help you write more efficient code when dealing with database records.

>### In General
>
>- When counting the number of results from the database, use size unless you know you’ll be loading all of the results later on, then use length.
>- When checking if there are any results from the database, use any? or its opposite empty? unless you know you’ll be loading all of the results later on, then use present? or its opposite blank?.

In a nutshell, calling `#size` could result in an extra call to the db.

>### LENGTH
>
As mentioned above, length is a plain Ruby method which functions like count when called on an array. The difference is that ActiveRecord does not define its own length method. So if you call length on an ActiveRecord::Relation object, it will be converted to an array, and will count the array’s objects. This is equivalent to the slower count method above:…

`#size` behaves differently depending on whether or not the record has been loaded.

>In this case, it behaves differently depending on whether or not the relation’s records have been loaded from the database yet.

