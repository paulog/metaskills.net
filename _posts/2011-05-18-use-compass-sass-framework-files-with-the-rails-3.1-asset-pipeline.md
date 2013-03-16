---
layout: post
title: Use Compass Sass Framework Files With The Rails 3.1 Asset Pipeline
categories: 
  - ruby-rails
---

<aside class="flash_info">
  UPDATE: Now that Rails 3.1 is out, just use the <a href="http://rubygems.org/gems/compass">latest compass</a> or pre-release. No hacks needed!
</aside>

<p>
  The Sprockets 2 gem along with the Tilt gem make it really easy to write JavaScript or CSS using any templating language you desire. The rails defaults are CoffeeScript and Sass. About the <a href="https://github.com/chriseppstein/compass/tree/stable/frameworks/compass/stylesheets">best collection of Sass framework files</a> for easy cross-browser CSS authoring are packaged in the compass framework. Compass even has <a href="http://compass-style.org/reference/compass/css3/">great documentation</a> for using their Sass framework. But what if you want to use those within the asset pipeline provided by Rails? Easy enough!
</p>

<p>
  First, bundle up the compass, but do not require it. Add this to your <code>Gemfile</code>.
</p>

```ruby
gem 'compass', :require => false
```

<p>
  Next, add a Sass initializer in <code>config/initializers/sass.rb</code> and fill it in with the code below. This will add two more load paths to the Sass engine. The first is your default rails/sprockets asset path for stylesheets. It simply let's you build a deep folder structure in that directory and use relative paths from each file. The second will put the entire compass Sass framework files into the Sass load path.
</p>

```ruby
Sass::Engine::DEFAULT_OPTIONS[:load_paths].tap do |load_paths|
  load_paths << "#{Rails.root}/app/assets/stylesheets"
  load_paths << "#{Gem.loaded_specs['compass'].full_gem_path}/frameworks/compass/stylesheets"
end
```

<p>
  Now in your rails <code>app/assets/stylesheets/foo.scss</code> file you can use Sass' <code>@import</code> with paths to the compass framework.
</p>

```css
@import "compass/css3/opacity";
#mylogo { @include opacity(0.5); }
```

<p>
  That is an example loading up the opacity helpers. Your generated css file will look like this! CSS is never going to be the same again!
</p>

```css
#mylogo {
  -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=50)";
  filter: progid:DXImageTransform.Microsoft.Alpha(Opacity=50);
  opacity: 0.5; }
```




