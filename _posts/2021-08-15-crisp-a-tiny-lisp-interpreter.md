---
layout: post
title: "Crisp: A Tiny Lisp Interpreter"
tags: programming
---


## Introduction

### In-fix Expressions and Operator Precedence

In arithmetic expressions, the operator is always placed *in between* the operands. For example, an expression to multiply `2`, `3` and `4` would be written as follows: `2 * 3 * 4`. Because the operator is sandwiched between operands, these expressions also called **in-fix** expressions. 

But when there are different operators in a single expression, it can result in some confusion. Take for example the expression, `2 * 3 + 4`. If `2` and `3` are multiplied first and then `4` is added, the result would be `10`. However, if `3` and `4` are added first and then multiplied by `2`, the result would be `14`. 

To prevent this kind of confusion, there is a collection rules that have evolved ever since the introduction of algebraic notation, that dictate the order of precedence among operators. This is called **operator precedence**, which is often abbreviated as BEMDAS which briefly encapsulates the order among some of the common operators: Brackets, Exponents, Multiplication, Division, Addition, Subtraction).

Just like in arithmetic expressions, it can get equally confusing for the compiler of a programming language to evaluate expressions with multiple operations. To this end, most programming languages come with their own rules regarding operator precedence, to ensure consistency and uniformity, while executing . For example, in the case of JavaScript,  the assignment operator has one of the lowest precedence while expressions inside a grouping operator (`( )`) have the highest precedence.

### Pre-fix and Post-fix Expressions

Clearly, operator precedence is a set of (arbitrary) rules which are necessary to avoid the confusion that results from reading an in-fix expression. This makes prior knowledge of operator precedence vital: if one does not have prior knowledge of operator precedence, one cannot correctly evaluate an expression. Is there a way to represent expressions that does not rely on operator precedence to be evaluated?

There are two alternatives to in-fix expressions: **pre-fix** and **post-fix** expressions. In a pre-fix expression, the operator is written *before* the operands. Thus, the expression `2 * 3 + 4` would be expressed as `+ 4 * 2 3` This clarifies that the multiplication operation should be applied on `2` and `3` and their result should be added to `4`. A pre-fix expression removes any ambiguity regarding the application of operators on operands. This removes the need for developing elaborate conventions or rules to dictate operator precedence. 

Similar to a pre-fix expression, in a post-fix expression, the operator is placed *after* the set of operands. Thus, the above expression would be expressed as: `2 3 * 4 +`.

### S-expressions

Given the advantages of pre-fix expressions over traditional in-fix expressions, some programming languages solely rely on them to write logic. Typically, in such programming languages, expressions are divided into one or more 'parts' or blocks. Each such part or block will be contained within a pair of parenthesis. Such expressions are called **s-expressions**. Thus, s-expression is a special kind of pre-fix expression, where every operation is contained within a pair of parenthesis. Much like arithmetic expressions, it is possible to have nested expressions in a single s-expression. 

An s-expression essentially contains the following building blocks: **lists** and **atoms**. An **atom** is the basic unit of the expression which is not divisible any further. Each atom is differentiated from another by the use of spaces. In the above examples, the numbers `2` `3` and `4` are atoms. Other data-types, like strings and `boolean` values can also be an atom. A variable can also be an atom. 

A **List** refers to each block or segment of an s-expression which is contained within a pair of parenthesis. The first atom in the list signifies a function or an operator. Thus, if `f` represents a function and `args` represent its arguments, the syntax of a list can be represented as follows: `(f arg1 arg2 ... argn)`. Following this syntax, a simple list would be `(+ 1 2)`. An s-expression may contain one or more such lists. This is because, as explained above, lists can be nested. So, a list may be composed of one or more atoms and/or lists. For example: `(* 2 3 (+ 1 2 11))`. Here's a list of s-expressions:

![Table of valid expressions](/assets/images/valid_expressions_crisp.png)


### Lisp

