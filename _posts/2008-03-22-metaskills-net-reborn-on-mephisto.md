--- 
layout: post
title: MetaSkills.net Reborn on Mephisto
disqus_id: /2008/03/22/metaskills-net-reborn-on-mephisto/
categories: 
  - heuristics
  - ruby-rails
---

<p><img src="/assets/drax.png" alt="Drax" class="floatr ml20" /> Well after a year of neglect, the MetaSkills.net blog has been <a href="/2008/03/22/metaskills-net-reborn-on-mephisto/">reborn on Mephisto</a>. Previously I was using Drupal and it finally got to a point where I was so deep into ruby that I did not even have the gumption to open up a PHP session to publish anything. The sad part is that I told myself that this PHP disdain would help me get off my butt and move to Mephisto. You know, eat my own dog food â€“ obviously procrastination won out â€“ but not forever. For the past week I worked hard to get the Meta theme for Drupal converted to Mephisto. You can use this theme yourself if you want, the source is available on <a href="http://github.com/metaskills/metatheme/tree/master">my github</a> and I am making updates often. Heck... feel free to fork the project and make some changes or let me know if you want me to incorporate them into mine.</p> 

<p>Here are a few things that I liked about rewriting the Meta theme for Mephisto. First, unlike Drupal, the administration of your Mephisto blog is not inline, but all tucked away in a private admin section. When you make a theme in Drupal, you are burdened to to the task of coding all the admin CSS for the inline admin features. The Meta theme had over 500 lines of CSS just for the Drupal administration. That is all gone and I love keeping theme code focused on doing nothing but presenting the "user" experience.</p>

<p>Also, when I first wrote the Meta theme I was just a JavaScript beginner. Nowadays I am pretty good at it and have moved away from the simple script like functions that pass arguments around into a full OO style that spawns from smart classes and a persistent state. The Meta theme is now 100% on Prototype and all the interactive features are tightly knit into the MetaTools, MetaSearch, and MetaContent classes. Take a look at the meta.js source if you want. Lastly, if you are on an old Drupal blog looking to get over to Mephisto, maybe this migration script will help.</p>


<h2>Migration from Drupal to Mephisto</h2>

<p>This script is what I used in the rails console to populate data from my old Drupal 4.7 blog to Mephisto. This code is untested on the schema of higher versions of Drupal.</p>

```ruby
# My Drupal 4.7 to Mephisto Script
# ----------------------------------------------------------------
# Things I did before
#   * Delete all rows from drupal node table where type != 'blog'
#   * Remove the type column from the drupal node table
# 
# Things I did afterward
#   * Found all comments in the Mephisto contents table that were 
#     from me and added user_id 1 to them.
# 
# Uncomment these if you are debugging the script and want to 
# start off on a clean mephisto install
# ----------------------------------------------------------------
# Article.find(:all).each(&:destroy)
# CachedPage.delete_all
# AssignedSection.delete_all
# Tagging.delete_all
# Tag.delete_all

@mysite = "http://www.metaskills.net/"
@home = Section.find_by_name('Home')

class DrupalArticle < ActiveRecord::Base
  establish_connection :drupal
  set_table_name  :node
  set_primary_key :nid
  has_many :comments, :class_name => 'DrupalComment', :foreign_key => 'nid'
  has_one :version, :class_name => 'DrupalArticleVerson', :foreign_key => 'nid'
  def created_at ; Time.at(self[:created]) ; end
end
class DrupalArticleVerson < ActiveRecord::Base
  establish_connection :drupal
  set_table_name  :node_revisions
  set_primary_key :vid
end
class DrupalComment < ActiveRecord::Base
  establish_connection :drupal
  set_table_name  :comments
  set_primary_key :cid
  belongs_to :article, :class_name => 'DrupalArticle', :foreign_key => 'nid'
end

DrupalArticle.find(:all).each do |da|
  # Creating the article
  na = Article.new
  na.site_id      = 1
  na.created_at   = da.created_at
  na.published_at = da.created_at
  na.updated_at   = Time.now
  na.title        = da.title
  na.body         = da.version.body
  na.excerpt      = da.version.teaser
  na.updater_id   = 1
  na.user_id      = 1
  na.save!
  na.sections << @home
  # Creating comments for this article
  da.comments.each do |dac|
    nac = na.comments.build
    nac.site_id       = 1
    nac.created_at    = Time.at(dac.timestamp)
    nac.published_at  = Time.at(dac.timestamp)
    nac.updated_at    = Time.at(dac.timestamp)
    nac.author        = dac.name
    nac.author_url    = dac.homepage
    nac.author_email  = dac.mail
    nac.author_ip     = dac.hostname
    nac.user_agent    = "Mozilla/5.0 (Macintosh; U; PPC Mac OS X; en) AppleWebKit/523.12.2 (KHTML, like Gecko) Version/3.0.4 Safari/523.12.2"
    nac.referrer      = @mysite
    nac.approved      = true
    nac.title         = dac.subject
    nac.body          = dac.comment
    nac.save!
  end
end
```

