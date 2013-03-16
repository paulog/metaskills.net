---
layout: post
title: StoreConfigurable - A Lesson In Recursion In Ruby
categories: 
  - heuristics
  - ruby-rails
---

<p>
  When ActiveRecord 3.2 was released there was a small addition called <a href="http://api.rubyonrails.org/classes/ActiveRecord/Store.html">ActiveRecord::Store</a> which bills itself as a simple key/value store for your models. The code below is pulled right from their example usage.
</p>

```ruby
class User < ActiveRecord::Base
  store :settings, accessors: [:color, :homepage]
end

@user = User.new(color: 'black', homepage: '37signals.com')
@user.color            # => 'black'
@user.settings[:color] # => 'black'
@user.settings[:remember_me] = true
```

<p>
  Most people know that I love simple tools. But when I found myself considering <code>ActiveRecord::Store</code>, I found it seriously lacking for my particular use case. What I wanted was a config for a user class that could do the following:
</p>

<ul>
  <li>Interchange property dot notation with indifferent string/symbol lookup.</li>
  <li>Automatically grow as needed. For namespaces and nested organization.</li>
  <li>Report state changes to the model from any node.</li>
</ul>


<h2>StoreConfigurable</h2>

<p>
  <span class="photofancy floatr ml20 mb10">
    <img src="http://cdn.actionmoniker.com/share/recursive_kitty_small.jpg" width="220" height="136">
  </span>
  The lack of these features is why I set out to build a little gem I call <a href="https://github.com/metaskills/store_configurable">StoreConfigurable</a>. A zero-configuration recursive Hash for storing a tree of options in a serialized ActiveRecord column which includes self-aware hooks that delegate dirty/changed state to your configs owner. Perfect right? Here is a simple example of its usage below. If you want to learn more, checkout the <a href="https://github.com/metaskills/store_configurable">project on github</a> with the full README. 
</p>

```ruby
class User < ActiveRecord::Base
  store_configurable
end

@user.config.remember_me = true
@user.config.sortable_tables.column    = 'created_at'
@user.config.sortable_tables.direction = 'asc'
@user.config.you.should.never.need.to.do.this.but.you.could.if.you.wanted.to = 'deep_value'
@user.config_changed? # => true
```

<p>
  Rather than talking about how StoreConfigurable might be a good fit for you, I would instead like to discuss how my first practical usage of recursion in Ruby is implemented in StoreConfigurable. That along with how to leverage default Hash values is the topic for this post.
</p>


<h2>Default Hash Values</h2>

<p>
  So first up, a little known feature of Ruby's <a href="http://www.ruby-doc.org/core-1.9.3/Hash.html">Hash</a> class is the ability for a default value to be returned when a key is missing. If you stop and think about this, the default is <code>nil</code>, which is a valid object in Ruby. But you can tell Hash objects to return other values when a key is missing. The first way to do this is to pass the default value as an argument to <code>Hash.new()</code>. The second is to pass a block to the new method which would return the default value. That block is passed in the current hash object and the key that is missing so you can do some fancy things if needed. Here are some quick examples of both techniques. 
</p>

```ruby
# Default value passed to new.

hash = Hash.new('default')  # => {}
hash[:foo]                  # => "default"

# Default value from new's block.

hash = Hash.new { |hash, key| "Missing #{key} for hash with id #{hash.object_id}." }
hash                        # => {}
hash[:foo]                  # => "Missing foo for hash with id 283829283."
```


<h2>Recursion</h2>

<p>
  Recurion in Ruby can take many forms. My solution for StoreConfigurable was to always make the hash returned by the proxy object self replicate itself for key misses. Here is a fundamental example to make your own Hash object return new instances of itself when keys are missing.
</p>

```ruby
class RecursiveHash < Hash
  Recursive = lambda { |h,k| h[k] = h.class.new }
  def initialize
    super(&Recursive)
  end
end

hash = RecursiveHash.new  # => {}
hash.class                # => RecursiveHash
hash[:foo]                # => {}
hash[:foo].class          # => RecursiveHash
hash[:a][:b][:c]          # => {}
hash                      # => {:foo=>{}, :a=>{:b=>{:c=>{}}}}
```

<p>
  In this example, I create a <a href="http://www.ruby-doc.org/core-1.9.3/Proc.html">Proc</a> object via the lambda keyword and I use this as the block argument when you create an instance of <code>RecursiveHash</code> via the super method. Here I am using the <code>&amp;</code> syntax to pass an existing Proc as a block. Hope I have not lost you so far :)
</p>

<p>
  So default hashes in Ruby and simple recursion techniques are what make up some of the key points of StoreConfigurable. I also use recursion when loading the hash values from the YAML stored in the database. Here is what that technique looks like.
</p>

```ruby
loader = lambda do |hash, key, value|
  if value.is_a?(Hash)
    value.each { |k,v| loader.call(hash.send(key), k, v) }
  else
    options.send "#{key}=", value
  end
end

data.each { |k,v| loader.call(config, k, v) }
```

<p>
  Hopefully this dive into Ruby or maybe StoreConfigurable is helpful!
</p>

<h2>Resources</h2>

<ul>
  <li><a href="https://github.com/metaskills/store_configurable">StoreConfigurable</a> - A zero-configuration recursive Hash for storing a tree of options in a serialized ActiveRecord column with self-aware hooks that delegate dirty/changed state to your configs owner.</li>
  <li><a href="http://api.rubyonrails.org/classes/ActiveRecord/Store.html">ActiveRecord::Store</a> - Simple key/value store for your models.</li>
</ul>


