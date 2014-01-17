--- 
layout: post
title: Rake task to read and write the ActiveRecord schema version table.
disqus_id: /2007/01/31/rake-task-to-read-and-write-the-activerecord-schema-version-table/
categories: 
  - ruby-rails
  - workflow
---


<p>
  After attending Rails Edge in Reston, Virginia I decided to move some common tasks from my <code>~/.irbrc</code> file and put them into Rake. I thought I would share a task that reads and writes the ActiveRecord schema table. Sometimes in migrations this is either good to know or manually change. Simply copy this in a foo.task file in your project/lib/tasks directory and use <code>rake -T</code> to see the description and usage. I have placed these tasks in the db:version namespace.
</p> 

~~~ruby
namespace :db do
  namespace :version do 

    desc "Read the current version of the database."
    task :read => :environment do
      puts "The #{RAILS_ENV}' database version is: #{ActiveRecord::Migrator.current_version}"
    end
    
    desc "Manually set the schema version to a specific target version with VERSION=x"
    task :write => :environment do
      if ENV['VERSION']
        if ActiveRecord::Base.connection.update("UPDATE #{ActiveRecord::Migrator.schema_info_table_name} SET version = #{ENV['VERSION'].to_i}")
          puts "SUCCESS: The '#{RAILS_ENV}' database version is now: #{ActiveRecord::Migrator.current_version}"
        else
          puts "FAILED: The '#{RAILS_ENV}' database version is still: #{ActiveRecord::Migrator.current_version}"
        end
      else
        puts 'You must specify a VERSION=n argument to this command. Use rake db:version:read to get the current version.'
      end
    end
    
  end  
end
~~~

