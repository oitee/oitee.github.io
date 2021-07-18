---
layout: post
title: Understanding var and let
tags: programming
---

The keywords `var` and `let` can both be used to declare variables. But are there any differences between the two?

`var` declares a variable, but the same variable can be declared multiple times in a program or a function. Thus, if you declare the same variable twice using `var`, the last declaration will overwrite the earlier one:

```js
var num = 6;
var num = 7;// now, num stores the value 7
```

This can cause some issues when working on larger and more complicated programs. Thus, to prevent such overwriting of variables, the keyword `let` can be used. It prevents the same variable from being declared twice in the same block of code, function or operation:

```js
let num = 6;
let num = 7; /* this will throw the following error message: ‘Identifier 'num' has already been declared’*/
```

It is also important to note, that `let` will not allow a variable to be declared if that variable was earlier declared using `var` command (and *vice versa*):

```js
var num = 7;
let num = 77; /* SyntaxError: Identifier 'myvar' has already been declared*/
let newnum = 99;
var newnum = 999;/* SyntaxError: Identifier 'newnum' has already been declared*/
```

## When and to what extent can you declare the same variable more than once
 
If there is a global variable declared outside a function, the same variable can be declared afresh, using either `var` or `let`. This would not lead to any errors:

```js
var global_var = 10; /*declaring a global variable, using var*/
let global_let = 10; /*declaring a global variable, using let*/
function myvar(){
   var local_var = 1; /*declaring a new variable inside the function myvar(), using var keyword*/
   var global_var = local_var; /* declaring the global variable (global_var) inside the function myvar(), using var*/
   console.log(global_var); /* output: 1*/
   local_var += global_var; /* adding the variables, and storing it as local_var*/
   console.log(local_var);/* output: 2*/
   console.log(global_var);/* output: 1*/
}
function mylet(){
   let local_var = 3; /* declaring the variable that was already declared in the myvar()function inside the function mylet(), using let keyword*/
   let anotherLocalVar = 4; /* declaring a new variable inside the mylet() function, using let keyword*/
   let global_var = local_var; /* declaring the global variable (global_var) inside the mylet() function using let*/
   local_var += global_var;/* adding local_var and global_var, and storing it inside local_var*/
   console.log(local_var);/*output: 6*/
   console.log(anotherLocalVar);/*output: 4*/
   console.log(global_var);/*output: 3*/
}
myvar();
mylet();
```

However, within the same function, the same variable cannot be declared more than once, using `let`. Thus:

```js
var global_var = 10;
let global_let = 10;
function mylet(){
   var local_var = 3;
   let anotherLocalVar = 4;
   let local_var = global_var; /* SyntaxError: Identifier 'local_var' has already been declared*/
   let anotherLocalVar = 33;/* SyntaxError: Identifier 'anotherLocalVar' has already been declared*/ 
   console.log(local_var);
   console.log(anotherLocalVar);
}
mylet();
``` 

## Block scope

Although the `let` keyword disallows repeated declarations within the same block, statement or operation, it allows a variable that is already declared, to be declared again, specifically only for the scope of that specific operation, statement or block within which it is declared again.

Thus, the `let` keyword enables a variable of the same name to hold two different values, depending on how and where it is declared.  For example, if a local variable is declared within a function using `let`, and the same variable is again declared in a `for-loop` statement within that same function, the value of the variable will not change for the rest of the function (i.e., outside the `for-loop`): 

```js
function myfunc(){
   let local_var = 10;
   for(let local_var = 10; local_var < 15; local_var++){
       console.log("value of local_var within the for-loop " + local_var);
   }
   console.log("value of local_var outside the for-loop " + local_var);
}
myfunc();
```

The output for the above program will be: 

      value of local_var within the for-loop 10
      value of local_var within the for-loop 11
      value of local_var within the for-loop 12
      value of local_var within the for-loop 13
      value of local_var within the for-loop 14
      value of local_var outside the for-loop 10

To highlight how this is different from the `var` keyword, if the variable `local_var` was declared using `var` instead, the above program would produce the following output:

      value of local_var within the for-loop 10
      value of local_var within the for-loop 11
      value of local_var within the for-loop 12
      value of local_var within the for-loop 13
      value of local_var within the for-loop 14
      value of local_var outside the for-loop 15

Similarly, in the case of nested functions, two nested functions can declare the same variable and assign different values to it using the keyword `let`:

```js
function first(){
   let myvar = 1;
   function second(){
       let myvar = 2;
       return myvar;
   }
   console.log("value of myvar within second(): " + second());
   console.log("value of myvar within first(): " + myvar);
   return myvar;
}
first();
``` 

Output for the above program will be: 

      value of myvar within second(): 2
      value of myvar within first(): 1

## In a nutshell

- Both `var` and `let` can be used to declare variables. 

- Both `var` and `let` can be used to declare local variables within a function.

- But, `let` does not allow for re-declaration of the same variable within the same block.

- However, `let` does allow for the same variable to be re-declared separately within another block.
