--- 
layout: post
title: Using Autotest For Rails Plugin Development
disqus_id: /2008/09/19/using-autotest-for-rails-plugin-development/
categories: 
  - heuristics
  - ruby-rails
  - workflow
---


<p>
  I love <a href="http://www.zenspider.com/ZSS/Products/ZenTest/">autotest</a>. I have event posted before how to extend the idea of <a href="/2008/4/6/autotest-playlist-for-red-green-feedback">autotest sounds to a red/green playlist</a> but now that I am taking more time to extract some of my work to plugins, I really wanted autotest to come with me. The problem is that the default autotest mappings do not play with rails conventions, the biggest being that test files for a lib match the name of the lib with <code>_test.rb</code> at the END of the filename.
</p>

<p>
  Today I posted a new project to my Github account, called <a href="http://github.com/metaskills/autotest_railsplugin/tree/master">Autotest::Railsplugin</a>. This project should be a great start for fleshing out all the typical mappings, ignore, libs, etc. that autotest needs to know about when developing a rails plugin. I invite you to download it and give it a try and provide feedback on missing details.
</p>


<h2>To Install</h2>

<ul>
  <li><a href="http://github.com/metaskills/autotest_railsplugin/tree/master">Download</a></li>
  <li>Rename the directory to "autotest"</li>
  <li>Place inside of your pluginâ€™s root directory.</li>
  <li>From your plugin root, you can now use autotest.</li>
</ul>

<pre class="command">
$ autotest
loading autotest/railsplugin
</pre>

<p>
  By default, autotest will scan your present working directory for an autotest folder that has a discover.rb in it. So it should find and display the <code>loading autotest/railsplugin</code> which means it found the plugin class just fine. If this is not the case make sure you have the directory named correctly. If you have a conflicting autotest directory for some reason you can always force the plugin class, again assuming you have it in the right, place using:
</p>

<pre class="command">
$ autotest -railsplugin
</pre>


<h2>Resources</h2>

<ul>
  <li><a href="http://github.com/metaskills/autotest_railsplugin/tree/master">The New Autotest::Railsplugin on Github</a></li>
  <li><a href="http://www.zenspider.com/ZSS/Products/ZenTest/">The Official ZenTest/AutoTest Site</a></li>
  <li><a href="/2008/4/6/autotest-playlist-for-red-green-feedback">Autotest Playlist For Red/Green Feedback</a></li>
  <li><a href="http://www.fozworks.com/2007/7/28/autotest-sound-effects/">Autotest Sound Idea from Jeremy Seitz @ FozWorks</a></li>
</ul>


