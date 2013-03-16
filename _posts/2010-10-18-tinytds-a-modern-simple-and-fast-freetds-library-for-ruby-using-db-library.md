--- 
layout: post
title: TinyTds - A modern, simple and fast FreeTDS library for Ruby using DB-Library
disqus_id: /2010/10/18/tinytds-a-modern-simple-and-fast-freetds-library-for-ruby-using-db-library/
categories: 
  - database
  - ruby-rails
---

<p>I just finished the first cut of learning C extensions for ruby and I <a href="http://github.com/rails-sqlserver/tiny_tds">present The TinyTds gem</a>. It is meant to serve the extremely common use-case of connecting, querying and iterating over results to Microsoft SQL Server databases from ruby. Even though it uses FreeTDS's DB-Library, it is NOT meant to serve as direct 1:1 mapping of that complex C API</p>

<p>The benefits are speed, automatic casting to ruby primitives, and proper encoding support. It converts all SQL Server datatypes to native ruby objects supporting :utc or :local time zones for time-like types. To date it is the only ruby client library that allows client encoding options, defaulting to UTF-8, while connecting to SQL Server. It also properly encodes all string and binary data. The motivation for TinyTds is to become the de-facto low level connection mode for the SQL Server adapter for ActiveRecord. For further details see the special thanks to Erik Bryn for his help, the authors/contributors of the Mysql2 gem for inspiration, and Yehuda Katz for articulating ruby's need for proper encoding support. Please read up on it here.</p>


