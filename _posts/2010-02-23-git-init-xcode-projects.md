--- 
layout: post
title: Git Init XCode Projects
disqus_id: /2010/02/23/git-init-xcode-projects/
categories: 
  - apple-os
  - heuristics
  - workflow
---

<p>
  Here is a little ZSH function I have been using for quickly setting up new XCode apps I call tire kickers, little play and learn apps. Being able to track your learning as you go with git.
</p>

~~~bash
if [[ -x `which git` ]]; then
  
	function ginit_xcode () {
	  git init
	  echo "\n\n# XCode\nbuild\n*.mode1v3\n*.mode2v3\n*.nib\n*.swp\n\
*.pbxuser\n*.perspective\n*.perspectivev3\n\n# OSX\n.DS_Store\n\n\
# TextMate\n*.tm_build_errors\n\n\n" >> .gitignore
	  git add .gitignore
	  git commit -m "Ignore Xcode stuff."
	  git add .
	  git commit -m "Initial Xcode project."
	}
  
fi
~~~

<p>The echo lines puts out a .gitignore file that will look something like this.</p>

~~~text
# XCode
build
*.mode1v3
*.mode2v3
*.nib
*.swp
*.pbxuser
*.perspective
*.perspectivev3

# OSX
.DS_Store

# TextMate
*.tm_build_errors
~~~

