---
layout: post
title: Memoization
tags: programming
---

In this post, I explore the concept of memoization and show how it can be a useful technique when dealing with functions that need to be called multiple times in a program. 

## What is Memoization?

It is an optimisation technique to store the return values of previous function calls. This technique builds a dictionary of the values returned by a particular function each time it is called. When the same function is called with the same argument(s) more than once, the function simply relies on the dictionary (called a 'memo'), and returns the corresponding value stored from the previous call, instead of executing the function for a second time. Memoisation can be very useful when calling the same function multiple times slows down the program.

#### Memoizing a one-argument function

In the following example, the function `oneArg` accepts an integer as an argument and returns its squared value:

```js
function oneArg(n) {
  return n * n;
}
console.log(oneArg(2)); //4
console.log(oneArg(2)); //4
```

As shown above, each time we call the function `oneArg` it will execute the operation inside the function and return its value, irrespective of whether the same argument was passed to it at an earlier occasion.

Here's how we can memoize `oneArg` such that it does not execute the operation inside it, more than once for the same argument:

```js
function memoizeOneArg() {
  let memo = {}; // to store the return values
  return function (n) {
    let strN = n.toString();
    if (memo.hasOwnProperty(strN)) {
      return memo[strN];
    }
    let temp = oneArg(n);
    memo[strN] = temp;
    return temp;
  };
}
let memoizedOneArg = memoizeOneArg();
```

In the above example, we have written a new function called `memoizeOneArg`. This function initialises an object called `memo`, which will store each pair of arguments and their return values, as key-value pairs.

Importantly, `memoizeOneArg` does not return the return value of `oneArg`. Instead, **it returns another (anonymous) function.** When this function is called, it will first check if the argument passed to it already exists as a property inside `memo`. If yes, the function will return the corresponding value of that property. Otherwise, it will call the actual function `oneArg`, store the return value inside `memo` and then, return that value to the caller.

This round-about way of returning a new function which calls the original function `oneArg` is necessary because **we want to initialise `memo` only once**. We cannot initialise `memo` inside `oneArg` because each time that function is called, it will re-initialise `memo` as a new (and empty) object. So, **we need a way to ensure that the object `memo` can retain the return values across multiple function calls**. This can of course be achieved if we make `memo` a global object. But this is not advisable when dealing with complicated programs, as it often leads to conflicts with other local variables that may happen to the same name.

So, we need a way to maintain a local variable, that can be accessed when we need to perform the operation done by the function `oneArg`. This can be done when we write a function (`memoizeOneArg`) that first initialises the object `memo` and then returns a function that checks `memo`'s properties before calling the actual function `oneArg`, in the manner explained above. Note that, the returned function will be initialised by a variable, and only this function should be called when the operation of `oneArg` needs to be executed. So, in the above example, the function `memoizedOneArg` should be called.

To clearly prove that the above technique (of storing return values by calling a returned function) actually works, we can write a new function `oneArgRandom` which will always return a random value. By using memoization, we can ensure that the same values are returned against the same argument:

```js
function oneArgRandom(n) {
  return Math.random() + n;
}
console.log(oneArgRandom(2)); // 2.39243349362445
console.log(oneArgRandom(2)); //2.8532267954230015

function memoizeOneArgRandom() {
  let memo = {};
  return function (n) {
    let strN = n.toString();
    if (memo.hasOwnProperty(strN)) {
      return memo[strN];
    }
    let temp = oneArgRandom(n);
    memo[strN] = temp;
    return temp;
  };
}
let memoizedOneArgRandom = memoizeOneArgRandom();
console.log(memoizedOneArgRandom(2)); //2.2444724212882425
console.log(memoizedOneArgRandom(2)); //2.2444724212882425
```

However, in terms of use-case of memoisation, it should only be used for pure functions, i.e., functions which will always return the same values for the same inputs, irrespective of the number of times it is called. `oneArgRandom` is not a pure function, as it is expected to return different values each time it is called. So, this function is not a fit-case for memoisation; nevertheless, this example was used to show that our memoization technique is working.

#### Memoizing a two-argument function

