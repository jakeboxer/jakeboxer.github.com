---
layout: post
title: "Prototype analog to jQuery's $(document).ready"
date: 2010-01-23 20:25
comments: true
categories: 
---

I have a lot of experience with jQuery, but less with Prototype. Recently, I needed to add some event handlers to some elements in a Ruby on Rails app I'm building. I searched for how to do the equivalent of jQuery's `$(document).ready()` function in Prototype so that I could add the handlers after the document loaded, but most of the guides I found were out of date (I'm running Prototype 1.6.0.3, and I don't know which version these guides were for, but they all made my Javascript console angry).

Eventually, I was able to piece it together after digging through [the](http://api.prototypejs.org/dom/document.html#observe-class_method) [depths](http://api.prototypejs.org/dom/event.html#observe-class_method) of the Prototype API documentation. It's actually very simple:

``` javascript
document.observe('dom:loaded', function(){
	// do yo thang...
});
```

Wrap whatever you're doing with that, and it won't be run until the document is loaded.