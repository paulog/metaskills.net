--- 
layout: post
title: Shell Script To Delete All Invisible ._ Resource Files
disqus_id: /2006/02/24/shell-script-to-delete-all-invisible-_-resource-files/
categories: 
  - apple-os
  - workflow
---


<p>
  <img src="/assets/network_drive.png" alt="Network Drive" width="102" height="102" class="floatr ml20" /> If you have ever accessed your website using a network protocol such as the Apple Filing Protocol (AFP), Samba (SMB), Web-based Distributed Authoring and Versioning (WebDAV), or Network File System (NFS) using your  Mac â€“ I am sure you have run into this problem before â€“ dreaded invisible resource fork files. These are the files that begin with a <code>._</code> and they are normally not seen from within the finder. My understanding is that these files are not even created  on your local Mac hard drive since the <a href="http://en.wikipedia.org/wiki/HFS+">HFS+ file system</a> is smart enough to keep your data and resource forks together in one single file. But even if you are accessing your website from one Mac HFS+ volume to another Mac HFS+ volume, these files will still be created by programs like DreamWeaver and TextMate because the various protocols to access that remote share and/or the file systems themselves will need to split them to cope. So ultimately when ever you use these other types of file systems or network protocols, you will eventually be creating lost of invisible resource fork files.
</p>
    

<h2>Hey! ._ Files Are Not Really Bad</h2>
    
<p>
  No they are not. By nature your Mac should deal with these just fine be it on you local drive or on a network volume. That said, I think it is important to point out what these invisible resource fork and other invisible files actually do. The most common one on any Mac is the <code>.DS_Store</code> file. This stores preferences on window settings. If you open a window, and choose icon view and then move them around, the <code>.DS_Store</code> file stores remembers all those settings. When you open that folder again, you see it the way it was left, thanks to that <code>.DS_Store</code> file. I have  never had a problem with these files on a remote volume. They can live :)
</p>
    
<p>
  The files that are prefixed with <code>._</code> are resource fork files that have a one to one match with another file that you can see. For example, let's say you have a file called <code>index.php</code> on your web server and you have made some changes to it with your editor such as <a href="http://macromates.com/">TextMate</a>. Once you save that file, TextMate will create another file called <code>._index.php</code> on your web server. Why? Well Textmate, just like the Finder, likes to save your preferences. The most common is what line number your cursor was on. Great feature, I hate scrolling back to that  line I just edited. However convenient this is, I have had <code>._</code> files cause technical and visual problems on my web server. Most of the time these problems happen when a PHP application like <a href="http://drupal.org/">Drupal</a> or <a href="http://www.pmachine.com/">Expression Engine</a> needs to scan directories for files to include or enable in its application. The reason these files can be detrimental to your web application can be numerous and if you are reading this article, you probably don't care, so let's just get rid of them.
</p>
    
    
<h2>The Killer Resource Script</h2>
    
<p>
  This shell script consists of three commands. The first is the UNIX <code>find</code> command. It is told to look for all files in a directory that have a name which starts with ._ and then print them to your window. The second part of the command is called <code>xargs</code> and basically this takes each line form the first command and applies the  command immediately following it. In this case, remove, as indicated by the <code>rm</code> at the end. The <code>-t</code> after <code>xargs</code> is optional, it is useful if you are running this from your terminal since it echos the command to you before it is executed. The final shell script looks like this.
</p>
		
```bash
#! /bin/bash
find /Volumes/Your_NetworkVolume -name '._*' -print | xargs -t rm
```

<p>
  To customize this script you will need to change &quot;<code>Your_NetworkVolume</code>&quot; where it appears above to the name of your web server share as it appears on your desktop.
</p>
    
<p>
  I find it useful to put this script in my AppleScript menu. If your apple script menu is not on menu bar at the top of your screen, you can turn it on using the &quot;AppleScript Utility&quot; program located in your (/Applications/AppleScript) directory. Just check the box labeled &quot;Show Script Menu in menu bar&quot; and unless you want a really long  list of extra pre installed example scripts by Apple, I suggest keeping &quot;Show Library scripts&quot; unchecked.
</p>
    
<p>
  <span class="photofancy floatl mr20"><img src="/assets/applescript-menu.jpg" alt="AppleScript Menu" width="422" height="267" /></span> Then using TextMate, create a file named something like this &quot;DELETE ._ Files-on-MySvr.sh&quot; and save it in your home (~/Library/Scripts) folder. While the file is still open in TextMate we need to make it executable by going to the menu (Automation Â» Run Command Â» Unix Shell Â» Make Script Executable). Once that is done you can close the script and you should see it appear in your scripts menu like this.
</p>

<p>
  This way, you can get in the habit of running this command when you are done working on the web server. Of course, you could setup this script on the web server itself and then execute it via the crontab every so often. It's really up to you.
</p>
    
<p>
  One last thing, in some cases you may have to delete invisible files on your web server that have spaces or special characters that might cause the script above to fail. Now why you would have files  on a web server that are non web friendsly is completely against my understanding, but if that is the case then you will have to use this modified version. It includes what I call a shielded version of the -print and xargs commands so they can understand how to escape or deal with the spaces and special characters.
</p>

```bash
#! /bin/bash
find /Volumes/Your_NetworkVolume -name '._*' -print0 | xargs -t0 rm
```


