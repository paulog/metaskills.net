---
layout: post
title: From Test::Unit & Shoulda To MiniTest::Spec & MiniShoulda
categories: 
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: If you want to use <a href="/2011/03/26/using-minitest-spec-with-rails/">MiniTest::Spec with Rails please read my updated post and how to use the minitest-spec-rails gem</a>.
</aside>

<p>
  It seems the MiniTest replacement for Test::Unit in Ruby 1.9 has been presenting itself more and more in my daily readings. Recently was Jamis Buck's article titled <a href="http://37signals.com/svn/posts/2742-the-road-to-faster-tests">The road to faster tests</a> where he talks about optimizing basecamp's suite to run faster. And there again, some of the comments below mentioned MiniTest as a solution. That got me thinking on what would it take for me to switch from Test::Unit to MiniTest.
</p>

<p>
  I love testing! But I am not nuts about large libraries. Looking at you RSpec! <a href="http://evan.tiggerpalace.com/articles/2010/12/18/ruby-test-unit-sucks-and-why-i-still-use-it/">Like others</a>, I believe that you do not need a bloated test framework to get good coverage with a lean and succinct style. A handful of assertions and a simple context structure provided by libraries like <a href="https://github.com/thoughtbot/shoulda">Shoulda</a> is really all I need for both unit and a certain level of functional testing. It is my firm believe that large libraries and semi-magical test code based on things like RSpec will wane in the coming years. Those that base very large test suites on them will suffer the penalty for when an upgrade or test refactor is needed. I could be wrong, time will tell. For now let's cover how to move to MiniTest for those that like simple Test::Unit and or Shoulda.
</p>

<p>
  MiniTest provides a drop in replacement for Test::Unit, so I wont cover the details of those basics. What most do not know is that MiniTest comes with a simple MiniTest::Spec DSL that uses <code>describe</code> and <code>it</code> blocks much like Shoulda's <code>context</code> and <code>should</code> methods. The similarities were so close that I set out to move all my projects to MiniTest::Spec. The goal would be that none of my exiting suites context/should/before/after code would require a change. The result...
</p>


<h2>MiniShoulda</h2>

<p>
  <a href="https://github.com/metaskills/mini_shoulda">MiniShoulda</a> is a small ruby gem that puts a Shoulda DSL on top of MiniTest::Spec. All it really does it alias a few key methods. Here is the core of it now.
</p>

~~~ruby
class MiniTest::Spec < MiniTest::Unit::TestCase
  
  class << self
    alias :setup    :before
    alias :teardown :after
    alias :should   :it
    alias :context  :describe
  end
  
  def self.should_eventually(desc)
    it("should eventually #{desc}") { skip("Should eventually #{desc}") }
  end
  
end
~~~

<p>
  For now, MiniShoulda performs only one monkey patch to MiniTest. There is a critical bug in MiniTest that does not allow the describe/context blocks to use the existing scope of the parent class. I have a <a href="https://github.com/seattlerb/minitest/pull/9">pull request</a> in for the project. Please help out and add a thoughtful comment based on the <a href="https://github.com/metaskills/minitest/commit/e7cde5bd9e61bc1ac13c7326ef4de23382e3467b">merits of the patch</a> and the <a href="https://gist.github.com/793330">description of the problem</a>. No author likes to see simple +1 comments.
</p>


<h2>The MiniTest Switch</h2>

<pre class="command">
$ gem install mini_shoulda
</pre>

<p>
  Our gem spec will automatically pull in MiniTest version 2.0.2 or greater. Assuming you have a test helper file, the top requires will looks something like this. MiniTest needs that autorun file required or none of your tests will run.
</p>

~~~ruby
require 'mini_shoulda'
require 'minitest/autorun'
~~~

<p>
  What should I expect? For starters, expect a few of your tests to fail. Especially ones that were happenstantially running linearly and not exercised in isolation. I think there is a technical term for this? Anyways, MiniTest mixes up the test suite each run trough it using a random seed. So it is highly like to expose a few poorly written tests that depended on state leftover from a previous test. Also the MiniTest::Unit underneath MiniTest::Spec does not have a few assertions you may have used. One is <code>assert_raise</code> which is now the standard convention <code>assert_raises</code> with an "s". And finally <code>assert_nothing_raised</code> has been pulled completely since Test::Unit's documentation alluded to its uselessness for a long time.
</p>

<p>
  What does MiniTest::Spec give you besides clean and powerful describe/it organization with before/after setups and teardowns? I highly recommend the <a href="http://cheat.errtheblog.com/s/minitest/1">MiniTest cheat sheet</a> to find out. See the spec and mock methods at the bottom? As much as I have always like the basic unit testing assertions, I am looking forward to using the must/wont object assertions.
</p>


<h2>Resources</h2>

<ul>
  <li><a href="https://github.com/metaskills/mini_shoulda">MiniShoulda Github Project</a></li>
  <li><a href="https://github.com/seattlerb/minitest/pull/9">The MiniTest Describe Pull Resuest</a> - HELP OUT! COMMENT!</li>
  <li><a href="http://cheat.errtheblog.com/s/minitest/1">MiniTest Cheat Sheet</a></li>
</ul>
