--- 
layout: post
title: RESTful AJAX with Forgery Protection
disqus_id: /2008/06/18/restful-ajax-with-forgery-protection/
categories: 
  - heuristics
  - ruby-rails
  - javascript
---

<p>
  Writing the new <a href="http://www.homemarks.com/">HomeMarks</a> has been a great exercise. I've learned that the <a href="/2008/05/24/the-ajax-head-design-pattern/"><em>AJAX Head <strike>Design Pattern</strike> Implementation</em></a> is more akin to developing a service-oriented application (SOA) since I have moved all client-side coupling from the controllers, like RJS, and only respond with HEAD or JSON data. Today I learned about <a href="http://halcyon.rubyforge.org/">Halcyon</a> which is self described as a JSON web application framework built on Rack. If you take a look at their example, it looks a lot like HomeMarks v2, pretty cool!
</p>

<h2>Getting To The Point</h2>

<p>
  Last week I finally got around to doing the controller CRUD for the user's column objects. The core <code>homemarks/app.js</code> JavaScript class contains the one (and only one) AJAX dispatcher/handler for the entire application. One problem that I ran into was how to continue to use forgery protection in rails to protect from CSRF attacks while relying solely on ad-hoc JavaScript requests to a RESTful back-end. I could have turned off <a href="http://api.rubyonrails.com/classes/ActionController/RequestForgeryProtection/ClassMethods.html#M000300"><code>protect_from_forgery</code></a> but that did not feel right. I wanted RESTful AJAX with forgery protection.
</p>

<h2>My Solution</h2>

<p>
  Since the HomeMarks v2 app has one AJAX dispatcher/handler for the entire app, all I had to do was to get the authenticity token into each and every request. It took a bit of digging in the rails source to find the methods I wanted but here both the ERB snippet I am using in my layout and the helper method found in my <code>application_helper.rb</code> file.
</p>

~~~html
<script type="text/javascript">
  <%= auth_params_js_var %>
</script>
~~~

~~~ruby
def auth_params_js_var
  unless RAILS_ENV == 'test'
    %|var authParams = $H({#{request_forgery_protection_token}:#{form_authenticity_token.inspect}});|
  end
end
~~~

<p>
  What does this do? It creates a JS variable called <code>authParams</code> at the window level (right term?) that all other functions can use. That variable returns a JavaScript hash that will have a single key/value pair for the rails authenticity token, see below. Basically at this point all you have to do is merge this hash with your <code>parameters</code> option for your AJAX object. You can reference the HomeMarks app.js file in the resources section if you want to see how I did this on my app, code snippet below too.
</p>

~~~javascript
doAjaxRequest: function(elmnt,finishMethod) {
  // ...
  var method = elmnt.method || 'post';
  new Ajax.Request(elmnt.action,{
    onComplete: function(request){
      this.completeAjaxRequest(request);
      if (finishMethod) { finishMethod.call(this,request) };
    }.bind(this),
    parameters: parameters.merge(authParams),
    method: method
  });
}
~~~


<h2>Resources</h2>

<ul>
  <li>The Latest HomeMarks <a href="http://github.com/metaskills/homemarks/tree/master/public/javascripts/homemarks/app.js">app.js</a> Class. See doAjaxRequest() function.</li>
</ul>

