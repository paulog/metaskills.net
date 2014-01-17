--- 
layout: post
title: Don't Be A Plinko Programmer
disqus_id: /2008/03/26/don-t-be-a-plinko-programmer/
categories: 
  - miscellaneous
  - ruby-rails
---


<p class="clearfix">
  <img src="/assets/plinko.png" alt="Plink Game" class="floatr ml20" />
  One of the things that I have really grown persnickety about is the placement of code. For example, I am a huge advocate that controllers in a rails project should read like a mini Domain Specific Language (DSL) and that as much logic as possible be delegated to the models. In my opinion the best way to do that in a Rails project is to learn the proper usage of ActiveRecord Association Extensions. You can check out the Rails API <a href="http://api.rubyonrails.com/classes/ActiveRecord/Associations/ClassMethods.html">on this page</a> and scroll down to the section called "Association Extensions" if you read the official docs. In short:
</p>

<blockquote>
  The proxy objects that control the access to associations can be extended through anonymous modules. This is especially beneficial for adding new finders, creators, and other factory-type methods that are only used as part of this association.
</blockquote>

<p>
  Now this brings me to the topic of my article, what is a Plinko Programmer. If you have no class and don't even know what Plinko is, <a href="http://en.wikipedia.org/wiki/Plinko">Wikipedia has a great write up</a> on it. A Plinko Programmer is someone that writes code which smells in a few ways. For instance they use excessive arguments in their methods and unnecessarily pass objects around as arguments. They like to bake their own <a href="http://www.ruby-doc.org/core/classes/Enumerable.html">Enumerable</a> methods vs using the ones readily available. They even like to create large class level methods, or even worse controller actions, that really should be factory methods in 2 or more classes. The analogy is akin to much of the Java code I have rewritten in Rails. Plinko code is long and full of if/else conditions, it's just like the game. You drop an object in the top and just watch it "by chance" work it's way thru the method/function. It is a nasty way to write code and if for anything else it is illegible and hard to test.
</p>


<h2>What Is The Right Way?</h2>

<p>
  Here is a short example of how to use Association Extensions. This is a simple example, but when you get used to really using association extensions you will begin to see just how much of your code really belongs there. Let's assume you have a simple invoice and items class.
</p>

~~~ruby
class Invoice < ActiveRecord::Base
  has_many :items, :class_name => 'InvoiceItem', :foreign_key => 'invoice_id'
end
class InvoiceItem < ActiveRecord::Base
  belongs_to :invoice
  def total
    # Some complex stuff
  end
end
~~~

<p>
  Now let's say that you want to have a clean little method for getting the total of the Invoice object. Resist the temptation to simple add an instance method to the Invoice class. Although it is logical to have <code>@invoice.total</code> it is better to add it too the association extension. Why? Well think about it, what are you doing? The answer is that you are working with a "collection" of InvoiceItems. It turns out that this is the first part of what the association extension is for, an easy way to work with a collection that has the benefits of knowing how to proxy to methods that can reflect back down to the original caller. It's hard to explain but I'll just leave you with my persnickety code example. Your general rule should be if you are working with the collection in part or in total, then the association extension is the place for it. Keep in mind that so far I have only talked about has_may association extensions, you can do these for one-to-on belongs_to and has_one associations as well.
</p>

~~~ruby
class Invoice < ActiveRecord::Base
  has_many :items, :class_name => 'InvoiceItem', :foreign_key => 'invoice_id' do
    def total
      proxy_target.map(&:total).sum
    end
  end
end
class InvoiceItem < ActiveRecord::Base
  belongs_to :invoice
  def total
    # Some complex stuff
  end
end
# Would yeild code like:
@invoice.items.total
~~~

<p>P.S. Here lately I've been creating an app/concerns directory where I put modules that encompass mixed in behavior in so many ways for top level models. Typically these modules/concerns shared instance and class methods with two or more primary classes. They have become an excellent home for association extensions since many large applications will define the same association from different models in the object. To keep the code from duplicating in those different models it is better to do something like this</p>

~~~ruby
# This file "invoice_item_concerns.rb" would reside in app/concerns
module InvoiceItemConcerns
  module AssociationExtensions
    def total
      proxy_target.map(&:total).sum
    end
  end
end

class Invoice < ActiveRecord::Base
  has_many :items, :class_name => 'InvoiceItem', :foreign_key => 'invoice_id', :extend => InvoiceItemConcerns::AssociationExtensions
end

class PackingSlip < ActiveRecord::Base
  has_many :shipments, :class_name => 'InvoiceItem', :foreign_key => 'packing_slip_id', :extend => InvoiceItemConcerns::AssociationExtensions
end

class InvoiceItem < ActiveRecord::Base
  belongs_to :invoice
  def total
    # Some complex stuff
  end
end

# Would yeild code like:
@invoice.items.total
@packing_slip.shipments.total
~~~



