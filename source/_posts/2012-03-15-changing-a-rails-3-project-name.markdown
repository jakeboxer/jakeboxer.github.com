---
layout: post
title: "Changing a Rails 3 project name"
date: 2012-03-15 12:00
comments: true
categories: 
---
I've had to change Rails 3 project names a few times now. It's too bad there's no `rake` command for it. Here's the next best thing: a step-by-step guide.

For the sake of this example, let's say I'm changing my name name from "Teach" to "Learn".

## Update your app's module name

In `/config/application.rb`, change the name of the module:

``` ruby /config/application.rb
module Learn # Used to be `module Teach`
  class Application < Rails::Application
    # ...
  end
end
```

## Update references to your app's module name

Your app's module name should appear in `/config.ru`:

``` ruby /config.ru
# This file is used by Rack-based servers to start the application.

require ::File.expand_path('../config/environment',  __FILE__)
run Learn::Application # Used to be `run Teach::Application`
```

And in `/Rakefile`:

``` ruby /Rakefile
#!/usr/bin/env rake
# Add your own tasks in files placed in lib/tasks ending in .rake,
# for example lib/tasks/capistrano.rake, and they will automatically be available to Rake.

require File.expand_path('../config/application', __FILE__)

Learn::Application.load_tasks # Used to be `Teach::Application.load_tasks`
```

Your app's module name should also appear in a bunch of files under the `/config` directory. It should appear in `/config/environments.rb`.

``` ruby /config/environments.rb
# Initialize the rails application
Learn::Application.initialize! # Used to be `Teach::Application.initialize!`
```

It should also appear in all the environment-specific configurations in `/config/environments/*.rb` (which should be `development.rb`, `production.rb`, `test.rb`, and any other environments you have configured). Change them all:

``` ruby /config/environments/*.rb
Learn::Application.configure do # Used to be `Teach::Application.configure do`
  # Settings specified here will take precedence over those in config/application.rb
  # ...
end
```

And in your initializers (under `/config/initializers/`). At the very least, `/config/initializers/secret_token.rb` and `/config/initializers/session_store.rb` should have it:

``` ruby /config/initializers/secret_token.rb
# Your secret key for verifying the integrity of signed cookies.
# If you change this key, all old signed cookies will become invalid!
# Make sure the secret is at least 30 characters and all random,
# no regular words or you'll be exposed to dictionary attacks.
Learn::Application.config.secret_token = 'garbagestring'
# Used to start with `Teach::Application`
```

``` ruby /config/initializers/session_store.rb
Learn::Application.config.session_store :cookie_store, key: '_learn_session'

# Use the database for sessions instead of the cookie-based default,
# which shouldn't be used to store highly confidential information
# (create the session table with "rails generate session_migration")
# Learn::Application.config.session_store :active_record_store

# For the sake of cleanliness, I changed it from `Teach::Application` in both the actual
# line of code and the comment. Also, `:key` used to be '_teach_session'.
```

And in your `/config/routes.rb` file:

``` ruby /config/routes.rb
Learn::Application.routes.draw do # Used to be Teach::Application.routes.draw
  # Routes in here
end
```

Finally, for consistency's sake, if the names of your databases include your app name, you should update those in `/config/database.yml`

``` yaml /config/database.yml
development:
  adapter: mysql2
  encoding: utf8
  reconnect: true
  database: learn_development # Used to be teach_development
  username: root
  password:
  
test:
  adapter: mysql2
  encoding: utf8
  reconnect: true
  database: learn_test # Used to be teach_test
  username: root
  password:
  
production:
  adapter: mysql2
  encoding: utf8
  reconnect: true
  database: learn_production # Used to be teach_production
  username: root
  password:
```




## Search your app and lib folders

`grep` through your `/app` and `/lib` folders to find any mentions of your project name and change them. This might be a little tedious if you have a name that conflicts with a bunch of Ruby/Rails stuff (like "def" or something stupid like that), but you only have to do it once.

## Done!

That's it! Now you have a renamed Rails 3 app.

Note: I did this on Rails 3.2.1. It's possible some things will be different on other versions.