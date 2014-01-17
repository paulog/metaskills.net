--- 
layout: post
title: How To Stop Delivery Of An Email Within An ActionMailer Method
disqus_id: /2008/04/03/how-to-stop-delivery-of-an-email-within-an-actionmailer-method/
categories: 
  - heuristics
  - ruby-rails
---


<p>
  OK, so you want to keep your code placement REALLY organized. You have <a href="/2008/03/26/don-t-be-a-plinko-programmer/">read about my persnicketyness</a> and now you want to practice the best in concern placement and keep those controllers of yours really slim. Like me, you may want to try and keep controller feature additions to very specific one liners of code. Organizing your controller code to do just that with ActiveRecord models or even your own custom classes is a pretty easy task, but how do you keep things simple when dealing with controller actions that have to send email AND you want that single email link of code to be responsible for everything in it's own encapsulated way.
</p>

<p>
  The answer is to push the logic back to the model again. Your controllers should not be concerned with the logic that deals with your own business rules for sending email. The problem with ActionMailer is that most people just use the dynamic <code>MyMailerClass.deliver_</code> methods that utilize <code>method_missing</code> to instantiate a mailer classs, and send an email in one fell swoop. I like this usage too, but the challenge is how to tell an instance of a the ActionMailer::Base class that it needs to stop delivery within a method definition that is all set to deliver? The answer is Ruby singleton method magic.
</p> 

~~~ruby
module ActionMailer
  class Base
    
    # A simple way to short circuit the delivery of an email from within
    # deliver_* methods defined in ActionMailer::Base subclases.
    def do_not_deliver!
      def self.deliver! ; false ; end
    end
    
  end
end
~~~

<p>
  I suggest placing this code into your <code>lib/core_ext/action_mailer.rb</code> file. It will allow you to write mailer methods that can short circuit themselves to stop delivery. For instance:
</p>

~~~ruby
class MyMailerClass < ActionMailer::Base
  def user_notification(user)
    # ... 
    do_not_deliver! if user.email.blank?
  end
end
# In your controller code.
MyMailerClass.deliver_user_notification(@user)
~~~

<p>
  Now you do not have to worry about placing delivery concerns in the controller and even worse duplicate that code when you have to use the same mailer method in multiple places. This works by letting the instance of the ActionMailer::Base class define it's own deliver! method which trumps the delier! method called by that object normally defined in ActionMailer::Base.
</p>


<h2>Resources</h2>

<ul>
  <li><a href="http://www.ruby-doc.org/docs/UsersGuide/rg/singletonmethods.html">About Singleton Methods</a></li>
  <li><a href="http://redhanded.hobix.com/inspect/methodsThatSelfDestruct.html">Methods That Self-Destruct</a></li>
</ul>