The same technique can be used to memoize a two-argument function:

```js
function twoArg(n, m) {
  return n * m;
}
function memoizeTwoArg() {
  let memo = {};
  return function (n, m) {
    let strNM = n.toString() + m.toString();
    if (memo.hasOwnProperty(strNM)) {
      return memo[strNM];
    }
    let temp = twoArg(n, m);
    memo[strNM] = temp;
    return temp;
  };
}
let memoizedTwoArg = memoizeTwoArg();
console.log(memoizedTwoArg(2, 3)); //6
console.log(memoizedTwoArg(2, 3)); //6
```

#### Memoizing functions with n arguments

The memoizing function (namely, `memoizeOneArg` and `memoizeTwoArg` in the above examples) which returns the memoized version of another function, is, in fact, quite independent of the original function that is being memoized. In other words, the return value of the memoizing function is not dependent on the contents of the function being memoized, except for the number of parameters to be passed to it. This means that the `memoizeOneArg` and `memoizeTwoArg` would work just the same way for all one-argument functions and two-argument functions, respectively. We just have to replace the references to `oneArg` and `twoArg` with the names of the functions we want to memoise. Better still, we can pass the functions we want to memoize, as arguments to the memoizing function.

We can also make one memoizing function that works with _any function_, irrespective of their requisite parameters, by using the spread syntax (`...`):

```js
function memoize(f) {
  let memo = {};
  return function (...args) {
    let strArgs = args;
    if (memo.hasOwnProperty(strArgs)) {
      return memo[strArgs];
    }
    let temp = f(...args);
    memo[strArgs] = temp;
    return temp;
  };
}
let memoizedMultiply = memoize((n, m) => n * m);
let memoizedAverage = memoize((n, m, o) => (n + m + o) / 2);
console.log(memoizedMultiply(2, 3)); //6
console.log(memoizedAverage(2, 3, 5)); //5
```

#### What about recursive functions?

Intuitively, memoization appears to be ideal when we have to work with recursive functions, as the chances of calling the same function with the same set of arguments multiple times, will be relatively high.

However, **if we simply pass a function which calls itself recursively, the benefit of memoization will be limited.** This is because, once the passed function is called, the link with the memo object snaps and that function independently keeps calling itself without first checking if a return value for each such call already exists inside the memo.

```js
function fibonacci(n) {
  if (n == 0 || n == 1) {
    return n;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
}
let memoizedFibo = memoize(fibonacci);
```

In the above example, which returns the *n*th fibonacci value, the `memoizedFibo` will check the properties of `memo` only once: that is, when the `fibonacci` is being called with `n` as a parameter. But the return values of each recursive call happening inside `fibonacci` is neither being stored in `memo` nor is `memo` being checked before executing each recursive call. For example, if we pass `50` to `memoizedFibo`, it will first check whether `memo` contains `'50'` as a property. If not, it will directly call the passed function, `fibonacci`. Now, `fibonacci` will recursively call itself for all values from 0 till (n-1) multiple times, to arrive at the 50th fibonacci number. But the return values of these recursive calls will never be saved in the `memo`. (This is why, if we pass `50` to `memoizedFibo`, the computer will not be able generate a value within reasonable time, as the number of repeated recursive calls will be extremely high).

To solve this, we need to ensure that during each recursive call, the function first checks the `memo` object. This is possible, if, **instead of calling the original function, the function recursively calls the memoized version of itself**.


```js
let memoizedFibo = memoize(function (n) {
  if (n == 0 || n == 1) {
    return n;
  }
  return memoizedFibo(n - 1) + memoizedFibo(n - 2);
});
console.log(memoizedFibo(50)); //output: 12586269025 [generated in 1.109 seconds]
```

## Takeaway

- Memoization is an efficient tool to save return values of functions, which can save time when the same function is expected to be called multiple times with the same set of arguments.

- Memoization is a **practical implementation of functional programming:** we are passing functions as arguments to another function, and we are setting functions as return values of other functions. Thus, memoisation shows [how functions can be treated as first-class citizens in JavaScript.](https://oitee.github.io/2021/07/11/higher-order-functions.html)
