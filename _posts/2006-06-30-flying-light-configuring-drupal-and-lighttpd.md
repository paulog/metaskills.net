--- 
layout: post
title: Flying Light - Configuring Drupal and LightTPD
disqus_id: /2006/06/30/flying-light-configuring-drupal-and-lighttpd/
categories: 
  - miscellaneous
---

<p>
  <img src="/assets/light_logo.png" alt="LightTPD Logo" width="249" height="239" class="floatr ml20" /> So the WebSvr mini here at my home-based NOC (named ActionMoniker.com) is now running LightTPD as the web server. The benefits are that I can now run my PHP-based Drupal blog in FastCGI mode while also allowing virtual hosting under the same server/IP for my RAILS projects. The end result has turned out quite well and I am actually loving the speed improvements and the <a href="http://lighttpd.net/documentation/">simple configuration</a> for LightTPD, which has a more natural feeling for me when it comes to configuring a web server.
</p>

<p>
  Below is a code snippet of my current configuration file for this Drupal host.  Please be aware, that this is not a full LightTPD configuration file which would likely have further restrictions on how your web server operates and secures itself. That said, this snippet incorporates the following configurations which are mostly necessaries for any Drupal blog using the Drupal supplied .htaccess file for Apache.
</p>

<ul>
  <li>A vhost default so that both the www and the root domain point to the same directory.</li>
  <li>URL access rules that deny access  to common Drupal install and app files.</li>
  <li>URL rewrite rules for my <a href="http://haveamint.com/">mint statistics,</a> multi site setup in a <code>/meta-theme-for-drupal</code> directory which, and the standard drupal rewrites for clean ULRs and a few system functions. It is important that these rewrites be in the correct order with the root drupal install being last.</li>
  <li>URL expires directives that help browser caching for all page assets that are core Drupal and found in the Meta Theme. </li>
  <li>A compress cache directory, server error log, and access log using the web server root variable set at the tops of the vhost config. I like that LightTPD can set variables.      </li>
  <li>Finally a LightTPD declaration for spawning FastCGI process for PHP.</li>
</ul>

```c
simple-vhost.server-root      = "/Library/WebServer/hosts/"
simple-vhost.default-host     = "default"

$HTTP["host"] =~ "^(metaskills.net)|(www.metaskills.net)$" {
  simple-vhost.default-host     = "metaskills.net"
  url.access-deny = ( 
    ".engine", ".inc", ".install", ".module", ".sh", ".sql", ".pgsql", 
    ".theme", ".tpl.php", ".xtmpl", ".code-style.pl", ".Repository", ".Root"
    )
 url.rewrite-once = (
   # hard-coded rewrites so the drupal catch-alls do not get them.
   "^/mint/$" => "/mint/index.php",
   "^/mint/\?(.*)" => "/mint/index.php?$1",
   # drupal rules for (meta-theme-for-durpal) multi-site, these MUST BE FIRST.
   "^/meta-theme-for-drupal/system/test/(.*)$"  => "/meta-theme-for-drupal/index.php?q=system/test/$1",
   "^/meta-theme-for-drupal/search/node/(.*)$"  => "/meta-theme-for-drupal/index.php?q=search/node/$1",
   "^/meta-theme-for-drupal/([^.?]*)\?(.*)$"    => "/meta-theme-for-drupal/index.php?q=$1&$2",
   "^/meta-theme-for-drupal/([^.?]*)$"          => "/meta-theme-for-drupal/index.php?q=$1",
   "^/meta-theme-for-drupal$"                   => "/meta-theme-for-drupal/index.php",
   # drupal rules for default site behavior.
   "^/system/test/(.*)$"        => "/index.php?q=system/test/$1",
   "^/search/node/(.*)$"        => "/index.php?q=search/node/$1",
   "^/([^.?]*)\?(.*)$"          => "/index.php?q=$1&$2",
   "^/([^.?]*)$"                => "/index.php?q=$1"
   )
  expire.url = ( 
    "/favicon.ico"              => "access 3 days", 
    "/files/"                   => "access 3 days", 
    "/misc/"                    => "access 3 days",
    "/themes/meta/css/"         => "access 3 days",
    "/themes/meta/images/"      => "access 3 days",
    "/themes/meta/js/"          => "access 3 days",
    "/themes/meta/meta-black/"  => "access 3 days",
    "/themes/meta/meta-grey/"   => "access 3 days",
    "/themes/meta/meta-paper/"  => "access 3 days",
    "/themes/meta/meta-pink/"   => "access 3 days" 
    )
  compress.cache-dir            = webserver-root + "tmp/cache_metaskills/"
  server.errorlog               = webserver-root + "logs/metaskills.error.log"
  accesslog.filename            = webserver-root + "logs/metaskills.access.log"
  fastcgi.server                = ( ".php" =>
                                    ( "localhost" =>
                                      ( 
                                        "socket" => "/tmp/php-fastcgi.socket",
                                        "bin-path" => "/opt/local/bin/php-fcgi",
                                        "max-procs" => 4,
                                        "bin-environment" => ( 
                                          "PHP_FCGI_CHILDREN" => "16",
                                          "PHP_FCGI_MAX_REQUESTS" => "10000"
                                        ),
                                        "bin-copy-environment" => (
                                          "PATH", "SHELL", "USER"
                                        ),
                                        "broken-scriptfilename" => "enable"
                                      )
                                    )
                                  )
}
```

