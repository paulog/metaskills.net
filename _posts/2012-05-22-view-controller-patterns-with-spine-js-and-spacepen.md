---
layout: post
title: View Controller Patterns With Spine.JS &amp; SpacePen
categories: 
  - ruby-rails
  - javascript
---

<h2>The VC in 'MVC'</h2>

<p>
  As Addy Osmani points out in his <a href="http://speakerdeck.com/u/addyosmani/p/scaling-your-javascript-applications">Scaling Your JavaScript Applications</a> presentation, all JavaScript 'MVC' frameworks interpret MVC differently. These differences are an academic rabbit hole and if you are really interested about them, I recommend reading some of the resource links at the bottom of this post. One in particular by Jonas Nicklas really outlined how I think client side JavaScript applications should be developed. It is titled <a href="http://elabs.se/blog/33-why-serenade-js">Why Serenade.js</a> and in it, Jonas describes his reasons for yet another JavaScript MVC framework. In doing so, he outlines the responsibilities of the MVC triad as follows:
</p>

<p>
  <strong>Views:</strong> Present data from the model and update it if it changes, notify controllers of user interaction events. <strong>Controllers:</strong> React to user interaction events by instructing the model to perform certain actions. <strong>Models:</strong> Handle business logic and persistence, notify the view of any changes to the data.
</p>

<p>
  <!--<span class="floatr" style="margin:0 0 15px 15px; -moz-box-shadow: 5px 5px 5px rgba(0,0,0,0.5); -webkit-box-shadow: 5px 5px 5px rgba(0,0,0,0.5); box-shadow: 5px 5px 5px rgba(0,0,0,0.5); -moz-transform: rotate(-2deg); -webkit-transform: rotate(-2deg); transform: rotate(-2deg);">
    <img src="/assets/todomvc.png" alt="TodoMVC JavaScript Frameworks" width="354" height="263" />
  </span>-->
  Seems straight forward right? In actuality it is easy to stray from and there is a lot of wiggle room for both the MV* framework authors and their developers to re-interpret one or all of these components. The two that I think are most open for debate are the View and the Controller stacks. For example, how many <a href="http://addyosmani.github.com/todomvc/">JavaScript frameworks</a> do you know allow you to choose your own view framework? How many times have you heard too that views should be "logic-less" and how does each of these play into helping newcomers understand the proper way to code these precious views and controllers together? My answer, ask Apple!
</p>

<h2>Apple's View Controller Programming Guide</h2>

<p>
  Some might be surprised that JavaScript applications for the browser can and should be coded much like modern desktop applications. Having developed iOS applications before, many of the same MVC principals and application structure apply. Apple has tons of documentation about general application design and one of my favorites is their <a href="http://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/Introduction/Introduction.html">View Controller Programming Guide</a>. I highly recommend anyone developing MVC JavaScript applications to read this guide. I'll paraphrase a few points:
</p>

<blockquote style="margin-left: 95px; margin-right: 95px;">
  A view controller manages a discrete portion of your app’s user interface. Often that view is the root view for a more complex hierarchy of other views. View controllers typically fall in one of two categories, either content or container. Content view controllers are great for things like lists and images. Container view controllers coordinate with other view controllers like tab view controllers or navigation controllers.
</blockquote>

<p>
  It is also worth reading both the <a href="https://developer.apple.com/library/ios/#documentation/uikit/reference/UIViewController_Class/Reference/Reference.html">UIViewController</a> and <a href="https://developer.apple.com/library/ios/#documentation/uikit/reference/uiview_class/UIView/UIView.html">UIView</a> class references too. So what should be your major take away from reading those? The first is that every controller in your JavaScript application manages a view. That view is going to be a DOM node that will most likely have a complex hierarchy of other views possibly managed by other controllers which themselves champion a node with a specific application concern. Second is that views are themselves valid objects built by classes with their own methods. It's best to think of UIViews as an instance of a jQuery object. Views should not be dumb either. They sometimes have handles back to model objects so they can establish bindings for updates. Views can also expose an interface that canonize how controllers should work with them and most often will have helper methods that break up their complexity. All of this is a good thing. If you disagree, stop reading now!
</p>

<h2>SpacePen - Markup On The Final Frontier</h2>

<p>
  I found the <a href="https://github.com/nathansobo/space-pen">SpacePen project on github</a> a few months ago and I have never looked at another templating solution since. For the first time my JavaScript application's view and controller stacks feel natural. No longer do I find myself writing any view related code in my controllers. SpacePen allows me to write view objects that encapsulate their concerns, stay DRY and build Spine.JS web applications that hit the MVC mark. So what is SpacePen? Basically a CoffeeScript subclass of a jQuery object.
