---
layout: post
title: "Crisp: A Tiny Lisp Interpreter"
tags: programming
---

<style>
.img-container { 
  float: left; 
  width: 33.33%; 
  padding: 5px; 
}
.clearfix::after {
  content: "";
  clear: both;
  display: table;
}

</style>

## Introduction

## Expressions and Operator Precedence

When we write arithmetic expressions, we place the operator in between the operands. For example, if we want to write a simple expression to multiply `2` with `3`, we will express it as: `2 * 3`. The same goes when we have multiple operands: `2 * 3 * 4` It is clear what this expression represents: we need to apply the operator (`*`) with the operands (`2, 3, 4`). 

But what happens when we have two different kinds of operands, like, `2 * 3 + 4`? Do we multiply `2` and `3` first, and then add the result of that operation with `4`? This would give us a result of `10`. Or, do we first add `3` with `4` first and multiply the result with `2`, resulting in `14`? This would make evaluating expressions with multiple operators very unpredictable. To prevent this kind of confusion, there is a collection rules that have evolved ever since the introduction of algebric notation, that dictate the order of precedence among operators. This is called operator precedence, which is often abbreviated as BEMDAS which briefly encapsulates the order among some of the common operators: Brackets, Exponents, Multiplication, Division, Addition, Subtraction. Thus, any expression within a pair of parenthesis will be evaluated first. Similarly, the multiplication operation would precede the addition operation which would precede subtraction. 

These set of rules were a borne out of necessity, as otherwise, it can be (as shown above) very difficult to precisely evaluate an expression. Just like in arithmatic expressions, it can get equally confusing for the compiler of a programming language to evaluate expressions with multiple operations. To this end, most programming languages come with their own rules regarding operator precedence, to ensure consistency and uniformity, while executing . For example, in the case of JavaScript,  the assignment operator has one of the lowest precedence while expressions placed inside a grouping operator (`( )`) have the highest precedence.

#### Is there a way to avoid reliance on operator precedence?

Because the operator is sandwiched between operands in expressions discussed above, these expressions are also called 'in-fix' expressions. Clearly, operator precedence is an arbitrary set of rules which are necessary to remove confusion that results from reading an in-fix expression. Is there a way to represent expressions that do not cause the confusion in the first place?

Largely, there are two alternative ways to write an expressions, **pre-fix and post-fix**. In a pre-fix expression, the operator is written *before* the operands. Thus, the expression `2 * 3 + 4` would be expressed as `+ 4 * 2 3` This clarifies that the multiplication operation should be applied on `2` and `3` and their result should be added to `4`. A pre-fix expression removes any ambiguity regarding the [*the operands on which an operator would apply*]. Thus, the need for elaborate conventions or rules to dictate operator precedence does not arise when expressions are expressed in this manner. In a post-fix expression, the operator is placed *after* the set of operands. Thus, the above expression would be expressed as: `2 3 * 4 +`.

#### S-expressions

Given the advantages of pre-fix expressions over traditional in-fix expressions, some programming languages solely rely on them to write logic. Typically, in such programming languages, expressions are divided into one or more 'parts' or blocks. Each such part or block will be contained within a pair of parenthesis. In other words, each segment of an expression will start with an opening bracket and end with a closing bracket. Also, there can be *nested* blocks: a block can sit inside another larger segment. Such expressions are called **s-expressions**.

An expression essentially will contain the following building blocks: **lists** and **atoms**(also called 'tokens'). An **atom** is the basic unit of the expression which is not divisible any further. Each atom is differentiated from another by the use of spaces. In the above examples, the numbers `2` `3` `1` `11` are atoms. A string or a `boolean` value can also be an atom. **List** refers to each block or segment of an s-expression which is contained within a pair of parenthesis. The first atom in the list signifies a function or an operator. Thus, if `f` represents a function and `args` represent its arguments, the syntax of a list can be represented as follows: `(f arg1 arg2 ... argn)`. Following this syntax, a simple list would be `(+ 1 2)`. An s-expression may contain one or more such lists. This is because, as explained above, lists can be nested. So, a list may be composed of one or more atoms and/or lists. For example: `(* 2 3 (+ 1 2 11))`. 

#### Lisp

