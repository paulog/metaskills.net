--- 
layout: post
title: A MacPort/RubyODBC Update
disqus_id: /2010/07/19/a-macport-rubyodbc-update/
categories: 
  - apple-os
  - database
  - heuristics
  - ruby-rails
---


<aside class="flash_warn">
  <a href="https://github.com/rails-sqlserver/tiny_tds">TinyTDS</a> is the upcoming de facto raw connection method for the SQL Server adapter. Please read the <em><a href="https://github.com/rails-sqlserver/activerecord-sqlserver-adapter/wiki/Using-TinyTds">Using TinyTDS</a></em> wiki page on the adapter for switching. No longer do you have to worry about compiling ODBC layers!
</aside>

<aside class="flash_info">
  I have changed my mind about the UTF-8 version of RubyODBC. Please <a href="http://groups.google.com/group/rails-sqlserver-adapter/browse_thread/thread/367cf70b2b48d4a0?hl=en">read that section</a> in the the rails 3 announcements for the SQL Server adapter.
</aside>


<p>
  Quite a while ago I wrote a <a href="/2009/9/5/the-ultimate-os-x-snow-leopard-stack-for-rails-development-x86_64-macports-ruby-1-8-1-9-sql-server-more">soup to nuts article on getting the full multi-ruby development stack installed</a> for those using the SQL Server adapter. The base package management system used there was MacPorts. In it I described how to edit the outdated Portfile for the rb-odbc package and exclaimed how important it was to use the +utf8 variant. I was totally wrong about that part.
</p>

<p>
  This past week I started heavily exploring <a href="http://rvm.beginrescueend.com/">RVM</a> at the advice of friend while we visited the Boston.rb user grooup. As an aside, I finally feel my unix-fu is strong enough to cover the edge cases needed to get the SQL Server stack happy with RVM. Good news, but that's next weeks blog post. Anyways I found that while updating my core MacPort's rb-odbc package that +utf8 was blowing up. I am currently using MacPorts 1.9.1 and was targeting 0.99991 of RubyODBC. Here is the updated port file. To edit your current port file run <code>mate $(port file rb-odbc)</code> and paste this new one in. Details afterward.
</p>

```text
# $Id: Portfile 30250 2010-07-18 02:16:17Z ken@metaskills.net $
PortSystem    1.0
PortGroup     ruby 1.0

ruby.setup          {odbc ruby-odbc} 0.99991 extconf.rb {README doc test} 
maintainers         nomaintainer
description         An extension library for ODBC from ruby.
long_description    Extension library to use ODBC data sources from Ruby. \
                    Supports Ruby 1.6.x and 1.8 on Win32 OSes and UN*X 
checksums           md5 64eaf6089e7ca17eeff54c4fe052ac96
homepage            http://www.ch-werner.de/rubyodbc
master_sites        http://www.ch-werner.de/rubyodbc
categories-append   databases
platforms           darwin 

configure.cmd             ${ruby.bin} -rvendor-specific -Cext extconf.rb
build.pre_args-append     -C ext
destroot.pre_args-append  -C ext

variant utf8 {
  configure.cmd             ${ruby.bin} -rvendor-specific -Cext/utf8 extconf.rb
  build.pre_args-delete     -C ext
  build.pre_args-append     -C ext/utf8
  destroot.pre_args-delete  -C ext
  destroot.pre_args-append  -C ext/utf8
}
```

<p>
  So where did I go wrong on my old article? My first big mistake was thinking that the <code>+utf8</code> port variant was needed. Not only is it <strong>NOT NEEDED</strong>, it may not work right at all. In fact, the old Portfile in MacPorts trunk technically did not even configure/make/install the utf8 version either! Honestly â€“ I spent all day learning the MacPort's Portfile syntax and tested this. Installing that variant just breaks with an error like <code>LoadError: dlsym(0x1010cd3b0, Init_odbc): symbol not found</code>.
</p>

<p>
  So, even though my updated Portfile above now fixes that issue and supports actually building a utf8 version of RubyODBC, <strong>YOU DO NOT NEED IT!</strong> In fact the entire SQL Server stack is tested with the plain non-utf8 package and passes with flying colors. This includes passing tests where unicode columns return correctly utf8-encoded strings, among others. Though I have not tested it, I believe the utf8 version would do more damage than good. The RubyODBC documentation says it would make every string utf8 encoded. Not good for an adapter that has mixed data types. For my own personal notes, here is a diff of the Makefile below. I think the Init_odbc error was a result of a missing flag in the Makefile.
</p>

```diff
# My notes
# +CPPFLAGS = -DHAVE_LONG_LONG
# +CPPFLAGS = -DHAVE_VERSION_H
# +CPPFLAGS = -DHAVE_TYPE_SQLBIGINT
# -CPPFLAGS = -DHAVE_SQLCONFIGDATASOURCEW
# -CPPFLAGS = -DHAVE_SQLINSTALLERERRORW
# -CPPFLAGS = -DHAVE_SQLWRITEFILEDSNW
# -CPPFLAGS = -DHAVE_SQLREADFILEDSNW
--- Makefile(+utf8)  2010-07-18 12:00:24.000000000 -0400
+++ Makefile(-utf8)  2010-07-18 12:49:32.000000000 -0400
@@ -47,7 +47,7 @@
 CFLAGS   =  -fno-common -O2 -arch x86_64  -fno-common -pipe -fno-common $(cflags) -arch x86_64
 INCFLAGS = -I. -I. -I/opt/local/lib/ruby/1.8/i686-darwin10 -I.
 DEFS     = 
-CPPFLAGS = -DHAVE_SQL_H -DHAVE_SQLEXT_H -DHAVE_TYPE_SQLTCHAR -DHAVE_TYPE_SQLLEN -DHAVE_TYPE_SQLULEN -DHAVE_ODBCINST_H -DHAVE_SQLCONFIGDATASOURCEW -DHAVE_SQLWRITEFILEDSNW -DHAVE_SQLREADFILEDSNW -DHAVE_SQLINSTALLERERROR -DHAVE_SQLINSTALLERERRORW -I/opt/local/include -D_XOPEN_SOURCE -D_DARWIN_C_SOURCE  -I/opt/local/include
+CPPFLAGS = -DHAVE_VERSION_H -DHAVE_SQL_H -DHAVE_SQLEXT_H -DHAVE_TYPE_SQLTCHAR -DHAVE_TYPE_SQLLEN -DHAVE_TYPE_SQLULEN -DHAVE_ODBCINST_H -DHAVE_SQLINSTALLERERROR -DHAVE_TYPE_SQLBIGINT -I/opt/local/include -D_XOPEN_SOURCE -D_DARWIN_C_SOURCE  -I/opt/local/include -DHAVE_LONG_LONG
 CXXFLAGS = $(CFLAGS) 
 ldflags  = -L. -L/opt/local/lib -arch x86_64
 dldflags = 
@@ -89,7 +89,7 @@
 LIBS = $(LIBRUBYARG_SHARED) -lodbcinst -lodbc  -lpthread -ldl -lobjc  
 SRCS = init.c odbc.c
 OBJS = init.o odbc.o
-TARGET = odbc_utf8
+TARGET = odbc
 DLLIB = $(TARGET).bundle
 EXTSTATIC = 
 STATIC_LIB =
```


