---
layout: post
title: Using Dotenv In Rails
categories: 
  - ruby/rails
---

Environment variables as a configuration means are everywhere in Ruby. For instance, ActiveRecord will use the single `DATABASE_URL` environment variable for every part of it's configuration, no database.yml needed! If you are not on board with environment variables, check out [The Twelve-Factor App](http://12factor.net/config) for configuration. This is exactly how good software platforms like [Heroku](https://www.heroku.com) work, all through environment variables.

But using environment variables in a Rails application can be tricky. During local development, you may not want to set everything in your Bash or ZSH profile. Perhaps you want per project settings. And then there is the trouble of using different environment variables when running your tests. Enter the [Dotenv](https://github.com/bkeepers/dotenv) project. 

In my examples below of how to integrate Dotenv, I will be using @ecbypi's fork until this [pull request for overloading environment variables](https://github.com/bkeepers/dotenv/pull/61) is accepted. This pull request is key to overriding development or local settings for our Rails test environment. If you like how these examples look below, weigh in on that pull request and lobby for a fresh hot Dotenv release. 

Let's get started. First, let's add the gem to your Rails project's Gemfile.

~~~ruby
# In Gemfile
gem 'dotenv', github: 'ecbypi/dotenv', branch: 'overload-environment-variables'
~~~

Next up, open your `config/application.rb` and add this line right after the require 'rails/all' line. This will require the Dotenv gem, then load your own per-developer local settings (if present), then a .env file that matches the current `Rails.env`.

~~~ruby
# In config/application.rb
require 'rails/all'
require 'dotenv' ; Dotenv.load ".env.local", ".env.#{Rails.env}"
~~~

Next, create individual `.env.RAILS_ENV` files with your configurations. YMMV on this one, but the idea is to check all of these in. The `.env.development` will hold all the default configurations for a new developer to get up and running quickly. The `.env.test` will hold all the configurations for your test runs. Past that, how you handle the production file or not, is left up to you. In my examples below, we are setting the environment variable for the Awesome gem.

~~~
$ echo "AWESOME_GEM_URL=https://awesome.dev"          > .env.development
$ echo "AWESOME_GEM_URL=https://sandbox.awesome.com"  > .env.test
$ echo "AWESOME_GEM_URL=https://awesome.com"          > .env.production
~~~

This covers our bases, but what if a developer wants to override a configuration. They can use good ol' environment variables and get the behavior they expect. Or they create their own `.env.local` file and set their configurations there. The benefit of the local file is that your Rails application via the console or the development server (for example Pow) will automatically get the same thing when using the .env.local file. Oh yea, it is a good idea to add the this file to your `.gitignore`.

~~~
$ echo "AWESOME_GEM_URL=https://my-awesome.dev"       > .env.local
~~~

Now, one last thing. We need to make sure that our tests do not use any real, development or local environment settings. This is where @ecbypi overload feature comes in handy. I have added this to the top of the Rails `test/test_helper.rb` file. 

~~~ruby
# In test/test_helper.rb
ENV["RAILS_ENV"] = "test"
+require 'dotenv' ; Dotenv.overload ".env.test"
~~~

Happy environment usage! Have you been solving environment and application configuration in a different way (don't say YAML), if so, I would love to hear about it!

