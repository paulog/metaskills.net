--- 
layout: post
title: Hell'OO HomeMarks
disqus_id: /2008/08/18/in-hell-oo-for-homemarks/
categories: 
  - heuristics
  - ruby-rails
  - javascript
---


<p>
  Well HomeMarks v2.0 is done and ready for the public. You can download it from my <a href="http://github.com/metaskills/homemarks/tree/master">Github project</a> page. It has a real simple <code>rake app:bootstrap</code> task that I came up with over the weekend will have you running a local copy in only a few seconds, give it a try. Sometime over the next few days, I'll move over the live site to this code base too. If you have not yet kept up on the implementation mantra I set down for the HomeMarks v2 project, you might want to <a href="/2008/05/24/the-ajax-head-design-pattern/">read an older post</a> as well as this excerpt from the project README.
</p>

<blockquote>
  HomeMarks was built using the Ruby on Rails framework with a heavy emphasis on object oriented JavaScript to make AJAX requests to a RESTful back-end. Unlike most Rails applications it does not use any inline JavaScript helpers nor does it rely on RJS (Remote JavaScript) for dynamic page updates. Instead it is nearly 100% unobtrusive JavaScript which uses simple HEAD or JSON responses to communicate to the objects on the page. This has yielded very slim controller code which is decoupled from the views and easily testable in isolation at a functional level.
</blockquote>

<p>
  So what do I think of this implementation and writing all this object oriented JavaScript? Well - its been real hard, I've been in JS Hell-OO for the better part of a few weeks as I came closer to finishing the project. Coming from a self inflicted addiction to unit/functional testing in rails, it is very hard to get into the habit of opening the browser to test view code. A big part of me hates writing all this JS without one bit of unit testing. I've heard that Prototype v2 will have unit testing, that would be awesome! 
</p>

<p>
  <span class="photofancy floatr ml20"><img src="/assets/oojs.gif" alt="Hell'oo for HomeMarks" width="384" height="237" /></span>
  So, bitching aside, I am very happy that my JS-fu is stronger. The end result besides all those buckets of water up the hill is that I can now really appreciate the ruby object model and how prototype strives to make JS class construction follow its example. Kudos really goes to Sam Stephenson for bridging the two object models. As I learned more about how prototype's classes work, I actually learned quite a bit about ruby too. Using that knowledge I gave a talk at the <a href="http://757rb.org/">Hampton Roads Ruby Users Group (757rb)</a> on the topic. You can download the PDF version of the slides <a href="http://www.slideshare.net/metaskills/oo-java-script-class-construction"> from the files section of the Google group</a> page. Look for the file titled <em>OO JavaScript Class Construction</em>
</p>


