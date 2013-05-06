--- 
layout: post
title: Authenticated S3 GETs For Private Objects Using Paperclip
disqus_id: /2009/11/23/authenticated-s3-gets-for-private-objects-using-paperclip/
lastmod: 2011-10-18T12:00:00Z
categories: 
  - heuristics
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: [10/18/2011] I now use Fog vs AWS::S3 with Paperclip. I have included a Fog specific example below.
</aside>

<p>
  Yea I know, I am probably the last person on earth that is just getting around to using <a href="http://github.com/thoughtbot/paperclip">Paperclip</a>. To be honest, most of my file upload code was written way before Paperclip or even AttachementFu was ever conceived. And frankly, I do not do much social app coding on the side - so the need never came up. But that changed recently and I wanted a really really good way of leveraging AWS::S3 storage with the best local app security while maintaining tight control over the files.
</p>

<p>
  So the Paperclip wiki has a few links that already dealt with some ways of protecting your app's attachments. One mentions a method I totally love called security through obscurity. It uses a secure random token as part of the filename which combined with the original filename and the id partition makes for great random URLs. The other is a great walk thru on how to use the :url option of paperclip to point access control back to your own application for your normal biz logic.
</p>

<p> 
  The problem I see with both of these methods is that they do not allow you to maintain app control past the final URL handoff/redirect. It also requires that your S3 bucket is public. For instance, if you were to use the <code>s3_permissions => :private</code> option of Paperclip, then that URL given to you by Paperclip is pretty much worthless. I knew AWS::S3 had authenticated GETs that generated an automatically expiring URL, but saw no way of accessing its features using the abstract <code>Paperclip::Attachment</code> object. So this is what I did.
</p>

```ruby
class MyDownload
  
  has_attached_file :attachment, 
                    :storage => :s3, :bucket => 'mybucket',
                    :s3_credentials => {...}, :s3_protocol => 'https', 
                    :s3_permissions => :private,
                    :path => lambda { |attachment| ":id_partition/#{attachment.instance.random_secret}/:filename" },
                    :processors => [:noop]
  
  before_validation_on_create :set_random_secret
  
  
  def attachment_url
    "#{self.class.tableize}/#{id}/#{attachment_file_name}"
  end
  
  def authenticated_s3_get_url(options={})
    options.reverse_merge! :expires_in => 10.minutes, :use_ssl => true
    AWS::S3::S3Object.url_for attachment.path, attachment.options[:bucket], options
  end
  
  private
  
  def set_random_secret
    self.random_secret = ActiveSupport::SecureRandom.hex(8)
  end
  
end
```

<p>
  Let me walk you thru some of the highlights of that class, the general concept following is that we are going to use the best of both examples in security mentioned above. First, the secure token, that is what #set_random_secret will generate for each instance. The <code>:path</code> option for Paperclip uses a proc to make sure each instance uses that attribute in the string that will be later interpolated further down. You can also see how I use the id partition too. Next, I have added two public instance methods. The first is #attachment_url and it will need a bit of explaining.
</p>

<p>
  Currently in Paperclip, if you use the :url option and your :storage is set to :s3, then it is ignored. This could totally be intentional. So in a typical setup where you want the file download running through your own access control, you wold have a :url option like this <code>:url => '/:class/:id/:filename'</code>. So this is what #attachment_url mimics, it simply gets around that shortcoming and points the download action back to your own controller. How that controller would work is beyond the scope of this article, see the resources section below for those links.
</p>

<p>
  The last example method is #authenticated_s3_get_url which dips right on down to the AWS::S3 library to get the URL for the object in the bucket. AWS::S3's doc mention that it will automatically generate a secure GET url that expires in 5 minutes. However in my example, you can see where I am changing that to 10 minutes and forcing the HTTPS protocol. This would be the URL that your own controller would do the final redirect to. This URL is for your private objects in your S3 bucket and will only work for the amount of time you want it too! Meaning your app stays in complete control. Putting it all together one more time...
</p>

```ruby
# A MyDownload instance.
>> dl = MyDownload.find(4)

# This is totally useless for private buckets/objects.
>> dl.attachment.url
=> "https://s3.amazonaws.com/mybucket/000/000/004/147681c16fddc1e5/private.pdf?1258989107"

# This is what you use in your own views.
>> dl.attachment_url
=> "my_downloads/4/private.pdf"

# Your controller would redirect to this secure GET.
>> dl.authenticated_s3_get_url
=> "https://s3.amazonaws.com/mybucket/000/000/004/147681c16fddc1e5/private.pdf?AWSAccessKeyId=0HJD3NS9CVWN2JV89K02&Expires=1258990967&Signature=8aWsq4o5gXfpIrRZyeETddnOeFw%3D"
```


<h2>What Is That Noop Processor</h2>

<p>
  Good eye! Did you see that I have a processor called Noop in the has_attached_file declaration? The default processor in Paperclip is the Thumbnail processor, which no matter what calls the ImageMagick identify command to see if it can do something to the file. I did not want that or any processing, just simple attachments. So I created this simple processor that just straight returns the file object. I made a <a href="http://github.com/thoughtbot/paperclip/issues/#issue/118">ticket on the Paperclip's issue page</a> that hopefully would allow a <code>:processors => false</code> option one day that would do this as well. So maybe one day it'll be a feature.
</p>

```ruby
module Paperclip  
  class Noop < Processor
    
    def make
      file
    end

  end
end
```


<h2>UPDATE: Fog Example</h2>

<p>
  Here is an updated <code>#authenticated_s3_get_url</code> that I use with Fog as the backend storage for S3. This example illustrates 2 things. First that the bucket is named as your CNAME record which is also enforced in the <code>:fog_host</code>. I do this so that I can generate URLs via Paperclip or Fog and always have my custom domain in place. When I gsub the "s3.amazonaws.com" out, I am left with just my CNAME domain/bucket name. Lastly, if you want to generate "https" URLs, use the <code>#get_https_url</code> method.
</p>

```ruby
class MyDownload

  has_attached_file :attachment, 
                    :storage => :fog, 
                    :fog_credentials => {...},
                    :fog_directory => 'mybucket.domain.net',
                    :fog_public => false,
                    :fog_host => 'http://mybucket.domain.net',
                    :path => lambda { |attachment| ":id_partition/#{attachment.instance.random_secret}/:filename" }  
  
  def authenticated_s3_get_url(expires = nil)
    expires ||= 10.minutes.from_now
    url = attachment.send(:directory).files.get_http_url attachment.path, expires
    url.gsub! /s3.amazonaws.com\//, ''
  end
  
end
```


<h2>Resources</h2>

<ul>
  <li><a href="http://almosteffortless.com/2009/03/22/randomize-filename-in-paperclip/">Security Through Obscurity: Randomize Filename in Paperclip</a></li>
  <li><a href="http://thewebfellas.com/blog/2009/8/29/protecting-your-paperclip-downloads">Your Access Control: Protecting Your Paperclip Downloads</a></li>
  <li><a href="http://github.com/thoughtbot/paperclip/issues/#issue/118">Allow False Option For Processors</a></li>
</ul>




