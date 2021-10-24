---
layout: post
title: Valid Parenthesis
tags: programming
image: /assets/images/valid_parenthesis.png
---

In this post, I discuss how I solved a problem on LeetCode on the correct order of Parentheses.

## The Problem

Quoting the [problem-statement](https://leetcode.com/problems/valid-parentheses/):

> Given a string s containing just the characters `(`, `)`, `{`, `}`, `[` and `]`, determine if the input string is valid. An input string is valid if:

> Open brackets must be closed by the same type of brackets.

> Open brackets must be closed in the correct order.

For example, if the input string is `[{}][]`, it should return `true`. Conversely, if the input string is `[)[{)]`, it should return `false`.

## My attempt

I was stuck with this problem for one whole day. I broke down the problem cases (i.e., cases when the return should be `false`), in the following three buckets:

1. When the number of characters in the string is odd (because this would imply that there is at least one incomplete pair of braces).
2. When the number of open brackets do not equal to the number of closed brackets (i.e., if the input is `[[[]` it should return `false`).
3. Each open bracket should be closed by its corresponding closing bracket. But, if there is any open bracket placed within a pair of brackets, then that bracket should be properly closed _before_ closing the first pair.

While the first two problem cases were easy to eliminate, it was proving to be rather difficult to write the logic for eliminating incorrect bracket-closings, i.e., strings such as `[(])`.This example had an even number of braces, and the number of opening brackets equals the number of closing brackets. Heck, even the nature of closing brackets match that of the opening brackets! Its _just the order_ which is misplaced.

Eventually, I was about to give up and look into the discussion section. However, incidentally, I was reading a book on data structures, that briefly introduced the concept of stacks and queues. While reading the part on stacks, it struck me that **brackets are always arranged in an last-in-first-out order**. As I had not yet done a deep-dive on stacks and learnt the intricacies of this data structure, I had initially thought that I should delay this problem till I knew enough about this concept.

But I couldn't help myself. Having invested an entire day, and now having visualised this potential solution, I just _had to see_ if I could solve the problem using the last-in-first-out breakthrough. So, this time, for point no. 3 above, I decided to do the following:

- For every open bracket I come across in the string (going left-to-right), I would push its complimentary closing bracket in an empty array.
- For every closing bracket, I would first see if it matches the element that was last pushed into the aforesaid array. If yes, I would delete the last pushed element from that array and continue with the rest of the string. If not, it would imply that the closing bracket is of a kind not matching the last open bracket. This would mean the supplied string was not correctly arranged.

Here's what I wrote:

```js
var isValid = function (s) {
  let arr = [...s];
  let open = 0;
  let close = 0;
  let correctClose = [];
  // checking if the supplied string has an even number of characters
  if (arr.length % 2 != 0) {
    return false;
  }
  for (let i = 0; i < arr.length; i++) {
    // for every open bracket, pushing the corresponding closed bracket into the array named 'correctClose'
    switch (arr[i]) {
      case "{":
        correctClose.push("}");
        open++;
        break;
      case "(":
        correctClose.push(")");
        open++;
        break;
      case "[":
        correctClose.push("]");
        open++;
        break;
      // for every closed bracket, checking if it matches the last pushed element into 'correctClose'
      case "]":
      case ")":
      case "}":
        close++;
        if (arr[i] == correctClose[correctClose.length - 1]) {
          correctClose.pop(correctClose.length - 1);
        } else {
          return false;
        }
    }
  }
  // Finally, checking if the number of open brackets and closed brackets are equal to each other
  return open == close;
};
```

This solution was accepted by LeetCode. I do not know if this is the most elegant way to solve this problem or if this is the correct implementation of the 'last-in-first-out' concept. I will read more to find out. But for now, I am very proud with myself for being able to solve this problem without relying on the discussion section of LeetCode.
