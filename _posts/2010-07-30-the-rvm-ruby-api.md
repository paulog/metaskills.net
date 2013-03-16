--- 
layout: post
title: The RVM Ruby API - Setting Up A CI System For The SQL Server Adapter
disqus_id: /2010/07/30/the-rvm-ruby-api/
categories: 
  - apple-os
  - ruby-rails
---

<p>
  A few weeks ago I started looking into the <a href="http://rvm.beginrescueend.com/">Ruby Version Manager (RVM)</a> project to help me build a better testing setup for both my day job and the <a href="http://github.com/rails-sqlserver/activerecord-sqlserver-adapter">ActiveRecord SQL Server Adapter</a>. In a <a href="/2009/9/5/the-ultimate-os-x-snow-leopard-stack-for-rails-development-x86_64-macports-ruby-1-8-1-9-sql-server-more">previous article</a> I covered details of how to get a development stack up and running for Rails with SQL Server using MacPort's. This article will cover some new additions to that goal, but first and primarily, I wanted to talk about the wonders of RVM and it's new ruby API.
</p>
    
<p>
  So like any good agile software gardner tasked with building a continuous integration system, I wanted to do it in such a way that was completely automated using rake. My first cut at said rake task used ruby's Kernel.system to issue <code>rvm</code> commands down to the shell. This failed horribly! Basically no matter how hard I tried, I could not hit the rvm shell function from ruby's system command. It kept using the rvm binary which can not change the local shell environment and hence do very little magic that RVM allows. Thankfully <a href="http://twitter.com/wayneeseguin">@wayneeseguin</a> pointed me to the new RVM ruby API and <a href="http://blog.ninjahideout.com/posts/the-path-to-better-rvm-and-passenger-integration">this article on how to use it for passenger</a>. I immediately started to switch my rvm rake tasks to use the new RVM API and was just floored with how well it did. Below is a copy of that rake task. Take a look over it and read below for details and how I have used this with the SQL Server stack.
</p>
    
