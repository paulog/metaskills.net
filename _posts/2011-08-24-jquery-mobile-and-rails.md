---
layout: post
title: jQuery Mobile &amp; Rails
categories: 
  - ruby-rails
  - javascript
---

<p>
  <img class="floatr" src="/assets/jquery_mobile_rails.png"/>
  I just finished my first dive into using <a href="http://jquerymobile.com/">jQuery Mobile</a> with a Rails application and wanted to share some techniques that came out along the way. Hopefully these will help you if your are using jQuery Mobile with Rails or want to test your mobile application's integration layer. This post assumes you are somewhat familiar with jQuery Mobile and its basic concepts. So let's jump right in with a series of helpful tips.
</p>



<h2>A Mobile Layout</h2>

<p>
  In my application, I decided to use use a route namespace of "mobile" vs a sub-domain for all controllers and views to reside in. Do what works for you, but I find that using namespaces both helps keep my code organized and simple to maintain. Either way, you should have a layout specific for your mobile application. Mine is in <code>app/views/layout/mobile.html.haml</code>. As you can see here, I use HAML, so all view examples will be using it vs ERB. Here is a general layout.
</p>

~~~ruby
!!! 5
%html{:lang => 'en'}
  %head
    %meta{:charset => 'utf-8'}
    %meta{:name => 'viewport', :content => 'width=device-width, initial-scale=1'}
    %meta{:name => 'format-detection', :content => 'telephone=no'}
    %title= 'My Mobile App'
    %link{:rel => 'stylesheet', :href => 'http://code.jquery.com/mobile/1.0b2/jquery.mobile-1.0b2.min.css'}
    %script{:src => 'http://code.jquery.com/jquery-1.6.2.min.js'}
    %script{:src => 'http://code.jquery.com/mobile/1.0b2/jquery.mobile-1.0b2.min.js'}
  %body
    = yield(:layout)
~~~

<p>
  A few key points here. First, the meta tag for the <code>viewport</code> is recommended as a good base. The other meta tag for <code>format-detection</code> is to disable automatic detection and linking of phone numbers. Phone number detection is way too aggressive and often just links random numbers with periods and hyphens. This means if you want a phone number to call when touched, you will have to use the <code>tel:</code> link format with the phone number afterward. I recommend the aid of something like the <a href="https://github.com/floere/phony">phony</a> gem for validating and parsing phone number formats. Lastly, the title attribute in the head is really moot. The jQuery Mobile framework will dynamically change the page title as new page DOM elements are loaded in.
</p>



<h2>Mobile Page IDs</h2>

<p>
  A typical jQuery Mobile app will have one full page load. All other pages thereafter are loaded via AJAX and inserted into the DOM. Their docs suggest that every page (and form) have a unique id attribute. These ids can be used to link to the DOM page element when navigating around preloaded pages. Simple apps can get by with a few ids like "#home", "#contact_us", "#etc". But if you have a large application, you need a better system to keep track of things.
</p>

<p>
  Thankfully if you are building your Rails applications in a RESTful manner, it is easy to leverage the route link helpers to generate your page ids. In the example below, I created a method called <code>mobile_page_id</code> that I placed into a <a href="http://weblog.jamisbuck.org/2007/1/17/concerns-in-activerecord">shared concerns module directory</a>. From here I mix this into my test helper, and application's view helper. It is that damn useful! You are going to want to use this everywhere!
</p>