Programming languages which follow this "*[fully parenthesized prefix notation](https://en.wikipedia.org/wiki/Lisp_(programming_language))*" (s-expressions) are called Lisp, which is loosely derived from the term "List Processor". Lisp was first developed by [John McCarthy in 1960](http://jmc.stanford.edu/articles/lisp/lisp.pdf). Thereafter, there have been many flavors and dialects that have emerged. Of these, Common Lisp and Clojure are two of the most well-known dialects of Lisp currently.  

Clojure is highly prevalent in modern times as seen in the latest [Stackoverflow survey](https://insights.stackoverflow.com/survey/2021#technology-top-paying-technologies). It is a dynamic programming language built on top of Java which provides a lisp syntax for writing programs which will run on Java Virtual Machine. This project is uses some of the syntax used in Clojure.

## Goal

The goal of this project is to write a tiny version of Clojure REPL, which can perform certain basic operations. 

REPL stands for read-eval-print-loop. It refers to an interactive programming environment, that takes an input from the user, evaluates it and prints the result. Once a result is printed, the environment goes back to the read state to accept fresh inputs from the user, thereby creating a loop. The commandline shell on Linux uses REPL. 

The aim of the project is to generate a REPL, which accepts an s-expression from the user, evaluates it and prints the result. 

## Scope of the Project

The project will evaluate **valid s-expressions**. If expressions are incomplete (eg., unclosed parenthesis) or if they are not written as per the syntax of an s-expression, appropriate error message will be displayed.

For the sake of simplicity, the project currently only supports **basic mathematical operations**, namely, multiplication, division, addition and subtraction. Thus, any s-expression containing any of these operations will be treated as valid. However, if an expression contains any other operator, the entire expression will be treated as invalid.

The project also **handles variables**. To initialise a variable, the keyword `def` should be used. This keyword acts as an operator and accepts  the following two arguments: name of the variable and its value. Here's how variables can be initialised: `(def nameOfVariable 9)`. The same variable can be re-initialised with a new value in a single s-expression. Further, during a REPL session, once a variable has been initialised in one s-expression, it can be re-used in subsequent s-expressions. 

An s-expression may contain two disjointed s-expressions, such that there is more than one resultant value at the end of evaluation. For example, in the following expression, `(+ 7 8) (* 1 10)`, there are two values: `15` and `10`. The project is implemented to only display the **right-most result**. In the above example, `10` will be displayed.

To quit a REPL session, `q` should be entered. 


## Running the Project

To execute this project, t[his Github Repository](https://github.com/oitee/crisp) will need to be cloned. To clone a Github repository, the following command needs to be executed on the commandLine:

```sh
git clone git@github.com:oitee/crisp.git
```

After the repository has been cloned, the necessary dependencies to run the project needs to be installed. To do this, the following command should be executed on the CommandLine:

```sh
npm install
```

Finally, to run the project, the following file in the repository should be executed on node.js:

```sh
node src/index.js
```


## Demo

Here's a demo of the project:

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


## Implementation details

### How expressions are read

The aim is to convert the input string (containing a sequence of characters) into a tree that expressly sets out the structure for evaluating each list in a valid expression. This is called **LR parsing**.  To do this, the input string is read from left-to-right, without ever going back. 

While traversing through each character in an expression, each collection of characters forming an atom is grouped together. This can achieved as spaces demarcate atoms from one another. Also, a closing parenthesis implies the end of an atom (as it is not mandatory to use a space between an operand and closing parenthesis). 

When an atom is parsed, it will be pushed to a stack. Once all the atoms of a list has been parsed and the list is complete, all the atoms belonging to that list will be popped from the stack. As every list is enclosed between a pair of parentheses, whenever a closing parenthesis is detected (while going from left-to-right of the input string), it would imply the end of a list. Thus, when a `)` is detected, all the atoms pushed to the stack (thus far) will be popped out of the stack. As only the atoms of the current list should be popped, the popping should stop the moment a `(` is popped. This would signify the beginning of the current list. 

Once all the atoms of a list are popped, the list is evaluated. This is done by passing the collection of atoms comprising the list to an evaluator function. This function will first interpret the meaning of each atom. For example, if an atom contains digits, it will call the appropriate function to convert that atom into an integer. Similarly, it will convert the operator symbol to its corresponding function. Once this process is complete, it will call the relevant operator function and pass the set of operands belonging to the current list.  The return value will be pushed back into the stack.

In this way, a **syntax tree** will be created incrementally and from bottom-up. For example, if the input expression is `(* 1 (* 5 6) (+ 7 8 9) 10)` , the atoms and lists will be built as follows:
First, the `+` operator will be pushed to the stack, along with the opening parenthesis `(`. Then, the following atoms will be pushed: `(` `*` `5` and `6`. (For the sake of simplicity the parentheses are not shown on the syntax tree).


![Syntax Tree and stack 1](/assets/images/syntax_tree_stack_1.png)


Now that there is a `)`, it will signify the end of the current list. So, all the atoms of the present list will be popped, namely  `6` `5` `*` and `(`. In place of these atoms, the value of `(* 5 6)` will be pushed into the stack. The syntax tree would now look like this:

![Syntax Tree and stack 2](/assets/images/syntax_tree_stack_2.png)

In a similar manner, the next set of atoms will be pushed, i.e., `(+ 7 8 9` will be pushed:

![Syntax Tree and stack 3](/assets/images/syntax_tree_stack_3.png)

Again, the presence of `)` would signify the end of the current list. So, all the atoms of this list will be popped and their result will be pushed into the stack. 

![Syntax Tree and stack 4](/assets/images/syntax_tree_stack_4.png)

Finally, the penultimate atom, `10` will be pushed and then all the remaining values in the stack will be passed to the addition operation along with `1`. Now that there is no other element left in the expression, the return value of this function call will be the final output, i.e., `7200`.


### Code arrangement

The project is divided into 8 modules, which carry out specific operations

- **index(*should this be changed to REPL?)*:** Hosts the REPL, which accepts the input string, passes it to the interpreter and displays the final result.
- **interpreter:** Parses the input string from left-to-right and pushes each atom to a stack. Once a list is completed, it pops all the atoms belonging to that list and calls the `eval` module to evaluate the list. Finally, it pushes the result of each list back into the stack.
- **eval:** Converts the operator symbol of each list to its respective function. This is done by calling the `operators` module. It calls the  `tokenize` module and passes every atom to it. Finally, it evaluates the list expression, by calling a reduce function and passing the operator function and the respective operands to it.
- **tokenize:** Converts each atom containing digits into an integer.
- **operators:** Maintains the table of operator functions and returns the corresponding operator function for a given operator symbol.  It maintains a variable table; each operator function accesses this table to access the value of each initialised variable.
- **reduce:** Contains a reduce function, which passes each atom of a list to the respective operator function and cumulatively stores the respective return values. Once every atom in a list has been passed to the operator function, the cumulative value is returned.
- **stack:** Maintains a stack data-structure to store the atoms of the input expression and to store the values of each list expression.
- **utils:** Contains necessary utility functions for carrying out the above operations.


### System Design

![System Design](/assets/images/system_design_crisp.png)

The project can be divided into three main components: 

**REPL:** REPL runs the interactive environment. It accepts inputs from the user, passes it to the parser component and displays the final result, once it is generated. 

**Parser:** The parser component parses each atom from the input string, by going from left-to-right. Each atom is stored in a stack data-structure. Once a list is formed, it transfers all the atoms of that list from the stack to the evaluator component. The value generated by the evaluator is pushed into the stack. 

**Evaluator:** The evaluator component receives all the atoms of each list in the input expression. It checks if any of the atoms contains any digits and converts them into integers, accordingly. It converts the operator symbols and intialised variables to their respective values. Once each atom in a list has been tokenized, it evaluates the list expression and returns the result. 

#### Test Coverage

To implement test cases for this project, the [jest](https://jestjs.io/) testing framework was used. Here is the test coverage report:

file:///home/ot/projects/crisp/coverage/lcov-report/index.html



## Further Improvements

While the project supports initialisation and re-initialisation of variables, it does not support initialisation of local variables, within a given scope. For example, in Clojure, the keyword `let` allows for initialisation of local variables within a given lexical context. The project does not support this feature.

The project does not currently envisage intialisation of functions. Currently, the program is built to only execute functions that are a part of the symbol table. The user is not provided the liberty to define her own functions.