---
layout: post
title: Understanding const and Immutability
tags: programming
---

In the [last post](https://oitee.github.io/2021/07/08/understanding-var-and-let.html), I explored the keyword `let` and how it is different from the keyword `var`. In this post, I will explore the keyword `const` and the manner of declaring immutable objects in javascript.

## `const` vs `var` and `let`

Like `var` and `let`, `const` is also a keyword that can be used to declare a variable. However, variables declared by `const` are read-only. This means that the value assigned to such a variable cannot be changed. Also, for a variable to be declared by `const`, it needs to be assigned a value as well. Thus, unlike `var` and `let`, which allow for variables to be declared without any value being assigned to them, one needs to assign a value to the variable being declared by `const`.

```js
var declaredByVar; /* value of declaredByVar is undefined*/
declaredByVar = 0; /* assigning a value to declaredByVar*/
let declaredByLet; /* value of declaredByLet is undefined*/
declaredByLet = 0; /*assigning a value to declaredByLet*/
const NO_VALUE_CONST;/* Syntax Error, as no value is assigned to NO_VALUE_CONST*/
const CONST_WITH_VALUE = 11;/* no error message, as a value has been assigned to CONST_WITH_VALUE*/
CONST_WITH_VALUE++;/* TypeError, as a new value cannot be assigned to CONST_WITH_VALUE*/
```

### Are they really 'variables'?

If `const` is used to declare data items that are immutable, are they really variables? _What is a variable?_ A variable is a [storage location with a symbolic name](<https://en.wikipedia.org/wiki/Variable_(computer_science)>) (like an envelope, representing the letter it contains inside). Thus, the name of the variable acts as a mere _identifier_ to the information it is assigned at a given time. The bifurcation betweeen the symbolic representation (or the name of the variable) and the information it represents, helps variables to be used in an expression or a block of code, independent of the information it holds. As a corollary to this, the information a variable represents can keep changing throughout a code.

```js
let a = 9;
a++;
```

Thus, in the above simple two-line code, the variable `a` represented `9` on first line and it represented `10` on the last line. Clearly, this would not be permissible if `a` was declared by `const`. Hence, it will be a misnomer to call `a` a variable, if it were declared by `const`.

Thus, `const` _does not_ declare variables that happen to be immutable. Instead, it declares a constant, or more precisely, _constant identifier to a value_. Thus, unlike a variable that can act as an identifier to a dynamic set of values, a constant is an immutable identifer to a specific value.

### Naming constants

Because constants declared by `const` are immutable, it is customary to name them in all-caps and separate each word with an underscore

## Objects and arrays are still mutable

Although re-assignment is prohibited for constants declared by `const`, the values of objects and arrays declared by `const` can still be altered. This is because, as discussed above, the keyword `const` assigns [a constant reference to a specific value](https://www.w3schools.com/js/js_const.asp). Now, when `const` is used to declare an array, a constant reference is established to that array. This reference cannot be altered. Thus, you cannot re-assign a new value (whether another array or otherwise) to that constant. However, you can alter the contents of that array itself. (Note that, as the values of arrays and objects declared by `const` are in fact mutable, the camelCase nomenclature is used for them).

```js
const constArray = [1, 2, 3];
constArray = [1, 2, 3, 4]; /*Type Error, as re-assignment is disallowed*/
constArray.push(4); /* constArray is now mutated to: [ 1, 2, 3, 4]*/
```

Same goes for objects declared by `const`:

```js
const constObject = {
  first: "1",
  second: "2",
};
constObject = {
  first: "1",
  second: "2",
  third: "3",
}; /*Type Error, re-assignment is not allowed*/
constObject["fourth"] = 4; // constObject is now mutated to: { first: '1', second: '2', fourth: 4 }
```

## Is there a way to preserve the properties of objects?

As shown above, the properties of an object that is declared by using the keyword `const` can be modified. So, declaring an object by `const` does not help, if the aim is to make it immutable. However, this may become necessary in certain circumstances. So, is there a way to make objects immutable as well?

### Freezing objects

Yes, objects can be _frozen_ thanks to the `Objects.freeze()` method. When an object is passed to this method, its properties and values cannot be modified. No new property can be inserted; no existing property can be removed. _Basically, the object becomes un-changeable_. However, this method can be used on any object, irrespective of whether it is declared using `const` or not.

```js
var obj = {
  prop1: "A",
  prop2: "B",
  prop3: "C",
};
obj.prop4 = "D"; /*modifying obj, by adding a new property.*/
Object.freeze(obj);
obj.prop5 = "E"; /*This attempt will fail, as the object is frozen*/
console.log(obj);
```

The output of the above program will be:

      { prop1: 'A', prop2: 'B', prop3: 'C', prop4: 'D' }

Thus, as shown above, if an object that has been _frozen_ (by passing it through `Objects.freeze()`) is attempted to be modified, that attempt will fail. But this will happen _silently_, that is, there would not be any error message displayed at the time of executing the code. However, if `strict mode` is invoked, then a `Type Error` will be displayed, as shown below.

```js
"use strict"; /* invoking strict mode*/
var obj = {
  prop1: "A",
  prop2: "B",
  prop3: "C",
};
Object.freeze(obj);
obj.prop5 = "E";
console.log(obj);
```

The output of this program will be:

      TypeError: Cannot add property prop5, object is not extensible

It is important to note that if an array is contained within a frozen object, the contents of that array can still be modified. Similarly, if an object is contained within an object that is frozen, that nested object can be modified, so long as it is not frozen as well.

```js
var objectWithArrayAndObjects = {
  array: [1, 2, 3],
  nestedObject: { prop1: "A", prop2: "B", prop3: "C" },
  variable: "1",
};
Object.freeze(ojectWithArrayAndObjects);
objectWithArrayAndObjects.array[3] = 4; // This will work
objectWithArrayAndObjects.nestedObject.prop4 = "D"; //This will work
objectWithArrayAndObjects.variable = 1; //This will fail silently
console.log(objectWithArrayAndObjects);
```

Output:

      { array: [ 1, 2, 3, 4 ],
      nestedObject: { prop1: 'A', prop2: 'B', prop3: 'C', prop4: 'D' },
      variable: '1' }

### Sealing Objects

Between fully change-able objects and frozen objects, there exists a middle ground: _sealed objects_. The method `Object.seal` is similar to the method `Object.freeze()` insofar as it prevents removal of existing properties and insertion of new properties to the object being passed to it. However, `Object.seal()` allows the existing properties of the object so sealed, to be modified.

```js
var obj = {
  A: "1",
  B: "2",
  C: "3",
};
Object.seal(obj);
obj.D = "4"; //This will fail
obj.C = 3; // This will suceed, as the property 'C' exists inside obj
console.log(obj);
```

Output:

      { A: '1', B: '2', C: 3 }
