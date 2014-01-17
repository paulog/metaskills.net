---
layout: post
title: Rails &amp; Spine.JS - Jasmine Testing Part 1
categories: 
  - ruby-rails
  - javascript
---

<p>
  In my <a href="/2012/01/15/rails-and-spine-js-using-the-coffeescript-source/">previous article</a> I talked a little bit about why I decided to use Spine.JS and how to include the CoffeeScript source into your Rails project using git submodules. Now I would like to talk about testing your brand new Spine.JS application. Afterward, be sure to <a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">read the second part</a> to this article which covers more advanced aspects of your Spine.JS application specs.
</p>


<h2>Testing JavaScript</h2>

<p>
  OK, so like any good programmer, you want to test your JavaScript web application, but how? Like most, I kept finding that <a href="http://pivotal.github.com/jasmine/">Jasmine</a> was the de facto testing framework that most Rails developers were using. For the newly aquatinted, Jasmine describes itself as behavior-driven and sports a clean spec style using <code>describe</code> and <code>it</code> blocks similar to RSpec or MiniTest::Spec. But maybe you, like me, quickly dismissed Jasmine since you sure as hell were not going to hit refresh or F5 in your browser every time you wanted to run your damn specs. After all, this is 2012 and Rails developers do not test with a browser! So why should I start now?
</p>

<p>
  Luckily I am a big fan of Ruby's <a href="https://github.com/guard/guard">Guard</a> gem, a simple library that responds to file system events. The guard project literally has TONS of other guard gems that automate everything from running test files to restarting your development server. Thankfully my search for JavaScript testing with Guard in mind brought me right back to Jasmine. Enter the <a href="https://github.com/netzpirat/guard-jasmine">guard-jasmine</a> gem and the wonderful world of a headless JavaScript testing piped right down to your terminal window!
</p>



<h2>Guard, Jasmine &amp; Jasminerice</h2>

<p>
  So this is our holy trinity and to be honest, there are a lot of moving parts under the stack. Things will seem to get complicated quick, but don't worry. I will give you a brief overview of the moving parts and then get right down to the basics of how you can start using Guard and Jasmine to test your Sine.JS application. 
</p>

<p>
  First, let's cover Guard. It is a simple gem that uses a <code>Guardfile</code> at the root of your project to control how other guards are triggered. I'll give you an example Guardfile later. But for starters, <a href="https://github.com/guard/guard">read the documentation</a> on what special libraries may be needed for file system events or notifications on your specific platform. In my case, I us Mac OS X and purchased the latest Growl 1.3. So my example <code>Gemfile</code> below will have the <a href="https://rubygems.org/gems/ruby_gntp">ruby_gntp</a> gem included in the spec.
</p>

<p>
  Next up is the guard-jasmine gem. My instructions assume you are running a Rails 3.1 or 3.2 app and that you are taking full advantage of the asset pipeline and CoffeeScript. Many of these details can be found on the <a href="https://github.com/netzpirat/guard-jasmine">guard-jamine's Rails 3.1 setup</a> section of their readme page. The underlying components of guard-jasmine are two a libs named <a href="https://github.com/bradphelan/jasminerice">Jasminerice</a> and <a href="http://www.phantomjs.org/">PhantomJS</a>. Jasminerice is a simple Rails engine that brings in the Jasmine source files to the asset pipeline while running a rack app mounted to <code>/jasmine</code> to run your specs from your current application. PhantomJS is yet another headless WebKit based on Qt that has a rich JavaScript API which guard-jasmine delegates to. Is your head spinning? Mine was too.
</p>


<h2>Put It All Together</h2>

<p>
  Here is the bullet train to getting this stack up and running. First, you will need to get PhantomJS installed. If Homebrew is your thing, just do <code>$ brew install phantomjs</code>. Or you can <a href="http://code.google.com/p/phantomjs/downloads/list">download one of their precompiled binaries</a> for your specific platform. This is what I opted to do and I just placed the phantomjs in my <code>PATH</code> somewhere.
</p>