```ruby
MYPROJECT_RUBIES = {
  'ruby-1.8.6-p388'   => {:alias => 'myprj186', :odbc => '0.99991'},
  'ruby-1.8.7-p299'   => {:alias => 'myprj187', :odbc => '0.99991'},
  'ruby-1.9.1-p378'   => {:alias => 'myprj191', :odbc => '0.99991'},
  'ruby-1.9.2-head'   => {:alias => 'myprj192', :odbc => '0.99992pre3'},
  'ree-1.8.7-2010.02' => {:alias => 'myprjree', :odbc => '0.99991'}
}

namespace :rvm do

  task :setup do
    unless @rvm_setup
      rvm_lib_path = "#{`echo $rvm_path`.strip}/lib"
      $LOAD_PATH.unshift(rvm_lib_path) unless $LOAD_PATH.include?(rvm_lib_path)
      require 'rvm'
      require 'tmpdir'
      @rvm_setup = true
    end
  end

  namespace :install do

    task :all => [:setup,:rubies,:odbc,:gems]

    task :rubies => :setup do
      installed_rubies = RVM.list_strings
      MYPROJECT_RUBIES.keys.each do |rubie|
        if installed_rubies.include?(rubie)
          puts "info: Rubie #{rubie} already installed."
        else
          with_my_environment_vars do
            good_msg = "info: Rubie #{rubie} installed."
            bad_msg = "Failed #{rubie} install! Check RVM logs here: #{RVM.path}/log/#{rubie}"
            puts "info: Rubie #{rubie} installation inprogress. This could take awhile..."
            RVM.install(rubie,rvm_install_options) ? puts(good_msg) : abort(bad_msg)
          end
        end
        RVM.alias_create MYPROJECT_RUBIES[rubie][:alias], "#{rubie}@myproject"
      end
    end

    task :odbc => :setup do
      rvm_each_rubie do
        odbc = "ruby-odbc-#{myproject_current_rubie_info[:odbc]}"
        RVM.chdir(Dir.tmpdir) do
          RVM.run "rm -rf #{odbc}*"
          puts "info: RubyODBC downloading #{odbc}..."
          RVM.run "curl -O http://www.ch-werner.de/rubyodbc/#{odbc}.tar.gz"
          puts "info: RubyODBC extracting clean work directory..."
          RVM.run "tar -xf #{odbc}.tar.gz"
          RVM.chdir("#{odbc}/ext") do
            puts "info: RubyODBC configuring..."
            RVM.ruby 'extconf.rb', "--with-odbc-dir=#{rvm_odbc_dir}"
            puts "info: RubyODBC make and installing for #{rvm_current_name}..."
            RVM.run "make && make install"
          end
        end
      end
    end

    task :gems => :setup do
      puts "info: Installing our app gems."
      rvm_each_rubie do
        myproject_gem_specs.each { |spec| rvm_install_gem(spec) }
      end
    end

  end

  task :remove => :setup do
    myproject_rubies.each { |rubie| RVM.remove(rubie) }
  end

end


def myproject_rubies
  MYPROJECT_RUBIES.keys.map{ |rubie| "#{rubie}@myproject" }
end

def myproject_current_rubie_info
  MYPROJECT_RUBIES[rvm_current_rubie_name]
end

def myproject_gem_specs
  [
    ['rails','2.3.8'],
    ['activerecord-sqlserver-adapter','2.3.8'],
    ['erubis','2.6.6'],
    ['haml','3.0.13'],
    ['mocha','0.9.8'],
  ]
end

def rvm_each_rubie
  myproject_rubies.each do |rubie|
    RVM.use(rubie)
    yield
  end
ensure
  RVM.reset_current!
end

def rvm_current_rubie_name
  rvm_current_name.sub('@myproject','')
end

def rvm_current_name
  RVM.current.expanded_name
end

def rvm_gem_available?(spec)
  gem, version = spec
  RVM.ruby_eval("require 'rubygems' ; print Gem.available?('#{gem}','#{version}')").stdout == 'true'
end

def rvm_install_gem(spec)
  gem, version = spec
  if rvm_gem_available?(spec)
    puts "info: Gem #{gem}-#{version} already installed in #{rvm_current_name}."
  else
    puts "info: Installing gem #{gem}-#{version} in #{rvm_current_name}..."
    puts RVM.perform_set_operation(:gem,'install',gem,'-v',version).stdout
  end
end

def for_macports?
  `uname`.strip == 'Darwin' && `which port`.present?
end

def rvm_install_options
  {}
end

def my_environment_vars
  if for_macports?
    {'CC' => '/usr/bin/gcc-4.2',
     'CFLAGS' => '-O2 -arch x86_64',
     'LDFLAGS' => '-L/opt/local/lib -arch x86_64',
     'CPPFLAGS' => '-I/opt/local/include'}
  else
    {}
  end
end

def rvm_odbc_dir
  for_macports? ? '/opt/local' : '/usr/local'
end

def set_environment_vars(vars)
  vars.each { |k,v| ENV[k] = v }
end

def with_my_environment_vars
  my_vars = my_environment_vars
  current_vars = my_vars.inject({}) { |cvars,kv| k,v = kv ; cvars[k] = ENV[k] ; cvars }
  set_environment_vars(my_vars)
  yield
ensure
  set_environment_vars(current_vars)
end
```


<h2>RVM Rake Task Breakdown</h2>

<p>
  This is the fun part - lets start from top to bottom. I'll focus only on the parts that are centric to why the RVM ruby API is so bad ass and the rake task in general. The following section is dedicated to RVM with the SQL Server Adapter stack. First, the :setup task, this is called before every other task. It simply sets up the load path so that the RVM API file can be required. That api file is located in your rvm repo path, typically in your ~/.rvm directory. Now every other task can use the <code>RVM</code> module which implements method missing for many commands.
</p>

