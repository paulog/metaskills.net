---
layout: post
title: Use Compass Sass Framework Files With The Rails 3.1.0.rc5 Asset Pipeline
categories: 
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: Now that Rails 3.1 is out, just use the <a href="http://rubygems.org/gems/compass">latest compass</a> or pre-release. No hacks needed!
</aside>

<p>
  This is a simple update to my <a href="/2011/05/18/use-compass-sass-framework-files-with-the-rails-3.1-asset-pipeline/">original article</a> for using compass's <code>.scss</code> files with the asset pipeline. This assumes you had this setup working in Rails 3.1.0.rc4, but should be helpful to anybody. The good news is that most everything is wired up.
</p>

<p>
  First, you are going to need to update your Gemfile. Follow the latest conventions and make sure you have a <code>:assets</code> group. Use the example below. With the exception of the compass addition to their "rails31" branch, this is what a Rails 3.1.0.rc5 project would generate. It is a good idea to put all your asset pipeline related gems in this group. As the comment says, these would not be required for production as the assumption is would precompile your static assets on deploys. Note, make sure your bundle for sass-rails installs the matching rc5 too.
</p>


~~~ruby
# Gems used only for assets and not required
# in production environments by default.
group :assets do
  gem 'sass-rails', "~> 3.1.0.rc"
  gem 'coffee-rails', "~> 3.1.0.rc"
  gem 'uglifier'
  gem 'compass', :git => 'git://github.com/chriseppstein/compass.git', :branch => 'rails31'
end
~~~

<p>
  Now here is a crazy gotcha. Rails 3.1.0.rc5 has changed the application.rb bundler line a little to account for this new :assets group. Find the bundler line and make sure it looks like this.
</p>

~~~ruby
Bundler.require *Rails.groups(:assets) if defined?(Bundler)
~~~

<p>
  Lastly, we need to get the compass files in the the load path. The sass-rails gem has set us up for this. I choose to hook into it using my <code>config/initializers/sass.rb</code> like so. I imagine that compass will be doing this on its own at some point in the future too. After this you should be all set!
</p>

~~~ruby
Rails.configuration.sass.tap do |config|
  config.load_paths << "#{Gem.loaded_specs['compass'].full_gem_path}/frameworks/compass/stylesheets"
end
~~~

