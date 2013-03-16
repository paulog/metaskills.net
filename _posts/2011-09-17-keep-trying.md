---
layout: post
title: Keep Trying
categories: 
  - heuristics
  - ruby
  - apple-os
---

<p>
  <span class="photofancy floatr ml20 mb20">
    <img src="/assets/evilruby.jpg" alt="Ruby Is Evil!" width="248" height="195" />
  </span>
  One part of Objective-C that I like is being able to send messages to nil objects safely and more so their <a href="http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html">KVC</a> and <a href="http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html">KVO</a> patterns. In Ruby I often use the <code>#try</code> method to safely send messages to objects that may be nil at runtime. But one thing I always wanted was a nice way to send a key path, basically a string of methods signatures, to an object in the same way. I give you my simple <code>#try_keypath</code> method :)
</p>

```ruby
class Object
  def try_keypath(methods, *args, &block)
    methods.to_s.split('.').inject(self) { |result, method| result.try(method) }
  end
end

class NilClass
  def try_keypath(*args)
    nil
  end
end
```

<p>
  Yup, it is that simple. Let's see how this would work. Here are a few basic classes that randomly return nested objects. So staring with the <code>Foo</code> object, we have the possibility to get to some deeply nested info.
</p>

```ruby
class Foo
  def bar
    rand(2) == 1 ? Bar.new : nil
  end
end

class Bar
  def batz
    'wohoo'
  end
  def deep
    Deep.new
  end
end

class Deep
  def info
    {:winning => true}
  end
end
```

<p>
  Finally, here is how it would look and return different results. Man I love Ruby!
</p>

```ruby
Foo.new.try :bar # => #<Bar:0x108d7dfd0>
Foo.new.try :bar # => #<Bar:0x108d7ddf0>
Foo.new.try :bar # => nil
Foo.new.try :bar # => nil
Foo.new.try :bar # => #<Bar:0x108d7d4b8>
Foo.new.try :bar # => nil

Foo.new.try_keypath 'bar.batz' # => "wohoo"
Foo.new.try_keypath 'bar.batz' # => "wohoo"
Foo.new.try_keypath 'bar.batz' # => nil
Foo.new.try_keypath 'bar.batz' # => nil
Foo.new.try_keypath 'bar.batz' # => nil
Foo.new.try_keypath 'bar.batz' # => "wohoo"

Foo.new.try_keypath 'bar.deep.info' # => nil
Foo.new.try_keypath 'bar.deep.info' # => {:winning=>true}
Foo.new.try_keypath 'bar.deep.info' # => nil
Foo.new.try_keypath 'bar.deep.info' # => nil
Foo.new.try_keypath 'bar.deep.info' # => {:winning=>true}
Foo.new.try_keypath 'bar.deep.info' # => nil
```