</p>

```ruby
class SpacePort extends View

  @content: ->
    @div =>
      @h1 "Space Ships"
      @ol =>
        @li "Apollo"
        @li "Soyuz"
        @li "Space Shuttle"
```

```ruby
view = new SpacePort
view.find('ol').append('<li>Star Destroyer</li>')

view.click 'li', ->
  alert "You clicked on #{$(this).text()}"
```

<p>
  This simple example shows how your SpacePen view class defines its markup in a <code>@content</code> class function. This function yields a builder syntax where you can easily add attributes and values. My advice is to do basic structure only in your <code>@content</code> function and rarely take advantage of the fact that it also takes the same parameters argument passed to your constructor. I will explain more on that advice later. But first, I did say that views should expose interfaces to controllers and are first class code citizens that champion their own concerns right? Let's look at a common pattern, views building new nodes within itself.
</p>

```ruby
class SpacePort extends View

  @content: ->
    @div =>
      @h1 "SpacePort"
      @ol outlet: 'shipList'

  addShip: (ship) ->
    @shipList.append "<li>#{ship.name}</li>"
```

```ruby
view = new SpacePort
view.addShip "Enterprise"
```

<p>
  The example has been changed slightly so that the view now exposes a public function called <code>addShip()</code> which takes an object to add to the view. It can easily implement this because I have declared the ordered list to be an outlet called <code>shipList</code> which will automatically be assigned as a property on each view instance. Calling append on that node is possible because like the view object itself, any outlet is a jQuery object as well.
</p>

<p>
  Though contrived, this example illustrates a powerful technique. If the view was refactored to some other structure besides an ordered list with items, the controller interface would stay the same. Pulling the thread, it is easy to see how a complex view hierarchy could have views building and managing messages to their subviews too. This is the crux of my article, but first, lets talk about controllers.
</p>

<h2>Spine.JS Controllers</h2>

<p>
  Spine.JS is my goto JavaScript MVC framework and if you are unfamiliar with it, check out their <a href="http://spinejs.com/docs/introduction">introduction</a> or read <a href="http://destroytoday.com/blog/reasons-for-spinejs/">why others</a> have chosen to use it. The main reasons I love Spine.JS can be summed up like so. First, it is authored in CoffeeScript, which means I can read the code to learn it. Second, all model instances will reflect changes by any other instance. Lastly, the controllers are minimally implemented and nicely abstract managing a view and the way controllers respond to events that bubble up to it.
</p>

<p>
  A Spine.JS controller always has an element associated to it which can be accessed via the <code>el</code> property. This element represents the view that controller manages. It will either be created for you or it can be assigned via the constructor. Many patterns in Spine.JS rely on appending elements to a controller and most of the functions for adding these subviews either take an element or an object that has an <code>el</code> property. This means building complex view/controller hierarchies by stacking Spine.JS controllers is both idiomatic and almost identical to how iOS view controllers work.
</p>

<h2>Using SpacePen Views With Spine.JS Controllers</h2>

<p>
  With the overview of these components out of the way, let's jump right into how they can work together to punch the VC up a notch in your next JavaScript MVC application. These examples below are pared down version of a scheduler application I just completed. The model is similar to an iCal day interface for calendar events. In such, there would be a view controller that manages the presentation of your schedule "by day". This controller's view would have a header for the current day and a set of controls for going to the previous day, today and the next day. This controller would act as a navigation controller and correlate to Apple's notion of a "container" view controller. When needed, it would find or create a day controller and append it to one of its subviews. The code for that container controller and view is out of the scope of this article, but its one of many day "content" controllers and view are perfect examples. Here we go!
</p>

```ruby
# In app/assets/javascripts/app/controller/day.js.coffee

class App.Controllers.Day extends Spine.Controller
  
  constructor: (params) ->
    @view = new App.Views.Day
    super el: @view
```

```ruby
# In app/assets/javascripts/app/views/day.js.coffee

class App.Views.Day extends View
  
  @content: (params) ->
    @div class: 'day', =>
      # ...

  initialize: (params) ->
    @attr 'data-date', params.moment.format('YYYY-MM-DD')
```

<p>
  So rule number one, always create an instance of a SpacePen view and assign that to the <code>@view</code> property of every Spine.JS controller. That <code>@view</code> will be assigned to the controller's <code>@el</code> property when it supers up. 
</p>

