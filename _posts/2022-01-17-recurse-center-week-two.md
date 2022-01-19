---
layout: post
title: "Recurse Center: Week Two"
tags: personal
---

This post is about my second week at the [Recurse Center](https://www.recurse.com).

## Clojure

One of my primary goals at the Recurse Center, is to learn how to write programs in Clojure. To this end, I had spent the first week familiarising myself with the basic syntax of Clojure. This week, I could delve deeper and wrote some not-so-simple programs.

Before the start of the retreat, I had asked for suggestions for getting started with Clojure. [Kapil Reddy](https://twitter.com/KapilReddy) was kind enough to point me to an on-boarding course, [clojure-by-example](https://github.com/inclojure-org/clojure-by-example), that he, [Aditya Athalye](https://github.com/adityaathalye) and a few others had designed.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Start from <a href="https://t.co/YwAhlj1HlN">https://t.co/YwAhlj1HlN</a> This is a good starting point. You don&#39;t have to figure out editor setup yet. Shoutout to <a href="https://twitter.com/jysandilya?ref_src=twsrc%5Etfw">@jysandilya</a>, <a href="https://t.co/LIVollpHAT">https://t.co/LIVollpHAT</a> and <a href="https://twitter.com/in_clojure?ref_src=twsrc%5Etfw">@in_clojure</a> for course design <a href="https://twitter.com/hashtag/Clojure?src=hash&amp;ref_src=twsrc%5Etfw">#Clojure</a></p>&mdash; Kapil (@KapilReddy) <a href="https://twitter.com/KapilReddy/status/1230720191417352193?ref_src=twsrc%5Etfw">February 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The week began with a long and super helpful Clojure pairing session with [Aditya](https://github.com/adityaathalye), who gave me a detailed walk-through of [clojure-by-example](https://github.com/inclojure-org/clojure-by-example). Until then, I had only read the first three chapters of [Clojure for the Brave and True](https://www.braveclojure.com/clojure-for-the-brave-and-true/). While I had a theoretical overview of some of the basic semantics of Clojure, this pairing session gave a more hands-on take on how to read, evaluate and write expressions in Clojure. This was the perfect start to my week!

During the rest of the week, I concentrated on completing the next three chapters of Clojure for the Brave and True ([Core Functions in Depth](https://www.braveclojure.com/core-functions-in-depth/), [Functional Programming](https://www.braveclojure.com/functional-programming/), and [Organizing Your Project: A Librarian’s Tale](https://www.braveclojure.com/organization/)). Learning about sequence abstraction was truly fascinating

> It doesn’t matter how a particular data structure is implemented: when it comes to using seq functions on a data structure, all Clojure asks is “can I `first`,`rest`, and `cons`it?” If the answer is yes, you can use the seq library with that data structure [source](https://www.braveclojure.com/core-functions-in-depth/)

While reading [Clojure for the Brave and True](https://www.braveclojure.com/clojure-for-the-brave-and-true/), I learnt about some of the key concepts of functional programming. Specifically, reading about pure functions and their use-case was a lot of fun! I would like to spend some time reading more on concepts (like 'referential transparency', and 'functional composition') that are fundamental to functional programming. Also, reading about higher-order functions was a lot of fun as it reminded me of my early adventures with [map, reduce and filter](/2021/07/11/higher-order-functions.html).

I had struggled to visualise the importance of lazy sequences while writing programs. So, I ended up spending considerable amount of time, playing around with lazy sequences. In the end, I published a [blog post (with a cheeky title) summarising my findings](/2022/01/17/lazy-clojure.html).

As reading several pages of [Clojure for the Brave and True](https://www.braveclojure.com/clojure-for-the-brave-and-true/) can get a bit monotonous, I would often take a detour to implement some of the examples and concepts explained in the book. It was particularly satisfying to take up [Daniel's](https://twitter.com/nonrecursive) challenge on [Chapter 4](https://www.braveclojure.com/core-functions-in-depth/):

> If you want an exercise that will really blow your hair back, try implementing map using reduce and then do the same for filter and some after you read about them later in this chapter.

Here's my implementation of `map-using-reduce`:

```clojure
(defn map-using-reduce
  [map-fn xs]
  (seq (reduce (fn f
                 [aggregator new-val]
                 (conj (vec aggregator) (map-fn new-val)))
               (empty xs)
               xs)))
```

Here's `filter-using-reduce`:

```clojure
(defn filter-using-reduce
  [filter-fn xs]
  (seq (reduce (fn
                 [aggregator new-val]
                 (if (filter-fn new-val)
                   (conj (vec aggregator) new-val)
                   aggregator))
               (empty xs)
               xs)))
```
And, `some-using-reduce`:

```clojure
(defn some-using-reduce
  [some-fn xs]
   (reduce (fn [aggregator new-val]
                  (if-not aggregator
                    (if (some-fn new-val)
                      (some-fn new-val)
                      aggregator)
                    aggregator))
                nil
                xs))
```


Here's my [GitHub repository](https://github.com/oitee/butterfly), hosting my white-board adventures with Clojure!

Also, as suggested by Aditya, I keep solving the 4Clojure problems mentioned at the end of each exercise of [clojure-by-example](https://github.com/inclojure-org/clojure-by-example). So far I have solved a handful of them; I intend to solve many more this week. Here's where I write down my solutions to 4Clojure problems: [https://github.com/oitee/clojure-by-example/blob/master/src/clojure_by_example/4-clojure-exercises.clj](https://github.com/oitee/clojure-by-example/blob/master/src/clojure_by_example/4-clojure-exercises.clj) 

## Using NGINX to Use a Custom Domain

While most of the week was spent on Clojure, I could sneak out some time to complete a long-standing task related to [Twirl](https://twirl.otee.dev). Twirl is a URL-shorening web app. As it is hosted on Heroku (free tier), the domain name of the app is pretty long(`oteetwirl.herokuapp.com`). This makes the short links generated by Twirl not so short after all. I used NGNIX on a Google Cloud VM I had spawned recently, to generate a custom short domain name for my application. Twirl is now accessible at: [https://twirl.otee.dev](https://twirl.otee.dev). This makes the resultant short links shorter! Here's a detailed [account of the steps I followed to complete this task](/2022/01/13/making-short-links-shorter.html)!

## Others

- I paired with [Kanyisa](https://kanyisa-ntombini.github.io/portfolio/), to continue working on building a REPL version of the [Mastermind game](https://en.wikipedia.org/wiki/Mastermind_(board_game)). We think we would need one more session to complete this.
- I attended a very insightful session hosted by Julia Evans on How DNS works and how it breaks.