<p>
  Obviously this configuration is dependant on php-fcgi being installed and If you followed along in my past <a href="/2006/05/29/my-own-soup-to-nuts-recipe-for-ruby-on-rails-on-os-x/">Soup to Nuts Recipe for Ruby on RAILS on OS X</a>, here is a few crib notes I kept on how to recompile a few things so that you can have your whole installation of LightTPD, Ruby, PHP, and MySQL all running from the Darwin Ports /opt directory.
</p>


<h2>Reinstall LightTPD with SSL</h2>

<p>If you do not plan on using SSL with Lighty, this step is optional.</p>

<pre class="command">
$ sudo port uninstall lighttpd
$ sudo port install lighttpd +ssl
</pre>


<h2>MySQL 5 Installation in Darwin Ports </h2>

<p>
  This part is definitely not for the week. If you are total noobie, stick with the <a href="http://dev.mysql.com/downloads/">compiled packages</a> offered by MySQL AB because doing this installation may break some of the tools you might be familiar with. Most can be fixed, but you will definitely loose your fancy start/stop button in the System Preferences.
</p>

<p>
  I installed MySQL 5 in this manner so that I could have PHP from Darwin Ports totally hooked into the MySQL installation by DP as well. I do not suggest you do this if you are new to this sort of stuff, again, these are just some quick notes I made. OK...
</p>

<p>
  First, add <code>/opt/local/lib/mysql5/bin</code> to the $PATH environment variable in your <code>.bash_profile</code>. When you do this you are linking to the REAL path for the MySQL binaries and not the Darwin Ports suffix5 sym links that it creates in<code>/opt/local/bin</code>
</p>

<p>
  Now issue these command. This will install mysql5 with Darwin Ports and the server startup scripts. The second command will load that startup script into your <code>launchd</code> list. and the third will install the default MySQL tables for permissions, etc.
</p>

<pre class="command">
$ sudo port install mysql5 +server
$ sudo launchctl load -w /Library/LaunchDaemons/org.darwinports.mysql5.plist
$ sudo -u mysql /opt/local/lib/mysql5/bin/mysql_install_db
</pre>

<p>
  So that you do not have to do the following steps, restart you computer which should let MySQL use the startup script you enabled in the step above. In this way you can bypass these first two commands below. The other two commands are some notes of mine on the usage of the wrapper LaunchDaemon installed by Darwin Ports for MySQL. For easy reading, check out the support pages for <a href="http://dev.mysql.com/doc/refman/5.0/en/unix-post-installation.html">UNIX Post-Installation Procedures</a> offered on the MySQL Reference Manual. Remember, I am pretty sure that you do not have to do these steps if you just restart your computer.
</p>

<pre class="command">
$ sudo -u mysql /opt/local/lib/mysql5/bin/mysqld_safe &
$ sudo /opt/local/lib/mysql5/bin/mysqladmin shutdown
$ sudo -u mysql /opt/local/etc/LaunchDaemons/org.darwinports.mysql5/mysql5.wrapper start
$ sudo -u mysql /opt/local/etc/LaunchDaemons/org.darwinports.mysql5/mysql5.wrapper stop
</pre>

<p>
  Here we are going to create the <code>my.cnf</code> file in one of the places this installation of MySQL likes to find it from the example files installed by MySQL. In this example I used the medium example file, but if you are expecting more traffic and have more RAM, use the my-huge.cnf file instead. Next we are creating a sym link to the my.cnf in the standard place the normal MySQL AB binaries install them on OS X, in the <code>etc</code> directory. In this way you can still use <a href="http://www.mysql.com/products/tools/administrator/">MySQL Administrator</a> buy be aware that I had to use 127.0.0.1 instead of localhost. Not sure why.
</p>

<pre class="command">
$ sudo -u mysql cp /opt/local/share/mysql5/mysql/my-medium.cnf /opt/local/var/db/mysql5/my.cnf
$ sudo ln -s /opt/local/var/db/mysql5/my.cnf /etc/my.cnf
</pre>

<p>
  Now to fix the MySQL gem that was installed the first time around. This install type uses the <code>mysql-config</code> as described in <a href="http://tmtm.org/en/mysql/ruby/">the fine manual</a>.
</p>

<pre class="command">
$ sudo gem uninstall mysql
$ sudo gem install mysql -- --with-mysql-config=/opt/local/lib/mysql5/bin/mysql_config
</pre>


<h2>Installing PHP5 using Darwin Ports MySQL </h2>

<p>
  Now you are ready to install PHP5 with its FastCGI support and Darwin Ports dependencies on the MySQL previously installed. The second command copies the php.ini installed by DP into the place the Darwin Ports PHP likes to find it.
</p>

<pre class="command">
$ sudo port install php5 +fastcgi +mysql5
$ sudo cp /opt/local/etc/php.ini-recommended /opt/local/etc/php.ini
</pre>

<p>
  Now add this line of code to the php.ini file somewhere at the end of the section called "Paths and Directories" that is about mid way down. See also this <a href="http://trac.lighttpd.net/trac/wiki/TutorialLighttpdAndPHP">web page</a> for more info on this.
</p>

```php
cgi.fix_pathinfo = 1
```



