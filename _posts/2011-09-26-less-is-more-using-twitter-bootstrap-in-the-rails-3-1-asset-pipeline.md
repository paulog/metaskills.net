---
layout: post
title: LESS Is More - Using Twitter's Bootstrap In The Rails 3.1 Asset Pipeline
categories: 
  - ruby-rails
  - javascript
---

<p>
  <span class="floatr ml20 mb20">
    <img src="/assets/less.png" alt="LESS - The Dynamic Stylesheet language" width="199" height="81" />
  </span>
  This weekend I decided to experiment with <a href="http://lesscss.org/">LESS CSS</a> by replacing the existing <a href="http://sass-lang.com/">Sass</a> and <a href="http://compass-style.org/">Compass</a> code that had been built thus far a small project. Why? Three basic reasons. First, I wanted to see how LESS stacked up. Second, I was intrigued by some of LESS' features, in particular <a href="http://lesscss.org/#-namespaces">their namespace support.</a> Lastly, I wanted to use <a href="http://twitter.github.com/bootstrap/">Twitter's Bootstrap</a> project as a baseline for my design. Since Rails 3.1 has been out for some time, I was expecting the move to LESS to go considerably smoother than my <a href="/2011/07/29/use-compass-sass-framework-files-with-the-rails-3.1.0.rc5-asset-pipeline/">pre-release attempts with Compass and Sass<a/>. I was wrong.
</p>

<p>
  So the tilt gem already supported LESS templates and that means that Sprockets and the asset pipeline rendering technically did too. But there was <a href="https://github.com/cowboyd/less.rb/issues/8">no official less-rails gem</a> that setup a standard configuration for other gems to hook into. This meant that libraries that distributed the Twitter Bootstrap assets were often hard to use or had inconsistent results. For example, it was not possible to load the Bootstrap's assets in such as way that let you build on top of the LESS mixins they had build. Nor were those assets namespaced in such a way that would allow you to have a <code>modal.less</code> asset as it would have conflicted with Bootstrap's version. So since I apparently have a bunch of free time and I hate building on top of a bad foundation, I set out to learn both <code>Rails::Railtie</code> and <code>Rails::Engine</code> to build the proper tools for LESS and Bootstrap in the Rails 3.1 asset pipeline.
</p>


<h2>New LESS Gems</h2>

<p>
  I introduce you to both the new <a href="http://github.com/metaskills/less-rails">less-rails</a> and <a href="http://github.com/metaskills/less-rails-bootstrap">less-rails-bootstrap</a> gems. Take a look at the source code if you ever wanted to learn how to implement a simple Rails::Railtie or Engine. But assuming you want to use Twitter's Bootstrap, here are a few examples to get you started. First just bundle up to less-rails-bootstrap, it will automatically pull in less-rails and less.rb via their dependencies. BTW, I am going to keep the semantic versioning of the less-rails-bootstrap gem in sync with the major and minor versions of the Bootstrap project.
</p>

~~~ruby
gem 'less-rails-bootstrap', '~> 1.3.0'
~~~

<p>
  From here, the easiest way to use Bootstrap is to require it in your <code>application.css</code> file. Doing so will compile the complete LESS libraries files for Bootstrap. Note how we namespace the files using a simple directory structure.
</p>

~~~css
/*
 *= require twitter/bootstrap
*/

#foo {
  /* Your styles... */
}
~~~

<p>
  Alternatively, in a file with the <code>.css.less</code> extension, you can import the entire Bootstrap LESS framework. This will allow you to use Bootstrap's variables and mixins in your CSS that follows. Remember, unlike other CSS frameworks, requiring or importing Bootstrap will include all the CSS for building a bootstrapped website. If you only want variables or mixins, you will have to import those discreet files, see next section.
</p>

~~~css
@import "twitter/bootstrap";

#foo {
  .border-radius(4px);
}
~~~

<p>
  Maybe all you want to use are the variables and mixins that come with Twitter Bootstrap. No problem, just import them individually from you own <code>.css.less</code> file. In this case only the <code>#foo</code> selector is output.
</p>

~~~css
@import "twitter/bootstrap/variables";
@import "twitter/bootstrap/mixins";

.myButton(@radius: 5px) {
  .border-radius(@radius);
}

#foo {
  .myButton(10px);
}
~~~

<p>
  Using the <a href="http://twitter.github.com/bootstrap/#javascript">Bootstrap JavaScript</a> files is just as easy. Again, you can include all them with a single directive from your <code>application.js</code> file. Optionally, you can require only the files you need like <code>require twitter/bootstrap/modal</code>.
</p>

~~~javascript
//= require twitter/bootstrap

$(document).ready(function(){
  //...
});
~~~

<p>
  The less-rails project has already started getting some good feedback. Soon we hope to implement all the features that you may have used in the sass-rails project, <a href="https://github.com/metaskills/less-rails/issues/1">like asset pipeline helpers</a>. One last thing, I wanted to say thanks to <a href="https://github.com/nmerouze">Nicolas Mérouze</a> for opening up the old less-rails gem space on rubygems.org for the new gem!
</p>

<h2>Resources</h2>

<ul>
  <li><a href="http://lesscss.org/">LESS - The Dynamic Stylesheet Language</a></li>
  <li><a href="http://github.com/metaskills/less-rails">LESS-Rails - Official Support For LESS In Rails</a></li>
  <li><a href="http://github.com/metaskills/less-rails-bootstrap">LESS-Rails-Bootstrap - CSS toolkit from Twitter For Rails 3.1 Asset Pipeline</a></li>
  <li><a href="http://github.com/cowboyd/less.rb">LESS.rb - LESS CSS Rendering From Ruby</a></li>
  <li><a href="http://sass-lang.com/">Sass - Syntactically Awesome Stylesheets</a></li>
  <li><a href="http://github.com/rails/sass-rails">Sass-Rails - Official Support For Sass In Rails</a></li>
  <li><a href="http://compass-style.org/">Compass - An Open-Source CSS Authoring Framework</a></li>
  <li><a href="http://twitter.github.com/bootstrap/">Bootstrap - A Toolkit From Twitter Designed To Kickstart Development</a></li>
</ul>

