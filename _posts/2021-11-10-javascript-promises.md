---
layout: post
title: "Promises in JavaScript and Clojure"
tags: programming
image: /assets/images/js_vs_clojure.png
---

In this post I discuss how promises are created in JavaScript, some of the confusions I had while working with JS promises and how Clojure handles promises.

## Need for a Promise in Twirl

While writing a specific test for [Twirl](https://github.com/oitee/twirl)—a web app that shortens URL—it was important that a request for shortening a specific link was sent at a later point of time. According to the [scope of the project](/2021/11/03/diwali-hack-week.html), the analytics table on the home of the app is required to display the top 50 links shortened by each user. These links are to be ordered according to their access count, that is, the more accessed a link, the higher it should be listed. Also, for links with the same access count, they should be ordered in a descending order of the date of creation. To test this latter part (i.e., whether links with the same access count are ordered according to their date of creation), it was important that a link was shortened at a subsequent point in time (as a part of the tests).

The simplest way to achieve this was to rely on the library function `setTimeout`. While this function accepts a call-back function which will be executed after a specified period of time, it does not return a pending promise. For example, for a simple implementation of the `setTimeout` function, namely, `setTimeout(() => console.log("time up"),10000)`, following value will be returned:

```js
Timeout {
  _idleTimeout: 10000,
  _idlePrev: [TimersList],
  _idleNext: [TimersList],
  _idleStart: 61901,
  _onTimeout: [Function (anonymous)],
  _timerArgs: undefined,
  _repeat: null,
  _destroyed: false,
  [Symbol(refed)]: true,
  [Symbol(kHasPrimitive)]: false,
  [Symbol(asyncId)]: 464,
  [Symbol(triggerId)]: 5
} 
```

Of course, the call-back function will eventually be executed. However, as `setTimeout` does not return a promise, we cannot use `await` on it. This means that the JavaScript engine will not pause execution till the call-back function is executed. As a result of this, the specific test discussed above was failing, as the test would complete before the future request for shortening a link could be executed.

To ensure that the test worked, we needed to create a new promise by invoking the `Promise` constructor. The `Promise` constructor expects a call-back function as a parameter. This call-back function accepts two further call-back functions, commonly called `resolve` and `reject`. A promise would be considered to be 'pending' till either of these two call-back functions are invoked, namely `resolve` or `reject`. If `resolve` is invoked, it would indicate that the promise was successfully resolved. If `reject` is invoked, it would indicate some error having taken place and the promise could not be resolved.

While trying to understand how promises are created, I had a few questions. First, *who calls the `resolve` function?* And, *what is `resolve`?*

### Who calls the resolve function?

I had (wrongly) assumed that the `resolve` function is called by the JavaScript engine when an asynchronous block of code is successfully executed. The reason for this confusion was that, I was looking at promises from the narrow lens of `setTimeOut`. In this case, there is a definitive way to know *when* the asynchronous code block is resolved: after the time-period passed to `setTimeOut` has expired and the call-back function passed to `setTimeOut` has been successfully invoked. I had (wrongly) expected that the JavaScript engine would call the `resolve`  function, once this was completed.

However, this is not how it works. The call-back function passed to the promise needs to call `resolve` to indicate to the constructor that the particular promise has successfully finished its execution. In the case of `setTimeOut`, the `resolve` function should be called from *within the call-back function* passed to `setTimeOut`.

### What is the resolve function?

Now, if we have to manually call the resolve function, what is it really? In every other case, when we write a function, we use variables to identify its parameters. It is the caller of that function which passes specific arguments, thereby giving concrete values to those named parameters. For example in the following function `sum`, the parameters `n` and `m` have no specific values. It is only when we call `sum(5, 6)` that the variables `n` and `m` have a certain value.

```js
let sum = (n, m) => n + m;

sum(5, 6); 
```

But in the case of promises, we **do not call the call-back function** passed to the promise. To quote from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise), the call-back function is _"executed by the constructor, during the process of constructing the new Promise object"_.

```js
function callbk(res, rej){
	setTimeout(() => res(), 1000);
}
let p = new Promise(callbk);
```

In the above example, we never call the `callbk` function with appropriate values for `res` and `rej`. If we do not call this function  by ascribing specific values to the `resolve` and `reject` parameters, what meaning do these parameters have?

It turns out that the call-back function passed to the `Promise` constructor is actually called by the constructor itself (and not the user). The value of `resolve` and `reject` have specific meaning to the constructor, and all it does is keep a look out for when these functions are called, for each promise. To quote from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise), _"At the time when the constructor generates the new Promise object, it also generates a corresponding pair of functions for resolutionFunc and rejectionFunc; these are "tethered" to the Promise object."_

Because these parameters have certain values ascribed by the constructor, it is not necessary that we call these function from inside the body of the call-back function itself. We can extract out these functions outside the block of the call-back function and it would still work:

```js
let resolve;
function callbk(res, rej){
	resolve = res;
}
setTimeout(() => resolve(4), 1000);
```

## Building a simple (Com)Promise constructor

To better visualise how promises are resolved under-the-hood, I thought of building a toy-version of the constructor.

```js
class compromise {
  constructor(callbk) {
    this.result = { status: false, value: null };
    let resultLocal = this.result;

    let res = (value) => {
      resultLocal.status = true;
      resultLocal.value = value;
    };
    callbk(res);
  }

  hasEnded() {
    return this.result.status;
  }
  value() {
    return this.result.value;
  }
}
```

