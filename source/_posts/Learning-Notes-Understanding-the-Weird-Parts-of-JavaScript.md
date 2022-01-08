---
layout: post
title:  Learning Notes - Understanding the Weird Parts of JavaScript
date:   2017-05-04 22:05:07 +08:00
category: 编程语言
tags: [JavaScript, 前端]
comments: true
---

The learning notes of the MOOC "JavaScript: Understanding the Weird Parts" on Udemy,including most important parts of JavaScript.

>* Course link: [JavaScript: Understanding the Weird Parts](https://www.udemy.com/understand-javascript/)
>* My Certification: [UC-CWVEBCC5](https://www.udemy.com/certificate/UC-CWVEBCC5/)

<!-- more -->

## Basic concept

### Conceptual Aside

Conceptual Aside:

- **Syntax Parser**: a program that reads your code and determines what it does and if its grammer is valid
- **Lexical Environment**: where something sits physically in the code you write
- **Execution Context**: a wapper to help manage the code that is running
- **Name/Value Pair**: a name which maps to a unique value
- **Object**: A collection of Name/Value pairs

- **undefined**: a special value/a special keyword that JavaScript has within it internally that means that the value hasn's been set.It means that this is the value that was initially set by JavaScript.
- **single threaded**: one command is being executed at a time.It may not be under the hood of the browser.
- **synchronous**: one at a time and in order.

- **dynamic typing**: you don't tell the engine what type of data a variable holds,it figures it out while your code is running.
- **operator**: a special function that is syntatically (written) differently.
- **coercion**: converting a value from one type to another.

- **by value vs. by reference**: all primitive types are by value and all objects are by reference.
- **array**: collections of anything.

- **inheritance**: one object gets access to the properties and methods of another object.
- **built-in function constructors**: objects created by them have all the properties and methods on the functions' `prototype` property and what's boxed inside of it is the primitive value itself.The built-in function constructors look like you're creating primitives but you're not,you are creating objects that contain primitives with a whole bunch of features.  

--------------

### Framework Aside

Framework Aside:

- **default value**: e.g. `window.libName = window.libName || "lib 1"`
- **namespace**: a container for variables and functions.
- **function overloading**: JavaScript doesn't have it.There are other appoaches to deal with it.
- **whitespace**: invisible characters that create literal "space" in your written code,like carriage returns,tabs, or spaces.
- **IIFEs and safe code**: by wrapping code in an immediately invoked function, does not interfere with crash into,or be interfered by any other code that might be included in your application.
- **function factory**: a function that returns or makes other things for you.

### Dangerous Aside

Dangerous Aside:

- automatic semicolon insertion: if you're going to return an object from a function,you need to type the `{` right after `return` rather than in a new line.
- `new` and functions: don't forget the key word `new` in front of the function constructors,or you'll probably get `undefined` returned and cause in trouble.You'd better always have a capital letter as the name of the constructor.
- built-in function constructors: strange things can happen during comparison with operator and coercion.In general,it's better not to use them.Use literals.
- arrays and `for..in`: in the case of arrays,don't use `for..in` because arrays are objects in JavaScrip and their items are added properties.


--------------

## Execution Context and Lexical Environment

### Execution Context

Execution Context(Global) was created at global level by the JavaScrip engine, and two things were also created for you:

- Global Object
- A special valiable called `this`

When you open the file inside your browser, `this` object is the `window` object,which refers to the browser window.That is to say:`this` object **equals** `window` object at global level in this case.

**Global means "Not Inside a Funciton".** Global variables and functions get attached to the global object.

![Execution Context](http://blog.qiniu.brianway.site/JavaScript_execution-context.png)

**hoisting**: variables and functions are to some degree available even though they're written later in the code.

The phenomenon is because the exection context is created in two phases.

1. creation phase: global object and `this` set up in memory,an outer environment, also **set up memory space for variables and functions**(this step called **hoisting**).
2. execution phase: runs the code you've written line by line,interpreting it,covering it,compling it,executing it.

![creation phase](http://blog.qiniu.brianway.site/JavaScript_creation-phase.png)

All variables in JavaScript are initally set to undefined,and functions are sitting in memory in their entirety.


### Function Invocation

**invocation**: running a function.In JavaScrip,by using parenthesis`()`.

**Execution Stack**: top execution context in the stack is the currently executing function.A new execution context is created and put on top of the stack for the function invoked,and popped off the stack when the function finished.

**Variable Environment**: where the variables live.And how they relate to each other in memory.

Every execution context has a reference to its **outer environment**.(that is to say,to its lexical environment)

execution context vs. outer environment reference:

- **execution context** is created when you invoke a function.
- **outer environment reference** is created for the execution context and it looks at where the code is physically written in the JavaScrip file.(A different way of thinking about it: the outer reference is to the execution context in which the function was created.)

Here is an example:

```javascript
function c(){
	console.log(myVar);//1 (4th line)
}

function b() {
	var myVar;
    console.log(myVar);//undefined (3rd line)
}

function a() {
	var myVar = 2;
    console.log(myVar);//2 (2nd line)
	b();
	c();
}

var myVar = 1;
console.log(myVar);//1 (1st line)
a();
console.log(myVar);//1 (5th line)

// result is (five lines): 1 2 undefined 1 1
```

**Scope Chain**:those links of outer environment references where you can access variables.

![Scope Chain](http://blog.qiniu.brianway.site/JavaScript_scope-chain.png)

**scope**: where a variable is available in your code.

**asynchronous**: more than one at a time.(Asynchronous part is about what's happening outside the JavaScrip engine rather than inside it.)

![asynchronous](http://blog.qiniu.brianway.site/JavaScript_asynchronous.png)

Any events that happen outside of the engine get placed into the **event queue**,an if the execution stack is **empty**,if JavaScrip isn't working on anything else currently,it'll process those events in the order they happend via the event loop **synchronously**.


## Types and Operators

There are six primitive types in JavaScrip.

**primitive type**: a type of data that represents a single value.That is,not an object.

- **undefined**: represents lack of existence
- **null**: represents lack of existence
- **boolean**: true or false
- **number**: floating point number.There's only one 'number' type.
- **string**: a sequence of characters.Both `'` and `"` can be used.
- **symbol**: used in ES6


precedence and associativity:

- operator precedence: which operator function gets called first.
- associativity: what order operator functions get called in: left-to-right or right-to-left.

> Reference: [Operator precedence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

`==` vs. `===`

- `==` will try to coerce the values if the two values are not the same type.
- `===` doesn't try to coerce the values.It's strict equality.

> Reference: [Equality comparisons and sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)

The example below shows the usage of coercion to check for existence and is used in a lot of really great frameworks and libraries.

```javascript
var a;
//goes to internet and look for a value

if(a){
    console.log("Something is there");
}
```

or opertator `||`: if you passed it two values that can be coerced to true and false,it will return the first one that coerces to true.

-------

## Objects and Functions

Object have properties and methods:

- Primitive "property"
- Object "property"
- Function "method"

![Object](http://blog.qiniu.brianway.site/JavaScript_object.png)


Both `[]` and `.` can find the property on the object and give you the value.

```javascript
var person = new Object();
person["firstname"] = "Brian";//use brackets []
person.lastname = "Way";//use dot .
```

Object literal vs. JSON string

- The object literal syntax uses the curly braces to define name and value pairs separated by colons.
- JSON stands for JavaScript Object Notation.It looks like object literal syntax except for some differences.For example,property names have to be wrapped in quotes.JSON is technically a subset of the object literal syntax.
    - `JSON.stringify({firstname : "Brian", student: true})`
    - `JSON.parse('{"firstname":"Brian","student":true}')`;


**First class functions**: everything you can do with other types, you can do with functions.(Assign them to variables,pass them around,create them on the fly.)

In JavaScrip,**functions are objects**.The function is an object with other properties,that is to say, a function is a special type of object.Two main other properties:

- it has a hidden **optional name property** which can be anonymous then if you don't have a name.
- we have **code property** that contains the code and that code property is **invocable** so we can run the code.

![function](http://blog.qiniu.brianway.site/JavaScript_function.png)


**expression**: a unit of code that results in a value.

**Statement just does work and an expression results in a value.**

mutate: to change something.

The keyword `this` points to what? Let's see an example.

```javascript
var c = {
    name: 'The c object',
    log: function() {

        this.name = 'Updated c object'; //this points to c object
        console.log(this);

        var setname = function(newname) {
            this.name = newname; //this points to the global object
        }
        setname('Updated again! The c object');
        console.log(this); //this points to c object
    }
}

c.log();
```

In the example above,the internal function `setname` when its execution context was created,the `this` keyword points to the global object,even though it's sitting kind of inside an object that I created.To avoid this case,just use `var self = this;`(set equal to by reference) as the very first line of your object method.


An array can be created in the following formmat:

- `var arr = new Array();`
- `var arr = [1, 2, 3];`

Array in JavaScript is zero based and dynamically typed,it figures out what type of things are on the fly.

`arguments`: the parameters you pass to a function.JavaScript gives you a keyword of the same name which contains them all.Although `arguments` doesn't contain the names of arguments,just the values,you could use it like an array.A new thing is called a spread parameter.`...other` means take everything else and wrap it into an array of this name "other".


### IIFEs

IIFEs - an immediately invoked function expressions

```javascript
// using an Immediately Invoked Function Expression (IIFE)
var greeting = function(name) {
    return 'Hello ' + name;
}('John');
```

To wrap your function in parentheses.This is when you want a function expression instead of normal function statement.

```javascript
//valid syntax but output nothing
(function (name) {
    console.log("Hello " + name);
});
// IIFE
(function (name) {
    console.log("Hello " + name);
}("Brian"));
//also OK
(function (name) {
    console.log("Hello " + name);
})("Brian");
```


### understanding closures

Even though the outer function ended/finished,any functions created inside of it when they are called will still have a reference to that outer function's memory.

![closure](http://blog.qiniu.brianway.site/JavaScript_closure-1.png)

Outer function is gone,the exectution context is gone.But what's in memory for that execution context isn't and JavaScript engine makes sure that the inner function can still go down the scope chain and find it.

![closure](http://blog.qiniu.brianway.site/JavaScript_closure-2.png)

In this way we say that the execution context has closed in its outer variables.And so this phenomenon,of it closing in all the variables that it's supposed to have access to,is called a **closure**. This is the feature of the language JavaScript,very important.


**callback function**: a function you give to another function,to be run when the other function is finished.

### call(),apply() and bind()

All functions in JavaScript also get access to some special functions/methods, on their own.So all functions have access to a `call` method,an `apply` method and a `bind` method.

- `bind()` creates a new copy of whatever function you're calling it on.And then whatever object you pass to this method is what the `this` variable points to by reference.
- `call()` invokes the function you're calling it on.Also let you decide what the `this` variable will be.Unlike `bind()` which creates a new copy,`call()` actually executes it.
- `apply()` does the exact same thing as `call()` except that it wants an array of parameters as the second parameter rather than just the normal list.

These functions can be used for function borrowing and function currying.

function currying: creating a copy of a function but with some preset parameters.

some open source libraries:

>* [underscore.js](http://underscorejs.org/)
>* [lodash](https://lodash.com/)
>* [moment.js](http://momentjs.com/)


## Object-Oriented JavaScript and Prototypal Inheritance

Classical vs. Prototypal Inheritance:

- Classical Inheritance: verbose
- Prototypal Inheritance: simple

All objects include functions hava a prototype property.The prototype is simply a reference to another object.

The prototype chain,the concept of prototypes is just I have this special reference in my object that says where to look for other properties and methods without manually going dot prototype.

![prototype chain](http://blog.qiniu.brianway.site/JavaScript_prototype-chain.png)

**reflection**: an object can look at itself,listening and changing its properties and methods.

*tip: the "for in" actually reached out and grabbed every property and method not just on the object but also on the object's prototype*

**extend**: here we just talk about the [`extend` function](http://underscorejs.org/docs/underscore.html#section-103) in [underscore](http://underscorejs.org/).You can **combine and compose objects** in this extend pattern with relection,not just the prototype pattern.


## Building Objects

**function constructors**: a normal function that is used to construct objects.(The `this` variable points a new empty object,and that object is returned from the function automatically.)

There are many approaches to building an object:

1. object literal syntax: `{}`
2. the key word `new`: `var john = new Person();`
    - first, an empty object is created.
    - then, it invokes the function with the `this` variable pointing to that empty object.
    - last, return the object if you don't return anything explicitly,or return what is returned in the function.
3. `Object.create`: creates an empty object with its prototype pointing at whatever you have passed in to `Object.create`.
    - you can simply override/hide properties and methods on those created objects by setting the values of those properties and methods on new objects themselves.
4. ES6 and classes: another approach but still works the same under  the hood,just syntactic sugar.
    - key word `class`: class is not the template or definition like other languages such as Java/C++,**class is also an object**.
    - key word `extends`: sets the prototype(`__prototype__`) for any of the objects created with this class.



`.prototype`: this `prototype` property of all functions is where the prototype chain points for any objects created using that function as a constructor.

- The `prototype` property on a function is not the prototype property(`__prototype__`) of the function.
- It's used only by the `new` operator.In other cases,it's just an empty object,hanging out there.
- It's the prototype of any objects created if you're using the function as a function constructor.
- takes up less memory space if you add method to the prototype because it'll only have one,while it'll have many copies if you add it to function constructor.


polyfill: code that adds a feature which the engine may lack.

```javascript
// polyfill
if (!Object.create) {
  Object.create = function (o) {
    if (arguments.length > 1) {
      throw new Error('Object.create implementation'
      + ' only accepts the first parameter.');
    }
    function F() {}
    F.prototype = o;
    return new F();
  };
}
```

syntactic sugar: a different way to type something that doesn't change how it works under the hood.


## Odds and Ends

initialization: when come accross a large initialization of objects,don't get overwhelmed by the syntax.

`typeof` and `instanceof`:

- `typeof`: it's an operator(essentially a function) that takes a parameter returns a string.It will tell you under most cases what something is.
    - `Object.prototype.toString.call(arr)` gets array type(`[object Array]`) returned,or you'll get `object` returned.
    - `typeof undefined` returns `undefined`
    - `typeof null` returns `object`.It's been a bug since forever.
- `instanceof`: tells if any object is down the prototype chain.It will tell you what something has in its prototype chain.

> [strict mode on the MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)


transpile: convert the syntax of one programming language to another.

>* [Typescript](http://www.typescriptlang.org)
>* [Traceur](https://github.com/google/traceur-compiler)(Traceur is a JavaScript.next-to-JavaScript-of-today compiler)
>* [features of ECMAScript 6](https://github.com/lukehoban/es6features)
