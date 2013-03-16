
= Todo

  * Implement Site Search: http://kwpolska.co.cc/


= Development Setup

Misc notes on setting up a development environment. Rarely needs to be done, but wanted to document my personal setup.
  
  1) Install some deps.
     $ sudo port install optipng
     $ sudo port install tidy
     $ sudo port install py27-pygments
       Add /opt/local/Library/Frameworks/Python.framework/Versions/Current/bin 
       to $PATH for pygmentize binary.
     $ bundle install

  2) Use this SCSS bundle. Requires nokigiri, in Gemfile. 
     $ cd ~/Library/Application\ Support/TextMate/Bundles
     $ git clone git://github.com/kuroir/SCSS.tmbundle.git "SCSS.tmbundle"
  
  3) Setup git hooks.
     .git/hooks/pre-commit
     #!/usr/bin/env zsh
     /Users/kencollins/Repositories/metaskills.net/tasks/jekyll
     .git/hooks/post-push
     #!/usr/bin/env zsh
     /Users/kencollins/Repositories/metaskills.net/tasks/deploy


= Developing

After all things are installed, consult each file in the tasks directory for 
development tips.


= CSS Conventions

  <span class="photofancy">
    <img src="/assets/exclude_from_timemachine.gif" alt="Time Machine Exclude Window" width="463" height="320" />
  </span>

  ```ruby
    ...
  ```

  <aside class="flash_info">...</aside>
  <aside class="flash_warn">...</aside>



= License

Blog content and design are Copyright (c) 2006-2011, Ken Collins. 
Use is strictly forbidden unless consent is given!
Only jekyll plugins and helpers are MIT Licensed.





