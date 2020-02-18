---
title: How to rename database columns in Rails
layout: post
---

Often when I begin a project, I find myself rolling back migrations, dropping the database, renaming tables and columns whilst I figure out the shape of the model. If I need to rename a column, I'll just update or delete the corresponding migration.

However, it is possible to change a column's name in a migration.

Say, I have a table `tasks` and I want to change the name of the `title` column to `name`. First I ask Rails to generate the migration:

```
rails generate migration rename_title_to_name_in_tasks
```

The generated migration looks like this:

```
class RenameTitleToNameInTasks < ActiveRecord::Migration[6.0]
  def change
  end
end
```

I then remove `change` and add the `up` and `down` migration like so:

```
class RenameTitleToNameInTasks < ActiveRecord::Migration[6.0]
  def up
    rename_column :tasks, :title, :name
  end

  def down
    rename_column :tasks, :name, :title
  end
end
```

`rename_column` takes three arguments. The first is the table name. The second is the existing column name. The third is the new name.

Above, I have defined the `up` and `down` migration. That is, the `up` migration does the renaming bit when we migrate the database. The `down` migration runs when we *roll back* the migration – in this instance, it reverts the column's name back to the original.

Before I run the migration, I give the schema a quick check.

```
  create_table "tasks", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "title", null: false
    t.text "notes", default: ""
    t.uuid "task_list_id"
    t.uuid "fulfiller_id"
    t.datetime "completed_at", precision: 6
    t.index ["fulfiller_id"], name: "index_tasks_on_fulfiller_id"
    t.index ["task_list_id"], name: "index_tasks_on_task_list_id"
  end
```

Then, after running `rails db:migrate`, I check it again to see if the migration has worked:

```
  create_table "tasks", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "name", null: false
    t.text "notes", default: ""
    t.uuid "task_list_id"
    t.uuid "fulfiller_id"
    t.datetime "completed_at", precision: 6
    t.index ["fulfiller_id"], name: "index_tasks_on_fulfiller_id"
    t.index ["task_list_id"], name: "index_tasks_on_task_list_id"
  end
```

All done!

Of course, five minutes later, I might decide that the new name probably isn't the best and I can run `rails db:rollback`. On doing so, I can see that the column name has reverted to `title`.

