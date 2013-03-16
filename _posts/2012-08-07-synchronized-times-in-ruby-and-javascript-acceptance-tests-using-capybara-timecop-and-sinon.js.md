---
layout: post
title: Synchronized Times In Ruby &amp; JavaScript Acceptance Tests Using Capybara, Timecop &amp; Sinon.JS
categories: 
  - ruby-rails
  - javascript
---

<p>
  Any good Ruby developer that tests time-dependent code has used the <a href="http://github.com/jtrupiano/timecop">Timecop</a> gem. Timecop provides dead simple time travel and freezing capabilities to Ruby's standard library. But what if you are working on a rich JavaScript application that is backed and tested by something like a Rails application and you want to alter the test browser's clock as well? The answer is pretty simple, but let's first examine all the parts at play here.
</p>


<h2>Rails &amp; Capybara</h2>

<p>
  My examples leverage a very simple Rails integration test setup using <a href="http://github.com/jnicklas/capybara/#the-dsl">Capybara's DSL</a> within a basic Rails integration test case. Much like <a href="http://techiferous.com/2010/04/using-capybara-in-rails-3/">Wyatt has described here</a> and I have shown in a <a href="http://github.com/metaskills/holygrail_rails31">demo project on github</a>. So if your integration test setup is different, transpose my code to fit your needs. Also, there are a few drivers for Capybara that support full JavaScript integration, most notably capybara-webkit. But the latest on the scene is <a href="http://github.com/jonleighton/poltergeist">Poltergeist</a> which uses the badass PhantomJS project. I highly recommend you switch to this driver! That said, any Capybara driver that fully supports Capybara's <code>#execute_script</code> should work just fine.
</p>


<h2>Ruby &amp; JavaScript Times</h2>

<p>
  JavaScript date values represent time in milliseconds since Unix Epoch. Many 3rd-party JavaScript date libraries use millisecond integers for both instantiating and altering these objects. Ruby on the other hand has a much higher precision and thanks to ActiveSupport's core extensions to Ruby's date and time classes we can easily represent these values for JavaScript. Specifically, ActiveSupport adds a <code>#to_i</code> method that returns an integer which represents that date or time in seconds since Unix Epoch. In my example code, you will see that I multiple this by 1000 to get the millisecond representation.
</p>

<p>
  ActiveSupport also provides an <code>#advance</code> core extension to all date and time classes. This method is Valuable As Fuck&trade; since it returns a new time instance that has moved backwards or forwards given a hash of options. For example, returning a time instance moved forward by 20 minutes would look like this <code>@time.advance(minutes:20)</code>. See the <a href="http://api.rubyonrails.org/classes/Time.html#method-i-advance">documentation</a> for all the options and remember, you can provide negative values to move backward. In short, the advance method is awesome!
</p>


<h2>Faking Time With Sinon.JS</h2>

<aside class="flash_info">
  <a href="http://pivotal.github.com/jasmine/#section-Mocking_the_JavaScript_Clock">Jasmine 1.2 now has a similiar mocking technique for the JavaScript clock.</a>
</aside>

<p>
  <a href="http://sinonjs.org">Sinon.JS</a> is a small stand-alone library that provides spies, stubs and mocks for your JavaScript. To be honest, I use Jasmine and supporting extensions for most of these features. However, Sinon.JS has one killer feature, faking time! Yup, it allows you to freeze JavaScript's clock to a specific time and tick it forward as needed. I want you to ponder the benefits of that for awhile. Imagine you have time sensitive JavaScript code that uses <code>setInterval()</code> or the like. Sinon.JS will actually allow you to tick time forward and still maintain compatibility with that code's behavior! Basically Sinon.JS is equal to our Ruby Timecop gem and then some! Check out their <a href="http://sinonjs.org/docs/#clock-api">clock API</a> or read the code if you want to learn more. Remember, it is safe to include Sinon.JS in any existing JavaScript project since it will not do anything unless you ask it too. So no fear in it clashing with your other JavaScript test setup.