<p>
  Past this my rvm namespace is broken up into three main install tasks. The primary concerns are <code>rvm:install:rubies</code> and <code>rvm:install:gems</code> each invoked by the rake task. Starting with the :rubies task, this iterates over a collection of ruby version strings first checking if RVM knows its installed then installing it otherwise.
</p>

<p>
  In the :gems task, things get a little interesting. I am using a few methods (seen toward the bottom) that give this a nice little DSL of my own around RVM. The first is a block method called <code>rvm_each_rubie</code>. This iterates over each of my project's ruby strings, tells RVM to use that ruby, hence dynamically switching to that ruby/gemset then yielding to the block. That means that each go around I will be in an completely different ruby/gem environment, each specific to my project using RVM gemsets. This allows the inner method for iterating over the required gems for my project and asking to install them with <code>rvm_install_gem</code>. This method uses <code>RVM.ruby_eval</code> to execute a string of ruby in the context of the current ruby version and gemset. In this case, finding out if a gem is installed already. If not, I use the <code>RVM.perform_set_operation</code> to install the gem, again in current context. I use perform_set_operation so I can read the STDOUT back to the rake task so a user sees exactly what is going of for the gem install.
</p>

<p>
  Past that there are a few other details. Most can be picked up by reading the helper methods toward the bottom. Using the RVM ruby API is a bit of a chore if you rely on documentation. Sure it has some, but nothing beats reading the code to see what is available to you. Remember you can find the API library by opening up the <code>~/.rvm/lib</code> directory. I'm sure you can also find help on the <a href="http://rvm.beginrescueend.com/support/irc/">RVM IRC channel</a>.
</p>

<p>
  Though it is not specific to general RVM API goodness, the :odbc task does show some great interfaces that RVM exposes for doing standard file system directory changing and running commands to the system.
</p>


<h2>Notes On RVM With The SQL Server Adapter Stack</h2>

<p>
  So you use the ActiveRecord SQLServerAdapter? That means you have some underlying components installed - namely FreeTDS, unixODBC, and RubyODBC right. If your like me and believe that MacPorts is the way to go and risking a Homebrew interleaved dependency with Apple's libraries is risky, this section is for you! So you have a MacPort base and you want to compile your RVM rubies in such a way that other dependencies such as Nokogiri and RubyODBC use your /opt/local installs. By default this does not happen because unless ruby was compiled the right way, it wont be able to allow built gems to know about your /opt/local directory. So this is what I came up with.
</p>

<p>
  Take a look at the <code>my_environment_vars</code> method. This works in-conjunction with the <code>with_my_environment_vars</code> block method. Basically it temporarily set's MacPort specific environment variables before installing a ruby version via RVM. The ones shown are what I have found work best for my system. I think the most important are <code>LDFLAGS=-L/opt/local/lib -arch x86_64</code> and <code>CPPFLAGS=-I/opt/local/include</code>. Once ruby is built with those, it can easily reflect with standard 3rd party install methods that use <code>RbConfig</code>. To date any gem that I have had to compile, most importantly RubyODBC, does so perfectly against my /opt/local ports. This includes MySQL, Nokogiri, everything! I love it. I totally encourage anyone to use RVM and to get to know it's great ruby API for automating all sorts of things.
</p>


<h2>Resources</h2>

<ul>
  <li><a href="http://rvm.beginrescueend.com/">Ruby Version Manager (RVM)</a></li>
  <li><a href="http://github.com/rails-sqlserver/activerecord-sqlserver-adapter">ActiveRecord SQL Server Adapter</a></li>
  <li><a href="/2009/9/5/the-ultimate-os-x-snow-leopard-stack-for-rails-development-x86_64-macports-ruby-1-8-1-9-sql-server-more">The Ultimate OS X Snow Leopard Stack For Rails Development - x86_64, MacPorts, Ruby 1.8/1.9, SQL Server, SQLite3, MySQL & More</a></li>
  <li><a href="http://wiki.github.com/rails-sqlserver/activerecord-sqlserver-adapter/">SQL Server Adapter Wiki</a></li>
</ul>


