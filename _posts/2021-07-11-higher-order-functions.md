---
layout: post
title: Higher Order Functions
---

In this post, I explore the concept of higher-order functions and discuss three common examples of higher-order functions.

## Functions as values

In the [last post](https://oitee.github.io/2021/07/10/functions.html), I explored the advantages of having functions in a program: they can reduce repetition and encapsulate specific tasks within a program. Apart from this, in Javascript, functions are treated as values as well. This means that functions can be stored in variables and constants. They can be contained as properties of objects and as elements of arrays as well. What is more important, functions can be passed as arguments to other functions. Functions can also be the return values of other functions. *Effectively,* functions are treated just as any other value is treated in Javascript. This is what makes functions first class citizens of Javascript.

Functions that accept other functions as parameters or return other functions, are called higher-order functions. Here is a simple example of a higher-order function:

```js
const operation = (operator, operand) => operator(operand);
console.log(operation(a => a * a, 5)); //output: 25
```

In the above program, the function `operation` accepts a value (`operand`) and a function (`operator`). It passes the `operand` to the `operator` function and returns the value so generated. In the previous post, it was shown how functions can _abstract_ operations and make lives easy. But in Javascript, functions can also _abstract_ other functions, by using higher-order functions.

## `map()`

The utility of the above function may not be obvious when we are dealing with a single set of values. But, when an operation needs to be applied on a series of values (like, the elements of an array), we can have a function called `map()` that does a simple iteration: it will pass each element of a data-set to any function (that is passed to it) and return the modified data-set. Thus the `map` function will carry out the following operation:

    map([a, b,...n], f()) = [f(a), f(b), … f(n)]

This makes life simple, as we can use `map()` to modify the elements of an array in a number of ways. All we need is the manner in which we need to modify the elements (i.e., a function expression) and the array itself. We no longer have to write a separate function, to manually carry out each kind of modification to each element of the array, thanks to `map()`. This will be clear from the following code:

```js
const map = function (f, data) {
  let result = [];
  for (let i = 0; i < data.length; i++) {
    result[i] = f(data[i]);
  }
  return result;
};
console.log(map((a) => a + 1, [1, 2, 3, 4, 5])); //output: [ 2, 3, 4, 5, 6 ]
console.log(map((a) => a % 10, [1, 2, 3, 4, 5])); // output: [ 1, 2, 3, 4, 5 ]
console.log(map((a) => a + a, [1, 2, 3, 4, 5])); // output: [ 2, 4, 6, 8, 10 ]
```

## `reduce()`

Another example to illustrate the utility of higher-order functions, is the function `reduce()`. The `reduce()` function accepts three parameters: a data-set (i.e., an array containing a list of elements), a function, and an accumulator. The `reduce()` function passes each element of the data-set to the passed function and cumulatively stores the respective return values in the accumulator. Finally, `reduce()` returns the value stored in the accumulator, at the end of the iteration. The `reduce()` function carries out the following operation:

    reduce(f, accumulator, data) = f(...(f(f(f(f(accumulator, data[0]), data[1]), data[2]), data[3])...)data[n])

Thus, the passed function is called once for each element of the data-set that is passed to it. The first time the passed function is called, the accumulator and the first element of the array is passed to it. The return value is stored in the accumulator. The second time the passed function is called, the second element of the array and the accumulator (which contains the return value of the last function-call) is passed. This continues till the last element of the array is reached. Thereafter, the value stored in the accumulator is returned.

```js
function reduce(f, acc, data) {
  for (let i = 0; i < data.length; i++) {
    acc = f(acc, data[i]);
  }
  return acc;
}
//Calling reduce() to multiply each element of an array
console.log(reduce((acc, item) => acc * item, 1, [1, 2, 3, 4])); // Output: 24
```

The function `reduce()` can be called to convert an array into a string, with each element of the array being separated by an “`and`”. (Note that, for the first item, the string “and” should not be appended):

```js
const concat = function (acc, item) {
  if (acc == undefined) {
    return item;
  }
  return acc + " and " + item;
};
console.log(reduce(concat, undefined, ["apple", "mango", "orange"])); //output:apple and mango and orange
```

A key difference between `map()` and `reduce()` is that, while `map()` will always return the data-set, with its elements modified as per the function that is passed to it, `reduce()` will return a single cumulative value, after passing each element and the accumulator to the passed function.

## Writing `map()` using `reduce()`

Although the return values of `map()` and `reduce()` are quite different, the function `map()` can be re-written such that it relies on `reduce()` to return the modified data-set.

This function, say `newMap()`, will accept two parameters: a function and a data-set. We know that `reduce()` will always return a single value, but the `map()` function will always return the entire data-set, with requisite modifications. Thus, to effectively utilise `reduce()` inside `newMap()`, we will need to set the accumulator as an empty array. The idea will be that `reduce()` will store each return value of the passed function, as a new element of that array. Once the iteration is complete, `reduce()` will return the entire array. Thus, `newMap` will carry out the following operation:

    map(f, data) = reduce((f, acc, item) => {acc.push(f(item)); return acc;}, [], data)

```js
function new_map(f, data) {
  var acc = [];
  const func = (acc, item) => {
    acc.push(f(item));
    return acc;
  };
  return reduce(func, [], data);
}
console.log(new_map((n) => n + 1, [1, 2, 3, 4])); // return: [2, 3, 4, 5]
```

## `Filter()`:

A third example of higher-order functions is `filter()`. `filter()` accepts two parameters: a function and a data-set. It passes each item of the data-set to the passed function. The passed function is expected to return only `true` or `false`. The `Filter()` returns an array containing only those elements for whom the passed function returned `true`.

```js
function filter(f, data) {
  var result = [];
  for (var i = 0; i < data.length; i++) {
    if (f(data[i])) {
      result.push(data[i]);
    }
  }
  return result;
}
console.log(filter((a) => a % 2 == 0, [0, 1, 2, 3, 4, 5, 6])); // output: [ 0, 2, 4, 6 ]
```

`filter()` is an effective tool to _filter_ out elements of an array that fail to pass a specific test or condition.
