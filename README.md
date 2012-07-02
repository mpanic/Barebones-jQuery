Introduction
==================

The goal of this project is to demonstrate some of the core jQuery
concepts using code snippets extracted from jQuery itself. 

Getting Started
------------------

jQuery itself is implemented as a small core, and numerous plug-ins can
ride on top of it. Because we focus on the core only, we can
significantly cut down on the amount of code needed. Let's break down
the code into five parts and examine them.

First, we define the main jQuery() function. It simply passes its parameter
to the jQuery.prototype.init() method, which does the actual processing.

    var jQuery = function(selector) {
        return new jQuery.prototype.init(selector);
    };

jQuery.prototype
------------------------

When working with jQuery, you typically spend most of your time with
so-called matched sets. For example, var set = $('a') will match all the
anchors in the document and store references to them in an object called
set. The structure of this object is similar to an array: it has a
length property, and individual matched elements are accessible via
numeric indexes. For example, set[0] will point to the first anchor
node, and so on.

Functionality of jQuery matched sets goes beyond that of standard
arrays, though. They provide a lot of useful methods which you can use
to manipulate elements in a set. For example, you can call set.hide() to
hide all the matched elements.

These methods are defined in the jQuery.prototype object – any method or
property that you attach there will be available on returned matched
sets later on.

Next listing shows the general structure of jQuery.prototype – we define
several properties and an important init() method presented in listing
below:

    jQuery.prototype = {
        jquery: "0.1 Barebones Edition",
        length: 0,
        init: function() { /* See next listing for full code */ }
    }

jQuery.prototype.init() is the method called by the jQuery() function
defined above. It is the main entry point of jQuery, which behaves
differently depending on the type of parameters passed in.

Common parameter types are function and selector string. If a function
is passed in, it is attached to page load event and if a selector string
is passed in, it is used to locate matching elements.

Here we only handle two types of selector strings: element names ('p')
and id attributes ('\#id'). It's not difficult to add support for other
basic selectors or integrate a full-blown selector engine like Sizzle
too.

    init: function(selector) {
        if(typeof selector === "function") {
            window.onload = selector;
            this[0] = document;
            this.length = 1;
        } else if(selector) {
            if (/^\w+$/.test(selector)) { // element name
                selector = document.getElementsByTagName(selector);
                return jQuery.merge(this, selector);
            } else { // #id                        
                var id = selector.substr(1);
                var elem = document.getElementById(id);
                if(elem) {
                        this[0] = elem;
                        this.length = 1;
                }
            }
        }
        return this;
    }

Utility methods
-----------------------

The methods we've talked about so far are designed to operate on matched
element sets. There's also another type of method in jQuery that are
more generic in nature. These methods are attached to the jQuery
function object itself, rather than the jQuery.prototype. We define two
such methods in the next listing.

jQuery.each() loops over an object, and executes a callback function
during each iteration. Note that we only loop over elements with numeric
property names. We also make sure that the numeric value of a property
name is less than the length property of the object passed in.

When jQuery.each() is applied to matched sets, these restrictions ensure
that we only operate on DOM nodes and not on any methods or properties
that we have added to jQuery.prototype ourselves. Finally note that
during each call, the value of this inside the callback is set to the
value of element for convenience.

jQuery.merge() simply combines two objects together.

    jQuery.each = function(object, callback) {
      for(var i = 0, length = object.length, value = object[0]; i < length; value = object[++i]) {
          callback.call(value, i, value);    
      }
      return object;
    }
    jQuery.merge = function(first, second) {
      var i = first.length, j = 0;
      while (second[j] !== undefined) {
        first[i++] = second[j++];
      }
      first.length = i;
      return first;
    }

Aliases
---------------

In next listing, we set up some useful naming shortcuts, and we do some
important work with prototype objects.

Naming changes are straightforward: we create the jQuery.fn shortcut for
jQuery.prototype, and the $ shortcut for jQuery (you can also add your
own shortcut here if you like).

Assigning jQuery.prototype to jQuery.prototype.init.prototype, on the
other hand, is a subtle but important change. It allows the
jQuery.prototype.init() method to use all the functionality we have
defined in jQuery.prototype.

    jQuery.prototype.init.prototype = jQuery.fn = jQuery.prototype;
    $ = jQuery;

Shortcuts & Prototypes

Believe it or not, we are done. However, we want this to be practical
rather than academical exercise, so we'll go ahead and create a few
plug-ins to help with testing our new library.

Plug-ins
----------------

jQuery plug-ins are really just methods attached to jQuery.prototype. We
mostly talk about plug-ins as third-party extensions, but internally
jQuery uses the same mechanism as well.

Next, we will define four plug-ins. The first two allow us to show and
hide elements in a matched set, while the other two deal with event
handling. Plug-in named bind is a general way of attaching event
handlers, while click is a shortcut specifically for handling click
events.

    jQuery.fn.hide = function() {
        return jQuery.each( this, function() { this.style.display = 'none'; } );
    };
    jQuery.fn.show = function() {
        return jQuery.each( this, function() { this.style.display = ''; } );
    };
    jQuery.fn.bind = function(eventType,fn) {
      return jQuery.each(this, function() {
        if(this.addEventListener) {
            this.addEventListener(eventType, fn, false);
        } else if(this.attachEvent) {
            this.attachEvent('on' + eventType, fn);
        }
          else {
            this['on' + eventType] = fn;
        }
      }); 
    };
    jQuery.fn.click = function(fn) { return this.bind('click', fn); };

Ideas for further exploration
-----------------------------

We are done with our code, but in terms of learning about jQuery
internals this is only the beginning. It's best to to approach this
hands-on: try to add or remove a feature, rewrite a section etc.
You could also pick an area you're interested in; for example, Ajax.
Skim through the relevant areas of the original jQuery source and try
to gradually incorporate that functionality into our small version
of the library.

A few additional resources:

* https://github.com/jquery/learn.jquery.com
** https://github.com/jquery/learn.jquery.com/blob/master/content/using-jquery-core/jquery-object.md
* https://github.com/jquery/jquery/blob/master/src/core.js
* http://james.padolsey.com/jquery/ jQuery source viewer
* http://zeptojs.com/