---
layout: post
title: Rails &amp; Spine.JS - Jasmine Testing Part 2
categories: 
  - ruby-rails
  - javascript
---

<p>
  So this is the third part to my mini series on Rails and <a href="http://spinejs.com/">Spine.JS</a>. Part one <a href="/2012/01/15/rails-and-spine-js-using-the-coffeescript-source/">covers an initial setup</a> and how to include Spine.JS into your Rails project while part two is actually the <a href="/2012/01/16/rails-and-spine-js-jasmine-testing-part-1/">first of a tome on how to test</a> your Spine.JS application. Assuming you have covered the bases there, lets get right down to business and review some of the elegant hacks &trade; yours truly came up with while testing my own Spine.jS application using <a href="http://github.com/guard/guard">Guard</a> and <a href="http://github.com/bradphelan/jasminerice">Jasminerice</a>. 
</p>


<h2>That Engine Is Gonna Need An Overhaul</h2>

<p>
  So did I mention that Jasminerice is a Rails engine and that you can run your tests by accessing the <code>/jasmine</code> URL in your development environment's browser? Good. As uncool as browser testing is, sometimes that it useful. For instance <code>console.log</code> statements will not show up in your Guard's test output. But hopefully that is an edge case and you are really using <a href="http://github.com/netzpirat/guard-jasmine">guard-jasmine</a> and watching your specs run in the terminal window. The question now becomes how can I leverage Jasminerice and its associated engine to really test my Spine.JS application with a full-fledged DOM? The answer is simple, let's hack the Jasminerice engine to load up our application while running the Jasmine specs.
</p>


<h2>A Review Of Jasmine And HTML Fixtures</h2>

<p>
  This might be a bit contraversial (or maybe just wrong) but I would like to examine why we might hack the Jasminerice engine for a few minutes. Jasmine bills itself as behavior-driven development for JavaScript, to which I disagree. I think Jasmine first and foremost is an all-purpose testing framework. Depending on how you use it determines what it becomes. In my case, I have been doing a lot of unit testing of my Spine.JS models and supporting libs as I learn the framework and build my system. So my usage now could be called TDD at both a unit and functional level. Later on, I plan on doing higher level integration testing with Jasmine, at this time it will be my BDD tool. Normally I do not get caught up in symantics but I think it is important to understand a few lexical terms before I start showing off how I use Jasmine to test my Spine.JS application. 
</p>

<p>
  Now that I have set my higher order bit for Jasmine as my unit, functional and itegration testing framework for Spine.JS &ndash; I would like to show how my solution below might differ from other practices. Experienced Jasmine users rave about extensions that allow you to load HTML fixtures to functionally test units of code. In fact, the Jasminerice gem includes a custom version of the <a href="https://github.com/velesin/jasmine-jquery">jasmine-jquery</a> library which among other things allows you to do just that. Let me be clear on this, there is nothing wrong with that! As with all things software, proper solutions depends on what you are doing and need. In my case, I believe that testing your Spine.JS application with Rails only needs to hook into your existing application without the need for fixtures and excessive mocking and stubbing.
</p>

<p>
  My basis for this argument is founded on a few principas that I believe all rich client side JavaScript applications should follow. Most important, the application is a single page load and all other calls to the server happen from that one page via AJAX. Every other part of your Rails application then becomes an API client to the JavaScript application. All views are client side only, most likely in a <code>JST</code> namespace.
</p>


<h2>Hacking Jasminerice</h2>

<p>
  Turns out this is really easy since Jasminerice is just a simple Rails engine with few moving parts. It has a <code>SpecController</code> that renders a basic layout which requires your application's JavaScript assets along with Jasmine. The goal is to tell it to render your single application page that loads up your Spine.JS app with a few additional hacks. When done, we are going to have Jasminerice in your full control. This means we get the same CSS and JavaScript as our real application along with a solid foundation to extend Jasminerice at our whim. This leaves us with a clean canvas suitable for Jasmine unit, functional and integraiton testing all wrapped up into one.
</p>


~~~ruby
# In: config/initializers/jasminerice.rb

module Jasminerice
  
  module MyApplication
    extend ActiveSupport::Concern
    included do
      delegate :foo_path, to: ::Rails.application.routes_url_helpers
    end
    def jasminerice?
      true
    end
    def current_user
      @current_user ||= User.find(1)
    end
  end
  
  ApplicationHelper.send :include, MyApplication
  
  class SpecController < Jasminerice::ApplicationController
    def index
      render template: 'users/home', layout: 'application'
    end
  end
  
