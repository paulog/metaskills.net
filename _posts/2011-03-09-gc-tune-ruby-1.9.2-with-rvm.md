---
layout: post
title: GC Tune Ruby 1.9.2 With RVM
categories: 
  - ruby-rails
---

<p>
  Here is a <a href="https://gist.github.com/856296">gist</a> by Sokolov Yura (funny-falcon) that allows you to GC tune Ruby 1.9.2 just like Ruby Enterprise Edition (REE). So all of us using <a href="http://rvm.beginrescueend.com/">RVM</a> have no reason not start using this right away. Here are the steps.
</p>

<pre class="command">
  $ curl https://gist.github.com/raw/856296/patch-1.9.2-gc.patch > ~/192-gc.patch
  $ rvm uninstall 1.9.2
  $ rvm install 1.9.2 --patch ~/192-gc.patch
</pre>

<p>
  I have used this RVM hook below for awhile now. It automatically sets and unsets the proper ENV vars to GC tunes my REE. I have now updated it to apply the same GC settings to my newly patched 1.9.2 as well. I recommend this go into <code>~/.rvm/hooks/after_use</code>. 
</p>

~~~bash
case "$rvm_ruby_string" in
  *ree*|*ruby-1.9.2*)
    export RUBY_HEAP_MIN_SLOTS=1000000
    export RUBY_HEAP_SLOTS_INCREMENT=1000000
    export RUBY_HEAP_SLOTS_GROWTH_FACTOR=1
    export RUBY_GC_MALLOC_LIMIT=1000000000
    export RUBY_HEAP_FREE_MIN=500000
    export RUBY_FREE_MIN=$RUBY_HEAP_FREE_MIN
  ;;
  *)
    unset RUBY_HEAP_MIN_SLOTS RUBY_HEAP_SLOTS_INCREMENT RUBY_HEAP_SLOTS_GROWTH_FACTOR RUBY_GC_MALLOC_LIMIT RUBY_HEAP_FREE_MIN RUBY_FREE_MIN
  ;;                                                                                                                                                                                                                                                                                                                        
esac
~~~





