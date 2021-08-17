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
