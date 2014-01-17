--- 
layout: post
title: How To Setup a Simple MySQL Backup Script
disqus_id: /2005/12/19/how-to-setup-a-simple-mysql-backup-script/
categories: 
  - database
  - heuristics
  - workflow
---

<p>Lately, I've been loving all things that can be solved by using <a href="http://archive.macosxlabs.org/rsyncx/rsyncx.html" title="Solving Problems With RsyncX">RsyncX</a>. It's my de facto backup utility and I just keep finding more and more tasks for it as each day goes by. It has become the hammer for all my system's needs. So when it came time for me to implement a nice little backup routine for the MySQL databases hosted here at ActionMoniker.com, it was the first tool I considered. My requirements were simple, I needed a SQL script of selected databases in logically named folders for each DB with time-stamps in the file name. Then step and repeat on a regular basis. </p>

<p>To use RsyncX for the job, I would have to copy the literal file system for the databases, typically these are located in <code>/usr/local/mysql/data/dbname</code> on your macintosh file system. Doing such would have been a technically valid method, but this solution would not have given me a portable backup file since RsyncX would copy the directory in whole, which includes separate files for each table in the database. Not to mention permission issues for restoring the files.</p>


<h2>Perhaps MySQL Administrator?</h2>

<p> So instead of square pegging RsyncX I turned to the one of the greatest inventions for the Open Source database community, the <a href="http://dev.mysql.com/downloads/administrator/index.html" title="Database Administration GUI for OS X MySQL Databases">MySQL Administrator</a> application. This wonderful program brings many features of administrating MySQL databases into a nice tidy graphical user interface that most users can understand by simply pointing and clicking around. It is very easy to understand and it is akin to the "Enterprise Manager" for Microsoft SQL Server running on Windows. The things you can accomplished with MySQL Administrator are past the scope of this article, but I do want to point out that MySQL Administrator does offer a manual and automatic backup solution for you by using your Mac's "user" crontab. If you are unfamiliar with cron, it is your Mac's (UNIX-Style) built-in job scheduling system. </p>

<p> If you have ever tried using MySQL Administrator's backup routine, DO NOT DO IT! The program fails miserably for most users since it uses your local system account, not the DB user account associated with the database you are tying to backup. If you are reading this article because you have tried using MySQL Administrator's backup system, you may have seen a system message like this: </p>

~~~text
From: kencollins@ken-pb.local (Cron Daemon)
To: kencollins@ken-pb.local
Date: Mon, 19 Dec 2005 23:04:01 -0500 (EST)
Subject: Cron <kencollins@ken-pb> "/Applications/MySQL Administrator.app/Contents/Resources/mabackup" --directory="/Users/kencollins/Desktop" --connection="localhost" --prefix="MetaDB" MetaDB
~~~

<p> Before I get to that simple solution and if you are one of the unlucky that tried backups with MySQL Administrator, let's digress for a moment and clean up the work it did to your system. First, if you have not already done so, install <a href="http://webmin.com/" title="The Swiss Army Knife of Mac OS X Admin Tools">Webmin</a> . Many of my tutorials will rely on it. This is a powerful tool that runs as a bundle of PERL modules accessible via a web browser on your local machine. It allows you to administrate many underlying system events and programs on your system. <strong>DISCLAIMER: It can be somewhat hazardous if you are not familiar with what you are doing.</strong> That said, the documentation on their website has a very straight forward  help file on installing the latest version. Now to the cleanup: </p>

<ul>
  <li><strong>Step 1</strong> - removing the MySQL Administrator cron entry. Open Webmin and navigate to "System -&gt; Scheduled Cron Jobs", directly beside "yourusername" you should see a command similar to the subject line in my example error above. Click on it to edit the job, and hit "delete" at the bottom of the page.</li>
  <li><strong>Step 2</strong> - removing any junk emails or system notices. Open Webmin and navigate to "Servers -&gt; Read User Mail", click on your username, then delete any messages similar to the ones above. You should have one message each time MySQL Administrator failed to do what you wanted.</li>
</ul>

<h2>The Simple Solution</h2>

<p>OK, now that our system has less junk processes, to the fun part, a simple and working MySQL backup. For this task I created a really simple script. Let's take a look at it below. </p>

~~~bash
#!/bin/bash
timestamp="$(date -u +%Y-%m-%d_%a)"
/usr/local/mysql/bin/mysqldump -uXXXX -pXXXX metadb > /Backups/MetaDB_${timestamp}.sql
~~~

<p>There are two parts to this script, the first line sets a variable called "timestamp" in the friendliest format I can think of (YYYY-MM-DD_Day), for example 2005-12-19_Mon. The second line of this script uses the mysqldump utility, gives it a username, password, and your database name. Replace XXXX with the username and password of your database and remember, if you are calling this script from a remote machine, it must have remote access to the database name, directly after your password. The last half of the second line writes your database backup file in a directory of your choosing with a filename using the <code>${timestamp}</code> variable of the system time from line one. In this example, this will generate a file called "MetaDB_2005-12-19_Mon.sql". </p>

<p> To run this script, save it to a text file like MyDB-Backup.command or .sh and make it executable. My favorite way of doing this is to use <a href="http://macromates.com/" title="The Best Text Editor for the Mac OS X">TextMate's</a> built-in UNIX Shell automation scripts, but if you do not have such a fancy text editor, simple use the Terminal and enter this command on the file <code>chmod +x yourfilename.sh</code> Lastly, you can schedule this job to run at any interval using the system's crontab. This can be added using Webmin or by opening up the crontab, located at <code>/etc/crontab</code> and then follow the example in the header of the file. </p>

