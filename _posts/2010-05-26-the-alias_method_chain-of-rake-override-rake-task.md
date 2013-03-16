--- 
layout: post
title: The alias_method_chain of Rake - Override Rake Task 
disqus_id: /2010/05/26/the-alias_method_chain-of-rake-override-rake-task/
categories: 
  - heuristics
  - ruby-rails
  - workflow
---


<p>
  Rake is cool. It is built so that multiple tasks with the same name run in a reverse defined series. This is great, but sometimes you want to override a task with your own behavior and conditionally call the earlier task. Especially if that task is defined deep somewhere else, like in a rails gem. I have had to solve this problem before in Rake. Awhile back I hacked something up that would totally trump a predefined rake task and allow you to replace it with a new one. Lately while <a href="http://wiki.github.com/rails-sqlserver/2000-2005-adapter/rails-db-rake-tasks">working on the SQL Server adapter</a>, I had a need to method chain some core rails :db namespaced tasks. So once again I googled others work and again resorted to hacking Rake. Below is what I was left with.
</p>

```ruby
Rake::TaskManager.class_eval do
  def alias_task(fq_name)
    new_name = "#{fq_name}:original"
    @tasks[new_name] = @tasks.delete(fq_name)
  end
end

def alias_task(fq_name)
  Rake.application.alias_task(fq_name)
end

def override_task(*args, &block)
  name, params, deps = Rake.application.resolve_args(args.dup)
  fq_name = Rake.application.instance_variable_get(:@scope).dup.push(name).join(':')
  alias_task(fq_name)
  Rake::Task.define_task(*args, &block)
end
```

<p>
  It's easy to use, just require it in your Rakefile. In the example below, I was able to programmatically method chain the core rails db:test:purge and only call the original if I needed to. Very cool!
</p>

```ruby
namespace :db do
  namespace :test do
    override_task :purge => :environment do
      ...
      # To invoke the original task add ":original" to its name
      Rake::Task["db:test:purge:original"].execute
      ...
    end
  end
end
```

<p>
  Lastly, thanks to <a href="http://www.taknado.com/">Eugene Bolshakov</a>, <a href="http://github.com/jwood">John Wood</a>, and <a href="http://github.com/markwfoster">Mark Foster</a> whom have tackled this rake problem before. My version above was based on their work, but correctly works with namespaces which was critical for my needs.
</p>