<p>
  Next, we need to get the gems in our <code>Gemfile</code>. Here is how mine are setup. I have them in both the <code>:develpment</code> and <code>:test</code> groups since Jasminerice runs in both of those Rails environments. I also have that ruby_gntp dependency since I am using Growl on Mac OS X, YMMV.
</p>

~~~ruby
group :development, :test do
  gem 'jasminerice'
  gem 'guard-jasmine'
  gem 'ruby_gntp'
end
~~~

<p>
  So that was easy, now on to our <code>Guardfile</code>. Here is mine below. Notice how I put my JavaScript related guards into a <code>js</code> group? This is a seldom used feature of Guard and it means I can monitor only my jasmine specs by starting guard off using <code>$ guard -g js</code> and my other guards in my <code>ruby</code> group, like minitest, will be ignored.
</p>

~~~ruby
group 'js' do

  guard 'jasmine' do
    watch(%r{app/assets/javascripts/myapp/index\.js\.coffee$})  { "spec/javascripts" }
    watch(%r{app/assets/javascripts/myapp/(.+)\.js\.coffee$})   { |m| "spec/javascripts/#{m[1]}_spec.js.coffee" }
    watch(%r{spec/javascripts/(.+)_spec\.js\.coffee$})          { |m| "spec/javascripts/#{m[1]}_spec.js.coffee" }
    watch(%r{spec/javascripts/spec\.js$})                       { "spec/javascripts" }
    watch(%r{spec/javascripts/spec_helper\.js$})                { "spec/javascripts" }
    watch(%r{spec/javascripts/jasmine-myapp.*})                 { "spec/javascripts" }
  end

end
~~~

<p>
  This setup assumes a few things. First that you are only testing your Spine.JS application and that those files are in the <code>app/assets/javascripts/myapp</code> directory. That myapp directory could just be <code>app</code> in your case if you used the <a href="http://github.com/maccman/spine-rails">spine-rails</a> gem without the <code>--app</code> option. In my case, that folder is named <code>homemarks</code>. This Guardfile also assumes that your JavaScript app and specs are CoffeeScript files and that specs are in the <code>spec/javascripts</code> folder specified by Jasminerice. You are going to want to follow some file naming convention now too. For example if you have a Spine.JS app file in <code>app/assets/javascripts/myapp/models/post.js.coffee</code>, then you are going to want the matching spec in <code>spec/javascripts/models/post_spec.js.coffee</code>. So saving each of those files will trigger that specific spec to run. There are also so some watchers on important root files like your spec.js sprockets manifest and your Spine.JS app index. Changing any of those files will result in your entire spec suite running again. 
</p>


<h2>To Be Continued...</h2>

<p>
  I will go into more details on the <code>spec_helper.js</code> and <code>jasmine-myapp</code> files above in <a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">the second part</a> of this article. For now you should be set to start writing specs like the one below and seeing them run by either visiting the <code>/jasmine</code> URL of your running Rails application or by using Guard in your terminal window.
</p>

~~~ruby
describe 'App', ->
  
  it 'sets el', ->
    expect(MyApp.Application.el).toEqual $('#app')

  it 'sets the userId as a property on itself', ->
    expect(MyApp.Application.userId).toEqual @bob.id
~~~

<p>
  <a href="/2012/01/17/rails-and-spine-js-jasmine-testing-part-2/">Continue to part two...</a>
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
  <li><a href="http://spinejs.com/">Spine.JS - Build Awesome JavaScript
  MVC Applications</a></li>
  <li><a href="http://pivotal.github.com/jasmine/">Jasmine - BDD for JavaScript. (I disagree, TDD too!)</a></li>
  <li><a href="http://github.com/netzpirat/guard-jasmine">Guard Jasmine - Automated Jasmine tests to your console.</a></li>
  <li><a href="http://github.com/bradphelan/jasminerice">Jasminerice - Pain free CoffeeScript testing under Rails 3.1</a></li>
  <li><a href="http://www.phantomjs.org/">PhantomJS - Headless WebKit with JavaScript API.</a></li>
</ul>

