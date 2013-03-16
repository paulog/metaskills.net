--- 
layout: post
title: Simple Script/Console Function
disqus_id: /2010/02/06/simple-script-console-function/
categories: 
  - heuristics
  - ruby-rails
---


<p>
  This is something simple I worked up today for my ZSH profile that let's me keep my simple <code>sc</code> alias and have it work with all versions of rails. If you did not know, all the script files in rails 3 are gone and the new all-in-one <code>rails</code> executable does all the heavy lifting. This little function even passes down all the arguments too.
</p>

```bash
sc () {
  if [ -f ./script/rails ]; then 
    rails console $argv
  else
    ./script/console $argv
  fi
}
```

<p>
  One other thing, the way rails uses IRB is different now. I had to change my <code>~/.irbrc</code> file to look like this below to get my simple prompt and history back. IMPORTANT NOTE: In order for this to work, you <a href="http://redmine.ruby-lang.org/issues/show/1556">have to apply this 2 line patch to your save-history.rb file</a>. Worked like a champ for me.
</p>

```ruby
# IRB history patch <http://redmine.ruby-lang.org/issues/show/1556>

require 'irb/completion'
require 'irb/ext/save-history'

IRB.conf[:USE_READLINE] = true
IRB.conf[:SAVE_HISTORY] = 500
IRB.conf[:HISTORY_FILE] = "#{ENV['HOME']}/.irb.hist"
IRB.conf[:PROMPT_MODE] = :SIMPLE
```

