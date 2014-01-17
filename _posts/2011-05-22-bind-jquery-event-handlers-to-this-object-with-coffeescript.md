---
layout: post
title: Bind jQuery Event Handlers To This Object With CoffeeScript
categories: 
  - javascript
---

<p>
  Friends have told me that rich domain objects are seldom wielded when using jQuery to enhance behavior on web pages. I myself have always loved JavaScript as a rich dynamic language first and something for the DOM second. Hence most of my client-side JavaScript follows a robust object-oriented approach similar to Ruby. This is the main reason I have used Prototype.js for so long.
</p>

<p>
  Since Rails announced both jQuery and CoffeeScript as the defaults in version 3.1, I decided it was high time I starting learning both. I had always known that jQuery bound the <code>this</code> keyword in event handlers to the DOM object. Something I found totally confusing and unacceptable to someone dealing with their own objects to encapsulate behavior. Today I found two ways of dealing with my problem, a jQuery way and a CoffeeScript way. First a code example. 
</p>


~~~ruby
class MyObject
  
  constructor: ->
    @myDomElement = $('#myDomElement')
    @._initBehavior()

  handler: (event) ->
    console.log(this)
    false

  _initBehavior: ->
    $(window).resize @handler

jQuery ->
  window.myObject = new MyObject();
~~~

<p>
  Here is a simple class using CoffeeScript's methods. It initializes itself with a static property <code>this.myDomElement</code> to some DOM element on the page with an id of "myDomElement". It then attaches an event handler to the window's resize event and logs <code>this</code> along the way. Simple stuff, the only problem will be that the object logged will not be an instance of MyObject, but the raw DOM element, in this case the window object. One way of fixing this is to use <a href="http://api.jquery.com/jQuery.proxy/">jQuery's proxy</a> function like so
</p>

~~~ruby
_initBehavior: ->
  $(window).resize jQuery.proxy(@handler,this)
~~~

<p>
  This works, but seems a little verbose to me and can clutter up your event initialization code. The other way is to use CoffeeScript's fat arrow operator. An <a href="http://jashkenas.github.com/coffee-script/">excerpt from their project page</a> explains it well.
</p>

<blockquote>
  The fat arrow => can be used to both define a function, and to bind it to the current value of this, right on the spot. This is helpful when using callback-based libraries like Prototype or jQuery, for creating iterator functions to pass to each, or event-handler functions to use with bind. Functions created with the fat arrow are able to access properties of the this where they're defined.
</blockquote>

<p>
  So all we have to do is change <code>-></code> to <code>=></code> for any of our callbacks or event handlers and now <code>this</code> is our own object and not the DOM element. Hot damn!
</p>

~~~ruby
class MyObject
  
  constructor: ->
    @myDomElement = $('#myDomElement')
    @._initBehavior()

  handler: (event) =>
    console.log(this)
    false

  _initBehavior: ->
    $(window).resize @handler

jQuery ->
  window.myObject = new MyObject();
~~~




