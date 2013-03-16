--- 
layout: post
title: A Review of Digg.com Traffic and CDNs
disqus_id: /2006/02/16/a-review-of-digg-com-traffic-and-cdns/
categories: 
  - heuristics
  - miscellaneous
---


<p>Thanks to all those that read my recent post about <a href="/2006/2/5/mini-network-with-a-big-xserve-style">networking 3 Mac Mini's</a> Hopefully it can help you create a network that is as close as possible to the  administrator's second home, the NOC. As an aside form the posts I had planned, I did want to share some of the statistics that the <a href="http://www.digg.com/">digg.com</a> exposure generated and some &quot;simple&quot; helpful tips to those interested in surviving high traffic/bandwidth peaks. I'll cover more &quot;technical&quot; ways of setting up your server(s) to handle this too, but that is for <a href="/2006/2/19/how-to-control-browser-caching-with-apache-2">my next post.</a></p>

<div class="center">
	<span class="photofancy"><img src="/assets/digg-homepage.gif" alt="Mini network story on the home page of digg.com" width="541" height="231" /></span>
</div>
		
		
<h2>Bandwidth Chart (7 Days) </h2>
		
<p>The story made it to the front page of digg about one week ago on Monday, February 6th. It was posted to Digg about 36 hours earlier on Saturday evening and within  1 hour of making it to the home page, using approximately 60 diggs to get it there, the bandwidth capacity here at ActionMoniker.com really took a dive. Below is a 7 day chart that goes from Sunday, February 5th to Saturday, February 11th.</p>

<div class="center">
  <span class="photofancy"><img src="/assets/network-traffic.gif" alt="7 Day Bandwidth Chart Showing Digg.com Traffic Usage" width="384" height="263" /></span>
</div>
		
<p>The first big spike in the chart was the initial onslaught of traffic on Monday afternoon from the digg.com homepage. That small gap you see there before the next big spike is where the WebSvr Mini had to be rebooted. In fact, I even upgraded the RAM to one gigabyte. You can see another spike  towards the end that correlates to the secondary traffic generated from the <a href="http://en.wikipedia.org/wiki/Blogosphere">blogosphere</a>, basically when other bloggers and news portals got around to posting the information about my article and then linking back to my site. Neat statistics, but let's take a look at why that WebSvr Mini crapped out.</p>
		
		
<h2>We All Need Cache</h2>
		
<p>Apache seemed to do well with the quantity of requests coming at it from Digg. That said, my post did have one HUGE draw back, the 10 large images on the article's page that totaled  124 Kilobytes in size. My assumption is that when the server started taking requests in mass   my 2.6 Megabit connection was not fast enough to serve those images in a timely manner. This caused a cascading effect on Apache by  keeping memory intensive threads open for longer and longer until the server could not take it any more.</p>
		
<p>This is where a good cache comes in handy. Not  browser or proxy caches, those are for <a href="/2006/2/19/how-to-control-browser-caching-with-apache-2">my next post</a> too, but rather  what are called gateway caches which are more commonly called Content Delivery Networks (CDNs). These are distributed caching servers throughout the internet that typically help deliver content to a client by geotargeting a visitors IP address and then finding a server close to that IP with the requested content. The best example of commercial CDN is <a href="http://www.akamai.com/">Akamai</a>, I think Apple bought or invested in these guys about 3 years ago, but I digress. The point is that CDNs can help distribute the load off of your local server by copying your content be it a whole web page or just certain media types and serving them up to your visitor's browser.</p>
		
<p><img src="/assets/map-world-gen-1.gif" alt="The Coral cache network map." width="248" height="107" class="floatr ml20" /> The most popular CDN that is freely used by many digg.com readers is a company called <a href="http://www.coralcdn.org/">Coral</a>. Many times a Digg or Slashdot article will take down an unsuspecting server (know as the slashdot effect)   forcing prospective visitors to view your content from a cached version on a sever very far away from your own. Often readers on both Slashdot and Digg try to &quot;coralize&quot; your article by linking to it from a comment thread using the Coral network as a cache. This is easily done by  adding <code>.nyud.net:8090</code> to the end of any hostname in a URL. For instance <code>http://www.metaskills.net/article.php</code> would become <code>http://www.metaskills.net.nyud.net:8090/article.php</code> but this is really only helpful to prospective readers if the coralized link was used as the initial link on Digg or Slashdot.</p>
		  
<aside class="flash_info">
  UPDATE: As of February 17th, Coral now uses both ports <strong>:8090</strong> and <strong>:8080</strong>
</aside>
		  
<p>In my case it was not and I did not like the idea of users having to read my article on somebody else's server nor relying on them finding a coralized link to my site buried in a comment thread. If that had happened, I would have lost traffic statistics, user comments, RSS subscriptions, lateral page visitation, and many  visitors altogether. I had to get the server running again and cope with my limited bandwidth. Wow, I just said 2.6 megabits was limited :) </p>
		

<h2>Just Cache Images or Large Media </h2>
		
<p>When I rebooted the WebSvr Mini, I immediately changed the the <code>&lt;img src=&quot;&quot;&gt;</code> tags of the 10 images in the content area that had the largest file sizes to appear from the Coral network. This was done by changing the image source tags from <code>&lt;img src=&quot;/images/mac-mini-network.jpg&quot;&gt;</code>  to a fully qualified domain source using the coral server which ended up looking something like this <code>&lt;img src=&quot;http://www.metaskills.net.nyud.net:8090/images/mac-mini-network.jpg&quot;&gt;</code> thus ensuring any future visits to my article received those images from  1 of any of the the <a href="http://coralcdn.org/maps/">260 Coral network servers</a>. In this way the largest aspect of my page was completely offloaded leaving my local Apache to handle the page source and other media request in a timely fashion. Best of all, this was completely transparent to the end user and no features were lost in their browsing experience. </p>

<p>I highly suggest that if you know ahead of time that an article of yours will be Slashdoted or Dugg, that you coralize as many of the images or large media files on your web server as possible. Your options on what that media content type is may vary, but choosing the ones with the largest file size first is usually the best idea. It can save your site from going down or to a creeping crawl. </p>


<h2>Do You Digg It ?</h2>

<p><img src="/assets/digg-hat.png" alt="Digg.com Hat" width="125" height="133" class="floatl mr20" />Well I do and to date we have had 34,000 reads to the Mac Mini Network article. With a success like that it was time to sport some gear. I had already changed my default home page from my <a href="http://www.google.com/ig">customized Google</a> with Slashdot articles to digg.com and now all I needed was a hat to tell the world. For the past 5 months I have been sporting a /. hat and matching retro LED digital watch from ThinkGeek.com and it was time for a change. I thought for sure that there would be a vendor out there selling &quot;I got Dugg&quot; or &quot;Digg It&quot; or even a logo embroidered hat, but I could not find one at all! I would have to say that this company has sure missed out on some sweet merchandising opportunities. So I will have to stay with my /. hat. </p>

<p>Above is a comp I put together at CafePress.com. I even mailed it to Digg.com and asked what they thought. They kindly asked that I refrain from using their mark on resale items until their lawyers finished helping them with their copyrights and licensing agreements. Maybe I will go back to /. </p>
		

