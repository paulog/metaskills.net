---
layout: post
title: The Browser Is Dead?
categories: 
  - ruby-rails
  - javascript
  - apple-os
---

<p>
  <span class="photofancy floatr mr10 mb30 ml30">
    <img class="" src="/assets/the_browser_is_dead.png" width="368" height="2969" alt="Converstation between DHH And Dave Thomas on Rails 3.1"/>
  </span>
  This <a href="https://twitter.com/#!/pragdave/status/62510978893492224">twitter post by Dave Thomas</a> sparked an interesting back and forth with DHH on how Rails 3.1 could be more opinionated towards web development for the browser. A short time before &ndash; it was announced that Rails would include CoffeeScript and Sass as defaults for JavaScript and CSS authoring. FWIW, both of these new defaults are in my opinion the best of the breed fore each task. If you have not done so, I suggest taking a quick read down this thread that I put together with screenshots from my Twitter.app.
</p>

<p>
  The topic is one I have thought often about over the past 6 years. I started writing Rails applications before 1.0 was released and I have seen the changes and felt each one. Having written both web and native iOS apps for <a href="http://homemarks.com">HomeMarks.com</a> and retooling the whole application for each major Rails version, I think it gives me a good insight into the pain here. For instance when <a href="http://ajaxian.com/archives/rails-rjs-for-ajax-101">RJS in Rails</a> first came out, HomeMarks took extreme advantage of it in very clever ways. Pushing rendered partials out and communicating to the browser's JavaScript environment was fun and rewarding at the time. It also felt really DRY at times.
</p>

<p>
  As I started to <a href="/2008/08/18/in-hell-oo-for-homemarks/">learn more advanced JavaScript techniques</a> for organizing my rich client code, I ended up rewriting HomeMarks again under the poorly named moniker <a href="/2008/05/24/the-ajax-head-design-pattern/">"The AJAX Head Design Pattern"</a>. It even caught a little bit of attention by the Ajaxian and <a href="http://voodootikigod.com/ajax-head-design-pattern">Chris Williams</a>. Essentially what I ended up building was my own minimal <a href="http://documentcloud.github.com/backbone/">backbone.js</a> in that I had a rich MVC layer all written in JavaScript. This usage of Rails as an API foundation on top of a rich JavaScript application was in my opinion a few years ahead of the average curve. Though it was very rewarding from a JavaScript as a language first, DOM second development perspective &amp; it yielded a very clean way to leverage the same RESTful controller actions used by JavaScript in the browser with a <a href="/2010/02/12/synchronizing-core-data-with-rails-3-0-0-pre/">native iOS app that integrated with Core Data</a>, it still felt like an over achievement or harder than it had to be.
</p>

<p>
  So as I embark on my third complete rewrite of HomeMarks, I have to think about my tools and techniques again and what the end goal may look like. I have thrown away thousands of lines of OO-JavaScript written in Prototype.js and <a href="https://gist.github.com/973483">started rewriting them in CoffeeScript</a>. Though there are still some tough decisions to be made on what my complete stack will look like and how HomeMarks v3 will hopefully again push the new web forward, I cant help but to agree with both Dave and DHH. I also share James A Rosen's opinion summed up in <a href="http://twitter.com/#!/jamesarosen/status/70544535373086720">this tweet</a>.

  <blockquote style="width:220px;">
    If I had known that RailsConf would be How-to-Use-Rails-as-a-Backend-for-Javascript-Conf, I would've gone!
  </blockquote>
</p>