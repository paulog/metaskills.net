--- 
layout: post
title: Installing REE With The Snow Leopard SQL Server Stack
disqus_id: /2009/10/27/installing-ree-with-the-snow-leopard-sql-server-stack/
categories: 
  - database
  - heuristics
  - projects
  - ruby-rails
---


<p>
  Today I noticed that Ruby Enterprise Edition 2009.10 was released and I have really been wanting to see if I could get the <a href="http://github.com/rails-sqlserver">SQL Server adapter</a> tested and running under it. I am really curious how the speed improvements might look and will share my results below. This article assumes that you read my previous guide titled <a href="/2009/09/05/the-ultimate-os-x-snow-leopard-stack-for-rails-development-x86_64-macports-ruby-1-8-1-9-sql-server-more/">The Ultimate OS X Snow Leopard Stack For Rails Development - x86_64, MacPorts, Ruby 1.8/1.9, SQL Server, SQLite3, MySQL & More</a> as I will be building on top of it and referencing certain steps. So let's get down to business.
</p>



<h2>Installing REE</h2>

<p>
  The guys at Phusion have done a rock solid job. My previous attempts to install REE were a bomb, but it worked perfectly this time around. Simply <a href="http://www.rubyenterpriseedition.com/download.html">follow their directions</a> and all will be OK. A snippet of my output is below. I liked how they picked a new default install directory of <code>/opt/ruby-enterprise-1.8.7-2009.10</code>. Wicked cool! By the way, the only issue I had during install was when they were trying to install the pg gem for me. I ignored that error for now.
</p>

<pre class="command">
$ sudo ./ruby-enterprise-1.8.7-2009.10/installer

Where would you like to install Ruby Enterprise Edition to?
(All Ruby Enterprise Edition files will be put inside that directory.)
[/opt/ruby-enterprise-1.8.7-2009.10] :
</pre>



<h2>Configuring For REE</h2>

<p>
  This is actually really easy. In my previous article, I talked about installing MacPorts and configure your profile. All I did this time around was add the REE path extensions after my MacPort extensions in my ZSH kit. So my path file now looks this below. If I ever want to just go back to my normal 1.8/1.9 toggle in <code>/opt/local/bin</code>, I just comment out the REE path extensions, source my profile and I'm back to my MacPort basics.
</p>

~~~bash
# MacPorts
path=(/opt/local/bin /opt/local/sbin /opt/local/apache2/bin ~/.zshkit/bin $path)
manpath=(/opt/local/share/man $manpath)
infopath=(/opt/local/share/info $infopath)

# REE
path=(/opt/ruby-enterprise-1.8.7-2009.10/bin $path)
manpath=(/opt/ruby-enterprise-1.8.7-2009.10/share/man $manpath)
infopath=(/opt/ruby-enterprise-1.8.7-2009.10/share/info $infopath)
~~~

<p>
  In the last guide, I mentioned how to switch between ruby 1.8/1.9 installed by MacPorts using a simple ZSH function. You can find this in my article title <a href="/2009/01/20/multiruby-the-macports-way-testing-your-rails-apps-with-ruby-1-9/">MultiRuby The MacPorts Way. Testing Your Rails Apps With Ruby 1.9</a>. I have updated this to include a condition that will noop the function if you are running REE. So to recap. I use my <code>chruby</code> function to switch between 1.8/1.9 installed by MacPorts and I use my profile extension to use REE. So what that profile change done... reload and/or open a new terminal window and test it out.
</p>

<pre class="command">
$ which ruby
/opt/ruby-enterprise-1.8.7-2009.10/bin/ruby
$ which gem
/opt/ruby-enterprise-1.8.7-2009.10/bin/gem
</pre>



<h2>Using REE With SQL Server</h2>

<p>
  Let's get the easy stuff out of the way and install the needed gems for the adapter to work. Again, always <a href="http://github.com/rails-sqlserver">follow the TODO on the adapter page</a> for the latest blessed versions of DBI and low level connectivity stuff. But just to recap the current needs.
</p>

<pre class="command">
$ sudo gem install dbi -v 0.4.1
$ sudo gem install dbd-odbc -v 0.2.4
$ sudo gem install activerecord-sqlserver-adapter
</pre>

<p>
  Now the hard part, but I have an easy solution. You can do this the hard way and recompile the UTF8 version of ruby odbc by hand or you can type one command. I tried compiling it by hand and found that I had a few bugs, even after following <a href="http://digitaljacob.riff.dk/2009/10/26/using-ruby-odbc-with-unixodbc-on-snow-leopard/">Jacob Riff's notes</a> when he had a similar issue when trying to hook ruby ODBC to use unixODBC in his <code>/usr/local</code> . My solution was much simpler. I just coped my ruby 1.8.7 ruby odbc to the REE's vendor ruby directory. Hence:
</p>

<pre class="command">
$ sudo cp /opt/local/lib/ruby/vendor_ruby/1.8/i686-darwin10/odbc.bundle /opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/vendor_ruby/1.8/i686-darwin10.0.0
</pre>

<p>
  Here are my notes below on where I failed on installing ruby odbc by hand under REE. Feedback welcome, but good to know that the above simple copy worked just fine.
</p>

<pre class="command">
# Installing Ruby ODBC By Hand Notes

$ ruby -Cutf8 extconf.rb
$ make -C utf8
$ sudo make -C utf8 install
  /usr/bin/install -c -m 0755 odbc_utf8.bundle /opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/site_ruby/1.8/i686-darwin10.0.0
$ sudo mv /opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/site_ruby/1.8/i686-darwin10.0.0/odbc_utf8.bundle /opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/site_ruby/1.8/i686-darwin10.0.0/odbc.bundle 

Loading development environment (Rails 2.3.4)
dlsym(0x102c6e330, Init_odbc): symbol not found - /opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/site_ruby/1.8/i686-darwin10.0.0/odbc.bundle
/opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/site_ruby/1.8/i686-darwin10.0.0/odbc.bundle
/opt/ruby-enterprise-1.8.7-2009.10/lib/ruby/site_ruby/1.8/rubygems/custom_require.rb:31:in `require'
</pre>




<h2>Benchmarking REE vs MacPorts</h2>

<p>
  Here is the good news, the above hacks and REE work! I have even tested the adapter and all is looking good from my end!!! Now how about those benchmarks? Here are the speed differences for running the full adapter test suite under 1.8.7 installed in MacPorts and the latest REE. Note, this is really non-scientific benchmarks, but I think they fun facts.
</p>

<pre class="text">
Ruby 1.8.7 MacPorts (patchlevel 174)
Finished in 158.574243 seconds.
2190 tests, 7395 assertions, 0 failures, 0 errors

REE 1.8.7 (2009.10)
Finished in 146.834762 seconds.
2190 tests, 7636 assertions, 0 failures, 0 errors
</pre>



<h2>Installing Passenger</h2>

<p>
  Since passenger was installed as part of the REE installation, all I had to do was run the apache2 installer again and paste in the new code into my httpd.conf file.
</p>

<pre class="command">
$ which passenger-install-apache2-module
/opt/ruby-enterprise-1.8.7-2009.10/bin/passenger-install-apache2-module
$ sudo passenger-install-apache2-module
</pre>



