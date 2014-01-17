--- 
layout: post
title: The "AJAX Head" Design Pattern
disqus_id: /2008/05/24/the-ajax-head-br-design-pattern/
categories: 
  - heuristics
  - ruby-rails
  - javascript
---


<aside class="flash_info">
  Chris Williams did a really great write up on this pattern with great details on when/how to use it. Please considering <a href="http://voodootikigod.com/ajax-head-design-pattern">reading it</a> afterward.
</aside>

<p>
  This is the first of a few articles covering the total rewrite of the <a href="http://homemarks.com/">HomeMarks.com</a> code base as I upgrade it to Rails 2.1. The "AJAX Head" pattern is the moniker I have assigned to methodology that has come about during said rewrite and the design decisions I choose early on. The rules were simple.
</p>

<ul>
  <li>Total Unobtrusive JavaScript (<a href="http://en.wikipedia.org/wiki/Tag_soup">No Tag Soup</a>)</li>
  <li>No RJS Controller Responses</li>
</ul>

<p>
  Rails makes it very easy to generate JavaScript with Ruby. The generator methods are extensive and cover both the Prototype and Scriptaculous libraries. The first version of HomeMarks used the generators exclusively to augment both the views and controller responses. In fact, I used a technique that is covered in the latest <a href="http://www.pragprog.com/titles/fr_arr/advanced-rails-recipes"><em>Advanced Rails Recipes</em></a> (way before it came out) under the "Replace In-View Raw JavaScript" chapter. Basically I created helper methods that generated JavaScript shared by both the views and controllers. It was a great technique but things <a href="http://github.com/metaskills/homemarks/tree/d33dab1405537db2d2dab86d79db415d9e7f56e5/app/helpers/application_helper.rb">got messy quick</a>. So life was good back then, I had a lot of Ruby code and a very dynamic site, but something did not seem right...
</p>


<h2>What Is The AJAX Head Pattern?</h2>

<p>
  It is an experiment into a vision to see what happens when you make the decision to totally go unobtrusive. Not just in your views but the controllers too! Imagine it this way &mdash; your controllers are <strong>slim APIs</strong>. They should do nothing but render a page on a GET request and when it comes to a POST/PUT/DELETE they should do nothing more than just say YES or NO (with errors).
</p>

<p>
  Now lets think about that. What does that mean for an application that uses exclusive AJAX calls for dynamic page updates? It means that your visual client (the browser) will need to know ALOT less from the controller about what to do. It's great, the browser knows what is on the page, from the GET request, and what it needs to do with it. The client only needs to tell the remote resource store what it did. In this pattern the controller is relegated to nothing more than acting on model data and letting the client know if it succeeded or failed. So in short.
</p>

<blockquote>
  The AJAX head design pattern forces the view and controller to work in isolation with the most minimal coupling possible. Kind of like a web service.
</blockquote>


<h2>Tools and Code Needed</h2>

<ul>
  <li>A application that does just about everything with AJAX.</li>
  <li>Fluency with ActionController::Base#head method and HTTP status codes.</li>
  <li>Site-wide exception rescues for ActiveRecord::RecordInvalid.</li>
  <li>Global client-side AJAX request and response handlers.</li>
  <li>Global client-side error handlers.</li>
</ul>

<p>
  This may seem like a daunting list, but its pretty simple when you break it down. Let's do each one at at time with some of the current code from HomeMarks. The HomeMarks app is my first check on the list. When I did this app just about EVERYTHING was done via an AJAX call to update the page. Now on to the others.
</p>


<h2>Head And Status Codes</h2>

<p>
  The <a href="http://api.rubyonrails.com/classes/ActionController/Base.html#M000454">ActionController::Base#head</a> method and <a href="http://dev.rubyonrails.org/browser/trunk/actionpack/lib/action_controller/status_codes.rb">HTTP status codes</a> are the second item on the list. The head method is very useful when you just want to respond with HTTP headers and no body content. This is typical behavior when an app responds to external web services requests. This is important to note, I chose to rely on head responses as a way to meet my design decision of not using any RJS responses from the controller.
</p>
  
  
<p>
  The basic usage of the head method is simple, you just pass a symbol to the head method that is the lowercase underscore equivalent to an HTTP status code. For instance, if you wanted to respond with success, choose a status code from the 200 range, the easiest being 'OK', which would look like <code>head(:ok)</code>. If you wanted to respond to bad authentication, you might use the 401 code like so, <code>head(:unauthorized)</code>. Here is an example from HomeMarks restful user signup action.
</p>

~~~ruby
class UsersController < ApplicationController
  # ...
  def create
    User.create!(params[:user])
    head :ok
  end
  # ...
