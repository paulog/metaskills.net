--- 
layout: post
title: "Protip: Exclude Your RVM Install From Time Machine Backups"
disqus_id: /2010/08/06/protip-exclude-your-rvm-install-from-timemachine-backups/
categories: 
  -- heuristics
---


<p>
  You may not realize it, but that <code>~/.rvm</code> directory is getting big and may be hurting your Time Machine backups. Especially if you are doing a lot of development. I just checked mine today.
</p>

<pre class="command">
du -skh ~/.rvm/gems
1012M	/Users/kencollins/.rvm/gems
</pre>

<p>
  Ouch! That's around 1gig of stuff that is pointless to backup just in the gems directory. I frequently blow away certain rubies and install them again using <a href="/2010/7/30/the-rvm-ruby-api">various rake tasks</a> and just found out today that you have to pass the --gems option to rvm remove to clean those old gemsets out. Either way, that whole rvm directory can just be ignored from Time Machine backups. Here is how, there is really only one tricky part.
</p>

<p>
  First, open up your System Preferences, then open up the Time Machine pref pane. From here click on the "Options...". Within this window you will be able to add a folder to be excluded from backup. Hit the "+" button to add a directory. Now here is the tricky part, your rvm directory is prefixed with a period and hence is invisible to the normal finder windows. Thankfully all Mac file windows can use a keyboard shortcut. Hit <code>Command+Shift G</code>. That will blind down a "Go to the folder:" window. Simply type in <code>~/.rvm</code> here and hit "Go". Now choose "Exclude" from the main file window. Your done!
</p>

<div class="center">
  <span class="photofancy">
    <img src="/assets/exclude_from_timemachine.gif" alt="Time Machine Exclude Window" width="463" height="320" />
  </span>
</div>


