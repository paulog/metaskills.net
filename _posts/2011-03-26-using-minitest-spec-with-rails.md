---
layout: post
title: Using MiniTest::Spec With Rails
lastmod: 2012-03-27
categories: 
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: I have renamed the gem that allows you to switch to MiniTest::Spec in Rails to minitest-spec-rails. 
  It is now considered a single gem install with a drop-in to switch to MiniTest::Spec. This article has been updated to reflect this change.
</aside>

<p>
  So after a few blog post on the subject of MiniTest::Spec, I finally have a simple solution for Rails 3 that leverages MiniTest's spec DSL. I introduce to you the <a href="http://github.com/metaskills/minitest-spec-rails">minitest-spec-rails</a> gem. MiniTestSpecRails defines a <code>Test::Unit::TestCase</code> class that subclasses MiniTest::Spec. It implements only what is needed to make Rails happy.
</p>

<p>
  Once you bundle the gem in your Rails application, it will satisfy the <code>require "test/unit/testcase"</code> from ActiveSupport's test case. Tricking it to use MiniTest::Spec instead of MiniTest::Unit. Here is an example Gemfile that shows the usage of MiniTestSpecRails. That is all you have to do!
</p>

```ruby
group :test do
  gem 'minitest-spec-rails'
end
```


<h2>Advantages?</h2>

<p>
  Since MinitTest::Spec is built on top of MiniTest::Unit which is technically a replacement for Test::Unit, there is not a lot that can go wrong. With minitest-spec-rails, we finally have a working solution by replacing MiniTest::Spec as the superclass for ActiveSupport::TestCase. This solution is drop dead simple and does not require you to recreate a new test case in your <code>test_helper.rb</code> or to use generators supplied by gems like <a href="https://github.com/blowmage/minitest-rails">minitest-rails</a>.
</p>
  
<p>
  This means two things. First, older Rails applications can switch to MiniTest::Spec now by simply installing this gem. Second, as Rails itself evolves to eventually leverage MiniTest::Spec, your test case code will not have to change. About the only gotcha is a few missing assertions available in Test::Unit that are changed or no longer available in MiniTest. For example, <code>assert_raise</code> vs <code>assert_raises</code> and there is no such thing as <code>assert_nothing_raised</code>. Code you would have had to change eventually anyway.
</p>


<h2>New Assertion Styles</h2>

<p>
  One thing that I try to do is to change my test assertions from the old Test::Unit style to the new MiniTest::Spec style. I commonly use this <a href="http://cheat.errtheblog.com/s/minitest/1">cheat sheet</a> to remember them. For instance:
</p>

```ruby
# This:
assert_not_nil @foo.bar

# Would Become This:
@foo.bar.wont_be_nil
```

<p>
  Likewise, MiniTest::Spec is consistently getting new features added. One patch that I advocated for now allows the <code>must_be</code> and <code>wont_be</code> low-level assertions to work with a predicate method symbol. Meaning, if the symbol ends in a question mark, then it will implicitly call that mehod on the subject for truthy or falsy matching. For instance:
</p>

```ruby
@user.must_be :valid?
```


<h2>Known Issues</h2>

<p>
  None that I know of yet. Try it out and <a href="http://github.com/metaskills/minitest-spec-rails/issues">report any issues to our github project</a> page. If you have been using minitest-spec-rails, I would love to hear about it too!
</p>


<h2>Resources</h2>

<ul>
  <li>MiniTest::Spec For Rails - <a href="http://github.com/metaskills/minitest-spec-rails">Drop in MiniTest::Spec support for Rails 3.</a></li>
  <li>MiniBacktrace - <a href="/2011/03/22/mini-backtrace-for-minitest-and-rails/">Make the Rails backtrace cleaner work.</a></li>
  <li>MiniTest Rails - <a href="http://github.com/blowmage/minitest-rails">Official minitest project for Rails that uses generators.</a></li>
</ul>


