--- 
layout: post
title: MultiRuby The MacPorts Way. Testing Your Rails Apps With Ruby 1.9
disqus_id: /2009/01/20/multiruby-the-macports-way-testing-your-rails-apps-with-ruby-1-9/
categories: 
  - apple-os
  - heuristics
  - ruby-rails
  - workflow
---

<aside class="flash_warn">
  UPDATE: In a recent article, I <a href="/2009/10/27/installing-ree-with-the-snow-leopard-sql-server-stack/">covered how to install REE</a> and have hence NOOP'ed this function to ignore the symlinking when REE is installed.
</aside>

<p>
  Ruby 1.9.1, the stable release, is just around the corner and if your like me, maybe you want to start playing around with it and perhaps test a few projects using 1.9 with edge rails 2.3. If so, and your on a Mac, then perhaps this installation method might appeal to you. I'll break this article up in two parts, the first will be on installing multiple versions of ruby and how to switch between them while the other will be some things I noticed when testing ruby 1.9 with edge rails on my favorite pet project <a href="http://github.com/metaskills/homemarks/tree/master">HomeMarks</a>.
</p>


<h2>Installing Ruby 1.9 & Change Ruby Function</h2>

<p>
  I love MacPorts and recommend using it even though OS X comes with ruby pre-installed. Doing so simply keeps things clean and separated in the <code>/opt</code> directory vs mixing things in /usr where they might be clobbered by Apple during a system update. So no to dependency hell! This article assumes you already have ruby 1.8 installed via MacPorts, if not change some of the paths in the provided code below. Now... thanks to MacPorts it is easier than ever to get ruby 1.9 preview release 2 installed in it's own separate space.
</p>

<pre class="command">
$ sudo port install ruby19
</pre>

<p>
  MacPort has just added these ruby 1.9 binaries to your PATH in <code>/opt/local/bin</code>. Nice and clean egh? From here you can start "playing" with 1.9 by just executing either <code>ruby1.9</code> or <code>irb1.9</code>, but that gets boring quick.
</p>

<ul>
  <li>erb1.9</li>
  <li>gem1.9</li>
  <li>irb1.9</li>
  <li>rake1.9</li>
  <li>rdoc1.9</li>
  <li>ri1.9</li>
  <li>ruby1.9</li>
  <li>testrb1.9</li>
</ul>

<p>
  Let's hack up your setup so that you can quickly switch your whole ruby install back and forth transparently. The first step is to move the corresponding ruby 1.8 binaries to the same naming convention as their 1.9 counterparts. We will then use symlinks to change which one gets executed. Step one:
</p>

<pre class="command">
$ sudo mv /opt/local/bin/erb /opt/local/bin/erb1.8
$ sudo mv /opt/local/bin/gem /opt/local/bin/gem1.8
$ sudo mv /opt/local/bin/irb /opt/local/bin/irb1.8
$ sudo mv /opt/local/bin/rake /opt/local/bin/rake1.8
$ sudo mv /opt/local/bin/rdoc /opt/local/bin/rdoc1.8
$ sudo mv /opt/local/bin/ri /opt/local/bin/ri1.8
$ sudo mv /opt/local/bin/ruby /opt/local/bin/ruby1.8
$ sudo mv /opt/local/bin/testrb /opt/local/bin/testrb1.8
</pre>

<p>
  So now that we have two versions of ruby installed, each in their own path suffix. So to switch back and forth without using these suffixes, we will need to create a symlink for each binary that points to the version we want to use. The users of the local ruby group <a href="http://757rb.org/">757rb</a> came up with this ZSH function below. Place it into your <a href="http://github.com/metaskills/zshkit/tree/master">.zshkit</a> or dotfile of choice and it will create/change symlinks for your ruby, irb, rake, etc.. that each point to the version you want to use. Calling it multiple times will simply toggle your ruby versions.
</p>

~~~bash
chruby () {
  ree=`ruby -e "puts RUBY_DESCRIPTION.include?('Ruby Enterprise Edition')"`
  if [ $ree = "true" ]; then 
    echo "NOOP: Currently using REE in your path."
  else
    v=`ruby -e "puts RUBY_VERSION.split('.')[0,2].join('.')"`
    if [ $v = "1.9" ]; then 
      cv="1.8"
    else
      cv="1.9"
    fi
    rubyexes=(erb gem irb rake rdoc ri ruby testrb autotest unit_diff rails)
    for i in $rubyexes; do
      sudo unlink "/opt/local/bin/${i}"
      sudo ln -s "/opt/local/bin/${i}${cv}" "/opt/local/bin/${i}"
    done
    echo "Now Running: "`ruby -v`
  fi
}
~~~

<pre class="command">
$ chruby
Now Running: ruby 1.8.7 (2008-08-11 patchlevel 72) [i686-darwin9]
$ chruby
Now Running: ruby 1.9.1 (2008-12-01 revision 20438) [i386-darwin9]
</pre>

<p>
  This works amazingly well at first. The one big gotcha is that you have to remember you have two gems directories. Any gem you installed under 1.8 will have to be installed again under 1.9. There are ways around this with some clever symlinking, but I'd recommend against it. Note too, this also applies to any site/vendored libs or bundles, for instance ruby odbc.
</p>


<h3>Things To Watch Out For</h3>

<p>
  If you install a gem under a certain version of ruby that places an executable in say /opt/local/bin then you are going to want to watch out for a bad she bang. For instance if you install the ZenTest gem while running under ruby1.9 suffix, then the she bang for <code>/opt/local/bin/autotest</code> would be incorrectly set to this <code>#!/opt/local/bin/ruby1.9 -ws</code>. The simple fix is to just change it to the true sym link.
</p>


<h2>Exploring Rails 2.3 & Ruby 1.9</h2>

<p>
  So far my HomeMarks project does not have any major issues with ruby 1.9. I'm not surprised, it's very very slim ruby code due to the <a href="/2008/05/24/the-ajax-head-design-pattern/">AJAX head design pattern</a>. I did however find a few new things about the changes an app will need when moving from 2.2 to 2.3.
</p>

<ul>
  <li>Your test/test_helper.rb file will need to change to from <code>class Test::Unit::TestCase</code> to <code>class ActiveSupport::TestCase</code>. I'm sure this change is related to future abstraction to choose Minitunit vs Test::Unit.</li>
  <li>They have finally changed <code>app/controllers/application.rb</code> to <code>app/controllers/application_controller.rb</code>. Finally their hacks on that bad name are now gone. I found that I had to do this change manually on my own.</li>
  <li>Disabling sessions for a single controller has been deprecated. Sessions are now lazy loaded. So if you don't access them, consider them off. You can still modify the session cookie options with request.session_options.</li>
</ul>

<p>
  Past that, about the only interesting thing I found in Ruby 1.9 was that Test::Unit is gone and is actually loading Miniunit. So things in rails that were requiring test/unit/error are going to fail.
</p>



