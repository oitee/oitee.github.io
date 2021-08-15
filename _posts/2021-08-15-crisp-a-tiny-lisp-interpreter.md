---
layout: post
title: "Crisp: A Tiny Lisp Interpreter"
tags: programming
---
## Demo

<div id = "outputdiv" style = "height: 110px; overflow: scroll; border: 1px solid black; width: 100%" >
</div>
<input type = "text" id = "userinput" style = "width: 100%">

<script>
  function repl (){
    const userInput = document.getElementById("userinput");
    const outputDiv = document.getElementById("outputdiv");
    let result="";
    try {
        result = userInput.value;
        
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
