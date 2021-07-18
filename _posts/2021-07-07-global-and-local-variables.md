---
layout: post
title: Understanding Global and Local Variables
tags: programming
---

In this post, I will briefly analyse the manner and implications of declaring a variable as a local variable and a global variable.

## Declaring local and global variables using `var` and `let`

When a variable is declared outside a particular function, (whether by using `var`, `let` or otherwise), that variable can be accessed by that function. This is because that variable would be considered as a 'global variable' from the perspective of that function: 

```js
var myvar = 1;
let mylet = 1;
justvar = 1;
function variables(){
   console.log(justvar); //output: 1
   console.log(mylet);// output: 1
   console.log(justvar);// output: 1
   }
variables();
```
Conversely, a variable that is declared within a function by using `let` or `var` keywords, will be considered as a 'local variable', accessible only within that function. 

## 'Global' by default

However, it is important to note that if a variable is declared without using the keywords `var` or `let`, that variable will be declared as a global variable, regardless of where they are declared. 

```js
function first(){
   var myvar = 1;
   let mylet = 1;
   justvar = 1;
   }
function second(){
   console.log(myvar);/* As myvar is defined within the function first(), this line throws
   an error message as myvar is not defined globally or within this function: 
   ReferenceError: myvar is not defined*/
   console.log(mylet);/* Similarly, this line throws an error message: ReferenceError: 
   mylet is not defined*/
   console.log(justvar);/* As the variable justvar was declared without the keywords var 
   or let, this line produces the following output: 1*/
   }
first();
second(); 
console.log(justvar); /* As the function first() has already been called,
this line produces the following output: 1*/
console.log(myvar);/* As myvar was not declared outside the function first(),this line 
throws an error message: ReferenceError: myvar is not defined*/
console.log(mylet);/* Similarly, this line throws an error message: ReferenceError: mylet 
is not defined*/
```

But to access a global variable in a function which is declared in another function, the latter function needs to be called earlier. This is because, where a variable is declared within a function, that variable will be so declared only when the specific line inside that function gets executed. As the lines of code encapsulated within a function will not be executed unless that function is called, it is necessary to have that function called, for the respective variable to be declared (whether globally or otherwise). Thus, in the above program, if the `function first()` (which declared the global variable) was not called before calling the `function second()`, a reference error would have been displayed. 

## Using strict mode

From the above discussion, it is clear that it is possible to declare variables without the `var` or `let` keywords. But such variables will automatically become global variables, which may cause certain issues while writing complex programs with multiple functions. Thus, to prevent declaring variables without `var` or `let`, one can invoke the ‘strict mode’, by typing the string `“use strict”` at the beginning of the program or a function. When the 'strict mode' is invoked, an error message will be displayed at each line where a variable is declared without using proper keywords. Thus, if the above program is run in strict mode, a `Reference error` will be displayed at the line declaring `justvar`. 

