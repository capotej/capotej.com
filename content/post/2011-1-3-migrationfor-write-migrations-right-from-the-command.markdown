---
title: "MigrationFor: Write migrations right from the command line!"
date: 2011-01-03T09:48:00Z
comments: false
url: /post/2583891119/migrationfor-write-migrations-right-from-the-command
tags:
---



As someone who mostly stays in the rails console, I’ve always hated forgetting a field, creating a migration, finding it among your other 500 migration files, then adding the one line you need to add, then running it. This is probably the most annoying part of the Rails experience. I’ve always wanted to write a better migration generator that could take a list of commands/fields and write the migration for you, since most of the time what you name a migration has all the info it needs (add_index_to_post_id). Thanks to the heavily refactored plugin/generator API in Rails 3, I was able to do just that.

Let’s take a look at how it works:

First, install it (only works for Rails 3)

`rails plugin install git://github.com/capotej/migration_for.git`

Then, you can create migrations like so:

`rails g migration_for add_index:posts:posts_id`

It would generate db/migrate/20110103182654_add_index_posts_posts_id.rb):

```ruby
class AddIndexPostsPostsId < ActiveRecord::Migration

  def self.up
    add_index 'posts','posts_id'
  end

  def self.down
    #waiting for reversible migrations in rails 3.1!
  end

end
```

Which you can then run normally with `rake db:migrate`

Let’s look at a more complex example:

`$ rails g migration_for create_table:posts add_column:posts:title:string add_column:posts:user_id:integer
`

Would generate:

```ruby
class CreateTablePostsaddColumnPostsTitleStringaddColumnPostsUserIdIntegeraddIndexPostsUserId < ActiveRecord::Migration

  def self.up
    create_table 'posts'
    add_column 'posts','title','string'
    add_column 'posts','user_id','integer'
    add_index 'posts','user_id'
  end

  def self.down
    #waiting for reversible migrations in rails 3.1!
  end

end
```

It uses a lookup table with all the [activerecord transformations](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html) and will only insert an expression into a migration if the method name is valid and it has the right number of arguments, so botched commands wont mess up the migration. Hope you enjoy it as much as I have!

Source available here: [https://github.com/capotej/migration_for](https://github.com/capotej/migration_for)
