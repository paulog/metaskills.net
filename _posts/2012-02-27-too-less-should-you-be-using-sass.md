---
layout: post
title: Too LESS? Should You Be Using Sass?
categories: 
  - 
---

<aside class="flash_warn">
  Previously I had updated this article to say that this <a href="http://github.com/cloudhead/less.js/commit/93b23d2c24936d5bd829ba1f725ef442e9475747">commit</a> looked like it gives you real variable properties in LESS. I was wrong! So even in LESS v1.3 you are still screwed for doing metaprogramming and working with a real CSS preprocessor. That may change for LESS v1.4 and if you want to help make that happen. I suggst you put your weight behind <a href="https://github.com/cloudhead/less.js/pull/698">this github pull request</a>.
</aside>


<p>
  First, a little bit of background. A while back ago I took a great deal of my personal time to try out Twitter's Bootstrap project for a Rails application of mine. This meant that I was willing to throw away some of the work I had done in Sass and rewrite it using LESS. No problem I thought, at first glance LESS and Sass look almost identical in functionality. So after some weeks of improving the toolchain for LESS with Ruby and then Rails, I set out to do just that &ndash; and what a miserable failure it was. My goal is to share my experiences to anyone considering using LESS and why it might be wrong for your Rails project.
</p>


<h2>Why Should My Opinion Matter?</h2>

<p>
  For staters, I am the author of <a href="http://github.com/metaskills/less-rails">less-rails</a>, the gem that allows you to effectively use LESS in the Rails asset pipeline. This gem made other gems of mine like <a href="http://github.com/metaskills/less-rails-bootstrap">less-rails-bootstrap</a> possible. So now full LESS frameworks such as Twitter's Bootstrap could be leverage natively within the Rails ecosystem. Both of these gems greatly reduced the barrier to using LESS for the average Rails developer and I wrote them for one simple reason. I wanted to make sure LESS was as close to par with Sass in the Rails asset pipeline as possible. So before I rewrote one single line of Sass to LESS, it was going to have a fair shake. This was no small feat! Especially considering that LESS is the only JavaScript based CSS language to be extended with Rails' asset helpers like <code>asset-data-url()</code>. Meaning you can hook into assets from any other place in the pipeline directly from LESS, just as you would with Sass.
</p>


<p>
  Other qualifications? Well I fancy myself a CSS nut and put my skills slightly above average. Most know me as an ActiveRecord or database nerd. But I am just as comfortable doing client side JavaScript and design implementation, especially using HTML5 and CSS3. Very few people know of my <a href="/2011/09/11/revisiting-my-design-past/">design background</a> nor that I have been writing proper presentation layer CSS for over {{ site.time | date: "%Y" | minus:2004 }} years. Here in the past few years, I have learned to leverage both Sass and the Compass framework like a pro, <a href="/2010/12/27/let-it-go-moving-from-mephisto-to-jekyll/">even for this blog</a>. Here lately, in the project I intended to move to LESS, my Sass usage has become quite impressive. I have extended Sass' color objects to allow <a href="http://gist.github.com/1932866">HSV from RGB</a> conversions for presentation code parity with similar iOS CoreGraphics drawing code in my native iPhone applications. I have even learned to <a href="http://gist.github.com/1932882">write Sass mixins</a> using custom functions and tricks I picked up while reading the Compass source. All this placed the bar pretty high for me on how LESS could take the place of Sass in my little experiment.
</p>


<h2>What Went Wrong?</h2>

<p>
  So my bar was pretty high. I set out to not only rewrite my current Sass to LESS, but I fully expected to duplicate much of the things I learned to love about the Compass framework in LESS as well. And like a good open-source citizen, I even prepared a new project that I dubed <a href="http://github.com/metaskills/protractor">protractor</a> on github to share all my work. So what went wrong? Plenty! Every day, mixin by mixin, line by line, I felt MORE pain. LESS was fighting me and it was winning. 
</p>

<p>
  <span class="photofancy floatr ml20"><img src="/assets/less_variable_property.png" alt="Does LESS have property interpolation?" width="400" height="199" /></span>
  Some of the bugs I encountered were fixed, like parsing errors for CSS3 keyframes. Others had horrible workarounds that used more code while at the same time lowering legibility. Finally, I came across the one issue on the hundreds of those that exists on LESS' github page that stopped me cold. It was titled <a href="http://github.com/cloudhead/less.js/issues/36">variable property</a> and after learning about it, I decided to stop using LESS for my Rails projects. What are variable properties anyway. Let me show you a common pattern both I and Compass use in a basic contrived example. You may also want to read my comments in that github issue link too. Imagine I have some global colors that I want to mixin to other classes based on a dynamic property. So given this SCSS below:
</p>

```css
$myWhite: rgb(240,240,240);
$myGray:  rgb(140,140,140);
$myBlack: rgb(30,30,30);

@mixin myColorClasses($property) {
  &.white   { #{$property}: $myWhite; }
  &.gray    { #{$property}: $myGray; }
  &.black   { #{$property}: $myBlack; }
}

.box {
  @include myColorClasses(background-color);
}
```

<p>
  It would output something like the following. Notice how I use the variable string <code>background-color</code> and generate dynamic properties?
</p>

```css
.box.white { background-color: #f0f0f0; }
.box.gray  { background-color: #8c8c8c; }
.box.black { background-color: #1e1e1e; }
```

<p>
  Astute observers may point out that you can achieve the same with sub classes and be more efficient in the generated CSS too. You would be correct. I did say this was a contrived example. The point is that LESS is not a CSS preprocessor and without the ability to dynamically define property values you will be limited in the ways your LESS based CSS framework can compete with frameworks like Compass. Simply put, LESS will never stack up to Sass.
</p>


<h2>In All Fairness</h2>

<p>
  I have mad respect for Alexis Sellier (<a href="https://twitter.com/#!/cloudhead">@cloudhead</a>). By far his skills and open source contributions well outrank my own. As software developers we need to recognize that there are no absolutes and as such, there is no one tool that is right. You have to weight a ton of other factors and choose the tools that are right for you. I firmly believe that as Ruby and or Rails developers that you should choose to learn Sass and Compass vs. LESS and I am basing my opinion heavily on three criteria.
</p>

<p>
  First is that from a Ruby perspective, Sass will always be more easy to extend and hook into. Second, that LESS lacks a critical feature that I require to author CSS. Lastly, that LESS is not a CSS preprocessors by design and I believe they are the way forward to a <a href="https://twitter.com/#!/chriseppstein/status/171697822012416000">better future via the CSSWG</a>. This is what Sass does very very well. I want to treat CSS as a language and use features like loops, lists and custom functions.
</p>

<p>
   Your mileage may vary and please make your own decisions. I would be more than happy to followup with any questions on this topic too, so please ask away in the comments below. Lastly, here are a few other articles that talk about LESS and Sass.
</p>

<ul>
  <li><a href="http://nittygrittyjs.com/blog/why-less-is-a-pain-in-the-sass/">Why LESS Is a Pain in the Sass</a></li>
  <li><a href="http://coding.smashingmagazine.com/2011/09/09/an-introduction-to-less-and-comparison-to-sass/">An Introduction To LESS, And Comparison To Sass</a></li>
</ul>