<p>
  Rule number two, keep the <code>@content</code> function for your views concerned with building the structure only. In my example, I set the date for the event using the <code>initialize</code> hook provided by SpacePen. This rule is important when you want to bind views to model events and update that view accordingly. All customization of the view to a model object should be done in a set/update helper of that view.
</p>

<h2>In Greater Detail</h2>

<p>
  Here is the same controller and view example with a bit more detail. First up, the view. This day view CoffeeScript file has a public <code>App.Views.Day</code> view and a private <code>EventView</code> that represents a subview and would be bound to a model instance. This public day view exposes a few public functions for a controller to hook into. The <code>addEvent</code> function will actually build a new SpacePen views for the passed model and append it as a subview. Notice too the <code>findEvent</code> helper function which can find the DOM element for a model object and then use the <code>view()</code> function provided by SpacePen to get to the SpacePen instance for this node.
</p>

```ruby
class App.Views.Day extends View
  
  @content: (params) ->
    @div class: 'day', =>
      # ...
      @div outlet: 'events', class: 'events'
      # ...
  
  initialize: (params) ->
    @date = params.moment.clone()
    @attr 'data-date', @date.format('YYYY-MM-DD')
    
  addEvent: (event) ->
    view = new EventView(event)
    @events.append
  
  updateEvent: (event) ->
    @findEvent(event)?.update(event)

  removeEvent: (event) ->
    @findEvent(event)?.remove()
  
  # Private
  
  findEvent: (event) ->
    selector = ".event[data-event-id='#{event.startsAtFormat()}']"
    @events.find(selector).view()

  afterAttach: -> 
    @startSpinner()
    @loadEvents()
  
  # ...

class EventView extends View
  
  @content: (event) ->
    @div class: '.event', 'data-event-id': event.id =>
      @div outlet: 'description'
      @div outlet: 'duration'
  
  initialize: (event) ->
    @update event
  
  update: (event) ->
    @attr 'data-starts-at', event.startsAtFormat()
    @description.text event.description
    @duration.text event.duration
```

<p>
  Next up, the expanded day controller example that manages the view above. In this example we bind to two of the the Spine.JS <code>CalendarEvent</code> model events. One for update and one for destroy. These events are then pushed down to the view and will do any work necessary. Just in case this controller is ever discarded for performance reasons, we hook into the <code>release()</code> function provided by Spine.JS controllers to remove the event listeners.
</p>

```ruby
class App.Controllers.Day extends Spine.Controller
  
  @events: 
    'click .event': 'selectEvent'
  
  constructor: (params) ->
    @date = params.moment
    @view = new App.Views.Day moment: @date
    super el: @view
    CalendarEvent.bind 'update', @updateEvent
    CalendarEvent.bind 'beforeDestroy', @destroyEvent
  
  release:
    CalendarEvent.unbind 'update', @updateEvent
    CalendarEvent.unbind 'beforeDestroy', @removeEvent
    super

  # Private

  selectEvent: ->
    # ...

  updateEvent: (event) ->
    @view.updateEvent event
    # ...

  removeEvent: (event) ->
    @view.removeEvent event
    # ...
```

<p>
  Hope you found this mini introduction to SpacePen and Spine.JS controllers useful. There is literally tons of ways you can use these tools together to make your controllers and views fun to build and maintainable. Be excellent and let me know what works for you!
</p>

<h2>Resources</h2>

<ul>
  <li><a href="http://elabs.se/blog/33-why-serenade-js">Why Serenade.js</a></li>
  <li><a href="http://addyosmani.com/blog/understanding-mvvm-a-guide-for-javascript-developers/">Understanding MVVM - A Guide For JavaScript Developers</a></li>
  <li><a href="http://addyosmani.com/blog/understanding-mvc-and-mvp-for-javascript-and-backbone-developers/">Understanding MVC And MVP (For JavaScript And Backbone Developers)</a></li>
  <li><a href="http://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/Introduction/Introduction.html">View Controller Programming Guide for iOS</a></li>
  <li><a href="https://github.com/nathansobo/space-pen">SpacePen - Markup On The Final Frontier</a></li>
  <li><a href="http://spinejs.com/docs/introduction">An Introduction To Spine.JS</a></li>
  <li><a href="http://destroytoday.com/blog/reasons-for-spinejs/">10 Reasons Why I Switched To Spine.JS</a></li>
  <li><a href="/2012/01/15/rails-and-spine-js-using-the-coffeescript-source/">Rails &amp; Spine.JS - Using The CoffeeScript Source</a></li>
</ul>
