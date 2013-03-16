---
layout: post
title: Rails &amp; Spine.JS - Using The CoffeeScript Source
categories: 
  - ruby-rails
  - javascript
---

<p>
  Our <a href="http://addyosmani.com/blog/short-musings-on-javascript-mv-tech-stacks/">options for JavaScript MVC frameworks</a> are numerous these days. While working on the third major rewrite of my personal bookmarking application, <a href="http://homemarks.com/">HomeMarks</a>, I decided to learn <a href="http://documentcloud.github.com/backbone/">Backbone.js</a>. Thankfully a local friend of mine <a href="https://twitter.com/#!/brennandunn/status/153487553062907905">recommended</a> that I try <a href="http://spinejs.com/">Spine.JS</a>. I was immediately hooked! 
</p>


<h2>Why Spine.JS?</h2>

<p>
  Spine.JS is is authored in <a href="http://coffeescript.org/">CoffeeScript</a> and that is a big deal for me. I will never write raw JavaScript again, which I consider, the deprecated syntax. So a JavaScript MVC framework that is written in CoffeeScript means I can read the source, learn from it and even debug it if necessary. Sure, I can read raw JavaScript or just rely on documentation. But nothing beats reading source code. A practice I think good developers follow. So here is an example of the <a href="https://github.com/documentcloud/backbone/blob/master/backbone.js#L151">Backbone.js model source</a> compared to reading <a href="https://github.com/maccman/spine/blob/master/src/spine.coffee#L83">Spine.JS model source</a> and I think you will see the difference.
</p>

<p>
  Source code legibilty is not the only reason to use Spine.JS. It is also very lightweight and requires no other JavaScript dependencies. Take for example Backbone.js which requies the use of <a href="http://documentcloud.github.com/underscore/">Underscore.js</a> and think for a moment why. Underscore.js makes mundane tasks in JavaScript like itterators and event binding much more friendly. But this is all moot when you are using CoffeeScipt's loops and comprehensions. And take my advice, CoffeeScript has so much more to offer. One of my personal favorites is the existential operator!
</p>



<h2>Coffee Time!</h2>

<p>
  So, are you with me and want to try out Spine? Great! But do not rush in and use the <a href="http://github.com/maccman/spine-rails">spine-rails</a> gem. Sure it gives you a nice way to require the Spine JavaScript files in the asset pipeline and some fancy generators. But when you break it down, there are better ways to get Spine's source files and who the hell uses generators? I mean, the only useful one is the initial <code>rails g spine:new</code> generator. Past the initial project setup, the spine-rails gem really does nothing but lock you down to an explicit version of Spine tied to that gem release. 
</p>

<p>
  I highly advise that new-comers to Spine start off with the spine-rails gem and its new project generator. Then quickly switch to just including the spine source using a git submodule. This will give you the benefit of using the source CoffeeScript files and tracking Spine's git repo which is getting good active development. So let's live on the edge and read some source. First uninstall the spine-rails gem if you have it and add the Spine project as a git submodule to your git repo.
</p>

<pre class="command">
$ mkdir -p vendor/assets/javascripts
$ git submodule add git://github.com/maccman/spine.git vendor/assets/javascripts/spine
</pre>

<p>
  This adds the Spine project to your <code>vendor/assets/javascripts/spine</code> directory. Which means it can now be leveraged by Rails asset pipeline using Sprockets. So if you had used the spine-rails generator above and had your spine requires in <code>app/assets/javascripts/app/index.js.coffee</code>, you would now be able to change what should have looked like this:
</p>

```ruby
#= require spine
#= require spine/manager
#= require spine/ajax
#= require spine/route
```

<p>
  To something like the following. Since the <code>spine/src</code> directory is where the source CoffeeScript files from our submodule above and Sprockets will render these just fine, it all just works!
</p>

```ruby
#= require spine/src/spine
#= require spine/src/manager
#= require spine/src/ajax
#= require spine/src/route
```

<p>
  So now you can easily update your Spine dependency using a simple git workflow with the added benefit that you can open up any of the source CoffeeScript files and learn Spine from the inside out. You can even change the source or put in console debugging to see what is happening and your application files will recompile via Sprockets on the next request.
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
  <li><a href="http://coffeescript.org/">CoffeeScript - A little language that compiles into JavaScript.</a></li>
  <li><a href="http://github.com/maccman/spine-rails">The Spine Rails Gem</a></li>
</ul>


