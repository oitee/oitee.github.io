---
layout: post
title: Defining functions
---

In this post, I expolore the ways of defining functions in javascript. But before that, I also attempt to understand the very nature and utility of functions in a program.

## What are functions?

A function is a snippet of ["self contained"](https://www.cs.utah.edu/~germain/PPS/Topics/functions.html#:~:text=Functions%20are%20%22self%20contained%22%20modules,the%20inside%20of%20other%20functions.) code that accomplishes a specific task. A function can be _called_ by other code (including another function), to perform that task. A function can also be called by itself also. Such functions are called 'recursive functions'.

Typically, a function would accept a certain number of values (called parameters), to perform a set of operations or expressions on them and `return` a value. However, it is permissible for functions to not accept any parameter as well. Similarly, not all functions have a `return` statement. In the following code, the function `noParameter()` does not accept any parameter, and merely logs `"Hi World"`. The function `noReturn()` adds the two parameters passed to it and stores it in the variable `a`, without returning any value.

```js
function noParameter() {
  console.log("Hi World");
}
function noReturn(a, b) {
  a = a + b;
}
noParameter();
noReturn();
```

### What is so special about functions?

Functions are like chunks of code, within a program. What is the point of having functions? Functions can be very useful, when the same task or operation needs to be performed multiple times in the same program. Thus, in the following code, the function `average()`, can be called as many times as may be required, and it will always return the average of the three parameters passed to it.

```js
function average(a, b, c) {
  return (a + b + c) / 3;
}
console.log(average(1, 2, 4)); // output: 2.3333333333333335
```

Apart from their ability of being re-used _ad infinitum_, functions can _encapsulate_ a specific task or operation. Thus, a complex program can be divided into specific sub-programs, each of which can be neatly encapsulated within a separate function. Thus, for example, if we need to write a program to find the average of all the elements of an array, as well as its largest and smallest elements, we can write three specific functions for each of these tasks, as shown below.

```js
function mean(data) {
  let sum = 0;
  for (let i = 0; i < data.length; i++) {
    sum += data[i];
  }
  return sum / (data.length - 1);
}
function greatestElement(data) {
  let greatest = data[0];
  for (let i = 0; i < data.length; i++) {
    if (greatest < data[i]) {
      greatest = data[i];
    }
  }
  return greatest;
}
function smallestElement(data) {
  let smallest = data[0];
  for (let i = 0; i < smallest.length; i++) {
    if (smallest > data[i]) {
      smallest = data[i];
    }
  }
  return smallest;
}
let sampleData = [1, 3, 5, 7, 9];
console.log(
  "The array: " +
    sampleData +
    " has " +
    sampleData.length +
    " elements. The average of these elements is " +
    mean(sampleData) +
    " with " +
    greatestElement(sampleData) +
    " being the largest element and " +
    smallestElement(sampleData) +
    " being the smallest element"
);
```

Output:

    The array: 1,3,5,7,9 has 5 elements. The average of these elements is 6.25 with 9 being the largest element and 1 being the smallest element

Thus, when the complexity of programs increase, it is highly useful to use functions to do specific operations. We need not spend time and energy to visualise the code that is written within a function, every time we have to call it. What is important, instead, is the operation, that function undertakes. As long as the operation encapsulated within a function is clear, it can be relied upon whenever that operation needs to be executed, without having to bother about _how that operation is indeed being executed_. This is called ‘abstraction’, and it helps in simplifying complex programs.

## Defining functions

There are two ways to define a function using the keyword `function`. _First_, a function can be _declared_ in the following manner (called function declarations):

    function name (parameter1, parameter2, ... parameterN){ ... }

_Second_, functions can be defined as a part of a larger statement of operation. One of the ways of defining functions using function expressions is as follows:

    const function(parameter1, parameter2... parameterN) { ... };

### function declarations v/s function expressions:

a) _Naming:_ Just as an identifier (or a name) is essential while declaring a variable, it is equally essential to name a function while declaring it. The name will act as a variable, storing the function itself.

```js
function sum(a, b) {
  return a + b;
} //Function declaration; 'sum' is the name of the function
console.log(typeof sum); //output: function
```

However, in a function expression, it is optional to name the function. A function without a name is called an anonymous function. Thus, function expressions can define two types of functions: named functions and anonymous functions.

But, it is important to note that, an anonymous function can, nevertheless, be stored in a variable or a constant:

```js
const ADD = function (a, b) {
  return a + b;
}; // function expression, containing an anonymous function
```

In the context of function expression, the name of the function is wholly irreleveant _outside_ the scope of that function. In other words, if a named function is defined in a function expression, that name will be accessible only within that function (i.e., it will be a local variable). To call that function outside its scope, the variable storing that function will need to be called.

```js
let f = function factorial(n) {
  if (n == 1) {
    return n;
  }
  return n * factorial(n - 1); //no error
};
console.log(f(5)); //no error
console.log(factorial(5)); // ReferenceError: factorial is not defined
```

b) _Hoisting:_ Function declarations can be hoisted. This means that it is not necessary to first declare a function before calling it (although it is advisable to do so). For example, the following code, will not throw any errors:

```js
console.log(sum(1, 3));
function sum(n, m) {
  return n + m;
}
```

However, function expressions cannot be hoisted. Thus, the following program will throw a _type error_ message:

```js
console.log(sum(1, 3));
var sum = function (n, m) {
  return n + m;
};
```

## Arrow functions

There is a third way to define functions: using a specific syntax that was introduced in ES6, called _arrow functions_. It is, in fact, a concise way to define a function expression. Thus, naming is not mandatory (as discussed above). The `function` keyword is not required either. Further, it is not mandatory to write `return`, if the body of the function contains only one line of code. The body of the function is separated from the parameters by an arrow (=>) and hence the name _arrow functions_.

The manner of defining arrow functions depends on the nature of the function expression itself, as elaborated below:

#### Functions without a parameter and with a single statement or expression:

    Syntax: () => expression;

_Function Expression:_

```js
let myFunc = function () {
  return true;
};
```

_Arrow Function:_

```js
let myFunc = () => true;
```

#### Functions with a single parameter and a single statement or expression:

    Syntax: parameter => expression;

_Function expression:_

```js
let myFunc = function (a) {
  return (a += 1);
};
```

_Arrow function:_

```js
let myFunc = (a) => (a += 1); /*no need to write the return keyword*/
```

#### Functions with multiple parameters and single statement or expression:

    Syntax: (parameter1, parameter2,...parameterN) => expression;

_Function expression:_

```js
let myFunc = function (a, b) {
  return (a = a + b);
};
```

_Arrow function:_

```js
let myFunc = (a, b) => a + b; // parameters to be written within parenthesis
```

#### Functions with multiple parameters and multiple statements and/or expressions

    Syntax: (param1, param2,...paramN) => {
      expression1;
      expression2;
      ...
      expressionN;
    }

_Function expression:_

```js
let newFunc = function (a, b) {
  let c = a - b;
  if (c < 0) {
    return b + " is greater than " + a;
  }
  return b + "either equal to or less than " + a;
};
```

_Arrow function:_

```js
let newFunc = (a, b) => {
  let c = a - b;
  if (c < 0) {
    return b + " is greater than " + a;
  }
  return b + "either equal to or less than " + a;
};
```
