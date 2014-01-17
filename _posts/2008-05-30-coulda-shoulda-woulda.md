--- 
layout: post
title: Coulda Shoulda Woulda
disqus_id: /2008/05/30/coulda-shoulda-woulda/
categories: 
  - heuristics
  - ruby-rails
  - workflow
---

<p>
  It has been about 6 months now since I started using the <a href="http://www.thoughtbot.com/projects/shoulda">Shoulda</a> testing plugin as my BDD/TDD tool of choice. Unlike a lot of other people, I did not flock to the RSpec bandwaggon. Personally I think RSpec is horribly bloated a sledgehammer for a simple issue, the need to have test code organized with nested setups and context blocks.
</p>

<p>
  If you are new to Shoulda, I highly urge you to take a look at the <a href="http://www.thoughtbot.com/projects/shoulda">Thoughbot project page</a> for it. If you have the time take a look at some of their other projects, like Paperclip. These guys are really smart. If you really want to see how shoulda can sing, check out <a href="http://mwrc2008.confreaks.com/12saleh.html">Tammer Saleh's presentation of it at the MountainWest RubyConf 2008</a>. In that presentation he covers how you can extend Shoulda and write your own macros that generate additional tests. This is what my blog post is about.
</p>


<h2>Sample Shoulda Macro For Functional Require Login</h2>

<p>
  Here is a simple module that you might find for any restful authentication system. This would be included in your main <code>test_helper.rb</code> file where it would add the normal login_as(user) method. The good part I want to point out is the <code>should_require_login(*actions)</code> macro method that shows off a neat example of how you could use Shoulda in your functional tests at the class level.
</p>

~~~ruby
module AuthSystem
  module TestHelper
    
    def self.included(receiver)
      receiver.extend ClassMethods
      receiver.send :include, InstanceMethods
    end
    
    module ClassMethods
      
      def should_require_login(*actions)
        actions.each do |action|
          should "Require login for '#{action}' action" do
            get(action)
            assert_redirected_to(login_url)
          end
        end
      end
      
    end
    
    module InstanceMethods
      
      def login_as(user)
        if u = users(user)
          @request.session[:user_id] = u.id
        end
      end
      
    end
    
  end
end
~~~

<p>
  Here is how it would look in the controller functional test. These are all contrived examples, but I think it illustrates how Shoulda can be used and in general how you can make your own macros that test at a higher level.
</p>

~~~ruby
class UsersControllerTest < ActionController::TestCase
  
  should_require_login :edit, :update, :etc
  
end
~~~




