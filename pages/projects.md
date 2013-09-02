--- 
layout: page
title: My Projects
categories: 
  - projects
---


<aside class="flash_info mt10">
  <img src="/assets/rubygems.png" class="ml5 floatr" width="50" height="38" />
  I write a lot of code! So this list can quickly become outdated. Please checkout my github accounts (<a href="http://github.com/metaskills/">metaskills</a> or <a href="http://github.com/rails-sqlserver/">rails-sqlserver</a>) or a list of my gems on <a href="https://rubygems.org/profiles/246">rubygems.org</a>.
</aside>


## Mocha-PhantomJS

[Mocha](http://visionmedia.github.io/mocha/) is a feature-rich JavaScript test framework running on node and the browser. Along with the [Chai](http://chaijs.com/) assertion library they make an impressive combo. [PhantomJS](http://phantomjs.org/) is a headless WebKit with a JavaScript/CoffeeScript API. It has fast and native support for various web standards like DOM handling, CSS selectors, JSON, Canvas, and SVG.

The [mocha-phantomjs](https://github.com/metaskills/mocha-phantomjs) project provides a mocha-phantomjs.coffee script file and extensions to drive PhantomJS while testing your HTML pages with Mocha from the console.


## MiniTest Spec Rails

The [minitest-spec-rails](https://github.com/metaskills/minitest-spec-rails) gem makes it easy to use the MiniTest::Spec DSL within your existing Rails 2.3, 3.x or 4.x test suite. It does this by forcing ActiveSupport::TestCase to utilize the MiniTest::Spec::DSL.


## Rails SQL Server

<aside class="flash_warn">
  [The SQLServer Adapter is looking for a maintainer.](https://groups.google.com/d/msg/rails-sqlserver-adapter/PWIobkN7FOg/a0LDfE1hN1UJ) However I still support and actively work on TinyTDS.
</aside>

The SQL Server adapter for rails is back for ActiveRecord 2.2 and up! We are currently passing all tests and hope to continue to do so moving forward. Includes tons of new features including unicode column support, pessimistic locking, date/time column casting, DDL transactions and way more.

The TinyTds gem is my first ruby C extension. It provides a fast, efficient and native interface to FreeTDS. This provides SQL Server users with the first low level connection mode that properly handles client db encoding, ruby primitives, and string encodings under 1.9 and up.

* [Rails SQL Server Adapter](http://github.com/rails-sqlserver/activerecord-sqlserver-adapter/)
* [TinyTds - A modern, simple and fast FreeTDS library for Ruby using DB-Library](http://github.com/rails-sqlserver/tiny_tds/)


## Less-Rails &amp; Less-Rails-Bootstrap

When [Twitter's Bootstrap](http://twitter.github.com/bootstrap/) was first released, I decided to give it a try and I assumed there was support in the Rails asset pipeline for it. I was mistaken. Even thought Tilt under Sprockets could render less files with the less.rb and therubyracer gems, there was no easy way to hook up the less load paths in a single place. Nor was there a way to expose helpers familiar to those using [sass-rails](https://github.com/rails/sass-rails) like `asset-data-url`. Thus was born these two gems. Both leverage `Rails::Engine` or `Rails::Railtie` models that hook into the low level internals of the Rails boot process to hook into the asset pipeline. I even take advantage of therubyracer to add functions to Less.js that hook right back into Ruby!

* [Less-Rails On Github](https://github.com/metaskills/less-rails)
* [Less-Rails-Bootstrap On Github](https://github.com/metaskills/less-rails-bootstrap)
* [LESS Is More - Using Twitter's Bootstrap In The Rails 3.1 Asset Pipeline](/2011/09/26/less-is-more-using-twitter-bootstrap-in-the-rails-3-1-asset-pipeline/)


## GroupedScope Plugin

GroupedScope aims to make two things easier in your ActiveRecord models. First, provide a easy way to group objects, second, to allow the group to share associated object via existing has_many relationships. See installation and usage for more details.

* [GroupedScope Plugin on Github](http://github.com/metaskills/grouped_scope/tree/master)
* [Jack has_many :things](/2008/09/28/jack-has_many-things/), an article introducing GroupedScope.


## Capistrano: Remote Shared Cache Strategy

This class greatly speeds up deploy times to multiple hosts by only updating your latest source code to a shared location accessible to all other servers. For instance, if you have 5 app servers in your cluster then the code update only happens with the primary or first server while all others use the shared cache to copy from. The remote shared cache can be updated by either a direct SCM export or a local export copy to target host. Notables on this project include:

* An add_rollback method that takes a block of code that can be added to the task_call_frames stack of existing transactions. Allows transactions to be added within transactions.
* A primary_app_server method that finds the primary or first app server.
* [http://github.com/metaskills/remote_shared_cache/tree/master](http://github.com/metaskills/remote_shared_cache/tree/master)


## HomeMarks: Simple Bookmark Start Page

[HomeMarks](http://homemarks.com/) was my first rails project when I was both learning to program in both Ruby and JavaScript. It allows you to create a simple bookmark start page that organizes links into colored boxes sorted in columns. A common practice by all early web developers I knew, this time you do not have to code your own HTML.

HomeMarks went thru its third rewrite a few months ago for Rails 3 and is currently deployed on Heroku. I also recently just finished my first native Objective-C iPhone app too. I am currently working on the native iPad version and will post updates here too. When that happens I will rewrite the rails application a 4th time using CoffeeScript & Spine.js.

* [Synchronizing Core Data With Rails (3.0.0.pre)](/2010/02/12/synchronizing-core-data-with-rails-3-0-0-pre/)
* [Hell'OO HomeMarks](/2008/08/18/in-hell-oo-for-homemarks/)
* [The "AJAX Head" Design Pattern](/2008/05/24/the-ajax-head-design-pattern/)