</p>


<h2>Putting It All Together</h2>

<p>
  So now the fun part, some code examples. First, you need to get Sinon.JS in your Rails JavaScript manifest. If you are smart, you have already setup a system where you can specify top level asset manifests for your JavaScript application per test environment. If not, you might want to take a look at two posts I previously published on setting up Jasminerice for testing Spine.JS applications.
</p>

<ul>
  <li><a href="/2012/01/16/rails-and-spine-js-jasmine-testing-part-1/">Rails &amp; Spine.JS - Jasmine Testing Part 1</a></li>
  <li><a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">Rails &amp; Spine.JS - Jasmine Testing Part 2</a></li>
</ul>

<p>
  These file snippets below assume you have an <code>integration.js</code> manifest which includes both a vendored Sinon.JS then a sub file which actually initializes Sinon.JS for our integration test run. In this case below, we are first processing a CoffeeScript file with ERB and then initializing Sinon.JS' fake timers to a default time. In my case, this is 8:30am central standard time. Now we can assure that Capybara's browser engine will be frozen at that time and ready to move forward for each test.
</p>

```javascript
// File app/assets/javascripts/integration.js

//= require sinon-1.4.2
//= require integration/sinon
//= require application
```

```coffeescript
# File app/assets/javascripts/integration/sinon.js.coffee.erb

<% central = ActiveSupport::TimeZone['Central Time (US & Canada)'] %>
<% today = central.parse('8:30am').to_i * 1000 %>

window.clock = sinon.useFakeTimers <%= today %>
```

<p>
  Now, the Ruby code. Here is slimmed down version of my base integration test case which <a href="/2011/03/26/using-minitest-spec-with-rails/">uses MiniTest::Spec</a> to drive Capybara tests. The first thing I do before any integration test is use Timecop to travel to 8:30am. This means that both Ruby and JavaScript are synced to the exact millisecond in time. Any test that needs to move time forward must call the <code>#advance_time</code> test helper. This method takes a hash of options which is passed directly to the <code>#advance</code> method I previously talked about. It measures the milliseconds between now and the advancement and sends that directly to Sinon.JS' fake timers using Capybara's <code>#execute_script</code> method. So calling <code>advance_time(seconds:20)</code> in Ruby now moves time forward in both Ruby and JavaScript. Epic win!!!
</p>

```ruby
# File test/test_helper_integration.rb

require "test_helper"
require "capybara/rails"

module ActionDispatch
  class IntegrationTest

  include Capybara::DSL

  before { travel '8:30am' }
  after  { Capybara.reset_sessions! }
  
  private

  def travel(parseable_time)
    Timecop.return
    Timecop.travel parse_time(parseable_time)
  end

  def parse_time(time)
    time.is_a?(String) ? Chronic.parse(time) : time
  end

  def advance_time(options)
    now_ms = Time.current.to_i * 1000
    Timecop.travel Time.current.advance(options)
    traveled_ms = (Time.current.to_i * 1000) - now_ms
    advance_sinon traveled_ms
  end

  def advance_sinon(ms)
    page.execute_script "if (window.clock) { window.clock.tick(#{ms}); }"
  end

end
```

<p>
  I have found the following technique critical to fully testing a recent time based JavaScript application I have developed. I hope you find this technique useful as well and as always, please contribute your thoughts or questions below. Cheers!
</p>


<h2>Resources</h2>

<ul>
  <li><a href="http://github.com/jtrupiano/timecop">Timecop - A gem providing "time travel" and "time freezing" capabilities, making it dead simple to test time-dependent code.</a></li>
  <li><a href="http://github.com/jnicklas/capybara/">Capybara - Acceptance test framework for web applications.</a></li>
  <li><a href="http://github.com/jonleighton/poltergeist">Poltergeist - A PhantomJS driver for Capybara.</a></li>
  <li><a href="http://sinonjs.org">Sinon.JS - Standalone test spies, stubs and mocks for JavaScript.</a></li>
</ul>



