--- 
layout: post
title: Jack has_many :things
disqus_id: /2008/09/28/jack-has_many-things/
lastmod: 2011-12-06T12:00:00Z
categories: 
  - database
  - heuristics
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: [12/06/2011] The <a href="https://github.com/metaskills/grouped_scope">GroupedScope</a> gem has been updated for Rails 3.1. Many new features have been added with a few mentioned below. Check the project's readme for complete details.
</aside>

<p>
  <span class="photofancy floatl mr20"><img src="/assets/jack.png" alt="Jack Has Many Things" width="320" height="214" /></span> I am Jack's sofa, stereo and wardrobe... I make Jack's life complete. I reside in a ActiveRecord table called "things" and Jack is the only one that has the key. This is Jack's life, and it's ending one minute at a time.
</p>
  
<p>
  As rails developers, we have done this simple relationship over and over again. I'm sure the has_many association is by far the most common in app/db design. It gives a single resource quick and easy access to others, but as your application grows, and depression sets in, we have to open up.
</p>


<h2>It's cheaper than a movie, and there's free coffee.</h2>

<p>
  <span class="photofancy floatr ml20"><img src="/assets/bob.png" alt="Jack With Bob" width="148" height="174" /></span> I am talking about groups. Not the underground ones that carve us out of wood, but ones where we share with those around us. This is healing. The time has come to for our objects to do the same, but how?
</p>

<p>
  The problem is that the ActiveRecord has_many association is scoped to an individual. No matter what conditions are tacked on, they are always <a href="http://en.wikipedia.org/wiki/Pwn">pwned</a> by the proxy owner. The things they own have ended up owning you! Sure we could model some groupable schema and go through it, ActiveRecord is beautiful that way. But what about our hard work in all those existing has_many associations and scopes? 
</p>


<h2>I felt like destroying something beautiful.</h2>

<p>
  The solution is on everyone's face, it is on the tip of everyone's tongue. I just gave it a name. It is called <a href="http://github.com/metaskills/grouped_scope/tree/master">GroupedScope</a> and it will fundamentally change the constructed SQL for the has_many associations you want to share. The best part, it will leave those associations untouched for continued use. GroupedScope even works with your existing association extensions and any scopes. Let's have a session.
</p> 

<p>First we need to install the gem. So bundle up in your Gemfile.</p>

~~~ruby
gem 'grouped_scope', '~> 3.1.0'
~~~

<p>Now let's open up our people schema and the Person model, so Jack can share. First we add a group_id column to the people table.</p>

~~~ruby
class PeopleCanChooseAGroup < ActiveRecord::Migration
  def up
    add_column :people, :group_id, :integer
  end
  def down
    remove_column :people, :group_id
  end
end
~~~

<p>Next we declare grouped_scope on a few associations.</p>

~~~ruby
class Person < ActiveRecord::Base
  has_many :things
  has_many :acquaintances
  has_many :problems
  grouped_scope :things, :problems
end

@jack = Person.find_by_name('Jack')
@bob  = Person.find_by_name('Bob')

@jack.update_attribute :group_id, 1
@bob.update_attribute :group_id, 1
~~~

<p>
  So now every Person object in our app is now ready to share their <code>:things</code> and their <code>:problems</code>. I have also just arbitrarily put Jack and Bob into the same group. Declaring <code>grouped_scope</code> in the model generates a new <code>group</code> instance method that will allow us to either reflect on the group or delegate to the associations we have declared as now haveing grouped scope.
</p>

~~~ruby
@jack.group   # => [#<Person id: 1, name: "Jack", group_id: 1>, #<Person id: 2, name: "Bob", group_id: 1>]
~~~


<h2>We all started seeing things differently.</h2>

<p>
  The object returned by the group method, is an instance of <code>GroupedScope::SelfGrouping</code>. It is far cooler than you think. It looks and acts as an enumerable array, but in reality it is a <code>ActiveRecord::Relation</code> object that can delegate to generated grouped association reflections which mimic their originals. Essentially giving the group access to all associated objects of its members, in this case their <code>:things</code> and <code>:problems</code>. Did I loose you? 
</p>

~~~ruby
@jack.problems.size   # => 6
@bob.problems.size    # => 2

@bob.group.problems        # => [#<Problem...>,#<Problem...>,#<Problem...>,#<Problem...>,....]
@bob.group.problems.size   # => 8
@jack.group.problems.size  # => 8
~~~

<p>
  Without going into the detail of Jack's and Bob's problems, we can see that within the group, they are all shared. This is what the GroupedScope gem is really all about. It allows existing has_many associations to be called on the group which changes the SQL generated to be owned by the group, essentially from <code>id = 1</code> to <code>id IN (SELECT id FROM people WHERE group_id = 1)</code>.
</p>

<p>
  The way it accomplishes this is pretty sweet. GroupedScope creates reflections that use a custom association scope method which uses a little Arel magic to build predicate conditions. Since it copies all your existing association reflection options, it can be really smart by maintaining all the logic in the existing association. So options like <code>:class_name</code>, <code>:foreign_key</code>, <code>:though</code>, and <code>:extend</code> will just work! It also lets you chain scopes existing scopes or use your custom association extensions on the original associations. Here is very contrived example:
</p>

~~~ruby
class Person < ActiveRecord::Base
  has_many :mental_issues, :class_name => "MentalState", :foreign_key => :name do
    def dangerous
      where(:snap_tolerance => 10)
    end
  end
  grouped_scope :mental_issues
end

class MentalState < ActiveRecord::Base
  scope :treatable_by, lambda { |doctor| where(:doctor_id = doctor.id) }
end

@jack.group.mental_issues.dangerous.treatable_by(@doctor) # => [#<MentalState...>,#<MentalState...>]
~~~


<h2>This is probably one of those "cry for help" things.</h2>

<p>
  <span class="photofancy floatr ml20"><img src="/assets/marla.png" alt="Marla" width="320" height="214" /></span>The GroupedScope gem is never quite done. It is however well tested and can do exactly all that I have outlined in ActiveRecord 3.1.0. I even have older gem versions that track our legacy 2-3-stable git branch.
</p>

<p>
  Also, you may have noticed that <code>GroupedScope</code> does not try to solve what your group business logic may look like. This is intentional and left to the end user to implement a <code>Group</code> object. This object would be the primary key owner of the <code>group_id</code> column in your models that declare grouped scope. You would also need to declare some sort of <code>belongs_to</code> association to your custom group model too.
</p>

<p>
  Lastly, if we you see something you might like in features or notice a bug, open a issue <a href="https://github.com/metaskills/grouped_scope">on our Github project</a> page.
</p>


<h2>I'd like to thank the Academy.</h2>

<ul>
  <li><a href="http://github.com/metaskills/grouped_scope/tree/master">GroupedScope Gem On Github.</a></li>
  <li><a href="http://en.wikipedia.org/wiki/Fight_Club">About the movie Fight Club.</a></li>
  <li><a href="http://decisiv.net/">My Workplace, Decisiv.</a></li>
</ul>

