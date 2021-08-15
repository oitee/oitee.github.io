---
layout: post
title: "Crisp: A Tiny Lisp Interpreter"
tags: programming
---

## Demo

<div id = "outputdiv" style = "height: 110px; overflow: scroll; border: 1px solid black; width: 100%" >
</div>
<input placeholder = "Type Lisp expression" type = "text" id = "userinput" style = "width: 100%">

<script>
function reduce(f, data = []) {
  let acc = data[0];
  for (let i = 1; i < data.length; i++) {
    acc = f(acc, data[i]);
  }
  return acc;
}

class Node {
  constructor(data) {
    this.data = data;
    this.next = null;
  }
}
class Stack {
  constructor() {
    this.head;
  }
  isEmpty() {
    return this.head == undefined;
  }
  push(x) {
    if (this.isEmpty()) {
      this.head = new Node(x);
      return this.head.data;
    }
    let pushedNode = new Node(x);
    pushedNode.next = this.head;
    this.head = pushedNode;
    return this.head.data;
  }
  pop() {
    if (this.isEmpty()) {
      throw "Underflow";
    }
    let popped = this.head.data;
    this.head = this.head.next;
    return popped;
  }
  peek() {
    if (this.isEmpty()) {
      throw "Empty stack";
    }
    return this.head.data;
  }
}

function tokenize(str) {
  if (typeof str !== "string") {
    throw "Present token is not a string " + str;
  }

  let charcodeMinus = "-".charCodeAt(0);
  let charcodeZero = "0".charCodeAt(0);
  let charcodeNine = "9".charCodeAt(0);
  let currentCharCode = str[0].charCodeAt(0);
  let num = 0;

  if (
    currentCharCode === charcodeMinus ||
    (currentCharCode >= charcodeZero && currentCharCode <= charcodeNine)
  ) {
    let i = 0;
    let isNegative = false;
    if (currentCharCode === charcodeMinus) {
      isNegative = true;
      i++;
    }
    for (i; i < str.length; i++) {
      currentCharCode = str[i].charCodeAt(0);
      if (currentCharCode >= charcodeZero && currentCharCode <= charcodeNine) {
        let digit = currentCharCode - charcodeZero;
        num = num * 10 + digit;
      } else {
        throw "Incompatible types: " + str;
      }
    }
    if (isNegative) {
      return -num;
    }
    return num;
  }
  return str;
}

const charCodeZero = "0".charCodeAt(0);
const charCodeNine = "9".charCodeAt(0);

function isDigit(char) {
  if (typeof char != "string" || char.length != 1) {
    return false;
  }
  let charCode = char.charCodeAt(0);
  return charCode >= charCodeZero && charCode <= charCodeNine;
}

function isInteger(n) {
  return typeof n === "number";
}

const charCodea = "a".charCodeAt(0);
const charCodez = "z".charCodeAt(0);
const charCodeA = "A".charCodeAt(0);
const charCodeZ = "Z".charCodeAt(0);

function isAlphabet(char) {
  if (typeof char != "string" || char.length != 1) {
    return false;
  }
  let charCode = char.charCodeAt(0);
  return (
    (charCode >= charCodea && charCode <= charCodez) ||
    (charCode >= charCodeA && charCode <= charCodeZ)
  );
}

function isString(str) {
  return typeof str === "string";
}

let variableTable = {};
function ifNumber(a) {
  if (typeof a == "number") {
    return a;
  }
  if (variableTable.hasOwnProperty(a)) {
    return variableTable[a];
  }
  throw "Operand is not a number: " + a;
}

const operators = {
  "*": function (n, m = 1) {
    return ifNumber(n) * ifNumber(m);
  },
  "+": function (n, m = 0) {
    return ifNumber(n) + ifNumber(m);
  },
  "/": function (n, m = 1) {
    return ifNumber(n) / ifNumber(m);
  },
  "-": function (n, m = 0) {
    return ifNumber(n) - ifNumber(m);
  },
  def: function (variable, value) {
    if (isString(variable)) {
      variableTable[variable] = ifNumber(value);
    } else {
      throw "Variable name is not a string: " + variable;
    }
    return ifNumber(value);
  },
};

function findOperator(str) {
  if (operators.hasOwnProperty(str)) {
    return operators[str];
  }
  throw "Invalid operator: " + str;
}

function lispEval(tokens) {
  if (tokens.length == 0) {
    throw "Empty expression is invalid";
  }
  let operationFn = findOperator(tokens[tokens.length - 1]);
  let operands = [];
  for (let i = tokens.length - 2; i >= 0; i--) {
    if (typeof tokens[i] === "number" || typeof tokens[i] == "string") {
      operands.push(tokens[i]);
    } else {
      throw "Operand is not valid: " + tokens[i];
    }
  }
  return reduce(operationFn, operands);
}

function lisp(expr) {
  let chars = [...expr],
    s = new Stack();
  let atom = "";

  for (let i = 0; i < chars.length; i++) {
    let presentChar = chars[i];
    if (presentChar != " " && presentChar != ")" && presentChar != "\n") {
      if (presentChar === "(") {
        s.push(presentChar);
        atom = "";
      } else {
        atom = atom + presentChar;
      }
    } else {
      if (atom !== "") {
        atom = tokenize(atom);
        s.push(atom);
        atom = "";
      }
      if (presentChar === ")") {
        let tokens = [],
          poppedValue = s.pop();
        while (poppedValue !== "(") {
          tokens.push(poppedValue);
          poppedValue = s.pop();
        }
        let value = lispEval(tokens);
        if (value != null) {
          s.push(value);
        }
      }
    }
  }

  let result = s.pop();
  while (!s.isEmpty()) {
    if (s.pop() == "(") {
      throw "Incomplete expression";
    }
  }

  if (isInteger(result)) {
    return result;
  }
  throw "Unexpected result";
}


  
  function repl (){
    const userInput = document.getElementById("userinput");
    const outputDiv = document.getElementById("outputdiv");
    let result="";
    try {
        result = lisp(userInput.value);
        
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
