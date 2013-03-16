--- 
layout: post
title: 757rb Memcached Talk
disqus_id: /2009/07/18/757rb-memcached-talk/
categories: 
  - heuristics
  - ruby-rails
---

<p>
  <img src="/assets/memcache_slides.png" alt="757.rb Memcached Presentation" class="floatr ml20" width="250" height="224" /> Earlier this week I gave a talk at our local ruby users group, <a href="http://www.757rb.org/">757.rb</a>, about Memcached. Here recently I started picking it up again so that I could keep track of large sets of PK/FK changes during a big database move. In doing so, I decided to dig deep into it's internals and get a better grasp of how I would use it when I decided to do some serious fragment caching again.
</p>

<p>
  The biggest thing I took away from the talk was -- First, that you can use an array of strings/objects for the key passed to various caching methods. If the object responds to #cache_key, the return of that method would be used. The default implementation of ActiveRecord's #cache_key method is to use the updated_at column for a unique timestamp. It is even smart enough to know weather the object is new or not. Thanks to Brennan Dunn for pointing that out! The second and most important thing was an article by <a href="http://www.motionstandingstill.com/starting-simple-with-rails-caching/2008-11-27/">Nahum Wild</a> that talked about item level caching. This seemed like a strange place to start as typically I would start with a page or big sections of fragments. But as the demo app shows, this is an incredibly good place to start. Especially if you have heavy hitting pages of paginated views that talk to deep nested associations.
</p>


<h2>Resources</h2>

<ul class="mt10">
  <li><a href="http://www.slideshare.net/metaskills/memcached-presentation-757rb">Slideshare Version of my Memcached Presentation</a></li>
  <li><a href="/assets/play_cache.zip">The Demo Rails App of my Memcached Presentation</a></li>
  <li><a href="http://www.motionstandingstill.com/starting-simple-with-rails-caching/2008-11-27/">Da Man, Nahum Wild's advice</a></li>
  <li><a href="http://github.com/andrewfromcali/mcinsight/tree/master">GUI Version Of Memcached For Mac</a></li>
  <li><a href="http://github.com/nkallen/cache-money/tree/master">CacheMoney Rails Plugin</a></li>
</ul>



