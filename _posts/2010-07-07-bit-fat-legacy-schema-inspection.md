--- 
layout: post
title: Bit Fat Legacy Schema Inspection
disqus_id: /2010/07/07/bit-fat-legacy-schema-inspection/
categories: 
  - ruby-rails
---

<p>
  Sometime in rails 2.x the #inspect method for an ActiveRecord class was changed to show you all the column names (attributes) of that class. This is fine when things are small but if your working on a big legacy schema and you want clean terse debugging, all those column names can be noisy. I just set this initializer up today to kill it. Now the class just shows the number of columns.
</p>

~~~ruby
module ActiveRecord
  class Base
    class << self
      
      def inspect
        if self == Base
          super
        elsif abstract_class?
          "#{super}(abstract)"
        elsif table_exists?
          "#{super}(#{columns.count} columns)"
        else
          "#{super}(Table doesn't exist)"
        end
      end
      
    end
  end
end
~~~

