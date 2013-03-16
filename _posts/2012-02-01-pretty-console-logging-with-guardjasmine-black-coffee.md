---
layout: post
title: Pretty Console Logging With Guard::Jasmine &amp; Black Coffee
categories: 
- ruby-rails
- javascript
---

<p>
  OK I know I promised that we would start the dive into <a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">testing your Spine.JS application using Jasmine(rice) in my last article</a>, but this is a good diversion. If you are new to my latest series on Spine.JS and Jasmine, scroll on down to the bottom to the related section and read back. However, for those that might be more familiar with Jasmine and specifically <a href="https://github.com/netzpirat/guard-jasmine">Guard::Jasmine</a> and ever felt the pain that debugging from that terminal window was lacking, read on! Even if your new to Guard::Jasmine and <a href="http://github.com/bradphelan/jasminerice">Jasminerice</a> I still suggest you setup these elegant hacks to make your testing go that much smoother.
</p>


<h2>So What Is The Problem?</h2>

<p>
  Guard::Jasmine allows you to continuously test your JavaScript right from the terminal window just like your Ruby code. The only drawback is that the console debugging is less than helpful. Guard::Jasmine will not allow you do view a string version of every object nor see line numbers of calling files. Both of these are invaluable when your stuck in a testing hole and just need to inspect a few objects. So if you are tired of seeing <code>[object Object]</code> in your Guard::Jasmine output, let's fix it right away.
</p>


<p>
  The first place we need to patch things up is Guard::Jasmine itself. In my <a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">last article</a> I covered how to monkey patch Jasminerice in a <code>config/initializers/jasminerice.rb</code> file. My Guard::Jasmine freedom patch will be placed in that same file. Pasted below, this does two things vs the <a href="http://github.com/netzpirat/guard-jasmine/blob/master/lib/guard/jasmine/runner.rb">original</a>. First it changes the <code>report_specdoc_logs</code> method to not pass <code>true</code> to the <code>format_log_message</code> method. Second, the <code>format_log_message</code> method itself now has the message regular expression changed to a multi-line scan. It will also look out for a custom prefix tag and allow it to pass through. This is for our pretty objects. Anything else now outputs the message with the file and line number, something previously stripped by Guard::Jasmine.
</p>

```ruby
# coding: utf-8
if defined?(Jasminerice) && Jasminerice.environments.include?(Rails.env)

  # Other Jasminerice patches from:
  # http://metaskills.net/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/
  # ...
  
  # Patch Guard::Jasmine to use a custom formatter for log messages that
  # allows multi-line objects to be printed with the line numbers.
  module Guard
    class Jasmine
      module Runner
        class << self
        
          def report_specdoc_logs(spec, options, level)
            if spec['logs'] && (options[:console] == :always || (options[:console] == :failure && !spec['passed']))
              spec['logs'].each do |log|
                Formatter.info(indent("    • #{ format_log_message(log) }", level))
              end
            end
          end
          
          def format_log_message(message)
            if message =~ /(.*?) in http.+?assets\/(.*)\?body=\d+\s\((line\s\d+)/m
              pp_prefix = '[myLog]'
              msg, file, line = $1, $2, $3
              if msg.starts_with? pp_prefix
                msg.sub pp_prefix, ''
              else
                "#{msg} in #{file} on #{line}"
              end
            else
              message
            end
          end
          
        end
      end
    end
  end
  
end
```


<h2>A JavaScript Pretty Print Library</h2>

<p>
  So now that we have Guard::Jasmine not stripping multi-line console messages to one line and printing file and line numbers, we are ready to hook into it. But first we need a JavaScript library that pretty prints objects for us and wrap this all up behind our own interface to <code>console.log</code>. During my research I found a library called jsDump and decided to <a href="https://github.com/NV/jsDump">use a fork of the project on github</a>. So go download that file and place it in <code>vendor/assets/javascripts/jsDump.js</code> of your Rails project. Next you will want to add it to your <code>spec/javascripts/spec.js</code> manifest like so. My below example is pulled right from my previous article and I have placed jsDump right after my jasmine files.
</p>

```javascript
#= require jasminerice
#= require mock-ajax
#= require jsDump
#= require application
#= require jasmine-myapp
#= require_tree ./lib/
#= require_tree ./models/
#= require_tree ./controllers/
#= require_tree ./views/
```


