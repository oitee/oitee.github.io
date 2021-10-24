---
layout: post
title: Creating Objects in JavaScript
tags: conceptual
image: /assets/images/objects_in_javascript.png
---

In my [earlier post](https://oitee.github.io/2021/07/14/data-types.html), I briefly touched upon how objects are an unordered collection of values. In this post, I will explore the nature of objects in JavaScript, how they are created and the concept of inheritance and prototypes.

## Objects in JavaScript

In a typical object-oriented programming language, like C++, objects are created by classes. A class is not an object by itself, but it contains constructors for creating new objects. They also determine the properties that their objects will contain. While the values of the properties of an object can be changed, it is not possible to add or delete any property that is provided by the class. 

In contrast, JavaScript is a prototype-based programming language. This means that, objects can be created by other objects. In fact, JavaScript provides three different ways of creating objects. 

- *First*, JavaScript allows us to create *ex nihilo* objects (i.e., objects created out of nothing). *Ex nihilo* objects are useful, when we need unique stand-alone objects in a program.
- *Second,* objects can also be created by constructor functions (much like in object-oriented programming languages). Unlike *ex nihilo* objects, this process of creating objects can be useful when we need to create objects that are similar to each other in some aspect (i.e., objects having certain common properties, like instances of classes in object-oriented programming languages).
- *Third,* objects can also be created by extending or cloning existing objects.

In addition to the above, unlike object-oriented programming languages, it is always permissible to create a new properties, after their creation. In other words, objects are *dynamic*  in JavaScript.

## How are objects created?

Largely, there are two ways of creating objects: by using object literals (for creating *ex nihilo* objects) and by using object constructors.

- **Object literals**: This is the simplest way to create an object. As per this method, we can declare a variable and assign it an object, which will be represented between two curly brackets. The object so assigned may contain a list of properties along with their respective names and values.

```js
let newObj = {};// empty object
let newObj2 = {name: "newObj2}; 
```

- **Using constructors**: An object can also be created by using the `new` keyword followed by a function invocation. The function so invoked is called a constructor. There are some built-in constructors:

```js
let obj = new Object();//will create an empty object
```

But one can write new constructors and create new objects using them as well. Constructors can set out the common properties that each object will have. They can also set their initial values for them. 

```js
const ConstructorOfObjects = function(name, date, serialNo) {
	this.name = name;
	this.date = date;
	this.serial = serialNo;
}
let object1 = ConstructorOfObjects("first", "July 16, 2021", 1);
```

## What are constructors?

These are functions that are used to create objects with some set properties. 

Inside the constructor function, `this` keyword is used to refer to the object being created. Thus, when `this.propertyName` is written inside a constructor, it will create a property called `propertyName` every time a new object is created by calling that constructor along with the `new` keyword. 

Essentially, the `new` keyword tells the JavaScript interpreter to create an empty object, and add the properties provided inside the constructor using the `this` keyword. As shown above, it is permissible to also pass arguments to constructors, which will be stored as values for the properties of the object (created by `this`). This way it is possible to create objects with shared properties, while also ensuring that their values are distinctly maintained.

As a corollary to the above, we can choose not to pass any arguments to the constructor function. The constructor function may also not provide for any properties. If that's the case, we will create an empty object using a constructor:

```js
let EmptyConstructor() = {};
let obj1 = new EmptyConstructor();

//A constructor that does not accept any parameter, but provides certain properties
let DefaultConstructor() = {
this.state = "default";
}
```

As shown earlier, an empty object can also be created by calling the built-in `Object` constructor. This way, we can create an *ex nihilo* object using a constructor:

```js
let exNihilo = new Object();
```

#### Naming constructors

Although constructors are technically functions, they are only used to create new objects. Thus, to distinguish them from ordinary functions (which perform specific operations), it is customary to begin the name of a constructor with an uppercase.

#### How to identify the constructors of an object?

When an object is created using a constructor, a property called `constructor` is added to that object, which carries the name of the constructor. Thus, it is very easy to know the name of the respective constructor by adding a `.constructor` after the name of that object:

  

```js
let object0 = new ANewConstructor(2441139, []);
console.log(object0.constructor == ANewConstructor);// true
```

Conversely, it may be important to test if an object is created by a constructor. This can be done using the keyword`instanceof`:

```js
object0 instanceof ANewConstructor; //True
```

#### What about object literals?

For objects created by object literals, the in-built `Object` constructor is used to create it. Thus, in the following example, the constructor of `obj1` is `Object`:

```js
let obj1 = {};
console.log(obj1.constructor == Object);//true
console.log(obj1 instanceof Object);//true
```

## Prototypes

Say, there are three objects created by a constructor `School`, namely `standardX`, `standardXI` and `standardXII`. 

```js
const School = function(subject, difficulty, medium){
	this.subject = subject;
	this.difficulty = difficulty;
	this.medium = medium;
}
let standardX = new School({"Math", "History", "Science"}, "easy", "English");
let standardXI = new School({"Math", "Computer Science", "Science"}, "intermediate", "English");
let standardXI = new School({"Math", "Computer Science", "Science", "Logic"}, "easy", "English");
```

But, say, all of `standardX`, `standardXI` and `standardXII,` belong to a school called `"Xavier High School"`. We could manually add this property to all three of them. But since it is a common property, we can simply add this property to `School.prototype`:

 

```js
School.prototype.schoolName = "Xavier High School";
console.log(standardXII.schoolName); // ""Xavier High School"
console.log(standardXII.schoolName); // ""Xavier High School"
console.log(standardXII.schoolName); // ""Xavier High School"
```

The above can be explained by the fact that every constructor has a default property called `prototype`. Just like the other properties provided by the constructor, the `prototype` property is also *inherited* by the objects created by the constructor. Thus, a prototype is a collection of a fall-back properties for an object. *It is common for all objects created by that constructor.* 

A prototype is, in fact, another object. Thus, apart from its own properties, each object also *inherits* the properties of another object, called a prototype*.* Every prototype, just like most objects, inherits properties of another prototype object. This goes on, till an object is reached which has `null` as a prototype. `Object.prototype`  is one such object: it has `null` as its prototype and, therefore, does not inherit any properties from any other object.

Thus, each object has two types of properties- properties defined directly to that object (called ‘own properties’) and properties inherited from their prototype(called 'prototype properties').

If a property is searched for inside an object, which it does not have, the JavaScript interpreter will first look into the properties of its prototype, and then the prototype of that prototype. This will continue till either the property being searched for, has been reached, or the chain of prototypes end. In the case of the former, the value of the property inside the respective prototype, will be returned. Else, `undefined`  will be returned.

#### How to check the prototype of an object?

The keyword **`isPrototypeOf`** can be used to check if an object is a prototype of another object:

```js
NameofPrototype.isPrototypeOf(nameOfObject);
```

#### What about *ex nihilo* objects? 

As explained above, objects created by an object literal is actually created by the built-in constructor, `Object`. Accordingly, the prototype of an *ex nihilo* object will be `Object.prototype`, i.e., the prototype of the `Object` constructor.

```js
let obj1 = {};
Object.prototype.isPrototypeOf(obj1);//true
```

### Prototypal inheritance

As noted in the beginning of this post, JavaScript is a prototype-based programming language, unlike other programming languages, such as C, which rely heavily on classes. This means that, at the heart of JavaScript, objects are class-less: they can be created *ex nihilo* (as discussed above) and they can also created using other objects.  

Now, given that the `prototype` property of a constructor refers to an object, we can use assign an existing object to it as well:

 

```js
//LivingBeingConstructor is a constructor
let LivingBeingConstructor = function(){
this.living = true;
}
//creating an object called animal, by calling LivingBeingConstructor()
let animal = new LivingBeingConstructor();

//Using animal as a prototype for the constructor Human
let HumanConstructor = function (){};
HumanConstructor.prototype = animal;

//The object man will inherit all the properties of animal
let man = new HumanConstructor();
```

Thus, we can clone or extend the properties of an existing object, by referring the `prototype` property of a constructor to that object.  

### Prototype cloning using `Object.Create`:

JavaScript provides a more direct way to assign objects as prototypes for newer objects: objects can be created by invoking the `Object.create` method. To do this, the prototype object can be passed as an argument to the method `Object.create`.  

So, in the above example, involving `standardX`, `standardXI` and `standardXII,`  we can create newer objects representing students studying in `standardX` by using `standardX` as a prototype.

```js

let phoebe = Object.create(standardX);// standardX is the prototype of phoebe
let anita = Object.create(standardX);// standardX is the prototype of anita
let rahul = Object.create(standardX);// standardX is the prototype of rahul

//We can create new objects with anita as a prototype as well
anita.likes = {"chocolate", "waffles", "umbrellas"};
let anitaBestFriend = Object.create(anita);

```

The same can be done with the example involving `man` and `HumanConstructor` :

```js
//creating an object elephant, that will inherit the properties of animal
let elephant = Object.create(animal);

//objects can also be created using the prototype property of another constructor
let woman = Object.create(HumanConstructor.prototype);
woman.gender = "female";

//creating annie, by using woman as a prototype
let annie = Object.create(woman);
```

In the above example, the object `annie` will inherit the properties of `woman` which inherited the properties of `HumanConstructor.prototype` which had inherited the properties of `animal` which had inherited the properties of `LivingBeingConstructor.prototype`. Finally, `LivingBeingConstructor.prototype` inherited the properties of `Function.prototype`. 
