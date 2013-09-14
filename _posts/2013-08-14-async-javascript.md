---
layout: post
title:  "Async Javascript"
date:   2013-05-01 21:00:00
categories: coding
---

##Function literals as expressions

###Elegant  

In Javascript, following code is equivalent to just writing `foo`.  

```javascript
function(x) { return foo(x); }
```  

So, following code  

```javascript
foo(function(x) { return bar(x); });
```  

is just an unneccesarily verbose way of writing  

```javascript
foo(bar);
```  

###Caution  

This elegance can quickly make you write incorrect code, unless you properly understand basic JavaScript functions' behavior.

```javascript
["1", "2"].map(function(x) { return parseInt(x);});
```

may be simplified to much cleaner code

```javascript
["1", "2"].map(parseInt);
```

But you'll not get the [result that you were expecting](http://www.wirfs-brock.com/allen/posts/166).

### Part 2

For example, following code  

```javascript
someArray.map(function(x){
  return foo(x.value);
});
```

can be understood as  

```javascript
someArray
.map(function(x){
  return x.value;
})
.map(function(value){
  return foo(value);
})
```

which can then be simplified as  

```javascript
someArray
.map(function(x){
  return x.value;
})
.map(foo)
```

We can further simplify the first part of the expression by refactoring following method:

```javascript
function getValue(x) {
  return x.value;
}
```

Which leaves our original expression as simple and elegant

```javascript
someArray
.map(getValue)
.map(foo)
```

###Functions that return functions

To make our `getValue` method more generic, we could write something like

```javascript
function get(attr) {
return function(object) {
  return object[attr];
}}
```

leaving our original expression as

```javascript
someArray
.map(get('value'))
.map(foo)
```