~~~ruby
# File: app/concerns/mobile_concerns.rb
module MobileConcerns  
  module Helpers
    def mobile_page_id(path = request.path)
      path.sub(/\A\/mobile\//,'').gsub("/",'_')
    end
  end  
end

# Mix into your applications helper.
module ApplicationHelper
  include MobileConcerns::Helpers
end

# Mix into your test helper of choice.
class MobileStoriesTest < ActionController::IntegrationTest
  include MobileConcerns::Helpers
end
~~~

<p>
  The <code>mobile_page_id</code> method is primarily for views. I will cover its usage in testing further down. When called without an argument, it will take the current request path and translate it to a string suitable for a page id. So if you were rendering a <code>/mobile/users/10/avatar</code> page, the id would be <code>users_10_avatar</code>. If your application follows RESTful resources in your routes, this can pay dividends. When needed, you can pass a <code>*_path</code> helper method to get the same id. In this example the same id would come back for <code>mobile_page_id(mobile_user_avatar_path(@user))</code>. Remember, I am using a "mobile" namespace in my examples and hence my method above strips that out, flavor this helper to your needs. Here is an example of what you might find in <code>app/views/users/avatar.html.haml</code> using the mobile_page_id. You can see here that I am also setting a title local that is used in the <code>%h1</code> tag and the <code>data-title</code> page element attribute. Doing this will show the same title in the header bar as well as the page title.
</p>

~~~ruby
- title = "User Profile"

%div{:id => mobile_page_id, :data => {:role => 'page', :title => title}}

  %div{:data => {:role => 'header'}}
    %h1= title
  %div{:data => {:role => 'content'}}
    edit your profile...
  %div{:data => {:role => 'footer'}}
    /...
~~~



<h2>Easy Data Attributes</h2>

<p>
  The jQuery Mobile framework will have you typing a lot of data-* attributes into your elements. It uses these from basic page behavior to UI themes. They are literally needed everywhere. If your typing out raw HTML these data attributes get old pretty quick. Thankfully both HAML and the latest Rails 3.1 tag helpers can keep things clean. Both allow hashes to be passed to the data attribute. Keys are dasherized and values are JSON-encoded except for string and symbols.
</p>

~~~ruby
# HAML Shorthand For:
# <div id="foo" data-role="page" data-title="Title" data-rel="dialog"></div>

%div{:id => 'foo', :data => {:role => 'page', :title => 'Title', :rel => 'dialog'}}

# Rails 3.1 Tag Helpers
# <div data-name="Stephen" data-city-state="[&quot;Chicago&quot;,&quot;IL&quot;]" />

tag("div", :data => {:name => 'Stephen', :city_state => %w(Chicago IL)})
~~~



<h2>Dynamically Setting Layout</h2>


<p>
  Remember how jQuery Mobile only loads the entire page once and then inserts all following pages using AJAX? This means that views after the initial page load do not need the layout when rendered. Depending on the setup of your mobile application, it could become a chore telling each action to conditionally render the layout or not. Imagine a page refresh or a user bookmarking a deep resource node of your mobile app. Thankfully with a little Rails-fu, we can conditionally set the layout to false if the request is an <code>HTTP GET</code> from an AJAX request. Below is an example of my base mobile controller that all mobile controllers inherit from. Now every action is able to load the entire mobile layout or not.
</p>

~~~ruby
class Mobile::BaseController < ApplicationController

  LAYOUT = 'mobile'.freeze
  layout LAYOUT
  around_filter :dynamically_assign_layout
  
  private
  
  def dynamically_assign_layout
    self.class.layout false if request.get? && request.xhr?
    yield
  ensure
    self.class.layout LAYOUT
  end
  
end
~~~



<h2>Integration Testing &amp; Capybara-Webkit</h2>

<aside class="flash_info">
  UPDATE: After this article I posted a gist titled <a href="https://gist.github.com/1172519">Never sleep() using Capybara!</a> that details how to deal with CSS animations and AJAX requests using Capybara. Check it out too!
</aside>

<p>
  Those that know me are familiar that I do not use RSpec or Test::Unit but instead opt for a simple testing framework built into Ruby 1.9, <a href="/2011/03/26/using-minitest-spec-with-rails/">MiniTest::Spec</a>. I also use the <a href="https://github.com/thoughtbot/capybara-webkit">Capybara-WebKit</a> driver for my acceptance testing in both <a href="https://github.com/metaskills/holygrail_rails23">Rails 2.3</a> and <a href="https://github.com/metaskills/holygrail_rails31">Rails 3.1</a>'s standard integration test layer. Reference the <code>integration_test_helper.rb</code> file in each link above to learn how to use Capybara-WebKit with your rails app. Thanks to Wyatt Greene for his <a href="http://techiferous.com/2010/04/using-capybara-in-rails-3/">original article</a> on the matter. 
</p>

<p>
  So why the fuss? Well Capybara-WebKit is a headless WebKit browser that you can direct right from your test suite. What makes it so awesome is that it renders web pages with full JavaScript support and is much faster than selenium based Capybara drivers. Since jQuery Mobile is entirely based on JavaScript with HTML &amp; CSS3 - Capybara-WebKit is a perfect candidate to acceptance test your mobile application. The only gotcha is scoping your Capybara actions to a certain page that is dynamically loaded into the DOM. No problem! This is easily solvable using the page ids from the <code>mobile_page_id</code> helper method I mentioned above. So whatever your testing framework, here are a few helpers that are critical. Assume these are mixed into your test helper.
</p>

~~~ruby
private

include MobileConcerns::Helpers # Mentioned above.

def current_page_id(path=nil)
  path = path || current_path
  "div##{mobile_page_id(path)}.ui-page-active"
end

def current_page(path=nil)
  find(current_page_id(path))
end

def within_current_page(path=nil)
  within(current_page_id(path)) { yield }
end
~~~

<p>
  So let's go over these. First is the <code>current_page_id</code> method. Most of the time you are going to pass a path argument to this method, since the Capybara's <code>current_path</code> will only work correctly on the first page load, not each following AJAX page load which would change the location hash. See how this is using the <code>mobile_page_id</code> helper mixed in and described above? Next is the <code>current_page</code> helper. It finds the passed path/id with Capybara's find method. The <code>within_current_page</code> leverages Capybara's <code>within</code> helper to scope your action to that particular DOM element. Here is a classic example using these.
</p>

~~~ruby
should 'be able to navigate to logged in user page and change email' do
  login_as @user
  visit mobile_homepage_path
  click_on 'My Account'
  user_path = mobile_user_path(@user)
  assert current_page(user_path)
  # Change email
  new_email = Forgery(:email).address
  within_current_page(user_path) do
    fill_in 'email', :with => new_email
    click_button 'Save Changes'
  end
  @user.reload.email.must_equal new_email
end
~~~

<p>
  Hopefully you can see how this simple page id foundation can help you better test your jQuery Mobile app with Capybara-WebKit. What an awesome tool! I sometimes find it hard to believe we are now at the point where we can easily test this much JavaScript in a headless browser directed by Ruby.
</p>



<h2>Resources</h2>

<ul>
  <li><a href="http://jquerymobile.com/">jQuery Mobile</a></li>
  <li><a href="http://weblog.jamisbuck.org/2007/1/17/concerns-in-activerecord">Concerns Module Pattern</a></li>
  <li><a href="https://github.com/thoughtbot/capybara-webkit">Capybara-WebKit</a></li>
  <li><a href="https://github.com/metaskills/holygrail_rails23">Rails 2.3 App w/Capybara-WebKit Integration</a></li>
  <li><a href="https://github.com/metaskills/holygrail_rails31">Rails 3.1 App w/Capybara-WebKit Integration</a></li>
  <li><a href="http://techiferous.com/2010/04/using-capybara-in-rails-3/">Wyatt Green's Using Capybara In Rails 3</a></li>
  <li><a href="/2011/03/26/using-minitest-spec-with-rails/">Using MiniTest::Spec With Rails</a></li>
</ul>




