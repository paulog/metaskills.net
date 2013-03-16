--- 
layout: post
title: Autotest Infinite Loop On Failure & Error
disqus_id: /2008/04/06/autotest-infinite-loop-on-failure-error/
categories: 
  - heuristics
  - ruby-rails
---


<p>
  I just had an issue pop up today that seemed to be an issue for a few others. It seemed that all of a sudden that my autotest was stuck in an infinite loop after a failure or error. At first I thought it was related to some additions to my <code>~/.autotest</code> file but after commenting out the whole lot of additions there, I realized it was something else. Here was my fix. Basically I think these errors are always related to a file that has changed during your test run. Now we just have to find out what that files are. Here are the steps I took to find out.
</p>


<h2>Step 1: Gather Changed FIle Info</h2>

<p>
  To find out what files are changing. To do this add the -v option when you start autotest. This will cause it to run in verbose mode. Now after you have failed a test and YOU KNOW YOU DID NOT SAVE ANYTHING watch what happens below your listing of test, assertions, failures, and errors. There will be an array dumped that will contain the files change that have caused autotest to start another test cycle. In my case here is what I saw.
</p>

```text
1) Failure:
test_truth(BookmarkTest)
[test/unit/bookmark_test.rb:6
test/unit/bookmark_test.rb:5]:
<false> is not true.

1 tests, 1 assertions, 1 failures, 0 errors
[["config/uuid.state", Sun Apr 06 13:58:55 -0400 2008]]
Dunno! config/uuid.state
```

<p>
  Ah ha... there it goes. I was using a UUID state file and it appears that file being written into the <code>config</code> directory is the culprit. By default autotest does not ignore that directory.
</p>


<h2>Step 2: Fix The Problem</h2>

<p>
  We have two options here. You can either add an autotest exception in you <code>~/.autotest</code> file or you can bail out. My option is to bail out because I should not be adding a file at run time to the config directory. It would seem a better place would be the <code>tmp</code> directory. However if you think that adding an exception to your <code>~/.autotest</code> file would be more appropriate, here is the syntax:
</p>

```ruby
Autotest.add_hook :initialize do |autotest|
  ['.svn','.hg','.git'].each { |exception| autotest.add_exception(exception) }
end
```











