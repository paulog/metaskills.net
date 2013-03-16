--- 
layout: post
title: PDF::Writer For Ruby 1.9
disqus_id: /2009/05/14/pdf-writer-for-ruby-1-9/
categories: 
  - heuristics
  - ruby-rails
---

<p>
  If you have legacy code written for ruby 1.8 and you want to run 1.9 and support your old <a href="http://ruby-pdf.rubyforge.org/pdf-writer/">PDF::Writer</a> code, then just jump right over to <a href="http://github.com/metaskills/pdf-writer/tree/master">my Github pdf-writer fork</a> and get it.
</p>

<pre class="command">
$ gem install metaskills-pdf-writer
</pre>

<p>
  If you are interested in knowing some of the dirty details about what pitfalls are under 1.9, read on. The biggest thing for me was getting used to the character encodings. Including string literals in your code that are say UTF-8 or some other encoding will blow up on you real quick. I highly highly suggest that you read <a href="http://blog.grayproductions.net/categories/character_encodings">James Edward Gray II: Everything About Ruby 1.9 Character Encoding Series</a> series. He covers all the basics. If you want to see what two commits I did for pdf-writer, see <a href="http://github.com/metaskills/pdf-writer/commit/ef9787c050a3b861089fc95f4f3af544f8e39219">here</a> and <a href="http://github.com/metaskills/pdf-writer/commit/8f0ed3f1bd312bf86da47d721ca2f2070724efd6">here</a>. The second one is very very hackish, it basically add a marshaling support to the Mutex class which 1.9 does not have. The reason this is needed was due to pdf-writer's use of transaction simple to roll back object state as it is drawing across multiple pages. If you are like me and wish all your PDF::Writer code was in Prawn, your not alone!
</p>


<h2>Resources</h2>

<ul>
  <li><a href="http://github.com/metaskills/pdf-writer/tree/master">PDF::Writer For Ruby 1.9</a></li>
  <li><a href="http://blog.grayproductions.net/categories/character_encodings">James Edward Gray II: Everything About Ruby 1.9 Character Encoding Series</a></li>
  <li><a href="http://eigenclass.org/hiki.rb?Changes+in+Ruby+1.9">General Changes in Ruby 1.9</a></li>
  <li><a href="http://prawn.majesticseacreature.com/">Prawn: Fast, Nimble PDF Generation For Ruby</a></li>
</ul>




