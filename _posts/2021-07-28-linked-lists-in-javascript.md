---
layout: post
title: Implementing Linked Lists
tags: programming
image: /assets/images/linked_lists.png
---

In the [previous post](/2021/07/27/data-structures-arrays-and-linked-lists.html), I wrote about the concept of linked lists and how they are different from arrays. In this post, I write about some of the basic operations that can be carried out on a linked list.

## Creating a linked list

To create a linked list, we need to first write a constructor function that will create each node of the list. Typically, a node will be a JavaScript object created by the constructor function. The constructor will accept one or more values as arguments for storing them as properties in each node object. The constructor will also create one or more properties that will act as pointers to one or more nodes in the list.   

In the example below, the constructor function `Node` will assign two properties to each node it creates, namely:  `this.data`, which will store the value passed to `Node` constructor, and `this.next` which will contain the link to the next object in this list. `this.next` will be initially be assigned to `null`.

```js
function Node(data){
    this.data = data;
    this.next = null;
}
```

When a new node is created by the constructor function, its `next` property will point to `null`. So, each time a subsequent node is created within a linked list, it is important to link the `next` property of the previous node to that new node. 

```js
let head = new Node(7);//head node
head.next = new Node(8);// second node
head.next.next = new Node(9);// third node
head.next.next.next = new Node(10);// fourth node
```

## Constructing a linked list from an array

While it is possible to create new nodes by manually linking them to the previous node's `.next` property, it is also possible to do this iteratively. For example, given an array, it is possible to create a linked list using a `for loop` and a temporary variable (say `tail`). During each iteration of the loop, `tail` can be used to point to the last node that was created in the linked list, and `tail.next` can be used to create the next new node in the list. Once the new node is created, `tail` should point to that node: `tail = tail.next`. This will be repeated for each element of the array. However, it is important to store the location of the first node, as a linked list is identified by its head node. Thus, we must create the head node outside the loop, and assign it to the `tail` variable, which can, then, be used to create the subsequent nodes in the list. 

Note that, we can use loops to change the `.next` properties of every node (which is, in fact, an object) by using the same variable (`tail`, in this example) because variables [are just names or identifiers to actual objects](/2021/07/09/understanding-const.html). Multiple variables can 'point' or 'refer' to the same object and each of them can be used to modify the properties of the object. Hence, we can use any variable (such as `tail`), to change multiple nodes in a linked list.   

```js
function arrayToLinkedList(arr) {
  let head = new Node(arr[0]);
  let tail = head;

  //creating the linked list iteratively
  for (let i = 1; i < arr.length; i++) {
    tail.next = new Node(arr[i]);
    tail = tail.next;
  }
  return head;
}
```

## Searching through a linked list

Given a linked list, it is possible to traverse through the entire list to find out if a particular data exists within that list. In the following example, the function `search` accepts two arguments: the head node of a linked list(`list`), and the value of to be searched for(`x`):

```js
function search(list, x) {
  while (list != undefined) {
    if (list.data == x) {
      return list;
    }
    list = list.next;
  }
  return;
}
//calling search by passing the head node of the linked list created in the first example
console.log(search(head, 9)); //Node { data: 9, next: Node { data: 10, next: null } }
console.log(search(head, 11)); // undefined
```

## Inserting a new node

As discussed in the [previous post](/2021/07/27/data-structures-arrays-and-linked-lists.html), we can simply add a new node at the beginning of the linked list:

```js
function insertAnyWhere(list, data){
let newNode = new Node(data);
newNode.next = list;
return newNode;
}
```

In the above example, the `newNode` points to the new node carrying `data`. The `.next` property of `newNode` points to `list` which is the original head node of the linked list. Finally, `insertAnywhere` returns `newNode` to signal to the caller that this is the new head of the linked list.

It is also possible to insert a new node at a specific location within a linked list. In the following example, the function `insertAfter` accepts three parameters: `list`, `data` and `newData`. It adds a new node carrying `newData` in `list`, after the node that carries `data`:

```js
function insertAfter(list, data, newData) {
  let newNode = new Node(newData);
  //calling the search function, to find out the node carrying 'data'
  let targetNode = search(list, data);
  // returning an appropriate message if 'data' does not exist within 'list'
  if (targetNode == undefined) {
    return data + "does not exist";
  }
  //checking if the head node is the targetNode
  //if yes, we will need to change the head node
  if (targetNode == list) {
    newNode.next = list;
    return newNode;
  }
  //if the targetNode is any other node in the list
  //newNode should point to the node after targetNode
  //and targetNode should point to newNode
  newNode.next = targetNode.next;
  targetNode.next = newNode;
  return list;
}
```

Similarly, it is possible to insert an element before a specific node in a linked list. For this, we cannot rely on the `search` function, as we need to know the location of node prior to the target node. Thus, we can create a new function `searchPreviousNode` that returns the node immediately prior to a specific node in a linked list:

```js
function searchPreviousNode(list, x) {
  //checking if list exists
  // and if list points to the last node of the list
  if (list == undefined || list.next == null) {
    return;
  }
  //loop to find out the node which points to the node that carries x
  while (list.next != null) {
    if (list.next.data == x) {
      return list;
    }
    list = list.next;
  }
  return;
}
```

Now, we can use `searchPreviousNode` to insert a new node immediately prior to a given node:

```js
function insertBefore(list, data, newData) {
let newNode = new Node(newData);
//checking if the head node contains 'data'
  if (list.data == data) {
    newNode.next = list;
    return newNode;
  }
//calling searchPreviousNode(), to find the (n-1)th node  
let previous = searchPreviousNode(list, data);
//checking to see if 'previous' is undefined
  if (previous == undefined) {
    return data + "does not exist";
  }
//if 'previous' is a node other than the head node, 
//the 'newNode' should point to the node after 'previous' node'
//and the 'previous' should point to the 'newNode'  
newNode.next = previous.next;
  previous.next = newNode;
  return list;
}
```

## Deleting a node

To delete a node carrying a specific value, we can rely on `search()` to find that node, and `searchPrevious()` to find the node immediately prior to that node. We can then link the *(n-1)*th node to the *(n+1)*th node.  

```js
function deletion(list, x){
    if (list.data == x){
        return list.next;
    }
    let targetNode = search(list, x);
    if (targetNode == undefined){
        return x + "does not exist";
    }
    let previous = searchPreviousNode(list, x);
    previous.next = targetNode.next;
    return list;
}
```