--- 
layout: post
title: Autotestify Brothers and Sisters
disqus_id: /2008/12/30/autotestify-brothers-and-sisters/
categories: 
  - heuristics
  - ruby-rails
  - workflow
---

<p>
  Based on my previous article <a href="/2008/09/19/using-autotest-for-rails-plugin-development/">Using Autotest For Rails Plugin Development</a>, <a href="http://brennandunn.com/">Brennan Dunn</a> wrote a ZSH function that helps him with his eager rails plugin work. Testing plugins is simply the most fun you will ever have. It's nice to test in isolation and easy to make a plugin test multiple rails versions, etc. Typically plugin testing is very fast to because you are not burdened with excessive tests in a big app that might use it. Just the ones needed to test the plugin. Enjoy this ZSH function.
</p>

~~~bash
function autotestify () {
  git clone git://github.com/metaskills/autotest_railsplugin.git
  rm -rf ./autotest_railsplugin/.git
  mv ./autotest_railsplugin ./autotest
}
~~~

