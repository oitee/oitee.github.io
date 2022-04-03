---
layout: post
title: "Understanding Tail Recursion"
tags: conceptual
image: 
---
<img src="https://user-images.githubusercontent.com/85887016/152996634-f4a8cf78-7139-480a-b7a5-0a6a6d2b7108.png" width="30%">

In this post, I try to explain how tail call optimization can be used to make recursive functions (of certain kind) more efficient. 

### Understanding Recursion

A recursive process can be divided in two parts:

1. A base case(s), which defines a simple case (such as the first item in a sequence)
2. A recursive step, where new cases are defined in terms of previous cases.

When a recursive process is called with the base case, it simply returns the result. If the process is called with a more complex case, it would break down the problem into two parts: a part it knows how to evaluate and a part which it does not. The latter part will resemble the original case, except that it will be a slightly smaller or simpler version of it. It would then call a fresh instance of itself on this latter part to work on the simpler case.

> *What is essential for a proper use of recursion is that the objects can be expressed in
terms of simpler objects, where "simpler" means closer to the basis of the recursion.* [[Source](https://faculty.uml.edu//klevasseur/ads2/c8/c8a.pdf)]
> 

To put it in another way, a recursive algorithm can be defined as solving a problem by solving a smaller version of the same problem (a sub-problem) and then using that solution to solve the original problem. To solve a sub-problem, we need to solve a still smaller version of that sub-problem and so on. To prevent an infinite series of sub-problems, we need the series of sub-problems to eventually terminate at a base case.

## Examples of recursive functions

We can use recursion to find the factorial of a positive natural number:

```js
function factorial(n){
    if(n <= 1)//.... (1)
        return 1
    return n * factorial(n - 1)// ... (2)
}
```

In the above example, line (1) represents the base case and line (2) represents the recursive call.

## Keeping Track of Recursive Calls

Other than the base case, a recursive function cannot compute the final result (of a given input) without first knowing the result of the same function with a smaller input. For example, in the case of the factorial function, for any value of `n` (where `n != 1`), we need to first know the value of `factorial(n - 1)`, and to calculate the value of `factorial(n - 1)` we need to compute `factorial(n - 2)` and so on, till `n == 1` is true. 

This means that for **every step of this recursive ladder, we must wait for return value of the recursive call**. Till this is done, we must pause our execution. Once we have this result, we can complete our evaluation by carrying out our operation(s) on it and return the result so computed. 

So, to find the factorial of 4, we need to suspend our execution till we know the result of the factorial of 3 (and so on).  

Also, note that at each step of the recursive process, **we do not carry out the same operation with the result we get from the recursive call**. Operations at each recursive step involves variables (not constants) relevant to that recursive step. In the case of `factorial`, for each step, we need to multiply the current value of `n` with the value we get from `factorial(n  - 1)`. 

> *The answer to the new factorial subproblem is not the answer to the original problem. The value obtained for (n - 1)! must be multiplied by n* to get the final answer. [[Source](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-31.html#%_sec_5.1.4)]
> 

From the above discussion we know that for a recursive operation-

- we need a series of invocations of the same function with different values starting from a given value to the base-case value;
- to compute the value at a given recursive step, we need to wait till all the function calls from the base case leading up to it are completed;
- while waiting, we need to store the state of the values at that recursive step.

To illustrate this, let’s take the example of `factorial(n)`, where `n = 4`: 

```jsx
factorial(4) = 4 * factorial(3)
factorial(3) = 3 * factorial(2)// ... factorial(4) waiting 
factorial(2) = 2 * factorial(1)// ... factorial(4), factorial(3) waiting
factorial(1) = 1//                ... factorial(4), factorial(3), factorial(2) waiting 

factorial(2) = 2 * 1//            ... factorial(3), factorial(4) waiting
factorial(3) = 3 * 2//            ... factorial(4) waiting
factorial(4) = 4 * 6
```

Since there is no *ex ante* way of knowing the number of recursive calls required to reach the base case for a given value, **we need to be able to store the states of a dynamic number of  recursive steps**. Also, as we can see above, the order in which the states of each recursive step is accessed is one of *Last-In-First-Out,* i.e., the **values of each recursive step are accessed in the reverse order of their insertion**. Because of this property of recursive function calls, we can use stacks to store the state of each recursive step. 

## Understanding Call Stack

Whenever a function is called, a memory block (known as ‘stack frame’ or ‘activation record’) representing its state (mainly, its parameters, local variables, its return address) is pushed to a call stack that is maintained during the run-time of the program. When the execution of the function is complete, it’s corresponding memory block is popped from the call stack. At any point in time, the call stack only contains memory blocks representing functions that have been called, but not yet been executed. (As an aside, most programming languages have the functionality to display the call stack at any particular point of time during run-time. This is called stack trace and it is a useful tool for debugging run-time errors) 

In the case of recursive functions, a function that is already part of the call stack, makes another call of the same function (with a smaller sub-problem). The original function cannot be executed till this newly called function returns its value. So, a new memory block representing the newly called function gets pushed on to the call stack. The process repeats itself till we reach the base case. Once we have reached the base case, we will have reached the top of the recursive call-stack. As soon as the base-case function call returns its value, it’s associated memory block gets popped from the stack. The function that made the base-case function call, gets popped next which returns its value to the next function that had called it. This goes on till we reach the bottom of the stack. The bottom-most block on the stack represents the original function call. By the time we reach there, we will have all the values needed to complete this evaluation. 

Here’s a visualization of a recursive call stack for the factorial of 5:

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1354bb4c-7405-491d-92c7-afa73cfe4916/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220403%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220403T144114Z&X-Amz-Expires=86400&X-Amz-Signature=f6c3862b71c183c3eb44fc88489dda5d54473d068681a9e77baa32230c74097f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject" width="100%">

## Computing Space Complexity for Recursive Calls

The space complexity for a recursive function depends on the maximum size of the call stack at any point of time during its execution. For a linear recursive function, i.e., a function that makes a single call to itself during each recursive step, the maximum size of the call stack will be the sum of the number of recursive calls made by each recursive step. Thus, for the `factorial` function, the maximum size of the call stack will be  `n` (as shown above).

For non-linear recursive functions, if we visualise the pattern of recursive calls as a *recursive tree*, where each node represents a recursive call, the maximum size of the call stack will be the height of the tree. Thus, for the following function generating `n`th Fibonacci number, the maximum size of the call stack will be `n`

```bash
function fibo(n){
if (n == 0)
	return 0
if(n==1)
	return 1
return fibo(n - 1) + fibo(n - 2)
}

```

In the above example, even though we are making two recursive calls at each step, we will still only require a call stack of size `n`. This is because at each node of the recursive tree, we need to wait for it’s children nodes to return their values, and the same goes for their children nodes as well. We essentially do a depth-first traversal from each node in the recursive tree, thereby requiring only a max stack size of the height (or depth) of the recursive tree.

## Stack Overflow

The size of call stacks are finite and limited (their exact size depend on multiple factors, including the programming language and the run time environment). This means that there are only so many recursive calls we can make for a given call stack.  When we exceed this size, we get a ‘stack overflow’ error. Let’s take the following function that recursively finds the sum of the first `n` natural numbers:

```bash
function sumOfNaturalNumbers(n){
    if(n == 1)
        return 1
    return n + sumOfNaturalNumbers(n - 1)
}
```

If we pass a relatively large number to the above function, say `100000`, we get the following error 

```bash
RangeError: Maximum call stack size exceeded
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:13:5)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
    at sumOfNaturalNumbers (/home/otee/projects/recursive_factorial.js:15:16)
```

However, if we write the same function using a `for` loop and pass the same parameter (`100000`), we do not get any errors.

```bash
function sumOfNaturalNumbersIteration(n){
    sum = 0;
    for(let i = 0; i <= n; i++){
        sum += i;
    }
    return sum;
}
```

The reason why we the above prograat ism does not break is because it takes constant space to run a `for` loop, irrespective of the number of iterations being executed. To run the above loop, all we need to keep track of are the current values of `n`, `i` and `sum`.

## Tail recursion

Tail recursion is a special kind of recursion where the recursive call is the last operation carried out in the recursive case and the result of the recursive call is not manipulated by the caller. 

> *A call from procedure f ( ) to procedure g( ) is a tail call if the only thing f ( ) does, after g( ) returns to it, is itself return. The call is tail-recursive if f ( ) and g( ) are the same procedure* (Steven S. Muchnick, ***Advanced Compiler Design and Implementation*,** p 461)
> 

The most important aspect of a tail call, is that a new frame does not need to be added to the call stack for every function call. For tail calls, there is no need to store the state of the function making the tail call (as all the operations inside that function are executed by the time the tail call is made and all that is left to do is to pass on the value so returned, to its original caller). Instead, the tail-called function returns the value directly to the *original* caller. As a result, tail recursion takes constant space to be executed. In fact, a tail-call is essentially a goto statement:

> *In this way, if the last thing a procedure does is call another (external) procedure, that call can be compiled as a GOTO.* *Such a call is called tail-recursive, because the call appears to be recursive, but is not, since it appears at the tail end of the caller.* [[Source](https://dl.acm.org/doi/10.1145/800179.810196)]
> 

The above implementation of the sum of natural numbers,`sumOfNaturalNumbers`, is not written in a tail-recursive style, as we do one operation after receiving the return value from the recursive call (namely, adding the current value of `n` to the returned value). Here’s a tail-recursive  implementation of that function:

```bash
function sumOfNaturalNumbersTailRecursive(n, acc = 0){
    if(n <= 0)
        return acc
    return sumOfNaturalNumbersTailRecursive(n - 1, acc + n)
 }
```

However, this implementation, too, fails when we pass `100000` to it:

```bash
RangeError: Maximum call stack size exceeded
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:34:42)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
    at sumOfNaturalNumbersTailRecursive (/home/otee/projects/recursive_factorial.js:37:12)
```

The reason why this implementation also results in stack overflow is because JavaScript (in the NodeJs runtime environment) [does not support tail call optimization](https://stackoverflow.com/questions/42788139/es6-tail-recursion-optimisation-stack-overflow/42788286#42788286). In fact, many other programming languages, including Java, do not support tail call optimization:

> *Not all programming languages require tail-call elimination. However, in functional programming languages, tail-call elimination is often guaranteed by the language standard, allowing tail recursion to use a similar amount of memory as an equivalent loop.* [[Wikipedia](https://en.wikipedia.org/wiki/Tail_call)]
> 

Clojure, being a functional programming language, supports tail call optimization.

### Tail Recursion In Clojure

Unlike compilers of some other functional languages, Clojure’s compiler will not automatically detect a recursive tail-call and optimise it accordingly. We need to make the tail call using a special form, `recur`, to explicitly utilise tail recursion optimisation. 

> *Many such languages guarantee that function calls made in tail position do not consume stack space, and thus recursive loops utilize constant space. Since Clojure uses the Java calling conventions, it cannot, and does not, make the same tail call optimization guarantees. Instead, it provides the recur special operator, which does constant-space recursive looping by rebinding and jumping to the nearest enclosing loop or function frame. While not as general as tail-call-optimization, it allows most of the same elegant constructs, and offers the advantage of checking that calls to recur can only happen in a tail position.*[[Source](https://clojure.org/about/functional_programming)]
> 

Here’s a non-tail recursive implementation of the sum of first `n` natural numbers:

```clojure
(defn sum-of-natural-numbers 
  [n]
  (if (<= n 0)
    0
    (+ n (sum-of-natural-numbers (dec n)))))
```

If we pass `100000` to this function, we get the following output on the REPL:

```clojure
(sum-of-natural-numbers 100000)
; Execution error (StackOverflowError) at flash.tail-recursive/sum-of-natural-numbers (form-init869174823328265034.clj:6).
; null
```

Here’s a tail recursive implementation of the above function:

```clojure
(defn sum-of-natural-numbers-tail-recursive
  ([n]
   (if (<= n 0)
     0
     (sum-of-natural-numbers-tail-recursive n 0)))
  ([n acc]
   (if (<= n 0)
     acc
     (recur (dec n) (+ acc n)))))
```

This function successfully completes evaluation when we pass `100000`

```clojure
(sum-of-natural-numbers-tail-recursive 100000)
;; => 5000050000
```

Here’s a tail-recursive implementation of finding the `n`th fibonacci number:

```clojure
(defn fibo-tail-recursive
  ([n]
   (if (<= n 0)
     0
     (fibo-tail-recursive n 0 1)))
  ([n prev curr]
   (if (= n 1)
     curr
     (recur (dec n) curr (+ prev curr)))))
```

The above tail recursion examples involve a common pattern:

- For each recursive call, we pass an accumulator parameter, in addition to the main argument.
- The initial value of this accumulator represents the base case
- At each recursive step, we modify the main argument as per the definition of the problem (in the case of `sum-of-natural-numbers-tail-recursive`, decrementing the value of `n`) and update the current value of the accumulator. We pass both these parameters along with the recursive call.
- The operations we were doing after the return of the non-tail recursive function, are now done before calling the tail recursive function and this is passed as the accumulator
- The value of the accumulator in the base-case, is the final result, which is returned by the base case.

## Takeaways

1. Representing a problem in a recursive manner, involves breaking down the problem into a series of simpler sub-problems and providing a result for the simplest version of the problem
2. Although recursion is intuitive and easy to reason about while solving computational problems, they take significantly more space than iterative processes
3. A special kind of recursion, namely tail recursion, solves the problem of stack overflow, when we can express our recursive calls as tail calls.
4. We can transform (ordinary) recursive functions to tail recursive functions, with the help of an additional accumulator parameter.