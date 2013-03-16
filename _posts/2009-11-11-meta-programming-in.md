--- 
layout: post
title: Meta Programming In...
disqus_id: /2009/11/11/meta-programming-in/
categories: 
  - ruby-rails
  - heuristics
---


<p>
  Last nights <a href="http://757rb.org/">757.rb</a> meeting was a great success. We did a talk titled <em>Introduction To Ruby & Rails</em> for all the new comers that might have been interested in learning more about both - from the ground up. Since Ruby was my first language and my toolbelt only extends to JavaScript and Objective-C, I'm always interested in how other languages do the things that I love so much in Ruby. One of the biggest pluses for Ruby, to me, is the support for meta programming. In a typical rails model, you might see something like this.
</p>

```ruby
class User < ActiveRecord::Base
  has_many                :articles
  named_scope             :active, :conditions => {:active => true}
  validates_uniqueness_of :email
  validates_presence_of   :email, :username, :twitter_handle
end
```

<p>
  For the untrained, those are class level methods writing code for an model that persists objects to a database. It looks like you are typing a spec or some declaration. In fact this is code writing other code. For newcomers to the language this is voodoo magic, but if you write programs and interfaces for others, meta programming is what it is all about. In fact, I use this technique all the time instead of writing the same code over and over again.
</p>

<p>
  This concept is built into the core of Ruby. In fact here is an example from my talk on the class method built into Ruby called <code>attr_accessor</code> which wraps a common paradigm of generating getter/setter methods for an instance variable. No magic here, the two class definitions below are equal and the interface of the object is the same. One is you do not have to write code over an over again in an vanilla class you care to have getter/setter methods for.
</p>

```ruby
class MyClass
  attr_accessor :foo
end

# Same as writing

class MyClass
  def foo
    @foo
  end
  def foo=(value)
    @foo = value
  end
end

# Same results

o = MyClass.new
o.foo           # => nil
o.foo = 'bar'   # => "bar"
o               # => #<MyClass:0x100172b80 @foo="bar">
```

<p>
  So here is where I want to learn. I know how to write Ruby well enough that I can show one way below of how to write <code>attr_accessor</code> if it did not exist in core Ruby. In my example below I made the method named <code>attribute_accessor</code> to avoid the name conflict. Can anyone show me how this would be done in PHP, Java, Anything? What is meta programming like in those languages? How about posting a code example on <a href="http://pastie.org/">pastie.org</a> and school me. I really am interested in knowing. BTW, my example only shows how the <code>attribute_accessor</code> takes on argument. In actuality in Ruby, the built in <code>attr_accessor</code> method takes multiple args and generates methods for each. For simplicity I narrowed down my examples and code to only take one argument so we can all stay focused.
</p>

```ruby
module MetaSkills
  
  def self.included(klass)
    klass.class_eval do
      include ClassMethods
    end
  end
  
  module ClassMethods
    
    def attribute_accessor(name)
      attribute_reader(name)
      attribute_writer(name)
    end

    def attribute_reader(name)
      define_method(name) do
        instance_variable_get "@#{name}"
      end
    end

    def attribute_writer(name)
      define_method "#{name}=" do |value|
        instance_variable_set "@#{name}", value
      end
    end
    
  end
  
end

Class.send :include, MetaSkills

class MyClass
  attribute_accessor :foo
end

o = MyClass.new
o.foo           # => nil
o.foo = 'bar'   # => "bar"
o               # => #<MyClass:0x100171c80 @foo="bar">
```

<p>
  Update: A friend was asking me about the namespaces you see in the module. It is technically possible to just open up the Class object and add my example additions. Typically in Ruby you are encouraged to create your own namespaced module and inject class/instance modules into the included class via the module's hook method <code>self.included</code>. The pattern is like so <a href="http://pastie.org/694019">http://pastie.org/694019</a>. So for instance, I could have done this <a href="http://pastie.org/694038">http://pastie.org/694038</a> without taking advantages of the modules hook method. 
</p>