end
~~~

<p>
  There is actually a number of things going on in those 2 lines of code above. If the user is created, the AJAX request will get a simple 'OK' message via the head method. Obviously the client will need to know what to do on its own. But if errors happen what to do? This brings us to...
</p>


<h2>Invalid Object Handling</h2>

<p>
  When Rails 2 came out, it gave us a very useful tool for our controllers, the <a href="http://api.rubyonrails.com/classes/ActionController/Rescue/ClassMethods.html">ActionController#rescue_from</a> method. It allows you to delegate exception handling to a method either per controller or site wide for a specific exception class. So enter the <a href="http://github.com/metaskills/homemarks/tree/master/lib/render_invalid_record.rb">RenderInvalidRecord</a> module, show below.
</p>

~~~ruby
module RenderInvalidRecord
  
  def self.included(base)
    base.rescue_from(ActiveRecord::RecordInvalid, :with => :render_invalid_record)
  end
  
  protected
  
  def render_invalid_record(exception)
    record = exception.record
    respond_to do |format|
      format.html { render :action => (record.new_record? ? 'new' : 'edit') }
      format.js   { render :json => record.errors.full_messages, :status => :unprocessable_entity, :content_type => 'application/json' }
      format.xml  { render :xml => record.errors.full_messages,  :status => :unprocessable_entity }
      format.json { render :json => record.errors.full_messages, :status => :unprocessable_entity }
    end
  end
  
end
~~~

<p>
  Once included in your application.rb file, every ActiveRecord::RecordInvalid exception will be handled by the render_invalid_record method. I use this pattern often and it is the primary reason I always use the <strong>bang methods</strong> that end with an exclamation point, such as <code>create!</code> or <code>save!</code>. This module above was inspired by <a href="http://github.com/josh/">Joshua Peek's</a> own module of the same name but has been edited here to fit my needs. Both the "xml" and "html" response formats are not used by HomeMarks but are included here to show how this module can work for any request type.
</p>

<p>
  Now, how this fits into the AJAX Head pattern. The design decision of not having RJS responses means one of two things needs to come from the response. First, if the response is a success, meaning in the 200 range, then it is assumed that the client only needs to know that all is 'OK'. Second, if the request has errors, then the response needs to communicate them. I choose JSON for the format of these errors since all AJAX request objects from Prototype automatically include a <code>application/json</code> accept header. So that is why I changed both the "js" and the "json" response type in the RenderInvalidRecord module to both serialize the objects full error messages into a JSON array with an HTTP status code of 422 "Unprocessable Entity" which seemed to make the most sense to me. This pretty much wraps up the server side part of the pattern. Now onto the client side. Everything from here on is personal taste on how your client will react to an "good" or "bad" response object.
</p>


<h2>Client-Side AJAX Handlers</h2>

<p>
  I can not stress enough how much you have to bake your own functions to handle the client side reaction to both good and bad AJAX responses. The client's ability to know what to do with errors will be tightly coupled to AJAX handlers and the design choices of your site. In the examples below I have broken up methods in the same JavaScript classes to show you the parts for a specific behavior in a more concise manner. Also please note that I am using Prototype module and class organization heavily in HomeMarks. This allows me to build a Base JavaScript module and classes that mimic Rails models which inherit/mixin Base methods. I also use the Scriptaculous Builder class for creating DOM fragments too. Now back to business.
</p>

<div class="center">
  <video title="HomeMarks v2 Signup" src="/assets/signup.mp4" width="617" height="473" controls preload></video>
</div>

<p>
  </a>The first part of the AJAX head pattern was to use unobtrusive JavaScript. Because of this design choice, there are no <code>remote_form_for</code> methods in the HomeMarks view code. All forms are simple <code>form_for</code> methods. Below is a sample of the HomeMarksSite JavaScript class that creates AJAX handlers for all site forms. The below example is slimmed down to only show the initialization of this behavior for the signup form. But no matter which form is given this behavior the process is the same.
</p>

<ul class="ml20 mr20 mb20">
  <li>The form's submit event is handled by <code>startAjaxForm()</code>. It does the following:</li>
    <ul>
      <li>Stops the normal submit behavior.</li>
      <li>Adds a loading spinner in the #form_loading span.</li>
      <li>Creates a new AJAX.Request using the forms own action. It also binds a function to delegate the logic of completing the form when the request is finished. More on that later in #2 below.</li>
      <li>Finally it disables the form.</li>
    </ul>
  <li>Once a response comes back from the server, the <code>delegateCompleteAjaxForm()</code> function will be called with both the form object and the AJAX request object. The logic of this function includes:</li>
  <ul>
    <li>Figuring out the "mood" of the request using getRequestMood() function which is part of my base module.</li>
    <li>Sends the form and mood to the <code>completeAjaxForm()</code> function. This function will replace the loading spinner with an image that reflects a good or bad response.</li>
    <li>If the mood is good, meaning the response status code was within a 200 range, then the function will delegate to a function specific for that forms success using a simple case statement.</li>
    <li>If this mood is bad, meaning the response status code not within a 200 range, the form will be enabled and the response JSON errors will be displayed. Currently I am using a modal class for this. Again more on that later. You may also note that my base module has a function called <code>messagesToList()</code> that process the response into a DOM &lt;ul&gt; list.</li>
  </ul>
