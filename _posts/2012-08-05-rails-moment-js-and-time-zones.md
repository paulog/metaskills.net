---
layout: post
title: Rails, Moment.js And Time Zones
categories: 
  - ruby-rails
  - javascript
---


<p>
  Here are a few quick tips for the time zone aware Rails developer that finds themselves deep into JavaScript date objects. First, use the <a href="http://momentjs.com">Moment.js</a> JavaScrpt date library! Moment.js has a very <a href="http://momentjs.com/docs/">rich API for parsing and working with times</a>, very similiar to ActiveSupport's extensions. However, it does not have a solid way of moving times across zones. Especially if those zones may or may not observer daylight savings time (DST).
</p>

<p>
  Many JavaScript time zone libraries require a huge set of geographic data to both identify zones and their observance of DST. These data files can add a significant overhead to JavaScript. But wouldn't it be great if there was a simple way of leveraging your Rails model's time zone settings? There is, but first we need to serialize an <code>ActiveSupport::TimeZone</code> object in JSON. Easy, just define an <code>#as_json</code> method like the one below. I suggest adding this to an initializer in your Rails <code>config/initializers/active_support.rb</code> directory. The key attribute here is the <code>utc_total_offset</code>. This will be a number in minutes that properly observes DST.
</p>

```ruby
module ActiveSupport
  class TimeZone

    def as_json(options=nil)
      { :name                => name, 
        :identifier          => tzinfo.identifier,
        :friendly_identifier => tzinfo.friendly_identifier,
        :utc_offset          => utc_offset,
        :utc_total_offset    => tzinfo.current_period.utc_total_offset }
    end

  end
end
```

<p>
  Now assuming you have serialized a models time zone attribute using the full <code>ActiveSupport::TimeZone</code> object, we can easily use this information client side via a quick extension to Moment.js' prototype. Here is a <code>moment.js.coffee</code> file I have required in my Rails applications.
</p>

```coffeescript
moment.fn.forTimeZone = (timeZone) ->
  currentOffset = (this.zone() * 60) * -1
  adjustedOfffset = if currentOffset > timeZone.utc_total_offset 
                      timeZone.utc_total_offset - currentOffset 
                    else 
                      currentOffset - timeZone.utc_total_offset
  this.clone().add 'seconds', adjustedOfffset
```

<p>
  So we let Ruby do all the hard work of telling us what time zones are observing DST without all the bloat to our JavaScript for parsing zone identifiers. Some sample output.
</p>

```javascript
eastern = {
  friendly_identifier: "America - New York",
  identifier: "America/New_York",
  name: "Eastern Time (US & Canada)",
  utc_offset: -18000,
  utc_total_offset: -14400
}

noonPacific = 1344279600000 // "2012-08-06T12:00:00-07:00"
format = 'MMMM Do, YYYY \\at h:mma'

moment(noonPacific).format(format)
// "August 6th, 2012 at 12:00pm"

moment(noonPacific).forTimeZone(eastern).format(format)
// "August 6th, 2012 at 3:00pm"
```


<h2>Resources</h2>

<ul>
  <li><a href="http://momentjs.com">Moment.js - A lightweight JavaScript date library.</a></li>
</ul>