end if defined?(Jasminerice) && Jasminerice.environments.include?(Rails.env)
~~~

<p>
  There we go, a simple example pulled from my current project. What we are doing here is freedom-patching that spec controller's index action to render both a template and layout from your application. I also have a <code>MyApplication</code> module which I include in the <code>Jasminerice::ApplicationHelper</code> for a few methods that my application view code would expect. In this case a <code>current_user</code> method and some delegation of URL helpers to the root Rails application. Do not get hung up on what user is returned there too. It is mostly moot as all AJAX calls will be stubbed. Lastly, I made a <code>jasminerice?</code> view helper and a matching one that I have in my own <code>ApplicationHelper</code> which returns false. I will show why and how this all fits together later on.
</p>

<p>
  Now that we have full control and some reflection for Jasminerice, let's reviw our appliation's main layout. Here is an example of my <code>app/views/layouts/application.html.haml</code> file.
</p>

~~~ruby
!!! 5
%html{:lang => 'en'}
  %head
    %meta{:charset => 'utf-8'}
    = csrf_meta_tags
    %title My Application
    - if jasminerice?
      = stylesheet_link_tag 'spec'
      = javascript_include_tag 'spec'
    - else
      = stylesheet_link_tag 'application'
      = javascript_include_tag 'application'
      %script jQuery(function(){ MyApp.App.Home.init(); });
  %body
    %section{id: 'app', data: {id: current_user.id}}
~~~

<p>
  It is really that simple! It has a single condition that says if Jasminerice is loading this page, use the top level <code>spec</code> CSS and JavaScript asset manifest. If not, render my main application's CSS and JavaScript manifests. You will also notice that I decouple my Spine's application initialization from the main JS files and only initialize the app on page load when not running in Jasminerce. This allows us to stub AJAX calls in specs then initialization the Spine.JS app when we are fully ready. So now, let us take a look at what both the <code>spec/javascripts/spec.css</code> and <code>spec/javascripts/spec.js</code> manifests may look like.
</p>

~~~css
/*
 *= require jasmine
 *= require application
 */
.jasmine_reporter { display: none; }
~~~

~~~javascript
#= require jasminerice
#= require mock-ajax
#= require application
#= require jasmine-myapp
#= require_tree ./lib/
#= require_tree ./models/
#= require_tree ./controllers/
#= require_tree ./views/
~~~

<p>
  These should be self explanatory. The <code>spec.css</code> manifest requires the jasmine styles, then our application's styles. It then hides the Jasmine reporter element. The <code>spec.js</code> manifest does something similiar, It first requires the jasminerice manifest, a Jasmine helper called <a href="https://github.com/pivotal/jasmine-ajax">jasmine-ajax</a> but poorly named mock-ajax, then your application's JavaScript followed by a personal Jasmine helper that we will discuss later. The last lines include all the spec files in each of the lib, models, controller and views directories located in the parent <code>spec/javascripts</code> directory.
</p>

<p>
  Congratulations! You are now in full control of both Spine.JS and Jasmine(rice) and how your specs are executed. You have a fully styled DOM that you can happily ignore as you test your unit code. And when the time comes, you can leverage that and you full Spine.JS application from a high level integration perspective. Tune in for my last part which will give some working examples of how to test your Spine.JS application with Jasmine.
</p>


<h2>Related</h2>

<ul>
  <li><a href="/2012/01/15/rails-and-spine-js-using-the-coffeescript-source/">Rails &amp; Spine.JS - Using The CoffeeScript Source</a></li>
  <li><a href="/2012/01/16/rails-and-spine-js-jasmine-testing-part-1/">Rails &amp; Spine.JS - Jasmine Testing Part 1</a></li>
  <li><a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">Rails &amp; Spine.JS - Jasmine Testing Part 2</a></li>
  <li><a href="/2012/02/01/pretty-console-logging-with-guardjasmine-black-coffee/">Pretty Console Logging With Guard::Jasmine &amp; BlackCoffee</a></li>  
</ul>


<h2>Resources</h2>

<ul>
  <li><a href="http://spinejs.com/">Spine.JS - Build Awesome JavaScript MVC Applications</a></li>
  <li><a href="http://github.com/bradphelan/jasminerice">Jasminerice - Pain free CoffeeScript testing under Rails 3.1</a></li>
  <li><a href="http://github.com/pivotal/jasmine-ajax">Jasmine AJAX - A library for faking Ajax responses in your Jasmine suite.</a></li>
</ul>