For the sake of simplicity, the above example only deals with the `resolve` parameter. Hence, this is a not a complete replica of the JS constructor `Promise`,  making it a **_com-promise_** (forgive the pun!) 

While our toy constructor does not have all the bells and whistles of `Promise`, it captures its core essence:

- It accepts a call-back function
- This call-back function accepts another call-back function, `resolve`
- When this `resolve` is called, the (com)promise is deemed to be complete. How to know if a promise is completed? The `hasEnded` method of the constructor will return `true` when a promise is completed its execution. Also, the `value` method of the constructor will return the passed to the `resolve` function by the original call-back function.

To test this out, let's write a simple call-back function `fn` which calls `resolve` from inside the call-back function of `setTimeout`:

```js
function fn(resolve) {
  setTimeout(() => resolve("Done"), 10000);
}
```

Now, when we create a promise using our constructor, the execution should be completed after 10 seconds, as per `fn`. To test this is working, we can write a `setInterval` function which periodically checks the state of the promise created by our constructor:

```js
let c = new compromise(fn);

setInterval(() => {
  if (c.hasEnded()) {
    console.log(`Resolved! The promise resolved to : ${c.value()}`);
  } else console.log(`Promise pending...`);
}, 1000);
```

The above example will generate the following result:

```js
Promise pending...
Promise pending...
Promise pending...
Promise pending...
Promise pending...
Promise pending...
Promise pending...
Promise pending...
Promise pending...
Resolved! The promise resolved to : Done
Resolved! The promise resolved to : Done
Resolved! The promise resolved to : Done
Resolved! The promise resolved to : Done
```

## Dealing with Asynchrony in Clojure

Having understood how promises are handled by JavaScript, it was illuminating to see how Clojure, a functional programming language, dealt with the same.

The fundamental difference between JavaScript and Clojure, when it comes to handling asynchronous blocks of code, is that **JavaScript execution proceeds asynchronously by default while Clojure execution proceeds in a synchronous manner unless explicitly told otherwise**. What this means is that, normally, when the Clojure compiler stumbles across an asynchronous code (i.e., a thread that will take a while to finish its execution), it will wait for the execution of that code to be completed, before proceeding further. By contrast, in the case of JavaScript, the default response of the compiler would be to create a new thread for the asynchronous code, and proceed further with the main thread. 

### Future: How to make something asynchronous

When we place the keyword `future` in a specific block, it indicates to the compiler that it does not need to wait for the current block to complete its execution. It can, instead, proceed forward without any waiting, after having created a new thread for executing the current block.

Let's take an example. In Clojure,  `Thread/sleep` works somewhat similar to a JavaScript's  `setTimeout` function: it pauses execution of the current thread for a specified period of time. Thus, when we simply use `Thread/sleep` in a block, it **pauses the execution of the entire code** for a specific period of time:

```clojure
(Thread/sleep 4000) 
(+ 1 2) 
(println "now") 
```

In the above example, the result of the expression `(+ 1 2)` will take 4 seconds to be executed and only after that, will the string `now` be printed. In case we want to avoid this and want `now` to be printed instantly, we can use `future` on the first part of the expression:

```clojure
(future 
	(Thread/sleep 4000) 
	(println 
		(+ 1 2))) 

(println "now")

```

This will print `now` instantly. After 4 seconds, the REPL will return the value of the expression `(+ 1 2)`

The keyword `deref` can be used to store the value returned after execution of an asynchronous block of code (indicated by the use of `future`). When we `deref` the result of a future, and if that future has not finished its compilation, it will pause the execution till the result is generated (much like how `await` pauses execution of an `async` function):

```clojure
(def x 
	(future 
		(Thread/sleep 3000) 
		(+ 1 2)))


(deref x)
```

In the above example, if we type the second statement `(deref x)` before  the future `x` is executed (i.e., before the passage of 3 seconds), the execution of the entire code will pause till the future `x` is executed.

But note that much like `await`, using `deref` does not result in calling the future again. It merely returns the value generated by the execution of the particular future.

### Promises in Clojure

To create a new promise, we need to use the key word `promise`:

```clojure
(def p (promise))
```

When we create a promise in the manner shown above, the promise is considered to be pending. In fact, we can use `realized?` to check if a promise has been realised or not:

```clojure
(realized? p)
```

The above will return `false`

Now, when we want to indicate that this promise has been realised (similar to calling `resolve` from the call-back function passed to the `Promise` constructor in JavaScript), we simply **`deliver` the promise:**

```clojure
(deliver p (+1 2))
```

This completes the promise. The second argument passed to `deliver` indicates the value that will be returned by that promise, once `deref` is used:

```clojure
(deref p)
```

This will return `3`. Now`realized?` will return true:

```clojure
(realized? p)
```

## Promises: Clojure vs JavaScript

For a beginner, I must admit, learning about how promises are implemented in Clojure, was much simpler and straight-forward. 

To create a promise in JavaScript, you need to first write a call-back function, and then call the first parameter of that call-back function (which is itself another call-back function)  when you want the promise to be realised.

```js
let p = new Promise((res, rej) => {
	...
	res(3);// this resolves the promise
})
```

In contrast, creating promises in Clojure is relatively simpler: 

```clojure
(def p (promise))
...
(deliver p 3)// this resolves the promise
```

The cognitive load for writing promises in JavaScript seems to be higher, than the first-level promise creation and resolution as provided by Clojure. May be there is good reason as to why the path to creating and resolving promises is so complicated in JavaScript. 

But for a novice like me, understanding how promises are treated in Clojure was much more simple and apparent. 