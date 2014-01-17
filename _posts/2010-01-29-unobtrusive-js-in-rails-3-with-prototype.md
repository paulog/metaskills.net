--- 
layout: post
title: Unobtrusive JS In Rails 3 With Prototype 
disqus_id: /2010/01/29/unobtrusive-js-in-rails-3-with-prototype/
categories: 
  - heuristics
  - ruby-rails
  - javascript
---


<aside class="flash_info">
  An updated version of UJS and forgery protection in Rails3 <a href="/2010/4/13/restful-ajax-with-forgery-protection-with-rails-3">here</a>.
</aside>

<p>
  Are you bleeding on the edge of rails 3 and need to shim up some unobtrusive JavaScript to work with your <code>link_to</code> code that uses a destructive :method option? I did today and here is what I did to solve it. If you are unfamiliar with the problem, and what has been happening in rails 3 with UJS, check out <a href="http://blog.solnic.eu/2009/09/08/unobtrusive-javascript-helpers-in-rails-3">Piotr Solnica's</a> blog for a good run down. Or you can check out the simple code block below.
</p>

~~~ruby
# This Ruby
link_to 'Logout', session_path, :method => :delete

# Will out put this HTML in Rails 3
# <a href="/session" data-method="delete" data-url "/session" rel="nofollow">Logoout</a>
~~~

<p>
  So no more tag soup. Yea! There was much rejoicing, but I could not find any illustrated examples of what type of JavaScript to use to back this up. There are two problems at play here. First, simple link <code>&lt;a&gt;</code> tags are not forms and hence methods like post/put/delete are not going to be solved with simple query params. Second, because link tags are not forms, there is no authenticity_token hidden in the generated form, like a <code>button_to</code> would generate. Let's ignore the second problem for a bit and see what I came up with for using with Prototype.
</p>

~~~javascript
var MyJsObject = {
  
  linkToDelete: function(event) {
    event.stop();
    var e = event.element();
    var a = e.readAttribute('href') ? e : e.up('a');
    var form = FORM({action:a.href, method:'post'},[
      INPUT({type:'hidden',name:'_method',value:'delete'}),
      INPUT({type:'hidden',name:'authenticity_token',value:authParams.get('authenticity_token')})]
    );
    form.submit();
    return false;
  }
  
};

document.observe('dom:loaded', function(){
  $$('a[data-method=delete]').each(function(a){ a.observe('click', MyJsObject.linkToDelete); });
});
~~~

<p>
  In this example I am only covering the DELETE action. This code could be abstracted to take a "method" argument and use it. As you can see, it's just a simple iteration over the links with custom "data-method" attributes and attaching a function to it. For brevity, I am using the Builder.js methods for building DOM objects. See anything odd up there? The authentication token? You got it, destructive actions like POST/DELETE/PUT will need that token. I wrote up a great article a year or so ago that still works today titled <a href="/2010/4/13/restful-ajax-with-forgery-protection-with-rails-3">RESTful AJAX with Forgery Protection (In Rails3)</a> that gives me that nice global var. Check it out for the last piece to this puzzle and have fun working on the edge!
</p>



