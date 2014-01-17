---
layout: post
title: Simple Memcached Reports In Rails
categories: 
  - ruby-rails
---

<p>
  <span class="photofancy floatr ml20">
    <img src="/assets/memcd.jpg" alt="Time Machine Exclude Window" width="152" height="182" />
  </span>
  The Pragmatic Bookshelf's <a href="http://www.pragprog.com/titles/memcd/using-memcached"><em>Using memcached: How to scale your website easily</em></a> is way old. But then again, memcached is still very useful. One of the things I remember reading a long time ago was the stats for your process and how to pull out meaningful reports like a hit ratio, memory and a get to set ration.
</p>

<p>
  I whipped these up today as a simple way of getting a report using <code>Rails.cache.report</code>, assuming your cache strategy is set to memcached. Hope others find it useful. BTW, I do not recommend the book, it is too low level for any useful meaning to the average Rails developer.
</p>

<div class="h20"></div>

~~~ruby
module ActiveSupport
  module Cache
    class MemCacheStore < Store
      
      include ActionView::Helpers::DateHelper
      include ActionView::Helpers::NumberHelper

      # Higher percentages, above 90%, are good.
      def hit_ratio
        stats.inject({}) do |memo, (server,stat)|
          value = (stat['get_hits'].to_f / stat['cmd_get'].to_f) * 100
          memo[server] = number_to_percentage(value,:precision=>2)
          memo
        end
      end

      # Another good statistic to look at is percentage of gets to sets. For a well-tuned application, 
      # there should be more gets than sets. High percents are bad, negatives are great.
      def gets_to_sets
        stats.inject({}) do |memo, (server,stat)|
          sets = stat['cmd_set'].to_f
          gets = stat['cmd_get'].to_f
          more_sets = sets - gets
          value = (more_sets / gets) * 100
          memo[server] = number_to_percentage(value,:precision=>2)
          memo
        end
      end

      def uptime
        stats.inject({}) do |memo, (server,stat)|
          started_on = stat['uptime'].seconds.ago
          utime = distance_of_time_in_words started_on, Time.now
          memo[server] = {:started_on => started_on, :uptime => utime}
          memo
        end
      end

      def memory
        stats.inject({}) do |memo, (server,stat)|
          memory = number_to_human_size stat['limit_maxbytes']
          used = number_to_human_size stat['bytes']
          memo[server] = {:memory_total => memory, :memory_used => used}
          memo
        end
      end
      
      def report
        stats.keys.inject({}) do |memo, server|
          rpt = memory[server].merge(uptime[server])
          rpt[:hit_ratio] = hit_ratio[server]
          rpt[:gets_to_sets] = gets_to_sets[server]
          memo[server] = rpt
          memo
        end
      end
      
    end
  end
end
~~~

