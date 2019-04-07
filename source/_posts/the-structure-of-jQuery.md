---
layout: post
title:  The Structure of jQuery
date:   2017-05-18 21:20:07 +08:00
category: 前端
tags: [JavaScript, jQuery, 前端]
comments: true
---

We examine famous frameworks and libraries,such as jQuery,to experience the open source education.This article mainly analyses the structure of jQuery.

> Course link: [JavaScript: Understanding the Weird Parts](https://www.udemy.com/understand-javascript/)
>
> (Section 8: Examining Famous Frameworks and Libraries)

<!-- more -->

## jQuery Introduction

jQuery is just a **JavaScript library**.What jQuery does essentially is let you **manipulate the DOM**.It handles the browser quirks but doesn't add any features to the browser or to JavaScript itself.


the `jQuery` function isn't a function constructor.It's just a function that returns an object,a function that returns a call to function constructor.

```javascript
// Define a local copy of jQuery
jQuery = function( selector, context ) {
	// The jQuery object is actually just the init constructor 'enhanced'
	// Need init if jQuery is called (just allow error to be thrown if not included)
	return new jQuery.fn.init( selector, context );
}
```

`jQuery.fn` is just an alias of `jQuery.prototype` which is just hanging out(just an empty object at fisrt) and is overwritten with a new object.

`jQuery.extend` and `jQuery.fn.extend` are the some function.This function takes properties and methods of one object and adds them to another.

jQuery extends itself with many useful functions,such as `map`,`each`,`type`,`isArray`,`makeArray`,and so on.

## Sizzle

`Sizzle`([http://sizzlejs.com/](http://sizzlejs.com/)) is a whole other engine inside of jQuery.

> A pure-JavaScript CSS selector engine designed to be easily dropped in to a host library.

The `jQuery.extend` invocation is inside an IIFE,and the property `Sizzle` is returned by another IIFE that returns a Sizzle object.That is to say,one IIFE can be inside another.

jQuery is using that Sizzle library.

```javascript
jQuery.find = Sizzle;
jQuery.expr = Sizzle.selectors;
jQuery.expr[":"] = jQuery.expr.pseudos;
jQuery.unique = Sizzle.uniqueSort;
jQuery.text = Sizzle.getText;
jQuery.isXMLDoc = Sizzle.isXML;
jQuery.contains = Sizzle.contains;
```

## init and jQuery prototype

`jQuery.fn.init` is the function constructor.At the end of the function constructor,it's returning a value,still giving back the empty object that was created with the `new` operator,but doing some stuff to it first.

```javascript
init = jQuery.fn.init = function( selector, context ) {
    //elide some codes
    return jQuery.makeArray( selector, this );
}
```

As for any new object created with the `init` function constructor, its prototype is the same memory spot as `jQuery.prototype`--the prototype of the jQuery object itself.

```javascript
//reminder: jQuery.fn = jQuery.prototype
// Give the init function the jQuery prototype for later instantiation
init.prototype = jQuery.fn;
```

The neat trick:

- you don't hava to call the `new` operator when you use jQuery.Instead this calls a function that then calls a function constructor,creating a new object.But that new object has access to all the properties and methods in `jQuery.fn`.
- And instead of always in this jQuery code typing in jQuery.fn.init.something,it's just the prototype of the jQuery object itself.


## Method Chaining

method chaining: calling one method after another,and each method affects the parent object.

When you execute a method or call a method,and it's found down the prototype chain,the `this` variable points at the originating object,the object that actually made the call.

e.g. Just see the return value of  `addClass` function:

```javascript
addClass: function( value ) {
    //elide some codes here
    return this;
}
```

## Expose Globally

Expose jQuery or `$` globally:

```javascript
// Expose jQuery and $ identifiers, even in
// AMD (#7102#comment:10, https://github.com/jquery/jquery/pull/557)
// and CommonJS for browser emulators (#13566)
if ( typeof noGlobal === strundefined ) {
	window.jQuery = window.$ = jQuery;
}
```

Both of `window.jQuery` and `window.$` are pointing at the same spot in memory as the jQuery object created inside this function that had all these properties and methods attached to it.
