---
layout: post
title: "Recurse Center: Week One"
tags: personal
image: 
---

A short post on my first week at the Recurse Center.

### Virtual RC

Despite the occasional Zoom fatigue and my groggy eyes, the first week was a lot of fun. 

It began with an introductory session where one of the facilitators of RC gave a quick tour of how Virtual RC works. [Virtual RC](https://www.recurse.com/virtual-rc) is a social place, which gives a snapshot of where each Recurser is currently situated: their virtual desk (indicating they are not attending any meeting), or a meeting with one or more Recursers or facilitators in any of the many Zoom rooms. It gives a real-time view of where your fellow Recursers are currently at, and, if you find other Recursers attending a specific Zoom room, you can always check out the event taking place in that room and decide whether you want to enter it. It is aimed at "facilitating the kind of [serendipitous, synchronous, and ephemeral interactions](https://www.recurse.com/virtual-rc)" that would take place naturally if the retreat was not taking place virtually.

<img src="/assets/images/virtual_rc.png" width="100%"/>
_Source:_ [_Recurse Center_](https://www.recurse.com/virtual-rc)

### Welcome Aboard!

The Virtual RC tour was followed by a short introduction to the retreat by [Nick Bergson-Shilcock](https://twitter.com/nicholasbs), [David Albert](https://twitter.com/davidbalbert), [Sonali Sridhar](https://twitter.com/jollysonali) and the other facilitators. Notably, we were told that RC thrives on its diversity and that everyone is here because they want us to be a part of this retreat (one shouldn't feel like an imposter). 

The only unifying quality among Recursers is that everyone wants to get dramatically better at programming. This is at the core of RC's philosophy: it is a place for self-directed learning, where [intrinsic motivation](https://www.recurse.com/about) is valued over coercive external motivation. So, there is no imposed curriculum or rigourous structure to follow; one is free to pursue their interests in programming, at their own pace. However, this can sometimes falsely lead us to believe that we have endless time on our hands which can keep us from actually completing our goals (see [planning fallacy](https://en.wikipedia.org/wiki/Planning_fallacy)). To this end, everyone at RC is strongly encouraged to regularly check-in with the rest of the batch, about what they are upto and what they plan to do. Most Recursers either post short check-in messages on a dedicated stream on Zulip or attend a check-ins meeting everyday, to talk about their progress.

### Social Rules

Perhaps the most interesting part of Day 1 involved David along with Rachel Vincent Petacat and Mai Schwartz, explaining and enacting the four light-weight social rules that everyone should follow at RC: 
- **No feigning surprise**: We should not act surprised if someone admits to not knowing anything, irrespective of how basic or fundamental that is. Feigning surprise only serves to prevent people from being transparent about what they know and do not know.
- **No well-actually's**: We should not interject someone only to correct them on something that has no actual bearing on the topic at hand. It just interrupts the flow of conversation without adding any value. 
- **No back-seat driving**: If we want to help someone out with a problem, we should engage with the problem entirely, instead of making sporadic or intermittent contributions from a distance.
- **No subtle-isms**: We should take care that we are not being indirectly or subtley racist, sexist, ageist etc. The [RC Manual](https://www.recurse.com/manual) provides a useful example of this: "It's so easy my grandmother could do it". 

It is expected that we will sometimes end up to breaking some of these rules, and that is alright. What is important is to gracefully take feedback, when someone points it out, and carry on with our conversation.

### Meet-and-Greets

We had two meet-and-greet sessions during the first week. We were randomly assigned Zoom breakout rooms in groups of 2, 3 and 4, where we got a chance to say hi and interact with fellow Recursers. In addition to the meet-and-greets, there is a coffee-chat bot, which, should you opt-in, randomly matches you with your fellow Recursers, for a quick coffee-chat. 

### Introductions to Fellow Recursers

On the third day, every Recurser and facilitator gave a one minute introduction of themselves. We mentioned our name, where we come from, our preferred pronouns and what we want to do during our time at the Recurse Center. The **diversity of the cohort really showed during this session**. Recursers' interests varied from wanting to learn full-stack development, deep-dive in multi-player game development to wanting to build programs that can do magic tricks, and explore the fundamentals of computer systems!  

Also, Recursers who are part of the Winter 1 batch (November 1, 2021 – February 11, 2022), were encouraged to share a piece of advice about their time at the retreat. Here are some common advice shared by many Recursers: find the right balance between focusing on your own projects and engaging in different events; experiment with new things; do stuff you enjoy; do not hesitate to quit projects you do not like; flailing is fine but flailing alone isn't; pair as often as you can.

### Pairing Workshop

We had a pairing workshop on the second day. As a self-learner, I have not had much experience pairing with other programmers. So, I was very excited about this workshop. In a pairing session, one person typically chooses to be the driver and the other takes the role of the navigator. The driver is the one who actually writes the code, and focusses on completing the task at hand. It is important for the driver to talk through their steps. The navigator observes the code being typed out, shares their thoughts and nudges the driver to correct inadvertent errors (like typos). The navigator should take a big-picture view while the driver should focus on the nitty-gritty of the problem. 

A pairing session should not last longer than 30-45 minutes. The driver and the navigator should ideally switch their roles mid-way through the pairing session. A few things to keep in mind while pairing: do not talk over each other, do not get defensive if an error is pointed out, do not pair on something that is trival to both the participants. Here's a helpful [article on pairing](https://martinfowler.com/articles/on-pair-programming.html#HowToPair). 

How to find pairing partners? There is a pairing bot on Zulip, which randomly matches Recursers looking for pairing partners. Also, there is a dedicated pairing stream on Zulip, where we can post the task we want to accomplish and request people who may be interested to collaborate. 

The pairing workshop ended with a pairing exercise. We would be randomly matched with another Recurser right after the workshop and we should try to build a program implementing a simple two-player game, called [Mastermind](https://en.wikipedia.org/wiki/Mastermind_(board_game)). The rules are simple:
- Player 1 chooses any combination of four colours (including duplicates) out of a total of six color options
- Player 2 has ten chances to guess Player 1's combination
- For each guess, the Player 1 will reward Player 2 with a **white** marble for every correct colour guessed by Player 2, that is in the correct position 
- Also, Player 2 will reward Player 2 with a **black** marble, for every correct color that is guessed by Player 2, that is not in the correct position

I was paired with [Kanyisa Ntombini](https://kanyisa-ntombini.github.io/portfolio/). We paired using [https://replit.com/](https://replit.com/). Although we could not complete the task in the 30 minutes allotted for the exercise (the goal was never to complete the task, but to get a sense of how pairing works), it was a lot of fun. We aim to complete the task in three sittings!


## Daily Leetcode

There's a 'Daily Leetcode' session that takes place at 11.30pm IST everyday. I attended two of these sessions. Everyone solves three problems of varying difficulty for an hour. Everyone then joins a Zoom room to discuss their solutions. If anyone approached a problem differently or if anyone was stuck at a particular point, they are welcome to share as well. I found these sessions extremely fun and I plan to attend them more regularly from hereon. 

On one of the sessions, I paired with [Erich Keil](https://twitter.com/erich_keil) to solve the [Binary Addition](https://leetcode.com/problems/add-binary/) problem. We tried the easiest method: using `parseInt` to convert the inputs to integers, add them and then convert the result back to binary (using the `toString` method). However this solution failed to pass some of the tests on LeetCode as the range of the length of the inputs ranged between 1 and 10^4, which meant that the results could exceed the upperbound of a number in JavaScript (i.e., 2^32 - 1). So, we had to write the long-winding (naive) solution that adds each digit, starting with the units place and keeps a track of the carry-over values (See the final solution [here](https://github.com/oitee/whiteboard/blob/main/leetCode/70_binary_addition.js)).

## Getting Started with Clojure

Even before I had started the retreat, I was fore-warned by [Avinash Sajjanshetty](https://twitter.com/iavins) (who was assigned to help me with my onboarding), that I should not expect much work to be accomplished on the first week owing to the number of introductory events and activities that are scheduled then. I was still hopeful that I would be able to sneak in some time to do my own stuff, in between the events and before the day started. Unfortunately, the cocktail effect of being up till the wee hours of the morning (Recurse Center's core hours are from 11am to 5pm EST, which translates to 9.30 pm to 3.30 am IST) and  attending back-to-back Zoom meetings, meant that I had very little bandwidth to put my head down and do the things I had planned for myself.

Nevertheless, I could get started with Clojure, during my first week. I plan to learn and get comfortable with writing programs in Clojure during the first half of my reatreat. During the first week, I set up [Calva](https://calva.io/) which provides an "integrated REPL powered environment" for writing programs in Clojure in Visual Studio Code. I also familiarised myself with [Calva Paredit](https://calva.io/paredit/), which provides a mechanism to navigate, edit and excute programs in Clojure, in a manner that makes use of the LISP structure of Clojure. I also completed the first three chapters of Daniel Higginbotham's [Clojure for the Brave and True](https://www.braveclojure.com/clojure-for-the-brave-and-true/).







