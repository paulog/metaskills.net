--- 
layout: post
title: Autotest Playlist For Red/Green Feedback
disqus_id: /2008/04/06/autotest-playlist-for-red-green-feedback/
categories: 
  - heuristics
  - ruby-rails
---


<p>
  Here is how to get a playlist of sounds that will be hooked to both your autotest :red and :green callbacks. Basically this gives you a folder of sounds that are played one after another, in a loop, as your tests pass or fail. See this move below for a quick example.
</p>

<div class="center">
  <object width="640" height="505"><param name="movie" value="http://www.youtube.com/v/HV_drKDclFA?fs=1&amp;hl=en_US&amp;color1=0x3a3a3a&amp;color2=0x999999"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="http://www.youtube.com/v/HV_drKDclFA?fs=1&amp;hl=en_US&amp;color1=0x3a3a3a&amp;color2=0x999999" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="640" height="505"></embed></object>
</div>


<h2>Step 1: Install QTPlay from MacPorts</h2>

<pre class="command">
$ sudo port install qtplay
</pre>


<h2>Step 2: Download My Autotest Playlist Files</h2>

<pre class="command">
$ cd ~
$ curl -O http://cdn.metaskills.net/downloads/autotest_playlist.tar
$ tar -xf autotest_playlist.tar
$ mv autotest_playlist .autotest_playlist && rm autotest_playlist.tar
</pre>


<h2>Step 3: Modify Your .autotest File</h2>

<p>
  User your favorite editor to open your <code>~/.autotest</code> file. And add this code below to it. Now you should be setup to have your own playlist of sounds that play using the <code>qtplay</code> binary installed above. If you want to play around with the sounds, then <code>open ~/.autotest_playlist/sounds</code> and start swapping out sounds. By the way, I made the :initialize and :quit sounds using the new Alex voice in Leopard. I hijacked some text to speech and then ran an audio tool called WavePad to speed change, flange, and chorus the voice a little bit.
</p>

```ruby
require '~/.autotest_playlist/playlist'
```


<h2>Credits:</h2>

<p>The idea for this came from <a href="http://www.fozworks.com/2007/7/28/autotest-sound-effects/">FoxWorks</a>. Shouts out "Thank You".</p>








