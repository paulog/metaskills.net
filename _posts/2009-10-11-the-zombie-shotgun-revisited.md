--- 
layout: post
title: The Zombie Shotgun Revisited 
disqus_id: /2009/10/11/the-zombie-shotgun-revisited/
categories: 
  - heuristics
  - ruby-rails
---

<p>
  <span class="photofancy floatr ml20"><img src="/assets/zombie_shotgun.jpg" alt="Resident Evil Zombie Shotgun" width="320" height="213" /></span> My how time flies. Over a year ago <a href="/2008/07/06/stop-exception-notifications-with-the-zombieshotgun/">I created a simple bit of code</a> that was useful for stopping ActionController routing errors from common Microsoft attacks from sending exception notification emails. Well now most people do not use exception notifications emails in favor of apps like Hoptoad. And hey, most code like this has moved to Rack middlewares.
</p>

<p>
  Yesterday I noticed a <a href="http://coderack.org">rack code competition</a> that encouraged "most useful and top quality Rack middlewares". Well the Zombie Shotgun is pretty useful to me, but I'm sure it's not top quality. That said, I did take the time to finally pick up on the rack internals and learn how to use <a href="http://github.com/brynary/rack-test">rack-test</a>. If you want to check out the new tested Zombie Shotgun middleware and how I tested it using Shoulda, <a href="http://github.com/metaskills/rack-zombieshotgun">go see the project on my github page</a>. Also, here is my <a href="http://coderack.org/users/MetaSkills/entries/15-zombie-shotgun">CodeRack entry</a>.
</p>

