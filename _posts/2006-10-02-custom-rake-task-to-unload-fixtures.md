--- 
layout: post
title: Custom Rake Task To Unload Fixtures
disqus_id: /2006/10/02/custom-rake-task-to-unload-fixtures/
categories: 
  - ruby-rails
  - workflow
---

<p>
  I made a revised version of a rake task that I have used quite often, for unloading existing DB tables into fixture data and thought I would share. This rake task is the in correct name space and adds a "rake db:fixtures:unload" command to your rails project when you put this in "lib/tasks/foo.rake". It can take an optional TABLES variable or if none is present the whole array of DB tables are converted. I find this rake task helpful when dealing with LARGE databases.
</p> 

```ruby
namespace :db do
  namespace :fixtures do
    desc 'Create YAML test fixtures from data in specifed tables. Set table names by TABLES=foos,bars,etc'
    task :unload => :environment do
      sql  = "SELECT * FROM %s"
      tables = ["schema_info"]
      ActiveRecord::Base.establish_connection
      (ENV['TABLES'] ? ENV['TABLES'].split(/,/) : ActiveRecord::Base.connection.tables).each do |table_name|
        i = "000"
        File.open("#{RAILS_ROOT}/test/fixtures/#{table_name}.yml", 'w') do |file|
          data = ActiveRecord::Base.connection.select_all(sql % table_name)
          file.write data.inject({}) { |hash, record|
            hash["#{table_name}_#{i.succ!}"] = record
            hash
          }.to_yaml
        end
      end
    end
  end
end
```

