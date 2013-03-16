--- 
layout: post
title: RESTful AJAX with Forgery Protection (In Rails 3)
disqus_id: /2010/04/13/restful-ajax-with-forgery-protection-with-rails-3/
categories: 
  - ruby-rails
  - javascript
---


<p>
  A <a href="/2008/6/18/restful-ajax-with-forgery-protection">while back ago</a> I wrote an article about how to use Rails built-in forgery protection in your RESTful AJAX calls. Normally AJAX requests, those responding true to <code>request.xhr?</code> in rails, are forgery whitelisted. But sometimes, and under what conditions I am not sure, AJAX methods are subjected to forgery protection. Maybe you have the <code>ActionDispatch::Request#forgery_whitelisted?</code> overridden to not include AJAX requests? Either way and for whatever reason â€“ if you like to use forgery protection in your RESTful AJAX calls to rails, then here is the new implementation under Rails 3 beta2.
</p>

<h2>Forgery Protection In Rails 3</h2>

<p>
  In a pre Rails 3 application all form tag helpers automatically create a hidden form field that contain the authentication token. These are passed when the form is serialized and if you viewed source, you could see them. However, Rails 3 is unobtrusive, which means you can count on them not polluting your markup with this kind of framework data. So where do the authentication params come from in a Rails 3 application? The answer is two places. First there is a new <code>ActionView::Helpers::CsrfHelper</code> module that allows you to use the #csrf_meta_tag helper in the head of your layout files. This helper will generate two meta tags, one for the authentication parameter name and the other for the value. The second place is in the new rails.js which reads these two meta tags for further interpolation into some generated form tags for all sorts of :remote links, buttons, etc. If your interested in reading more about these, look into the source, but I thought you might like to see some places where these are used by the framework now.
</p>

```html
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  <%= csrf_meta_tag %>
```

```javascript
document.observe("dom:loaded", function() {
  var authToken = $$('meta[name=csrf-token]').first().readAttribute('content'),
    authParam = $$('meta[name=csrf-param]').first().readAttribute('content'),
    formTemplate = '<form method="#{method}" action="#{action}">\
      #{realmethod}<input name="#{param}" value="#{token}" type="hidden">\
      </form>',
    realmethodTemplate = '<input name="_method" value="#{method}" type="hidden">';
  //... etc
```


<h2>Now Our RESTful AJAX with Forgery Protection</h2>

<p>
  OK, now that bit of background information is out of the way, lets look into how we can use this for ourselves. Since Rails 3 has provided us with a good convention, we can hook into this pretty easily. This prototype example below is pretty simple and follows my OOJS approach. I have a base JS object that is basically a module with a function called authParams. I then mixin this base JS object into all my Prototype classes. We are reading the same meta tags that rails now uses and create a Prototype Hash object with the name/value of the authentication token. Then in ever AJAX request I merge these parameters in with what ever I am serializing for that request. Pretty simple eh? Did I miss anything? 
</p>

```javascript
// A base JS object.

var MyBase = {

  authParams: function() {
    var params = {}
    var authParam = $$('meta[name=csrf-param]').first().readAttribute('content');
    var authToken = $$('meta[name=csrf-token]').first().readAttribute('content');
    params[authParam] = authToken
    return $H(params);
  }

};

// Mixed Into Other Classes.

var SomeClass = Class.create(MyBase,{

  doAjaxRequest: function() {
    new Ajax.Request(elmnt.action,{
      parameters: this.form.serialize(true).merge(this.authParams()),
    });
  }

});
```


