---
layout: post
title:  Rails Seed Data from YAML with Rake Task
date:   2014-10-10 22:22:22
categories: rails
---


This is inspired by [http://sixarm.com/about/rails-seed-data.html](http://sixarm.com/about/rails-seed-data.html) which i noticed had some errors.

### Seed Data in YAML files

We use the Rails convention of putting our seed data files here:

`/db/seeds/`

We create a YAML file for our seed users:

`/db/seeds/users.yml`

The users YAML file defines each user's fields like this:

{% highlight yaml %}
  alice:
    name: "Alice Adams"
    mail: "alice@example.com"
  bob:
    name: "Bob Brown"
    mail: "bob@example.com"
  carol:
    name: "Carol Cox"
    mail: "carol@example.com"
{% endhighlight %}

### Rake task to load seed data

We use the Rails convention of putting rake tasks here:

`/lib/tasks/`

We use a rake file for our seeds:

`/lib/tasks/db_seed_users.rake`

The task provides the rake namespace, runs any setup tasks, then calls a normal ruby method:

{% highlight ruby %}
  # rake db:seed:users
  namespace :db do
    namespace :seed do
      desc "Seed Users from /db/seeds/users.yml"
      task :users => :environment do
        db_seed_users
      end
    end
  end
{% endhighlight %}

> **TIP** - the *desc* is required if you want your task to show up with *rake -T*

The normal ruby method loads the YAML file and iterates on each user:

{% highlight ruby %}
  # Seed multiple users by loading the YAML file
  def db_seed_users
    path = Rails.root.join('db','seeds','users.yml')
    puts "Seeding file #{path}"
    File.open(path) do |file|
      YAML.load_documents(file) do |doc|
        doc.keys.sort.each do |key|
          puts "Seeding key #{key}"
          attributes = doc[key]
          create_a_seed_user(attributes)
        end
      end
    end
  end

  # Seed one user
  def create_a_seed_user(attributes)
    User.create(attributes)
  end
{% endhighlight %}

To run the rake task:

`$ rake db:seed:users`

To view your rake tasks:

`$ rake -T`

**Remember, your tasks will not show up unless you add a `desc`**

How to make seeds run more than once

We want to be able to run our seed task more than once and still get produce same set of users.

To do this, we add code that checks to see if the user already exists, and if so, skips creating that user:

{% highlight ruby %}
  # Seed one user
  def db_seed_user(attributes)
    mail = attributes['mail']
    user = User.where(mail: mail).first_or_create
    if user
      puts "This email address exists: #{mail}"
      # update the user with user.update(name: attributes['name'])
    else
      puts "This email address is new: #{mail}"
      User.create(attributes)
    end
  end
{% endhighlight %}
