---
layout: post
title: Implementing Stacks
tags: programming
---

Stack is a common data structure that is often used to retrieve and insert data items in a specific order. In this post, I will implement some of the common operations involving stacks.

## What are stacks?

Stacks are data-structures, where the data items are arranged and retrieved as per the last-in-first-out (LIFO) order. **This means that the last item added to the data structure will be the first item that can be accessed.** This kind of data structure is modelled after a real-world _stack_ of plates in a kitchen sink. It can also be thought of as a jar of biscuits, where one can only reach the first item before moving onto the next one. 

LIFO has many practical applications. For example, **the undo-operation (ctrl + z) that is available in various computer programs, follow the LIFO principle**, as we always need to access the last change carried out. [Brackets are opened and closed](/2021/07/25/valid-parenthesis.html) as per this order as well. 

Also, function calls are internally maintained as a stack by the computer. If we have three functions f, g and h and they are interwoven in their calls, i.e., f(g(h(x, y))), then the function f cannot be called until the function g has finished and returned, which cannot be called until h has finished and returned. Function h is the last function that was called and is the first that must be finished.

#### Common operations

Some of the common operations involving stack are as follows:

`isEmpty()`: returns `true` if a stack is empty, i.e., if it does not contain any data items.

`push()`: adds a new item at the top of the stack.

`pop()`: deletes the last added item from a stack and returns the deleted item

`peek()`: returns the last added item from a stack.

<img src="/assets/images/stackPushPop.jpg" alt="Pop and push operations" width="100%"/>

## Implementation

To create a stack, we need to first write an appropriate constructor function. Much like how linked lists are created, each new stack data structure can be created by invoking this constructor function along with the `new` function.

### Designing the constructor function

The constructor function will essentially create objects carrying certain common properties. Recall that, the [concept of class-based objects](/2021/07/19/creating-objects-in-javascript.html) can be incorporated in JavaScript, by using constructor functions, which can set the common properties which each object will possess.

Now, we know that we need to be able to insert and delete items as per the LIFO order. To do this, we will need to write functions that can carry out the aforesaid `push` and `pop` operations effectively. As these functions will be integral to this data structure, they will need to be available to every object created by the constructor function. Thus, the constructor will create these functions as properties for each object it creates. When functions sit inside an object (as properties of that object), they are called 'methods'.

Apart from creating methods, the constructor will also need to provide a space to store the data items, which the requisite methods can access. To this end, we can either rely on arrays or linked lists.

Thus, from the above discussion, it is clear that to implement a stack data structure, we need to write a constructor function, which will have two components: a) a set of methods, that carry out the requisite operations associated with stacks, and b) a container for holding the data items to be stored inside a stack.

Thus, the constructor function, that relies on linked lists, would look something like this:

```js
function Stack(){
this.head;//for storing the latest inserted node in the list
this.Node = function {...};// for creating the linked list to store the data-items
this.isEmpty = function {...};
this.pop = function(){...};
this.push = function(x){...};
this.peek = function {...};
}
let newStack = new Stack();// creates a new stack data structure by invoking Stack()

```

Similarly, the constructor function, that relies on arrays, will look something like this:

```js
function Stack(){
this.arr = [];//for storing the data items
this.index = -1; //for storing the index of the top-most item
this.isEmpty = function() {...};
this.pop = function(){...};
this.push = function(x){...};
this.peek = function(){...};
}
let newStackUsingArray = new StackUsingArray();
```

In the following part of this post, I will show how each of these methods will be written. (To see how a constructor function for linked lists is written, please see [my earlier post on this topic](/2021/07/28/linked-lists-in-javascript.html).)

### isEmpty()

The task of this method is to determine if a stack object (created by the constructor) is holding any data item presently. This method would not only be useful to the user working with stacks, but will also be relied upon by the other methods inside the stack object (as shown below). If a stack is empty, this method should return `true`. Else, it will return `false`.

_For stacks built using linked lists:_

We need to see if the `this.head` item is `undefined`. If yes, it would indicate either that no node has yet been created, or that every node that was created has since been popped out of the stack.

```js
this.isEmpty = function () {
  return this.head == undefined;
};
```

_For stacks built using arrays:_

In this case, a stack would be considered empty, if the value stored inside `this.index` is less than 0. The initial value of `this.index` will always be -1. With each item being pushed into the stack, the value of `this.index` will be increased by one. Conversely, with each item being popped out of the stack, the value of `this.index` will be reduced by one. Thus, when the value of `this.index` goes below 0, it will indicate either that no item has been pushed into the stack, or that every item that was pushed into the stack had been popped out.

```js
this.isEmpty = function(){
        return this.index < 0;
    }
```

### push()

This method will accept a data item as an argument and will push that item into the stack. The newly pushed item will now be the top-most item. So, the value of `this.head` (in the case of linked lists) or the value of `this.index` (in the case of arrays) will need to be changed accordingly, with each invocation of this method. Finally, this method should return the value of the item that has been pushed into the stack.

_For stacks built using linked lists:_

When a new item is to be pushed, a new node (carrying that item) should be inserted into the linked list. As shown in [my earlier post](/2021/07/28/linked-lists-in-javascript.html), the new node can be inserted at the head of the linked list.

```js
this.push = function (x) {
  if (this.isEmpty()) {
    this.head = new this.Node(x);
    return this.head.data;
  }
  let pushedNode = new this.Node(x);
  pushedNode.next = this.head;
  this.head = pushedNode;
  return this.head.data;
};
```

_For stacks built using arrays:_

In this case, we need to insert the new item at `this.index +1` of `this.arr`, i.e., the index right after the index of the last inserted item. Note that, while working with some programming languages such as C and Java, where arrays are of fixed capacity, we should first check if adding a new item to the array would exceed the capacity that was initially allocated for that array. However, this is not necessary in JavaScript, as arrays are always dynamically resized.

```js
this.push = function (x) {
  this.index++;
  this.arr[this.index] = x;
  return x;
};
```

### pop()

This method will delete the last item that was pushed into the stack and return that item to the caller. However, before carrying out this operation, it would be important to see if the stack is empty. This can be done by invoking the `this.isEmpty()` method. If the stack is empty, an appropriate error message should be returned.

_For stacks built using linked lists:_

If the stack is not empty, the node next to the head node should be replaced as the new head node of the linked list, i.e., `this.head = this.head.next`. But before carrying out this change, it would be important to store the data carried by `this.head`, as we will need to return that value:

```js
this.pop = function () {
  if (this.isEmpty()) {
    return "stack underflow";
  }
  let popped = this.head.data;
  this.head = this.head.next;
  return popped;
};
```

_For stacks built using arrays:_

For stacks using arrays, we need to return the data item at the `this.index` of `this.arr` and reduce the value `this.index` by one.

```js
this.pop = function () {
  if (this.isEmpty()) {
    return "stack underflow";
  }
  let popped = this.arr[this.index];
  this.index--;
  return popped;
};
```

### peek()

This method will return the last item that was pushed to a stack.

_For stacks built using linked lists:_

For stacks using linked lists, we need to return the value stored in `this.head.data`. But before doing this, we need to first check if the stack is empty.

```js
this.peek = function () {
  if (this.isEmpty()) {
    return "stack empty";
  }
  return this.head.data;
};
```

_For stacks built using arrays_:

We need to return the value stored at `this.index` of `this.arr` of the stack. But, if the stack is empty, we need to return an appropriate response.

```js
this.peek = function () {
  if (this.isEmpty()) {
    return "stack empty";
  }
  return this.arr[this.index];
};
```
