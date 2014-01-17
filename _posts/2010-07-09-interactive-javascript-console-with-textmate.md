--- 
layout: post
title: Interactive JavaScript Console With TextMate
disqus_id: /2010/07/09/interactive-javascript-console-with-textmate/
categories: 
  - apple-os
  - heuristics
  - textmate
  - javascript
---


<p>
  Last week I started reading <a href="http://oreilly.com/catalog/9780596517748"><em>JavaScript: The Good Parts</em></a> by Douglas Crockford. It was on my list of long overdue things to do. While reading it, I wanted to be able to kick some simple JavaScript examples around. As rubyist we have it good, <code>irb</code> let's us fire up an interactive console anytime we want. But with JavaScript, options are limited. Sure you could install Johnson/EnvJS, Rhino or some other JavaScript engine. Maybe even load up firebug or the web inspector. But who wants to load a browser to play with JS?
</p>

<p>
  Luckily if your are on a Mac, you do not have to worry about any of that. You already have a JavaScript engine installed. Where? Thanks to Safari's <a href="http://webkit.org/blog/214/introducing-squirrelfish-extreme/">SquirrelFish Extreme (SFX)</a>, it is right in your system's library at this full path <code>/System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Resources/jsc</code>. Instead of adding that to my path, I created a symlink to in my opt's bin directory (seeing how I have MacPorts). For instance.
</p>

<pre class="command">
sudo ln -s /System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Resources/jsc /opt/local/bin
</pre>

<p>
  Assuming you just did that command in your own bin path of choice, you can now just type <code>jsc</code> and start typing JavaScript just as you easily as if you were in an irb prompt. Fun, but we can do better.
</p>


<h2>Creating A TextMate Bundle To Run JavaScript</h2>

<p>
  So I was inspired by this article that explained how easy it is to make a <a href="http://www.phpied.com/jslint-on-mac-textmate/">TextMate bundle to run JavaScript thru JSLint</a> and decided to make one to run the same JavaScript in an the interactive console for SFX. I wont go thru the details of how to add a simple TextMate bundle command, but the following picture and code sample shows the <code>Run JSConsole</code> command I made. The command resides in my own bundle namespace.
</p>

<div class="center">
  <img src="/assets/jsc_tmbundle.gif" alt="TextMate Bundle Window" width="520" class="shadow" />
</div>

~~~ruby
#!/usr/bin/env ruby

require ENV['TM_SUPPORT_PATH'] + '/lib/escape.rb'

def terminal_script_filepath
  %|tell application "Terminal"
      activate
      do script "jsc -i #{e_as(e_sh(ENV['TM_FILEPATH']))}"
    end tell|
end

open("|osascript", "w") { |io| io << terminal_script_filepath }
~~~

<p>
  With that simple bundle command done, you can now use the <code>Command-R</code> keyboard shortcut to load the windows JavaScript file into a newly opened terminal window running in the SFX console. Pictures below.
</p>

<div class="center">
  <img src="/assets/jsc_textmate.gif" alt="TextMate Windows With JavaScript" width="508" class="shadow" />
</div>

<div class="center">
  <img src="/assets/jsc_terminal.gif" alt="Mac OS Terminal App Lauanched From TextMate JavaScript" width="520" class="shadow" />
</div>


<h2>Great, Now What?</h2>

<p>
  My simple script just loads the whole file, it does not take into account TM's selected text. Nor does it do anything with selected files. I would love to see someone write a TextMate bundle that could preload up jQuery/Prototype or a set of files. The ultimate would be a bundle command that executed and updated markers like the Ruby bundle can do. Sadly, I do not have that much time on my hands :)
</p>



