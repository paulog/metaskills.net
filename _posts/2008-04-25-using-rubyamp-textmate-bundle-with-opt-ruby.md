--- 
layout: post
title: Using RubyAMP TextMate Bundle With /opt Ruby
disqus_id: /2008/04/25/using-rubyamp-textmate-bundle-with-opt-ruby/
categories: 
  - apple-os
  - heuristics
  - ruby-rails
  - textmate
---

<p>
  I've been a TextMate user for a long time now and I'm still finding new things to do with it. Here recently I wanted to use the <a href="http://code.leadmediapartners.com/tools/rubyamp">RubyAMP TextMate Bundle</a> and was a little miffed to find that it was pointing to my OS X system ruby. The error message looked something like this when it went looking for my ruby/gems.
</p>

~~~text
No such file to load -- appscript (LoadError) from 
/System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/lib/ruby/1.8/rubygems/custom_require.rb:27:in `gem_original_requireâ€™
/System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/lib/ruby/1.8/rubygems/custom_require.rb:27:in `requireâ€™
...
~~~

<p>
  My problem is that I use ruby installed from MacPorts (yes I have Leopard). I do this because I put a lot of crap and extra dependencies into my opt direcotry and I do not trust Apple to NOT blow away the crazy dependencey hell I would have ended up creating in the standard bin directories if I chose not to use MacPorts. I would have thought that my <code>TM_RUBY</code> environment variable being set correctly to <code>/opt/local/bin/ruby</code> in my TextMate preferences would have given RubyAMP enough info to find my correct gem environment. Obviously not... and it took me quite a bit of digging around to learn what else I needed to do. For starters, here is more than you ever wanted to know about <a href="http://macromates.com/textmate/manual/shell_commands#search_path">how TextMate gets the $PATH information</a>. You can skip reading that and do these simple steps.
</p>

<ul>
  <li>Open /Developer/Applications/Utilities/Property List Editor.app</li>
  <li>Click on "New Root", now expand that node in the list view below.</li>
  <li>Click on "New Child", name it PATH</li>
  <li>The child row for PATH should be a String type</li>
  <li>Enter your PATH info here, should mimic your .bash_profile, without $PATH</li>
  <li>Save this file to your Desktop as environment.plist</li>
</ul>

<p>
  Now do this in the console, we need to move that file to an invisible <code>.MacOSX</code> folder in your home directory. In all likelyhood this folder does not exists, nor does the environment.plist file inside of it, if so, please do your own work to make sure that you do not overwrite existing information. Now:
</p>

<pre class="command">
$ mkdir ~/.MacOSX
$ mv ~/Desktop/environment.plist ~/.MacOSX
</pre>

<p>
  With the file in place, you now have to log out and back in. To be on the safe side, just reboot. Now your OSX apps, including TextMate and it's bundles will have the correct PATH information to get to your binaries. See below for an example of what my environment.plist file looks like. Note too that my full PATH info in that file is this string <code>/opt/local/bin:/opt/local/sbin:/opt/local/lib/mysql5/bin:/opt/local/apache2/bin</code>. Notice that path info after the apache2 path? This is where you should add the default path info for a Mac. Unlike the .bash_profile, you can not specify <code>$PATH</code> and have it expanded from within a plist file. So to be on the safe side, when I added this, I put the standard mac path info at the end of my own additions. That standard path info is <code>/usr/local/bin:/bin:/sbin:/usr/bin:/usr/sbin</code>
</p>

<div class="center">
  <img src="/assets/envplist.png" width="496" height="302" />
</div>


