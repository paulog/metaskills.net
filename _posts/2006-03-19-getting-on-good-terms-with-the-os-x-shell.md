--- 
layout: post
title: Getting On Good Terms With The OS X Shell
disqus_id: /2006/03/19/getting-on-good-terms-with-the-os-x-shell/
categories: 
  - heuristics
  - workflow
---

<p>
  <img src="/assets/iTerm.png" alt="iTerm Logo" width="135" height="116" class="floatl mr20" /> I will be the first to admit that I am really just learning how to tap into the power of my shell environment and to be honest, I've spent way to many hours  reading man pages and figuring out how to do some really neat things that help my automate my workflow and system administration. Mostly these are just basic tasks like my <a href="/2005/12/19/how-to-setup-a-simple-mysql-backup-script/">Simple MySQL Backup</a> and <a href="/2006/02/24/shell-script-to-delete-all-invisible-_-resource-files/">Deleting Invisible Resource Files</a> scripts. But in all seriousness, when you get right down to using a UNIX-based operating system, you cannot escape using the shell environment. This is a good thing, its your friend, and getting your feet wet sooner than later is a good idea.
</p>
		
<h2>Getting A Better Terminal Application</h2>
		
<p>
  Go <a href="http://iterm.sourceforge.net/index.shtml">download iTerm</a> from the sourceforge project page and install it. That default Terminal.app that comes with the Mac has got to go, it is dreadful looking and the features are really slack. Fortunately iTerm is a native Cocoa app that supports both the PowerPC and the new Intel Macs. The best feature of iTerm is that it supports tabbed windows, because you can never have to many terminal windows open. Here are some preferences I recommend after installing iTerm.
</p>

<ul>
  <li>Tab Position Top</li>
  <li>Selection copies text to clipboard &amp; Focus follows mouse  </li>
  <li>The &quot;Foreground&quot; and ANSI Colors pictured below, most are defaults.</li>
  <li>A dark background image with little detail so as not to interfere with your foreground text. </li>
  <li>In there &quot;Terminal&quot; tab of the profiles, set the &quot;Type&quot; to &quot;xterm-color&quot;. </li>
</ul>

<div class="center">
  <img src="/assets/iterm_prefs-all.png" alt="iTerm Profile Preferences" width="500" height="373" class="mt20" />
</div>



<h2>Customizing Your Bash Profile </h2>

<p>
  After you have a nice terminal program, its time to tweak your environment a bit. Here is a great set of customization commands that will change the way your shell interface is displayed when using the bash terminal, this is the standard shell environment on newer versions of OS X. Using these environment enhancements you will get a solid visual on the following information which is also colored coded to appear well on a dark background.
</p>

<ul>
  <li>The current user you are logged in as (green) for you (red) for root/others. </li>
  <li>The current machine name and working directory. </li>
  <li>A new entry line which is not obscured by the items up above.</li>
</ul>

~~~bash
# User colors (root colors in /etc/bashrc)
export TERM="xterm-color"
C1="\[\033[1;32m\]"
C2="\[\033[1;30m\]"
C3="\[\033[0m\]"
C4="\[\033[0;36m\]"
export PS1="${C2}(${C1}\u${C2}@${C4}\h${C2}) - (${C4}\A${C2}) - (${C4}\w${C2})\n${C2}-${C1}=>>${C3}"
~~~

~~~bash
# Root colors (user colors in ~/.bash_profile)
export TERM="xterm-color"
C1="\[\033[0;31m\]"
C2="\[\033[1;30m\]"
C3="\[\033[0m\]"
C4="\[\033[0;36m\]"
export PS1="${C2}(${C1}\u${C2}@${C4}\h${C2}) - (${C4}\A${C2}) - (${C4}\w${C2})\n${C2}-${C1}=>>${C3}"
~~~

<p>
  Here  is how to install these, they go into two separate places. The first customization commands go into your user home directory in a file called <code>.bash_profile</code> and if you have never modified or used this file, you will have to make a new one. The period that precedes the name is technically not allowed by most text editors since it makes the file invisible. I suggested using <a href="/2005/12/22/textmate-by-programmers-for-efficiency-experts/">TextMate</a> to create or modify this and the file below. Remember, the first file <code>.bash_profile</code> needs to go into your home directory. The more you use your terminal, the more you will start to modify this file. It too is your friend.
</p>

<p>
  The second set of customization commands go into the root <code>bashrc</code> file. Again, using <a href="/2005/12/22/textmate-by-programmers-for-efficiency-experts/">TextMate</a>, you can find this file in the /etc directory. If you are new to using TextMate to get to files in invisible directories, do this. When you have the &quot;Open...&quot; dialog window in view in TextMate, hold down <code>Command-Shift+G</code>. This will open a sub dialog window called &quot;Go to the folder:&quot; with a text field to type the directory structure into. Simply enter <code>/etc</code> and hit the &quot;Go&quot; button. This will take you right to the /etc directory and in there you should see the <code>bashrc</code> file. Open it add the lines from the second code sample box above and save it. From this point onward your terminal window in iTerm should look something like this.
</p>


<div class="center">
  <img src="/assets/my-iterm.gif" alt="My iTerm Window" width="523" height="415" class="shadow" />
</div>

<p>
  I never setup a server without putting these customization settings in place. It just makes me feel more at home on the command line. In some later articles, I'll start to cover some really sweet shell scripts for more workflow automation. Let me know if you like this or if you have any problems setting them up on your system.
</p>

