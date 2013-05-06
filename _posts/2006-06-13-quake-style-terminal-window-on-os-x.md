--- 
layout: post
title: Quake Style Terminal Window on OS X
lastmod: 2012-05-18T12:00:00Z
disqus_id: /2006/06/13/quake-style-terminal-window-on-os-x/
categories: 
  - apple-os
  - heuristics
  - workflow
---

<aside class="flash_info">
  UPDATE: (May 18th, 2012) The modern replacement for VisorTerminal is called TotalTerminal and cand be found <a href="http://totalterminal.binaryage.com/">on the binaryage site</a>
</aside>

<p>
  <img src="/assets/visor_screenshot.jpg" alt="Visor Screenshot" width="550" height="413" class="floatr ml20 shadow" /> Well this is far beyond cool but highly functional, a Quake like terminal implementation of Terminal.app that is a HotKey away from within any application. A friend turned me on to this after it showed up on the <a href="http://arstechnica.com/journals/apple.ars/2006/6/12/4291">Monday morning Apple links</a> post from arstechnica.com. Although I have never thought of this idea, it seems to have been a popular request for quite some time and after a <a href="http://episteme.arstechnica.com/eve/forums/a/tpc/f/8300945231/m/332004739731">public request</a>, the author of <a href="http://quicksilver.blacktree.com/">QuickSilver</a> stepped up to the challenge and coded this little goodie using the application enhancer method called <a href="http://www.culater.net/software/SIMBL/SIMBL.php">SIMBL</a> which was created by Mike Solomon, the creator of the PithHelmet plugin for Safari.
</p>

<p>So if you are itching to try this out here is what you do.</p>

<ul>
  <li><a href="http://www.culater.net/dl/files/SIMBL-0.8.1.tbz">Download and Install SIMBL</a> </li>
  <li><a href="http://download.blacktree.com/Visor.zip">Download Visor</a> and place it in the <code>/Library/Application Support/SIMBL/Plugins</code> folder.</li>
  <li>Logout and back into your account (maybe restart) </li>
  <li>Launch Terminal.app (close it if you want, also see my AppleScript below)  </li>
  <li>Set your Visor HotKey preferences in the new menu. </li>
</ul>

<p>
  Now, when you launch any other program and hit your HotKey (mine is Command ~) a terminal application will slide down from the top of your screen just like in Quake. The only down side is that if you are used to your iTerm display, you will have to set the preferences of Terminal.app to mimic that. I set mine with a black background, ProFont 10px, and 80% visibility.
</p>

<p class="clearfix">
  <img src="/assets/termina_applescript.gif" alt="Visor AppleScript" width="370" height="363" class="floatl mr20 shadow" />  Lastly, since you do have to have the Terminal application running for this to work and I was too lazy to open this al the time, I simple created a small AppleScript which can be compiled into an application and placed into your Login Item in the System Preferences. Here is a screenshot of the code. Notice that it tells window 2 to close. This is because once you have SIMBL and Visor installed, an invisible window opens for Terminal that you cannot see. So the window that you can see (window 2) is closed by AppleScript.
</p>


