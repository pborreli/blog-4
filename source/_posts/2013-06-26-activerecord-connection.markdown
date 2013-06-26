---
layout: post
title: "ActiveRecord Connections: A High Level Overview"
date: 2013-06-26 22:09
comments: true
categories: [activerecord, code-walkthrough, rails-4.0]
---

So, you feel pretty comfortable writing Ruby on Rails applications. Connecting to one (or more) databases and doing complex queries on complex relations doesn't faze you one bit. But it bothers you that you don't know more about how it works "behind the scenes".

Today I'll be starting the first in a series of posts on the internals of ActiveRecord, the part of Rails that allows us to connect to relational databases. This post will focus on the basics of how ActiveRecord establishes a connection.

When getting to know the internals of ActiveRecord, there's no substitute for [reading the source code](https://github.com/rails/rails/tree/v4.0.0/activerecord), but sometimes having some direction can certainly help. After reading and following along, I encourage you to read through the comments left by the Rails developers. They're quite clear and answer a lot of questions.

### Where to Begin?
When poking around ActiveRecord, a good place to start is in our old friend ActiveRecord::Base located in the [base.rb file](https://github.com/rails/rails/blob/v4.0.0/activerecord/lib/active_record/base.rb). ActiveRecord::Base has a nice overview of ActiveRecord features in the comments. The actual code is just a long list of modules included and extended into the class. We see familiar faces such as "scoping", "Validations", "Callbacks" and a whole lot more.

Our next step will be into the [ConnectionHandling module](https://github.com/rails/rails/blob/v4.0.0/activerecord/lib/active_record/connection_handling.rb) where, you guessed it, database connections are handled. `ActiveRecord::ConnectionHandling` is what gives our model classes such handy methods as `.establish_connection`, `.connection_pool` and `.connection_config`.

### General Concepts
Before we get into more details, it's important to know generally how ActiveRecord connects to databases. ActiveRecord uses either a hash or url describing how it should connect. By default Rails applications will specify this information in the config/database.yml file, which gets read during Rails initialization and passed to `.establish_connection`.

ActiveRecord uses the idea of a connection pool that coordinates threads'
access to a limited number of database connections. The connection pool is
smart and will allow only one thread per database connection. Once a thread is
done with a connection it will free it up for other threads to check out the
connection.

ActiveRecord also offers a connection reaper that acts to garbage collect old
connections. Since this is turned off be default, we won't cover it here.

# Basics of .establish_connection
For those not following along, .establish_connection looks like this:

``` ruby
def establish_connection(spec = ENV["DATABASE_URL"])
  resolver = ConnectionAdapters::ConnectionSpecification::Resolver.new spec, configurations
  spec = resolver.spec

  unless respond_to?(spec.adapter_method)
    raise AdapterNotFound, "database configuration specifies nonexistent #{spec.config[:adapter]} adapter"
  end

  remove_connection
  connection_handler.establish_connection self, spec
end
```

First, the method gets a connection specification resolver that simply is able to parse the various forms of database configs you can throw at it (e.g. hash, url, etc.). Then a specification object is returned that embodies the spec of how ActiveRecord will connect to the database. This object gets passed around a lot so ActiveRecord knows what to do.

Many terrible spellers (myself included) will notice the check for the response to `#adapter_method`, which raises if a non-valid database adapter such as "postgrsql" is given.

Next ActiveRecord removes any connection the model has to a database, before then connecting to the specified database.

# Wrap Up

This concludes our first look at ActiveRecord interals with a very high level look at how ActiveRecord connects to databases. Next time, we'll dive a little deeper into database connections with a look at database adapters and more on connection pools.
