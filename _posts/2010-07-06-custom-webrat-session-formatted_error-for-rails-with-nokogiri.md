--- 
layout: post
title: Custom Webrat::Session formatted_error For Rails With Nokogiri
disqus_id: /2010/07/06/custom-webrat-session-formatted_error-for-rails-with-nokogiri/
categories: 
  - ruby-rails
---

<p>
  I never liked how Webrat shows the complete response body for exceptions when integration testing rails applications. For the longest time I redefined the formatted_error method to simple do nothing. Today I used Nokogiri to parse out the good bits from that page. Here is the result. Not pretty, but it's working fine so far. Got a better example? Please share!
</p>


~~~ruby
module Webrat
  class Session
    def formatted_error
      doc = Nokogiri::HTML(response_body)
      exception_name = doc.css('head title').inner_html.squish
      exception_msg = doc.css('body h1').inner_html.squish
      exception_detail1 = "".tap do |detail|
        d = doc.css('body p')[0]
        detail << d.content.strip
        detail << d.next_sibling.content.squish
      end
      exception_detail2 = "".tap do |detail|
        d = doc.css('body p')[1]
        detail << d.content.strip
        detail << d.next_sibling.css('code').first.content.strip
      end
      app_trace = doc.css('#Application-Trace pre code').inner_html
      [exception_name, exception_msg, exception_detail1, exception_detail2, app_trace].join("\n")
    rescue
      "Could not format page exception. Perhaps try to use Nokogiri on this: \n#{response_body}"
    end
  end
end
~~~

