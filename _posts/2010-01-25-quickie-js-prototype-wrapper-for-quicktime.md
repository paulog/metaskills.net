--- 
layout: post
title: Quickie.js - Prototype wrapper for QuickTime
disqus_id: /2010/01/25/quickie-js-prototype-wrapper-for-quicktime/
categories: 
  - apple-os
  - heuristics
  - javascript
---


<aside class="flash_info">
  UPDATE: May 15th, 2011 - Today I decided to to re-write this class as my first dive into both CoffeeScript and jQuery. You can find an <a href="https://gist.github.com/973483">updated version in this gist on github</a>.
</aside>

<p>
  As some of you know, I am in the last steps of announcing my first iPhone application. We all know that every good iPhone application has a great marketing website with a screen cast. I myself was heavily inspired by the <a href="http://tapbots.com/convertbot">Tapbots Convertbot</a> website while building my own and wanted a good way of embedding the screen cast. I think the last time I did an object/embed tag was god... around 2003 or something, seriously.
</p>

<p>
  So after looking at the source of the Convertbot website, I was happy to find a nice little JavaScript wrapper for QuickTime called Quickie.js. This is a MooTools class created by Jose Prado that helps generate an object or embed tag appropriate for the requesting browser in an abstract way. Here is the link to the <a href="http://pradador.com/code/quickiejs/">original Quickie.js</a>. If you use MooTools, fine, it is real close to Prototype and has my respect, but if you use Prototype, here is my rewrite of the class.
</p>


```javascript
/*

Author:
  Ken Collins <metaskills.net>

Options:
  id - (string: defaults to 'Quickie_' + unique id) The id of the Quickie object.
  width - (number: defaults to 1) The width of the Quickie object.
  height - (number: defaults to 1) The height of the Quickie object.
  container - (element) The container into which the Quickie object will be injected.
  attributes - (object) QuickTime attributes for the element. See http://www.apple.com/quicktime/tutorials/embed.html for possible attributes.
  
Returns:
  - (element) A new HTML object Element with browser appropriate QuickTime embed code.

Example:
  var myQuickie = new Quickie('myMovie.mov', {
    id: 'myQuicktimeMovie',
    width: 640,
    height: 480,
    container: 'qtmovie',
    attributes: {
      controller: 'true',
      autoplay: 'false'
    }
  });

Credits:
	Mootools Implementaiton: http://pradador.com/code/quickiejs/
	
License:
	MIT-Style License

*/

var Quickie = Class.create({
  
  initialize: function(path, options){
    var time = Try.these(
      function() { return Date.now() },
      function() { return +new Date }
    );
    var defaultOptions = {
      id: null,
      height: 1,
      width: 1,
      container: null,
      attributes: {}
    }
    this.path = path;
    this.instance = 'Quickie_' + time;
    this.options = Object.extend(defaultOptions, arguments[1] || {});
    this.id = this.options.id || this.instance;
    this.container = $(this.options.container);
    this.attributes = this.options.attributes;
    this.attributes.src = this.path;
    this.height = (this.attributes.controller == 'true') ? this.options.height + 16 : this.options.height; 
    this.width = this.options.width;
    this._assignElement();
  },
  
  toElement: function() {
    return this.element;
  },
  
  _assignElement: function() {
    var build = (Prototype.Browser.IE) ? this._buildObjectTag() : this._buildEmbedTag();
    this.element = ((this.container) ? this.container.update('') : new Element('div')).update(build).down();
  },
  
  _buildObjectTag: function() {
    var build = "";
    build = '<object classid="clsid:02BF25D5-8C17-4B23-BC80-D3488ABDDC6B" codebase="http://www.apple.com/qtactivex/qtplugin.cab"';
    build += ' id="' + this.id + '"';
    build += ' width="' + this.width + '"';
    build += ' height="' + this.height + '"';
    build += '>';
    for (var attribute in this.attributes) {
      if (this.attributes[attribute]) {
        build += '<param name="' + attribute + '" value="' + this.attributes[attribute] + '" />';
      }
    }
    build += '</object>';
    return build;
  },
  
  _buildEmbedTag: function() {
    var build = "";
    build = '<embed ';
    build += ' id="' + this.id + '"';
    build += ' width="' + this.width + '"';
    build += ' height="' + this.height + '"';
    for (attribute in this.attributes) {
      if (this.attributes[attribute]) {
        build += ' ' + attribute + '="' + this.attributes[attribute] + '"';
      }
    }
    build += ' pluginspage="http://www.apple.com/quicktime/download/"></embed>';
    return build;
  }

});
```

<p>
  If you ask me this is far cleaner than the original. It encapsulates factory methods and all the initialize does is just setup vars, not do ALL the work in one big procedural way. So what does this class do? It's simple, it allows you to create JavaScript objects that will represent DOM objects for QuickTime movies. This object can be passed to any method that takes expects and element to Prototype since it has a toElement method. Here is the way I am using it in my upcoming project. Just like the Tapbots website, I am creating <code><div></code> tags that have all the attributes I need and are the container elements. On page load, I update all those containers with generate QuickTime source code. See the example below.
</p>

```javascript
/* 
Example Container
<div id="mymovie_container" class="quicktime_video" rel="src=/video/mymovie.mov|width=280|height=393|loop=true|autoplay=true|video_id=mymovie"></div>
*/

document.observe('dom:loaded', function(){
  $$('.quicktime_video').each(function(container){
    var attributes = {};
    attributes.controller = 'false';
    attributes.autoplay = 'true';
    attributes.loop = 'false';
    attributes.enablejavascript = 'false';
    container.readAttribute('rel').split('|').each(function(keyvalue) {
      pair = keyvalue.split('=');
      attributes[pair[0]] = pair[1];
    });
    var qt = new Quickie(attributes['src'], {  
      id: attributes['video_id'], 
      width: parseInt(attributes['width']), 
      height: parseInt(attributes['height']), 
      container: container, 
      attributes: attributes
    });
    container.update(qt);
  });
});
```


<h2>Resources</h2>

<ul>
  <li><a href="http://github.com/metaskills/quickie.js">Quickie.js on Github</a></li>
</ul>


