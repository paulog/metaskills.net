---
layout: post
title: MiniBacktrace - For MiniTest & Rails
categories: 
  - ruby-rails
---

<aside class="flash_info">
  UPDATE:  
  <a href="/2011/03/26/using-minitest-spec-with-rails/">Using MiniTest::Spec With Rails</a> 
  &amp;
  <a href="/2011/03/26/using-minitest-spec-with-rails/">From Test::Unit &amp; Shoulda To MiniTest::Spec &amp; MiniShoulda</a>
</aside>

<p>
  <a href="https://github.com/metaskills/mini_backtrace">MiniBacktrace</a> allows you to take advantage of the Rails.backtrace_cleaner when using MiniTest. This includes everyone using Rails 3 with Ruby 1.9.
</p>

<p>
  Just add 'mini_backtrace' to your Gemfile's :test group and your should automatically see a huge difference. Any additions to the Rails.backtrace_cleaner should now work.
</p>

~~~ruby
Rails.backtrace_cleaner.add_silencer { |line| line =~ /my_noisy_library/ }
~~~

