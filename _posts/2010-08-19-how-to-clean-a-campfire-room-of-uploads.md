--- 
layout: post
title: How To Clean A Campfire Room Of Uploads
disqus_id: /2010/08/19/how-to-clean-a-campfire-room-of-uploads/
categories: 
  - workflow
  - ruby-rails
---

<p>For us at work, our uploads to campfire are really transitory. Most of the time they are simple screenshots around a current topic. Every now and then vacation photos or even movies, at the end of the day, none of it has value after a certain amount of time. To us as the real value of campfire is our textual transcripts.</p>

<p>This morning after 4 years of campfire, we were real close to our 1GB limit of uploads. Time for a clean up. I found no easy way of automating this, so I turned to the Tinder gem. It has a nice interface to the campfire API using HTTParty as the backend. I found out that there was no easy way to delete uploads too. So after some good ole fashion DOM inspection and knowing rails application conventions, I found my own interface. Below is a little script I used to clean up our room this morning. It basically loops thru a rooms uploads, 5 at a time, and deletes them. Pausing for a quarter of a second between each so I don't freak out the new 37signals administrator :)</p>


```ruby
require 'rubygems'
require 'tinder'

class CampfireUploadCleaner
  
  CF_DOMAIN = 'mydomain'
  CF_ROOM   = 'My Room Name'
  CF_TOKEN  = ENV['MY_CF_TOKEN']
  
  def initialize
    @campfire = Tinder::Campfire.new CF_DOMAIN, :token => CF_TOKEN
  end
  
  def room
    @room ||= @campfire.find_room_by_name(CF_ROOM)
  end
  
  def connection
    room.send(:connection)
  end
  
  def delete_uploads
    uploads = room.send(:get,:uploads)['uploads']
    uploads.each do |upload|
      id = upload['id']
      name = upload['name']
      connection.post "/uploads/delete/#{id}?n=0"
      puts "Deleted: [#{id}] #{name}"
      sleep(0.25)
    end
  end
  
  def sweep_uploads
    while room.files.present?
      delete_uploads
    end
  end
  
end

cleaner = CampfireUploadCleaner.new

cleaner.delete_uploads  # => Deletes the top 5 uploads.
cleaner.sweep_uploads   # => Deletes all uploads.
```


<p>The upload hash actually contains much more than just the id and name of the upload. There is a timestamp, filetype and other attributes. So if you wanted to extend this script, you could. I did not spent a lot of time with it, but I never figured out how to get more than the top 5 uploads too. I'm sure some param hacking would yield some good results.</p>
