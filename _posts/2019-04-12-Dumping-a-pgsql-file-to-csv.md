---
layout: post
title: Converting a pgsql file to CSV
---

I had a request this morning from a client who wanted me to save some tables from their database as CSVs. They provided a `.pgsql` file which contains all the data and instructions for constructing the database. It's sort of a bunch of migrations and seeds in one place.

There might be a more straightforward way but I decided to create a new Rails app and build the database from the dump file. I could then export the tables from the console.

There weren't many steps to it. Firstly I created the database.

```
rails db:create
```

I went to `config/database.yml` to get the name of the database before running:

```
psql -d rails_app_development -f dumped_db.pgsql
```

It took a couple of tries. The command's output informed me I needed to create some users:

```
createuser --superuser postgres
createuser --superuser rails
```

Following `rails db:drop db:create` I ran the `psql` command again.

Having created the database I ran `db:schema:dump` to create the schema file. From that I could see which tables I needed to create corresponding models for.

I then went to the console and dumped my models to CSV files.

```ruby
def dump_csv(path, model)
  CSV.open(path, "wb") do |csv|
    csv << model.attribute_names
    model.find_each do |row|
      csv << row.attributes.values
    end
  end
end
```
