---
layout: post
title: Automating Heroku PG Backups
categories: 
  - ruby-rails
  - database
---

<p>
  On December 1st, Heroku <a href="http://blog.heroku.com/archives/2010/12/1/bundles-deprecation/">deprecated their bundles add-on</a> in favor of <a href="http://blog.heroku.com/archives/2010/11/16/pgbackups/">their new PG Backups</a>. Even though there are <a href="http://groups.google.com/group/heroku/browse_thread/thread/25402694098d393a">other solutions</a> for automating backups using this new add-on, none of them met my needs. I like to have a daily DB backup history, just in case you find something bad that happened "n" days earlier. Below is a simple rake task suitable to place in your rails <code>lib/tasks/heroku.task</code> file. I'll explain some things I learned below when writing this.
</p>

~~~ruby
require 'heroku'
require 'heroku/command'

HEROKU_BACKUP_BUCKET = "#{Heroku::Command::Base.selected_application}-backups"

namespace :heroku do
  
  desc "Use the `heroku pgbackups` with my S3 bucket."
  task :backup => :connect_to_s3 do
    info = capture_heroku_command 'pgbackups'
    if heroku_existing_backup?(info)
      last_backup_info = info.split("\n").last.split(" | ")
      last_backup_id = last_backup_info[0]
      last_backup_time = last_backup_info[1]
      puts "Deleting last backup - ID: #{last_backup_id} BackupTime: #{last_backup_time}"
      heroku_command 'pgbackups:destroy', last_backup_id
    end
    heroku_command 'pgbackups:capture'
    backup_url = capture_heroku_command 'pgbackups:url'
    backup_filename = "#{Heroku::Command::Base.selected_application}_#{Time.now.xmlschema}.dump"
    backup_data = Net::HTTP.get_response(URI.parse(backup_url)).body
    AWS::S3::S3Object.store backup_filename, backup_data, HEROKU_BACKUP_BUCKET
  end
  
  task :connect_to_s3 do
    AWS::S3::Base.establish_connection!(
      :access_key_id     => ENV['AMAZON_ACCESS_KEY_ID'],
      :secret_access_key => ENV['AMAZON_SECRET_ACCESS_KEY'])
    begin
      AWS::S3::Bucket.find(HEROKU_BACKUP_BUCKET)
    rescue AWS::S3::NoSuchBucket
      AWS::S3::Bucket.create(HEROKU_BACKUP_BUCKET)
    end
  end
  
end

private

def heroku_command(*cmds)
  Heroku::Command::Base.command(*cmds)
end

def capture_heroku_command(*cmds)
  stdout = STDOUT
  StringIO.new.tap do |out|
    def out.flush ; end
    $stdout = out
    heroku_command(*cmds)
  end.string.chomp
ensure
  $stdout = stdout
end

def heroku_existing_backup?(info)
  info !~ /no backups/i
end
~~~


<p>
  Once installed you can run something like <code>bundle exec rake heroku:backup</code> or if you are not using bundler, <code>rake heroku:backup</code>. Assuming you have your S3 credentials setup in the ENV variables, it will find or create a private bucket on your S3 account and upload an app-named and time-stamped dump file to that new bucket. If necessary, it <strong>will delete your latest backup</strong> on Heroku to make room for this new one. The code should be easy to change, so flavor to taste.
</p>

<p>
  I tried to use the Heroku commands built into their plugin without resorting to command interpolation. The only problem was that the Heroku gem always wants to print to standard out and flush the buffer. So I created a few private helper methods that temporarily shim in a <code>$stdout</code> replacement that does not flush. This let's me run the Heroku commands from code and capture what would have been printed to standard out.
</p>

<p>
  Lastly, since I could not find a way to automate this on Heroku via their cron add-on, I simply added a <code>launchd</code> plist to my desktop Mac that hit a shell script in my project folder to run the rake task. It is way past my skills to try and get RVM to work in the <code>launchd.plist</code> system since it is not a true shell. This is why the shell script uses my system ruby (installed via MacPorts). Here is the shell script below and the launchd plist which I placed in <code>~/Library/LaunchAgents</code> with a name like <code>com.actionmoniker.backupMyApp.plist</code>. Just run <code>launchctl load ~/Library/LaunchAgents/com.actionmoniker.backupMyApp.plist</code> and this will run at 4am every morning. If any one finds out how to automate the execution of this rake task on Heroku, please drop me a line!
</p>

~~~bash
#! /bin/zsh
source /Users/kencollins/.zshenv
cd /Users/kencollins/repos/myapp && /opt/local/bin/rake heroku:backup
~~~

~~~html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.actionmoniker.backupMyApp</string>
  <key>ProgramArguments</key>
  <array>
    <string>/Users/kencollins/repos/myapp/lib/bin/backup.sh</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict>
    <key>Minute</key>
    <integer>0</integer>
    <key>Hour</key>
    <integer>4</integer>
  </dict>
</dict>
</plist>
~~~



