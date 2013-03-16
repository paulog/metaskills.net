---
layout: post
title: How Do You Encapsulate Your JavaScript
categories: 
  - javascript
---

<aside class="flash_info">
  This page has been translated into <a href="http://www.webhostinghub.com/support/es/misc/encapsular-javascript">Spanish</a> language by Maria Ramos from <a href="http://www.webhostinghub.com/">Webhostinghub.com</a>.
</aside>

<p>
  I ask this question a lot! To Job candidates, friends, and almost any developer that says they work with JavaScript. I believe how you encapsulate your JavaScript is a good indicator on your level of expertise with the language. I find that most beginners have come to JavaScript via jQuery and often define their functions at the top level namespace in some application.js file. These functions are loosely organized and often have no way of sharing simple object state and behavior. A good object oriented programmer would never write their application code like that. JavaScript should be no different and I think jQuery has a part in the blame of those not knowing this.
</p>

<p>
  I've <a href="/2008/08/18/in-hell-oo-for-homemarks/">been writing rich object oriented JavaScript</a> for the better part of 4 years now using <a href="http://www.prototypejs.org/">Prototype.js</a>. That framework has always championed encapsulating your web application's behavior using a <a href="http://prototypejs.org/learn/class-inheritance">standard class and inheritance notation</a>. Sam Stephenson, the author of Prototype.js, did a good job trying to educate what real JavaScript Object Notation (JSON) is about and how you can encapsulate behavior using Prototype's Class structure while paying homage to <a href="http://dean.edwards.name/weblog/2006/03/base/">Dean Edward's Base.js</a> to which it owes its origins.
</p>

<p>
  So jQuery is the proper winner and choice for anyone wanting to use a JavaScript framework to make working with the DOM in web sites easier. It has awesome event handling and delegation. So many things that Prototype.js lacked. It even has great documentation. About the only bad thing about jQuery is the widely different interfaces to it's AJAX functions and the arguments passed to their callbacks. Oh, and they totally screwed new JavaScript developers by not sheperding them into learning some sort of way to encapsulate their object behavior. So this article is for those that are not using <a href="http://jashkenas.github.com/coffee-script/">CoffeeScript's class system</a>, <a href="http://documentcloud.github.com/backbone/">Backbone.js's class system</a>, or jQuery's deeply nested plugin class system that is hidden away. So lets get to fixing that. Here is 32 lines of dirt simple JavaScript inheritance.
</p>

```javascript
/* Simple JavaScript Class
   By John Resig. MIT Licensed. <http://ejohn.org/blog/simple-javascript-inheritance/> */
(function(){
  var initializing = false, fnTest = /xyz/.test(function(){xyz;}) ? /\b_super\b/ : /.*/;
  this.JQClass = function(){};
  JQClass.extend = function(prop) {
    var _super = this.prototype;
    initializing = true;
    var prototype = new this();
    initializing = false;
    for (var name in prop) {
      prototype[name] = typeof prop[name] == "function" && 
        typeof _super[name] == "function" && fnTest.test(prop[name]) ?
        (function(name, fn){
          return function() {
            var tmp = this._super;
            this._super = _super[name];
            var ret = fn.apply(this, arguments);        
            this._super = tmp;
            return ret;
          };
        })(name, prop[name]) :
        prop[name];
    }
    function JQClass() {
      if ( !initializing && this.init )
        this.init.apply(this, arguments);
    }
    JQClass.prototype = prototype;
    JQClass.prototype.constructor = JQClass;
    JQClass.extend = arguments.callee;
    return JQClass;
  };
})();
```

<p>
  It is <a href="http://ejohn.org/blog/simple-javascript-inheritance/">worth reading the comments</a> John Resig's blog post for this piece of work as well as the full documentation. My code example above names this <code>JQClass</code> to avoid namespace collisions with other frameworks. Here is a simple example of its usage.
</p>

```javascript
// Your namespace.

var MyApp = {
  models: {}
};

// Simple object example for page flash.

MyApp.models.flash = JQClass.extend({
  
  init: function(aPage) {
    this.page = $(aPage);
    this.alertDiv = this.page.find('div.flash.alert');
    this.noticeDiv = this.page.find('div.flash.notice');
  },
  
  alert:  function(content) { this._doFlashFor(this.alertDiv,content); },
  
  notice: function(content) { this._doFlashFor(this.noticeDiv,content); },
  
  clear: function() {
    # ...
  },
  
  _doFlashFor: function(flash, content) {
    $.mobile.silentScroll(0);
    this.clear();
    flash.html(content);
    flash.show();
  }
  
});
```


<p>
  This code example is pulled from a jQuery mobile project I recently finished. The real usage of the flash objects would be numerous as there would be one per page and I have a top level delegate object that finds the active page and finds or create the flash DOM elements. But this should be a good enough example to show why you might choose a simple class and object inheritance system vs putting all your logic in <code>$(document).ready({})</code> scopes. There are tons of ways to write good OO JavaScript and I hope some find this useful and explore some other ways.
</p>


