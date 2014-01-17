--- 
layout: post
title: Learn To Program in Ruby and Basic SQL
disqus_id: /2006/03/05/learn-to-program-in-ruby-and-basic-sql/
categories: 
  - database
  - heuristics
  - ruby-rails
---

<p>
  <img src="/assets/ruby-sql_books.png" alt="Learn To Program Ruby and SQL" width="204" height="182" class="floatr ml20" /> I've been learning to or &quot;trying to learn&quot; <a href="http://www.rubyonrails.org/">Ruby on Rails</a> for a few months now and things have always kept me from finishing the book that I purchased from those great publishers at the <a href="http://www.pragmaticprogrammer.com/">pragmatic bookshelf.</a> My problem has been that sometimes other work has gotten in the way, but mostly it was because I did not have the core understanding of the basics for building web applications. Especially in the areas of object oriented programming and database languages. For me this was a big problem, I'm typically a fundamentalist when it comes to learning and applying knowledge. Knowing the details helps me understand the big picture and more importantly the confidence to know what I am doing is correct. So rather than learning super high level code, I decided to revisit the 3-foot section of the pool again by reading these two books.
</p>


<h2><em>Teach Yourself SQL in 10 Minutes</em> by Ben Forta</h2>

<p>
  Just so you know, you are not going to  learn SQL in 10 minutes! This book is chopped up into &quot;10 minute&quot; exercises that take you into the language step by step. If you are new to databases, this book is great because it teaches you the core SQL syntax that most <a href="http://en.wikipedia.org/wiki/Dbms">database management systems</a> use to extract and input data into the database itself. It even covers idioms that are unique for each database language including SQL Server, PostgreSQL, MySQL, and Oracle.
</p>

<p>
  By the time I picked up <a href="http://www.amazon.com/gp/product/0672325675/sr=8-1/qid=1141569613/ref=sr_1_1/103-7179153-7483038?_encoding=UTF8"><em>Teach Yourself SQL in 10 Minutes</em></a>, I had already learned to tread water in a huge Microsoft SQL Server database that I work in every day. So hacking stored procedures and building some basic queries were in my capabilities, but after reading Ben's book, I was able to do a lot more. It really pays off to know the basics and I'll let you in on a little secret. If you really want to work with databases and plan on sticking with it, I suggest using <a href="http://www.dbsuite.com/mosg5/?action=product&actionid=sql4xmanagerj">SQL 4X Manager J 3</a> by InterServices New Media. It will allow you to be database agnostic and its features are so rich that it will continue to be a valuable tool long after you have mastered SQL. This application is written in pure JAVA and it uses ODBC connections to support every database I have come across. I simple love its console support, the ability to edit and run stored procedures, create views, and generate entity relationship models. Trust me, it's worth the price!
</p>

<div class="center">
  <a href="http://www.dbsuite.com/mosg5/?action=product&actionid=sql4xmanagerj" class="nobor">
    <img src="/assets/th_sql4xjmaininterface.png" alt="Main Interface for SQL 4X Manager J 3" width="528" height="308" />
  </a>
</div>


<h2><em>Learn to Program</em> by Chris Pine</h2>

<p>
  If you are new to programming, this is a <a href="http://www.pragmaticprogrammer.com/titles/fr_ltp/">great book</a> to pick up first. It is a modest 150 pages long and baby steps you into the concepts shared by most languages, using my favorite language, Ruby. That's right, I buy into that hype around this language because to me Ruby is like Apple Computer, its different! Here is an quote from its creator, Yukihiro Matsumoto, in the de facto Ruby resource guide,  the <a href="http://www.pragmaticprogrammer.com/titles/ruby/index.html"><em>Pragmatic Programmers' Guide to Programming Ruby</em></a>.
</p>

<blockquote>
  I believe that the purpose of life is, at least in part, to be happy. Based on this belief, Ruby is designed to make programming not only easy but also fun. It allows you to concentrate on the creative side of programming, with less stress.
</blockquote>

<p>
  So if you believe that JAVA is better thank Ruby or maybe PHP or any other language, please do not start to flame me. I think each language has its place. You can have yours and I'll have mine. I for one have always valued the underdog more. That said, <em><a href="http://www.pragmaticprogrammer.com/titles/fr_ltp/">Learn to Program</a></em> by Chris Pine is the place to start.
</p>

<p>
  Like most other programming books, this one is rich in examples and test scripts. The thing that I thought most unique about it was that it forced you to utilize the concepts you learned by asking you to write a program that did X, Y, and Z. It did this without giving you the answers which really forces you out of the academia of learning into real practice. So if you <a href="http://www.pragmaticprogrammer.com/titles/fr_ltp/">pick up this book</a>, or if decide to use <a href="http://pine.fm/LearnToProgram/?Chapter=00">the free tutorials</a> on Chris' website, check back here often and I'll keep posting updates on what code I came up with. Perhaps others can share how the faced the challenges since I could not find once single resource on the web that did this. Here are a few to start this off.
</p>


<h2>A Few Things To Try (bottles of beer program)</h2>

~~~ruby
bottles = 99

while bottles.to_i != 0
  puts bottles.to_s + ' bottles of beer on the wall. ' + 
    bottles.to_s + ' bottles of beer!'
  bottles = bottles.to_i-1
  puts 'You take one down and pass it around... you have ' + 
    bottles.to_s + ' bottles of beer on the wall.'
  puts ''
end
~~~


<h2>A Few Things To Try (roman numeral method)</h2>

~~~ruby
def romanize integer
  mdiv = integer/1000
  m = 'M'*mdiv
  d = 'D'*(integer%1000/500)
  c = 'C'*(integer%500/100)
  l = 'L'*(integer%100/50)
  x = 'X'*(integer%50/10)
  v = 'V'*(integer%10/5)
  i = 'I'*(integer%5/1)
  m+d+c+l+x+v+i
end

puts romanize(420)
puts romanize(9)
puts romanize(1)
puts romanize(278)
puts romanize(2156)
~~~


<h2>A Few Things To Try (how old are you, spank)</h2>

~~~ruby
puts 'What year were you born in? (number please)'
year = gets.chomp.to_i
puts 'What month were you born on? (number please)'
month = gets.chomp.to_i
puts 'What day were you born on? (number please)'
day = gets.chomp.to_i

bday        = Time.mktime(year.to_i, month.to_i, day.to_i)
time        = Time.new
diftime     = time - bday
diftoyears  = (diftime/60/60/24/365).to_i/1

puts 
diftoyears.times do
  puts 'SPANK!'
end
~~~
	

