--- 
layout: post
title: How to use Passenger (mod_rails) with rails edge 2.1
disqus_id: /2008/04/22/how-to-use-passenger-mod_rails-with-rails-edge-2-1/
categories: 
  - heuristics
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: This issue has <a href="http://github.com/FooBarWidget/passenger/commit/53301de464b323d364723854d3a8d293ab8327d6">now been resolved</a> in the official release.
</aside>

<p>
  If you are like me and have <a href="/2008/04/13/now-on-passenger-mod_rails/">been using passenger</a>, then you may have run into an issue when working with rails edge. I mean the REAL rails edge on Git, not that fancy rake task which I think is still pointing to a subversion snapshot. Let me take an aside on how to freeze rails edge to a project that you are managing in Git. This method is akin to using svn:externals. As a cop out, here are 3 links that you should read to learn how.
</p>

<ul>
  <li><a href="http://railsontherun.com/2008/4/16/freezing-rails-with-git">http://railsontherun.com/2008/4/16/freezing-rails-with-git</a></li>
  <li><a href="http://woss.name/2008/04/09/using-git-submodules-to-track-vendorrails/">http://woss.name/2008/04/09/using-git-submodules-to-track-vendorrails/</a></li>
  <li><a href="http://git.or.cz/gitwiki/GitSubmoduleTutorial">http://git.or.cz/gitwiki/GitSubmoduleTutorial</a></li>
</ul>

<p>
  OK... now that you know how to freeze the real rails edge to your git project, and if you have been using passenger, you may now have seen this error below. The basic problem is that passenger bypasses the work that the rails boot.rb does and in doing so, it only accounts for setting RAILS_ROOT during the ApplicationSpawner process and not the FrameworkSpawner process. In the latest rails, ActionPack is now relying on RAILS_ROOT to be set by calling Rails.root (shortcut method to that constant) when loading. So <a href="http://github.com/metaskills/passenger/commit/69afcd75425a89c9d17d1fc40c0a7571d6bd547c ">my fix was to add the RAILS_ROOT to the FrameworkSpawner class</a>.
</p>

```text
Framework that failed to load: Vendor directory: /Users/foo/project/vendor/rails 
Error message: Anonymous modules have no name to be referenced by 
Exception class: ArgumentError 
```


<h2>Resources</h2>

<ul>
  <li><a href="http://code.google.com/p/phusion-passenger/issues/detail?id=29&can=1&q=edge&colspec=ID%20Type%20Status%20Priority%20Milestone%20Stars%20Summary">Here is the Passenger Ticket</a></li>
  <li><a href="http://github.com/FooBarWidget/passenger/commit/53301de464b323d364723854d3a8d293ab8327d6">The official Passenger change set that fixed this issue.</a></li>
<ul>



