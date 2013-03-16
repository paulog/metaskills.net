--- 
layout: post
title: Visor Terminal on Snow Leopard
disqus_id: /2009/08/18/visor-terminal-on-snow-leopard/
categories: 
  - apple-os
---

<aside class="flash_warn">
  UPDATE: Hacks no longer needed, latest Visor/SIMBL is 64-bit Snow Leopard happy!
</aside>

<p>
  This is a similar process that I had to go through back in the day when I had to hack visor terminal in Leopard. Basically the steps are pretty easy. First you just install <a href="http://www.culater.net/software/SIMBL/SIMBL.php">SIMBL</a> and the <a href="http://visor.binaryage.com/">Visor.bundle</a> as a SIMBL plugin in <code>~/Library/Application Support/SIMBL/Plugins/Visor.bundle</code>. Once that is done here is the process to get this working in Snow Leopard.
</p>

<p>
  First, you are going to need a copy of the Terminal.app from Leopard. I have <a href="http://cdn.metaskills.net/VisorTerminal.zip">provided a copy</a> in the resources section below. This copy has a few key things changed in the app's Info.plist file. First I have changed the bundle identifiers and display names to VisorTerminal. This is how we are going to scope visor to use this particular app. It also allows us to set things like the LSUIElement to 1 so that this app does not show in the dock. A summary of the changes I made are below, all these are done already in the download file I provide. If you want to do these on your own copy of Leopard's Terminal.app then just right click on the app, show package contents, and edit the Info.plist file. Remember to rename the app to VisorTerminal.app.
</p>

```html
<key>CFBundleDisplayName</key>
<string>VisorTerminal</string>
...
<key>CFBundleIdentifier</key>
<string>com.apple.VisorTerminal</string>
...
<key>CFBundleName</key>
<string>VisorTerminal</string>
...
<key>LSUIElement</key>
<string>1</string>
```

<p>
  Now we need to edit the installed <code>~/Library/Application Support/SIMBL/Plugins/Visor.bundle</code> so that it focuses on the old Leopard's terminal app (now VisorTerminal.app). Again right click on it and show the package contents, here is the complete plist below. You can see where I changed 3 places to VisorTerminal.
</p>

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>CFBundleDevelopmentRegion</key>
  <string>English</string>
  <key>CFBundleExecutable</key>
  <string>Visor</string>
  <key>CFBundleIdentifier</key>
  <string>com.blacktree.visor</string>
  <key>CFBundleInfoDictionaryVersion</key>
  <string>6.0</string>
  <key>CFBundleName</key>
  <string>Visor</string>
  <key>CFBundlePackageType</key>
  <string>BNDL</string>
  <key>CFBundleSignature</key>
  <string>????</string>
  <key>CFBundleVersion</key>
  <string>Custom</string>
  <key>GoogleML</key>
  <dict>
    <key>TargetApplications</key>
    <array>
      <dict>
        <key>BundleIdentifier</key>
        <string>com.apple.VisorTerminal</string>
        <key>BundleVersionsRE</key>
        <array>
          <string>.*</string>
        </array>
        <key>ExecPattern</key>
        <string>*/VisorTerminal.app/Contents/MacOS/Terminal</string>
      </dict>
    </array>
  </dict>
  <key>NSPrincipalClass</key>
  <string>Visor</string>
  <key>SIMBLTargetApplications</key>
  <array>
    <dict>
      <key>BundleIdentifier</key>
      <string>com.apple.VisorTerminal</string>
    </dict>
  </array>
</dict>
</plist>
```

<p>
  Now when you launch VisorTerminal, it will be hidden from the dock and be the app that Visor.bundle focuses on. I really like how this app is removed from the doc too. More so I am just stoked it is working in Snow Leopard. As a last step, yoyou can add this to your login items so that it opens up automatically. I have found that it is a bit sticky when it first launches, but a few clicks in and out on first launch fixes that. One last note, I found it is easier to edit the custom properties for this terminal window like it's default color, font size, etc, if I turn off the LSUIElement.
</p>

<p>
 <strong>UPDATE:</strong> Thanks to Raptor007 for reminding us that if you want to get a head start on your terminal preferences looking the same for your VisorTerminal, you should copy <code> ~/Library/Preferences/com.apple.Terminal.plist</code> to <code> ~/Library/Preferences/com.apple.VisorTerminal.plist</code>. Thanks!
</p>

<h2>Resources</h2>

<ul>
  <li><a href="http://visor.binaryage.com/">Visor Project Page</a></li>
  <li><a href="http://www.culater.net/software/SIMBL/SIMBL.php">SIMBL Project Page</a></li>
  <li><a href="http://cdn.metaskills.net/downloads/VisorTerminal.zip">VisorTerminal.app (Leopard)</a></li>
</ul>