<h2>Logging Helpers - Using BlackCoffee</h2>

<p>
  In my last article I mentioned how I would talk more about the <code>jasmine-myapp</code> file in the manifest above. Now is the time. I recommend that this file is place to write all your top level helpers and functions for Jasmine or any other testing libraries in your JavaScript stack. Think of this file as your Ruby <code>test_helper.rb</code> or your own extensions to <code>ActiveSupport::TestCase</code>. My advice is that these functions all be written to pollute the time level namespace as other Jasmine helpers do, like the <code>beforeEach</code> and <code>clearAjaxRequests</code>. This makes things easier but be conscious of that decision and write functions keeping that in mind.
</p>

<p>
  The only thing working against is is Sprockets/Tilt rendering CoffeeScript files in their own closure. Which is something you should really not fight! But in this case I think it is fine to have this particular file avoid that. Which allows us to (a) write our helper code in CoffeeScript and (b) use these functions as helpers in the global space like other Jasmine helpers. So enter my <a href="http://github.com/metaskills/sprockets-blackcoffee">Sprockets BlackCoffee</a> gem. This is a simple gem that exposes a CoffeeScript template that uses the <code>--bare</code> option to keep your file from being wrapped in a closure. All you have to do is give the file a <code>.js.black_coffee</code> extension and it will just work. So let's assume you have a <code>spec/javascripts/jasmine-myapp.js.black_coffee</code> created and the gem installed like so.
</p>


```ruby
group :assets do
  # ...
  gem 'sprockets-blackcoffee'
end
```

<p>
  Finally, here are examples of my logging helpers in the <code>jasmine-myapp</code> file. The <code>myLogParser</code> uses jsDump to get back a pretty formatted string of any object if that object is not already a string. The primary logging helper <code>myLog</code> will prefix your message with <code>[myLog]</code> so the Guard::Jasmine recognizes the message and outputs only the object. The last helper <code>myLogLine</code> will do just like the other, but will allow the file and line number tobe printed too.
</p>

```ruby
myLog = (obj) ->
  console.log "[myLog]#{myLogParser(obj)}"

myLogLine = (obj) ->
  hmLogParser(obj)

myLogParser = (obj) ->
  if typeof obj is 'string' then obj else jsDump.parse obj
```


<h2>In Practice</h2>

<p>
  Here is an example of a Jasmine spec where I was using console.log before the patches above and what it would output to the terminal.
</p>

```ruby
#= include spec_helper

describe 'User', ->

  it 'has been configured with proper attributes', ->
    @user = new User id: 2, email: 'bob@test.com'
    console.log @user.attributes()
    expect(@user.email).toEqual 'bob@test.com'
```

<pre class="code">
User
  ✘ has been configured with proper attributes
    ➤ Expected 'foo@bar.com' to equal 'bob@test.com'.
    • [object Object]
ERROR: 1 specs, 1 failures
in 0.653 seconds  
</pre>

<p>
  There is the totally helpful <code>[object Object]</code>. But if we now change our code to leverage out patches and change console.log to <code>hmLog @user.attributes()</code> to use our helper. We will get this. If you need a pretty object with line numbers. Just use <code>hmLogLine</code>.
</p>

<pre class="code">
User
  ✘ has been configured with proper attributes
    ➤ Expected 'foo@bar.com' to equal 'bob@test.com'.
    • {
       "email": "foo@bar.com",
       "id": 2
    }
ERROR: 1 specs, 1 failures
in 0.653 seconds
</pre>


<p>
  I hope this is helpful to anyone using Guard::Jasmine with a desire to see better output. If you continue to follow my series, my next post will be a deeper dive into Jasmine testing of Spine.JS applications.
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
  <li><a href="http://github.com/metaskills/sprockets-blackcoffee">Sprockets BlackCoffee Gem</a></li>
  <li><a href="http://github.com/netzpirat/guard-jasmine">Guard::Jasmine - Automatically tests your Jasmine specs on Rails.</a></li>
  <li><a href="http://github.com/bradphelan/jasminerice">Jasminerice - Pain free CoffeeScript testing under Rails 3.1.</a></li>
</ul>

