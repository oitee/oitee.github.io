---
layout: post
title: "Recurse Center: Week Three"
tags: personal
---

This post is about my third week at the [Recurse Center](https://www.recurse.com).



## Functional Programming and Clojure

During this week, I continued reading Clojure for the Brave and True, and completed the next two chapters ([Clojure Alchemy: Reading, Evaluation, and Macros](https://www.braveclojure.com/read-and-eval/), and [Writing Macros](https://www.braveclojure.com/writing-macros/)). Reading about how Clojure syntax is read and evaluated was very interesting! It reminded me of my first project, [Crisp: a simple Lisp interpretor](https://github.com/oitee/crisp).

During the previous week, while reading Brave Clojure's 4th and 5th chapters, I was briefly introduced to some of the key tenets of functional programming. I found these concepts very fascinating. This week, I spent considerable amount of time reading about them. While I want to write a brief post on my findings in a separate post, here are some of the points I have taken on my notebook:
- Functional programming is characterised by its use of pure functions. 
- A pure function is one that has no side effects. 
- The values evaluated by a pure function are solely dependent on its arguments. The return value for a given set of arguments, will always be the same, irrespective of the number of times the function is called or when it is being invoked. This makes it easier to [do parallel programming, testing, and using higher-order functions](https://purelyfunctional.tv/issues/purelyfunctional-tv-newsletter-340-fewer-side-effects-is-better-than-more/)
- Because of their deterministic nature, a pure function can be replaced with its return value, in an expression (referential transparency)
- The use of impure functions is not prohibitted. But as a functional programmer, it is important to be [wary of using side effects](https://twitter.com/ericnormand/status/1384860792013705218?s=20).
- As changing the value of a data-structure can cause side effects, functional programming emphasizes the use of immutable data-structures. 
- As we need to need to [change the state of local variables](https://dzone.com/articles/functional-programming-recursion) to run a loop, recursion should be preferred over loops


## RemindMe: My First Clojure App

Now that I was getting slightly familiar with the world of Clojure, I thought it would be the perfect time to embark on a journey to build my very first app using Clojure. I decided to build an online version of flash cards, called **RemindMe**. 

Given that it would be my first project in Clojure, I thought it would be useful to first build the app using JavaScript, the language I am most familiar with. 

So, I spent a day writing the server-side logic of **RemindMe**, using Node.js. I then proceeded to build the front-end part of the project. To implement the front-end part, I used the Bootstrap framework. This [Boostrap tutorial](https://www.youtube.com/watch?v=4sosXZsdy-s&) was very helpful, to get started. (_Side note: Long after finishing the front-end part, I realized that the hamburger menu on the navigation bar is not working on a mobile display. I'll need to fix this; would love to pair with anyone who is good with Bootstrap/CSS!_). 

The Node.js version of RemindMe is hosted on [this GitHub repository](https://github.com/oitee/remind-me). 

After completing the app in Node.js, I proceeded to re-implement the backend of RemindMe, using Clojure. I used the Ring framework and it's jetty-adaptor to build my web service. I chose Compojure as a routing library, for better handling of requests. 

**RemindMe is deployed here:** [https://remind.otee.dev](https://remind.otee.dev).

Here's a detailed account of how I built RemindMe, using Clojure, Ring and Compojure: [https://otee.dev/2022/01/25/clojure-backend-using-ring-jetty-compojure.html](/2022/01/25/clojure-backend-using-ring-jetty-compojure.html)


## Others

- I solved these LeetCode problems during this week: [Calculate Money in LeetCode Bank](https://leetcode.com/problems/calculate-money-in-leetcode-bank/) ([my solution](https://github.com/oitee/whiteboard/blob/main/leetCode/75_calculate_money_in_Leetcode_bank.js)), [Determine Color of a Chessboard Square](https://leetcode.com/problems/determine-color-of-a-chessboard-square/) ([my solution](https://github.com/oitee/whiteboard/blob/main/leetCode/76_determine_color_of_a_chessboard_square.js)) [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)([my solution](https://github.com/oitee/whiteboard/blob/main/leetCode/78_longest_increasing_subsequence.js)), [Maximal Square](https://leetcode.com/problems/maximal-square/)([my solution](https://github.com/oitee/whiteboard/blob/main/leetCode/77_maximal_square.js))
- My article on [Lazy Sequences in Clojure](https://otee.dev/2022/01/17/lazy-clojure.html) got featured on **[Clojure Deref](https://clojure.org/news/2022/01/21/deref)**!
- This article was also a part of Recurse Center's [Joy of Computing Blog](https://joy.recurse.com/posts/1446-who-moved-my-cheese-laziness-in-clojure)! 