###Resources  
[Captain Obvious on JavaScript](https://github.com/raganwald/homoiconic/blob/master/2012/01/captain-obvious-on-javascript.md)  
[A JavaScript Optional Argument Hazard](http://www.wirfs-brock.com/allen/posts/166)  
[Async Javascript](https://leanpub.com/asyncjs)  
[Annotated EcmaScript](http://es5.github.com/)  
[JavaScript Garden](http://bonsaiden.github.com/JavaScript-Garden/)  

## JavaScript events are blocked until the thread is free.

Whenever an event fires, it gets _queued_. And this queue is processed only when _(single)_ thread is free.  

JavaScript runtime works as an *event loop*:

```javascript
RunScript();
while(atleastOneEventIsQueued)
  fireNextQueuedEvent();
```

Events can be queues while code is running, but they can't fire until runtime is free.

Consider below code:

```javascript
for(var i=0; i <3; i++) {
  setTimeout(function() { console.log(i); }, 0);
};
```

When `setTimeout` is called, a timeout event is queued, but the code inside the function is not executed; and it'll not be executed till runtime is free - meaning the for loop has finished running. Hence, result you'll get in this example is `3 3 3`.  

Consider another example:

```javascript
var start = new Date();
setTimeout(function() {
  var end = new Date();
  console.log('Time elapsed: ', end - start, 'ms');
}, 500);
while(new Date() - start < 1000) {} // essentially some code that takes 1000ms to run
```

Here, timeout event will get queued and then loop will run for next 1000ms. Only after this, the event will be fired and log statement will get a chance to print. Hence, it'll always print >1000ms, even though the timer was originally set for 500ms.  

[Async Javascript](leanpub.com/asyncjs)  
[John Resig - How JavaScript timers work](http://ejohn.org/blog/how-javascript-timers-work/)  

##JavaScript by nature is __synchronuous__.

If you call a function, your program can't continue till that function returns. An async function in JavaScript is simply a function that takes a callback which will not be executed till it returns, i.e. till we reach the event queue. __Non-blocking__ functions is perhaps a better term for their behavior.  

For `foo` to be async, it needs to pass the following test:  

```javascript
var a = false;
foo(function() { console.assert(a, 'a should be true'); })
a = true
```

What this means is that callback passed to `foo` will be put on the event queue; and [this event can't fire till the thread is free](https://gist.github.com/4606038).  

Both `setTimeout` and `setInterval` pass this test - being the only two in-built async functions.

##Maybe async, maybe not

Some functions are async sometimes, but not others. jQuery's `$` method can be used to delay a function till DOM has finished loading. But if you're not careful about how you structure your code, you may run into issues:

```javascript
var fontSize;
$(function() {
  // uses fontSize
});
fontSize = 11;
```

This works as intended - until browser loads the page from cache, causing the callback to be fired immediately.


Now, consider following code

```javascript
var connection = ssh.connect(address, function() {
  connection.send('hi');
});
```

Here, value returned by `connect` function is used inside the callback. If `ssh.connect` ran synchronuously, `connection` will be undefined in the callback, which mandates the function to be async.

However, from API designer's perspective, if connection is already available, it can directly call the callback and return the connection.

```javascript
ssh.connect = function(address, callback) {
  if(connection) {
    callback();
  } else {
    connection = makeConnection();
  }
  return connection;
}
```

Effectively, this function will behave async sometimes and sync other times. But as we saw above, __never__ define a "maybe async" function whose return value maybe useful in the callback. The simple solution is to wrap the callback in a `setTimeout`. It'll ensure that `connection` is returned before `callback` is invoked.

## Things to note about async behavior

### 1 Async output

JavaScript output is almost always async. For instance, you can make multiple updates to DOM, but user won't see any changes until control returns to event queue. implicitly, any change you make queues a 'redraw' event.  

### 2 Not all async behavior is obvious

For example, in webkit based browsers, console.log is async:  

```javascript
(function(){
    var obj = {};
    console.log(obj);
    obj.foo = "hello";
})();
```

Above code will log `{foo: 'hello'}` in chrome and safari, but `{}` in node.  

(This seems to have been corrected in chrome now.)

### 3 setTimeout and setInterval are imprecise

Both `setTimeout` and `setInterval` have a minimum delay of _4ms_, according to the [HTML spec](http://www.whatwg.org/specs/web-apps/current-work/multipage/timers.html#timers).

>"If the currently running task is a task that was created by the setTimeout() method, and timeout is less than 4, then increase timeout to 4."

### 4 Each event at bottom of stack

Implication of JavaScript's _event loop_ runtime is that each event that fires will be at the bottom of the stack. Hence, stack trace will not give much context about where the event was initiated.

## Distributed Events

_Distributed events_ - where a single incident can trigger reactions throughout your application.  

### PubSub  

It works like an aggregator; publishing one event to anyone who wants to subscribe. For example, Node's `EventEmitter`, jQuery's bind-like functions etc.  

>"
Though PubSub deals with async events, there's nothing inherently async about it. __Understand__: When user clicks a button, that's an async event, which gets pushed to event queue and is subsequently fired by the runtime. But everything that happens _as a result of that event_ is synchronous. All the handlers attached to that event fire in sequence, one after the other. This is important to understand because if too many handlers fire in resposne to an event, you risk blocking the thread and making the browser unresponsive.
"

### Evented Models  

as in Backbone.js, where a model publishes notifications when any data changes.  

## Promises

_A promise represents an ongoing task that may either succeed or fail._  

### Promises and Deferreds  

A promise exposes methods to attach handlers but not ones that change the state. [Deferred](http://api.jquery.com/jQuery.Deferred/) is a superset of Promise - with methods to change its state. Idea is to create a `Deferred` and then pass its `Promise` around the app, which is available by calling `Deferred.Promise()` method. This prevents other pieces from interfering with its state while enabling them to perform any action in response to change in deferred. All Ajax functions in jQuery return Promises, starting from v1.5.  

```javascript
resolve -> done
reject -> fail
notify -> progress
then - shorthand for adding all 3 (done, fail, progress) callbacks in one step
```

### Advantages  

#### Encapsulation  

Your application may want to do multiple, possibly unrelated, things in response to an Ajax call. Instead of handling everything in one humongous callback, it's much more elegant to pass a Promise around the app. Callbacks can be added even when the promise is in resolved/rejected state - they will execute immediately.  

#### DRY  

If something is shared across multiple ajax calls, it can be refactored as a new callback and added where required.  

#### Joining Promises  

```javascript
$.when(firstPromise, secondPromise)
  .then(onBothPass, onAnyFail)
```

`when` acts as a logical `AND` for promises. Resulting promise is resolved when all given promises are resolved, or rejected when any of the given promises is rejected. So in above code, `onBothPass` will run if both promises get resolved, or `onAnyFail` will run if either gets rejected.  

### Advanced topics

* Binding to future promises with 'pipe'
* Using Promises in place of callbacks

## Javascript execution context

Functions run in global scope ie. `this` points to global scope. Methods are functions in an object's directory and we invoke them using object reference ie. `this` points to the object on which the function is invoked. 

_It's important to understand that methods are still functions and can be invoked without object reference._

```javascript
var x = 0;

var obj = { 
  a : function() { 
    console.log('hello ' + this.x); 
  }, 
  x : 1 
};
obj.a(); // prints hello 1, uses x defined in global context

b = obj.a;
b(); // prints hello 0, uses x defined in scope of object
```

### Changing the function context

```javascript
var x = 0;
var obj = { x : 1 };

function foo() {
  console.log(this.x);
}

foo(); // prints 0
foo.call(obj); // prints 1
```

`call` and `apply` let you execute a function in a specific context ie. they let you explicitly specify the value of `this`.  

Every function in JavaScript - being an object - has its own methods, `call` and `apply` in addition to `toString` etc.  

[Yehuda Katz - Understanding JavaScript Function Invocation and “this”](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)  
[Call - MDN](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/call)  
[Apply - MDN](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/apply)  
[Ode to Code](http://odetocode.com/blogs/scott/archive/2007/07/04/function-apply-and-function-call-in-javascript.aspx)  
[JavaScript Refresher](http://odetocode.com/Articles/473.aspx)  

## Throttling and Debouncing

When, in special circumstances, we want to prevent a function from being called too often:  
**Throttling** - additional calls to the function are ignored for next _n_ ms.  
**Debouncing** - any call to the function is delayed till _n_ ms have passed since the last call.  

Both of these are easy to implement by creating _meta-functions_ that return a wrapped version of original function:  

```javascript
function throttle(func, delay) {
  var lastHitDate = 0;
  return function() {
    var now = new Date();
    if(now - lastHitDate >= delay) {
      lastHitDate = now;
      return func.apply(this, arguments); // call the function if _delay_ ms have passed since last call
    };
  };
}

function debounce(func, delay) {
  var timerId;
  
  return function() {
    // everytime the function is called, delay it by _delay_ ms
    clearTimeout(timerId);
    timerId = setTimeout(function() {
      func.apply(this, args);
    }, delay);
  };
}
```

Note that with debouncing, the function may never get called if it's called too often - may need to add a max delay. Also note that it's important to use the _same_ throttled/debounced instance of a function from every place that we want to limit calls from. Calling throttle/debounce methods again would just create a new timer/lastHitDate.  

Underscore.js provides `_.throttle` and `_.debounce`.
