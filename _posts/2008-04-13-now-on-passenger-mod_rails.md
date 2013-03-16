--- 
layout: post
title: Now on Passenger (mod_rails)
disqus_id: /2008/04/13/now-on-passenger-mod_rails/
categories: 
  - apple-os
  - heuristics
  - ruby-rails
---


<p>
  <img src="/assets/passenger_install.png" width="515" height="308" class="floatr ml20" /> Well this is working out well so far. I'm really liking the <a href="http://modrails.com/index.html">Passenger (mod_rails for Apache)</a> extension. Right now I have this Mephisto site running it and it seems to be doing really well. Also, most people do not do this, but I run a full development stack Apache/MongrelCluster to mimic production boxes the best way I can. Now I am running mod_rails on all my development hosts.
</p>


<h2>Some Things I Like</h2>

<ul>
  <li>I do not have to fuss with OS X launchd startup scripts for my mongrels, just Apache.</li>
  <li>Typically in a high volume site that runs mongrel behind an Apache proxy balancer, will get a large timeout and proxy error even if the mongrels are immediately available. Passenger has a <a href="http://www.modrails.com/documentation/Users%20guide%20Nginx.html#_redeploying_restarting_the_ruby_on_rails_application">nice way to restart</a> the app, just touch the tmp/restart.txt file.</li>
</ul>


<h2>Some Things I'm Waiting For</h2>

<ul>
  <li>The RailsEnv can not be set per virtual host. You have to set RAILS_ENV = 'development' in each app if you want to run mixed virtual hosts with different environments.</li>
  <li>Normally I would pass environment variables to the console when issuing mongrel cluster starts. I would really love to see Passenger support <a href="http://httpd.apache.org/docs/2.2/env.html">apache environment variables.</li>
</ul>