</ul>

~~~javascript
var HomeMarksBase = {
  getRequestMood: function(request) {
    return (request.status >= 200 && request.status < 300) ? 'good' : 'bad';
  },
  messagesToList: function(request) {
    var messages = request.responseJSON;
    var messageList = UL();
    messages.each(function(message){ 
      messageList.appendChild( LI(message.escapeHTML()) );
    });
    return messageList;
  }
};
var HomeMarksSite = Class.create(HomeMarksBase,{ 
  initialize: function() {
    this.signupForm = $('signup_form');
    this.initEvents();
  },
  startAjaxForm: function(event) {
    event.stop();
    var form = event.findElement('form');
    var options = Object.extend({imgSrc:'loading_invert.gif'}, arguments[1] || {});
    var loadArea = form.down('#form_loading');
    var imgTag = IMG({src:('/images/'+options.imgSrc)});
    $(loadArea).update(imgTag);
    new Ajax.Request(form.action,{
      onComplete: function(request){ this.delegateCompleteAjaxForm(form,request) }.bind(this),
      parameters: form.serialize(true)
    });
    form.disable();
  },
  delegateCompleteAjaxForm: function(form,request) {
    var mood = this.getRequestMood(request);
    this.completeAjaxForm(form,{mood:mood});
    if (mood == 'good') { 
      switch (form) { 
        case this.supportForm : this.completeSupportForm(request); break;
      }
    }
    else { 
      form.enable();
      if (!request.responseText.blank()) {
        var flashHTML = DIV([H2('Errors On Form!'),this.messagesToList(request)]);
        this.flashModal('bad',flashHTML);
      };
    };
  },
  completeAjaxForm: function(form) {
    var options = Object.extend({mood:'good'}, arguments[1] || {});
    var loadArea = form.down('#form_loading');
    var completeId = 'complete_ajax_form_' + form.id;
    var imgSrc = '/images/'+options.mood+'.png';
    var moodHtml = SPAN({id:completeId,className:'m0 p0'}, [IMG({src:imgSrc})]);
    $(loadArea).update(moodHtml);
    setTimeout(function() { $(completeId).fade(); },3000);
  },
  completeSignupForm: function(request) {
    var message = "Thank your for signing up for your own HomeMarks page. An email has been sent to " +
      "your address along with a link to activate your account. If you have not done so already, please " +
      "take a moment to read the HomeMarks documentation.";
    var flashHTML = DIV([H2('Signup Complete:'),P(message)]);
    this.flash('good',flashHTML);
  },
  initEvents: function() {
    if (this.signupForm) { this.signupForm.observe('submit', this.startAjaxForm.bindAsEventListener(this)); };
  }
});
~~~


<h2>Client-Side Error Handling</h2>

<p>
  Currently I have 3 client-side options that I can use to present errors back to the user from the JSON array in AJAX response. The first I started out with was using the <code>alert()</code> function. That got boring real quick, but I left my implementation below which is part of the base class, called <code>messagesToAlert()</code>.
</p>

<p>
  The second slightly more interesting way was to create a flash message that used the same XHTML/CSS already present in my layout file. This was real easy to implement. Basically using the ERB fragment below in my layout, I guaranteed that I had 3 unique DOM elements for each of my flash message styles, good/bad/indif. Now all I had to do was to create an accessor in my HomeMarksSite class. I choose <code>this.flashes</code>. I then created a <code>clearFlashes()</code> function and then a <code>flash()</code> function that would take the mood and the HTML to embed, viola â€“ I can now call <code>this.flash.('good',someHTML)</code> and I get the same type of flash one would have seen if I set it in the controller and rendered. Note, this is what I used on the the signup form success above.
</p>

~~~ruby
<% [:good,:bad,:indif].each do |key| %>
  <div id="flash_<%= key %>" class="flash_message" style="display:<%= flash[key].blank? ? 'none' : 'block' %>;">
	  <span><%= flash[key] %></span>
	</div>
<% end %>
~~~
<br />