Programming languages which follow this "*[fully parenthesized prefix notation](https://en.wikipedia.org/wiki/Lisp_(programming_language))*" (s-expressions) are called LISP, which is loosely derived from the term "List Processor". Lisp was first developed by [John McCarthy in 1960](http://jmc.stanford.edu/articles/lisp/lisp.pdf). Thereafter, there have been many flavors and dialects that have emerged. Of these, Common Lisp and Clojure are two of the most well-known dialects of Lisp currently.  

Clojure is highly prevalent in modern times as seen in the latest [Stackoverflow survey](https://insights.stackoverflow.com/survey/2021#technology-top-paying-technologies). It is a dynamic programming language built on top of Java which provides a lisp syntax for writing programs which will run on Java Virtual Machine. This project is uses some of the syntax used in Clojure.


## Goal

The goal of this project is to write a tiny version of Clojure repl, which can perform certain basic operations. 

Repl stands for read-eval-print-loop. It refers to an interactive programming environment, that takes an input from the user, evaluates it and prints the result. Once a result is printed, the environment goes back to the read state to accept fresh inputs from the user, thereby creating a loop. The commandline shell on Linux uses Repl. 

Thus, the aim is to write a program which accepts an s-expression from the user, evaluates it and prints the result.



## Implementation details

The aim is to convert the input string (containing a sequence of characters) into a tree that expressly sets out the order (or structure) of evaluating each list in a valid expression. This is called LR parsing.  To do this, the input string is read from left-to-right, without ever going back. While traversing through each character in an expression, each collection of characters forming an atom is grouped together. This can achieved as spaces demarcate atoms from one another. Also, a closing parenthesis implies the end of an atom (as it is not mandatory to use a space between an operand and closing parenthesis). When an atom is parsed, it is will be passed to a function (`tokenize`) to interpret its meaning. For example, if its a collection of digits, the `tokenize` function will convert it to an integer. Similarly, if it is an operator, the `tokenize` function will return the relevant function. Once an atom is *tokenized*, it will be pushed to a stack. 

When a list is complete, all the atoms belonging to that list will need to be popped from the stack. As every list is enclosed between a pair of parenthesis, whenever a closing parenthesis is detected (while going from left-to-right of the input string), it would imply the end of a list. Thus, when a `)` is detected, all the atoms which were pushed to the stack will be popped. Again, to limit the popping to only atoms of the current list, the popping should stop the moment a `(` is popped. This would signify the beginning of the current list. 

Once all the atoms of a list are popped, the operator function is called and all the operands (or arguments) are passed to it. The return value is pushed back to the stack. In this way, a **syntax tree** is created incrementally and bottom-up. For example, if the input expression is `(* 1 (* 5 6) (+ 7 8 9) 10)` , the atoms and lists will be built as follows:

First, the `+` operator will be pushed to the stack, along with the opening parenthesis `(`. Then, the following atoms will be pushed: `(` `*` `5` and `6`. (For the sake of simplicity the parentheses are not shown on the syntax tree).


<div class="clearfix">
  <div class="img-container">
  <img src="/assets/images/syntaxTree_1.png" style ="width:100%">
  </div>
  <div class="img-container">
  <img src="/assets/images/stack_1.png" style="width:50%">
  </div>
</div>

 

Now that there is a `)`, it will signify the end of the current list. So, all the atoms of the present list will be popped, namely  `6` `5` `*` and `(`. In place of these atoms, the value of `(* 5 6)` will be pushed into the stack. The syntax tree would now look like this:



<div class="clearfix">
  <div class="img-container">
  <img src="/assets/images/syntaxTree_1.png" width="100%">
  </div>
  <div class="img-container">
  <img src="/assets/images/stack_1.png" style="width:40%">
  </div>
</div>

In a similar manner, the next set of atoms will be pushed, i.e., `(+ 7 8 9` will be pushed:



<div class="clearfix">
  <div class="img-container">
  <img src="/assets/images/syntaxTree_2.png" width="100%">
  </div>
  <div class="img-container">
  <img src="/assets/images/stack_2.png" width="40%">
  </div>
</div>

Again, the presence of `)` would signify the end of the current list. So, all the atoms of this list will be popped and their result will be pushed into the stack. 

<div class="clearfix">
  <div class="img-container">
  <img src="/assets/images/syntaxTree_3.png" width="100%">
  </div>
  <div class="img-container">
  <img src="/assets/images/stack_3.png" width="60%">
  </div>
</div>

Finally, the penultimate atom, `10` will be pushed and then all the remaining values in the stack will be passed to the addition operation along with `1`. Now that there is no other element left in the expression, the return value of this function call will be the final output, i.e., `7200`.

<div class="clearfix">
  <div class="img-container">
  <img src="/assets/images/syntaxTree_4.png" width="100%">
  </div>
  <div class="img-container">
  <img src="/assets/images/stack_4.png" width="50%">
  </div>
</div>




## Demo

<div id = "outputdiv" style = "height: 110px; overflow: scroll; border: 1px solid black; width: 100%" >
</div>
<input placeholder = "Type Lisp expression" type = "text" id = "userinput" style = "width: 100%">

<script>
!function(t,e){"object"==typeof exports&&"object"==typeof module?module.exports=e():"function"==typeof define&&define.amd?define([],e):"object"==typeof exports?exports.crisp=e():t.crisp=e()}(self,(function(){return(()=>{"use strict";var t={d:(e,r)=>{for(var n in r)t.o(r,n)&&!t.o(e,n)&&Object.defineProperty(e,n,{enumerable:!0,get:r[n]})},o:(t,e)=>Object.prototype.hasOwnProperty.call(t,e),r:t=>{"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(t,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(t,"__esModule",{value:!0})}},e={};function r(t,e){for(var r=0;r<e.length;r++){var n=e[r];n.enumerable=n.enumerable||!1,n.configurable=!0,"value"in n&&(n.writable=!0),Object.defineProperty(t,n.key,n)}}function n(t,e){if(!(t instanceof e))throw new TypeError("Cannot call a class as a function")}t.r(e),t.d(e,{lisp:()=>c});var o=function t(e){n(this,t),this.data=e,this.next=null},i=function(){function t(){n(this,t),this.head}var e,i;return e=t,(i=[{key:"isEmpty",value:function(){return null==this.head}},{key:"push",value:function(t){if(this.isEmpty())return this.head=new o(t),this.head.data;var e=new o(t);return e.next=this.head,this.head=e,this.head.data}},{key:"pop",value:function(){if(this.isEmpty())throw"Underflow";var t=this.head.data;return this.head=this.head.next,t}},{key:"peek",value:function(){if(this.isEmpty())throw"Empty stack";return this.head.data}}])&&r(e.prototype,i),t}();"0".charCodeAt(0),"9".charCodeAt(0),"a".charCodeAt(0),"z".charCodeAt(0),"A".charCodeAt(0),"Z".charCodeAt(0);var a={};function u(t){if("number"==typeof t)return t;if(a.hasOwnProperty(t))return a[t];throw"Operand is not a number: "+t}var f={"*":function(t){var e=arguments.length>1&&void 0!==arguments[1]?arguments[1]:1;return u(t)*u(e)},"+":function(t){var e=arguments.length>1&&void 0!==arguments[1]?arguments[1]:0;return u(t)+u(e)},"/":function(t){var e=arguments.length>1&&void 0!==arguments[1]?arguments[1]:1;return u(t)/u(e)},"-":function(t){var e=arguments.length>1&&void 0!==arguments[1]?arguments[1]:0;return u(t)-u(e)},def:function(t,e){if("string"!=typeof t)throw"Variable name is not a string: "+t;return a[t]=u(e),u(e)}};function h(t){if(0==t.length)throw"Empty expression is invalid";for(var e=function(t){if(f.hasOwnProperty(t))return f[t];throw"Invalid operator: "+t}(t[t.length-1]),r=[],n=t.length-2;n>=0;n--){if("number"!=typeof t[n]&&"string"!=typeof t[n])throw"Operand is not valid: "+t[n];r.push(t[n])}return function(t){for(var e=arguments.length>1&&void 0!==arguments[1]?arguments[1]:[],r=e[0],n=1;n<e.length;n++)r=t(r,e[n]);return r}(e,r)}function s(t){if("string"!=typeof t)throw"Present token is not a string "+t;if(function(t){return f.hasOwnProperty(t)}(t))return t;var e="-".charCodeAt(0),r="0".charCodeAt(0),n="9".charCodeAt(0),o=t[0].charCodeAt(0),i=0;if(o===e||o>=r&&o<=n){var a=0,u=!1;for(o===e&&(u=!0,a++);a<t.length;a++){if(!((o=t[a].charCodeAt(0))>=r&&o<=n))throw"Incompatible types: "+t;i=10*i+(o-r)}return u?-i:i}return t}function p(t,e){(null==e||e>t.length)&&(e=t.length);for(var r=0,n=new Array(e);r<e;r++)n[r]=t[r];return n}function c(t){for(var e=function(t){if(Array.isArray(t))return p(t)}(l=t)||function(t){if("undefined"!=typeof Symbol&&null!=t[Symbol.iterator]||null!=t["@@iterator"])return Array.from(t)}(l)||function(t,e){if(t){if("string"==typeof t)return p(t,e);var r=Object.prototype.toString.call(t).slice(8,-1);return"Object"===r&&t.constructor&&(r=t.constructor.name),"Map"===r||"Set"===r?Array.from(t):"Arguments"===r||/^(?:Ui|I)nt(?:8|16|32)(?:Clamped)?Array$/.test(r)?p(t,e):void 0}}(l)||function(){throw new TypeError("Invalid attempt to spread non-iterable instance.\nIn order to be iterable, non-array objects must have a [Symbol.iterator]() method.")}(),r=new i,n="",o=0;o<e.length;o++){var a=e[o];if(" "!=a&&")"!=a&&"\n"!=a)"("===a?(r.push(a),n=""):n+=a;else if(""!==n&&(n=s(n),r.push(n),n=""),")"===a){for(var u=[],f=r.pop();"("!==f;)u.push(f),f=r.pop();var c=h(u);null!=c&&r.push(c)}}for(var l,d=r.pop();!r.isEmpty();)if("("==r.pop())throw"Incomplete expression";if("number"==typeof d)return d;throw"Unexpected result"}return e})()}));

  
  function repl (){
    const userInput = document.getElementById("userinput");
    const outputDiv = document.getElementById("outputdiv");
    let result="";
    try {
        result = crisp.lisp(userInput.value);
        
    } catch(e) {
        result = "Error: " + e;
    }

    outputDiv.innerText += "> " + result + "\n";
    userInput.value="";
    outputDiv.scrollTop = outputDiv.scrollHeight;
  }
  const userinput = document.getElementById("userinput");
    userinput.addEventListener("keydown", function(event) {
    if (event.key === "Enter") {
        repl();
    }});
  </script>