~~~javascript
var HomeMarksBase = {
  messagesToAlert: function(request) {
    var messages = request.responseJSON;
    var alertText = messages.join(".\n");
    if (alertText.endsWith('.')) { alert(alertText); } else { alert(alertText+'.'); };
  },
  messagesToList: function(request) {
    var messages = request.responseJSON;
    var messageList = UL();
    messages.each(function(message){ 
      messageList.appendChild( LI(message.escapeHTML()) );
    });
    return messageList;
  }
};
var HomeMarksSite = Class.create(HomeMarksBase,{ 
  initialize: function() {
    this.flashes = $$('div.flash_message');
  },
  clearFlashes: function() {
    this.flashes.invoke('hide');
    this.flashes.invoke('update','');
  },
  flash: function(mood,html) {
    this.clearFlashes();
    var moodFlash = this.flashes.find(function(e){ if (e.id == 'flash_'+mood) {return true}; });
    moodFlash.update(html);
    moodFlash.show();
    $('site_wrapper').scrollTo();
  }
});
~~~

<div class="center">
  <video title="HomeMarks v2 Signup" src="/assets/login.mp4" width="617" height="473" controls preload></video>
</div>

<p>
  Lastly, and probably the coolest, is the HomeMarksModal JavaScript class I created. This is my new default way of errors to a user. It can be seen in all my video examples on this page. To the right is another example of the login form that uses the same handlers described above. The HomeMarksModal JavaScript class is a bit large to paste inline in this article, but if you want to see the latest version, you can always get it <a href="http://github.com/metaskills/homemarks/tree/master/public/javascripts/homemarks/modal.js">from the master branch of the homemarks_core project</a> on Github.com. When you bind an instance of this object to the window/document, it will automatically crate the modal HTML using Builder. If you want to use it, make sure to get the CSS and images the project too. Below is a function that I put in my site class that allows me to call <code>this.flashModal('good',someHTML))</code>.
</p>

~~~javascript
var HomeMarksSite = Class.create(HomeMarksBase,{ 
  flashModal: function(mood,html) {
    var moodColor = this.flashMoodColors.get(mood);
    HmModal.show(html,{color:moodColor});
  }
});
~~~


<h2>Wrapping It Up</h2>

<p>
  This article turned out a lot longer than I had wanted. The AJAX head pattern is pretty simple and it is really fun to see a two line user signup action yield such interactive results. Something also feels good about not putting view code in the controller, yes you can look at inline RJS as view code. Sure it requires that you do a lot more JS work, but there are benefits.
</p>

<p>
  Image you have team of programmers, one person can stay in model/controller land while the other stays on the view/javascript. The model/controller person would write their own tests (PLEASE TAKE A LOOK AT MINE), while the view/javascript person might even take the initiative to use selenium to test the JavaScript. At the time of this writing, I have not done any selenium tests since the plugin for rails is not compatible for 2.0. That last benefit I could think of for this design pattern is that you could easily update view code without restarting your rails app. Just update the JavaScripts and your good to go.
</p>

<p>
  In closing, please let me know what you think. Perhaps you have a better name for the design pattern? Perhaps someone else has a name for it already?
</p>



<h2>Resources</h2>

<ul>
  <li>Original <a href="http://github.com/metaskills/homemarks/tree/d33dab1405537db2d2dab86d79db415d9e7f56e5/app/helpers/application_helper.rb">application_helper.rb</a> from HomeMarks v1. An overboard example of shared view/controller tagsoup/rjs.</li>
  <li>The latest <a href="http://www.pragprog.com/titles/fr_arr/advanced-rails-recipes"><em>Advanced Rails Recipes</em></a> book. See the chapter on "Replace In-View Raw JavaScript". I discourage this technique used in mass.</li>
  <li>Rails API for the <a href="http://api.rubyonrails.com/classes/ActionController/Base.html#M000454">ActionController::Base#head</a> method.</li>
  <li>Rails API for the <a href="http://dev.rubyonrails.org/browser/trunk/actionpack/lib/action_controller/status_codes.rb">HTTP status codes</a>.</li>
  <li>Rails API for the <a href="http://api.rubyonrails.com/classes/ActionController/Rescue/ClassMethods.html">ActionController#rescue_from</a> method.</li>
  <li>The latest <a href="http://github.com/metaskills/homemarks/tree/master/lib/render_invalid_record.rb">RenderInvalidRecord</a> module from the HomeMarks core.</li>
  <li>The latest  <a href="http://github.com/metaskills/homemarks/tree/master/public/javascripts/homemarks/modal.js">HomeMarksModal</a> JavaScript class from the HomeMarks core.</li>
</ul>






