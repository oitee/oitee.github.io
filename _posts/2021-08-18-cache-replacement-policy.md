---
layout: post
title: Cache Replacement
tags: conceptual
image: /assets/images/cache_eviction_fibonacci.jpg
---

It is said that "There are [two hard things in computer science](https://twitter.com/codinghorror/status/506010907021828096): cache invalidation, naming things, and off-by-one errors." This post will be on the first one.

In this post, I attempt to discuss what the term 'cache' actually means and delve into some of the common cache replacement policies. Also, inspired by this [Clojure repository](https://github.com/clojure/core.cache) on Github, I attempt to implement some of the common cache replacement policies. 

## What is cache?

Cache refers to the storage of data that is likely to be requested in the future. When an application needs to access some data (which is expected to be cached), it will first check if that data exists in its cache. If yes, it will simply retrieve that data and access it. Otherwise, it will generate that data, which will be stored in the cache for a future request.  

When the data requested from a cache is successfully found, it is called a *cache hit*. On the other hand, a *cache miss* occurs when the requested data is not stored in the cache. In the event of a *cache miss*, the data will need to be computed or located from a slower data source, which will take much longer time to generate, than a *cache hit*. For example, a browser may download web-pages that are most-frequently used by the user. When requests for those web-pages are generated by the user, the browser can simply retrieve the downloaded pages which will be much faster than loading other web-pages which are not cached.

Thus, it is clear that, ideally, the number of *cache hits* should, on an average, be greater than the number of *cache misses.* This makes the **hit ratio of cache** very crucial. 

## Problem with unrestricted memoization

In a world with infinite resources to store infinite amount of data, we can achieve a near-perfect hit ratio, as we can keep populating our cache with the data generated from previous requests, thereby restricting *cache misses* to only those requests that have never been created before. In my [previous post on memoization](https://otee.dev/2021/08/05/memoization.html), **I had implemented exactly this**. 

Recall that memoization refers to the storage of return values of previous function-calls. I had implemented memoization such that whenever a memoized function was called with a given set of arguments, that function would first check if a return value for that set of arguments already existed in a dictionary called `memo`. If the memo did not contain the return value, it would compute the value by calling the actual function, and **then *indiscriminately* store that value against the set of arguments**, before returning the value back to the caller. In this way, we simply stored *each and every input and return values*  of a memoized function. 

This approach proved to be particularly effective with functions which recursively called itself more than once (such as a function to generate the nth Fibonacci number). But, inadvertently, the memo dictionary (storing the return values), kept growing with each new set of inputs. For example, when we call the memoized Fibonacci function with `50` as an argument, the memo would store the return values of every integer from `0` till `50`. This is not ideal, because when dealing with larger data-sets and more complicated algorithms, **we will not have the luxury to store each and every return value**. 

## Cache Replacement Algorithms

There will always be a trade-off between the size of the cache and the speed of returning a data. Of course, a bigger cache can contain greater amount of data, thereby reducing the chances of re-computation. On the other hand, there will always be practical limitations of the total amount of memory we can allocate to the cache of a particular program. 

Thus, given a specific size of a cache, when we reach that maximum capacity, we will need to *replace* or *evict* an existing item in the cache in order to add a new item to it. To increase  efficiency of a program, we need to find the most optimum way to add and replace items in a cache, **such that the item that will be least likely to be requested again is always replaced** by an item that has a greater likelihood of a future request. This strategy is known as **Bélády's algorithm**. Clearly, to achieve this, we need to be aware of the nature of future requests. 

In the case of recursive calls, we can easily estimate the nature of future (recursive) calls. Take the example of the function returning the `n`th Fibonacci number [f(n) = f(n - 2) + f(n - 1)]. To find the `n`th Fibonacci number, we need to call the same function for `(n - 2)` and for `(n - 1)`. This will continue till `n == 0 || n == 1`, in which case it will return `n`. 

In this case, we can reach the optimum strategy proposed by Bélády, if we create a cache which can store just two items, such that **each new return value is added to the cache by evicting the older of the remaining values in the cache**. To illustrate how this works, it is important to note in the case of recursive calls, the final value is reached by going up the chain of recursive calls starting with the base-case. Thus, the cache will initially store the values of the `n = 0` and `n = 1`, which are the two base cases of the present function. When `n = 2`, the cache contains the return values of f(1) and f(2). To find the 2nd Fibonacci number, we need to know the values of f(2 - 2) and f(2 - 1). Both these values are already stored in the cache, so we will not need to compute either of these values. But, now that we have a new return value, i.e., f(2), it will be added to the cache, and the value of f(0) will be evicted. Now, when `n = 3`, the cache contains the values of f(1) and f(2), relying on which, the return value of f(3) can be generated. Again, the return value of f(3) will be replacing the return value of f(1). This will continue till we reach n. In the following table, this process has been illustrated for a case when n = 7.

<img src="/assets/images/cache_eviction_fibonacci.jpg" alt="Finding the 7th Fibonacci number, using a cache-size of 2 element" width="100%"/>

However, unlike recursive calls, in most cases, it is impossible to predict the nature of future calls or requests for data. Thus, it is not practically possible to write a cache replacement algorithm that can achieve the same level of efficiency as proposed by  Bélády. Instead, **we write approximation algorithms based on certain parameters** (such as order of insertion, recency and frequency of access etc), which can increase the *cache hit* probabilities for a given program. 

Following are some of the common and simpler types of cache replacement policies:

- **First-in-first-out (FIFO) policy**: The earliest inserted item in the cache will be evicted when a new item needs to be inserted.
- **Last-in-first-out (LIFO) policy**: The last item inserted in the cache will be evicted first.
- **Least-recently used (LRU) policy**: The item which is least recently used will be evicted first.  This is one of the most simple and common cache replacement policies. It assumes that the more recently an item is accessed or used, the more likely it is to be used or accessed again (e.g., switching between tabs of a browser)
- **Most-recently used (MRU) policy**: The item which is most recently used will be evicted first. This policy is useful, when the chances of repeating the same request in the near future is unlikely (like scrolling through a social media feed or flipping through a photo album).
- **Least-frequently-used(LFU) policy:** The cache algorithm maintains a counter on the number of times an item in the cache is accessed. It will evict the least frequently accessed item in order to add a new item.
- **Time-to-live (TTL) policy:** If an item remains in the cache beyond a given period of time without being accessed, the cache algorithm would discard it, to make room for a new item. Morever, if the size of the cache needs to be bounded, another cache replacement policy can be applied on top of TTL. For example, if there is no item in the cache which has exceeded the given time-period, the least frequently used item (LFU) may be evicted to insert a new item.
- **Random replacement (RR) policy**: The cache algorithm randomly selects an item to evict.

### Implementing LRU Policy

To implement LRU policy, we can write a function that accepts any function (which will carry out the actual task) and returns a memoized version of the same function. This returned function will first check if, for a given set of parameters, a return value exists in the respective cache object. If yes, it will access the value. Otherwise, it will compute the return value and store its value in the cache object. 

In addition to the cache object, the function should maintain another object which will keep a counter for each time a value is accessed or entered into the cache. This object will contain the same set of keys as the cache object. But the value of these keys will represent how recently they were inserted or accessed by the function.

```js
function lru(f) {
  let cache = {};
  let accessed = {};
  let counterCount = 0;
  return (...args) => {
    if (cache.hasOwnProperty(args)) {
      accessed[args] = ++counterCount;
      return cache[args];
    }
    let result = f(...args);
    if (Object.keys(cache).length >= 2) {
      let least = counterCount;
      let candidateKey;
      for (let key in accessed) {
        if (accessed[key] < least) {
          least = accessed[key];
          candidateKey = key;
        }
      }
      delete cache[candidateKey];
      delete accessed[candidateKey];
    }
    cache[args] = result;
    accessed[args] = ++counterCount;
    return result;
  };
}
```

 In the above example, the function `lru` creates two objects, `cache` (for storing the return values of each set of arguments) and `accessed` (for storing the counter values of each accessed item in the cache). If a return value exists within `cache`, that value is fetched and returned to the caller. But, before returning the value, the corresponding value for that set of arguments in `accessed` is updated. If a return value is not present, the actual function is called. The return value is stored in the `cache`. Before storing the new item, it is checked if the size of `cache` has already exceeded its maximum capacity (here, it has been kept as `2`). If so, the set of arguments with the lowest counter value in the `accessed` object is fetched. This set of arguments is deleted from both `cache` and `accessed` before the new return value is stored in the `cache`.


## Separating out the Cache Eviction

In the above implementation, it is clear that we will need to write separate functions for each type of cache eviction policy. More significantly, the memoization, computation and implementation of the cache algorithm are all taking place together in the same function, which is why we need to write an altogether new function if we want to implement a new policy for replacing cache. This is a limitation.

It would be more idomatic if we separate out the cache eviction algorithm from the rest of the function. This will give liberty to the user to choose a specific cache algorithm while memoizing a given function. To do this, we can write separate classes for implementing different cache algorithms. Objects created by this class will contain three methods (which carry out operations as per the rules of the corresponding cache eviction policies):

- **has:** This method will be used to look-up if the cache object contains the return values of a specific set of arguments
- **hit:** This method will be invoked when we know that the cache object contains a specific return value. This method will return the value. It can also carry out necessary operations (if any), required to be carried out while accessing an item in the cache. For example, in an LRU cache algorithm, the `hit` method will update the counter value of the set of arguments, before returning its values.
- **miss:** This method will be invoked when we know that the cache object does not contain the specific input. It will accept two parameters: i) the set of arguments, and ii) its corresponding return value. This method will be responsible for updating the cache, as per the corresponding cache eviction policy.

Thus, the LRU policy can be implemented by creating a new class called `LruCache` as shown below:

 

```js
//class for all creating cache objects
//following LRU policy
class LRUCache {
  constructor(n) {
    this.size = n;
    this.table = {};
    this.accessed = {};
    this.counterCount = 0;
  }
  has(key) {
    return this.table.hasOwnProperty(key);
  }
  hit(key) {
    this.accessed[key] = ++this.counterCount;
    return this.table[key];
  }
  miss(key, value) {
    if (Object.keys(this.table).length >= this.size) {
      let least = this.counterCount;
      let candidateKey;
      for (let k in this.accessed) {
        if (this.accessed[k] < least) {
          least = this.accessed[k];
          candidateKey = k;
        }
      }
      delete this.table[candidateKey];
      delete this.accessed[candidateKey];
    }
    this.table[key] = value;
    this.accessed[key] = ++this.counterCount;
  }
}
//Creating a new cache object
//by invoking LRUCache
let lruCacheForFibo = new LRUCache(3);

//Memoizing function:
// It will invoke has(), hit() and miss() of the cache object 
function cacheThisFunction(cache, f) {
  return (...args) => {
    if (cache.has(args)) {
      return cache.hit(args);
    }
    let result = f(args);
    cache.miss(args, result);
    return result;
  };
}

//Calling the above function to memoise the Fibonacci function
let memoizedFiboLru = cacheThisFunction(lruCacheForFibo, (n) => {
  if (n == 0 || n == 1) {
    return n;
  }
  return memoizedFiboLru(n - 2) + memoizedFiboLru(n - 1);
});
```

The above example contains three parts:

- A class that creates cache objects containing the respective `has` `hit` and `miss` methods
- A function that accepts another function and a cache object. This function returns a memoized version of the function that was passed to it. Importantly, **the definition or logic of this memoizing function will not need to change with different caching algoritms** being implemented by the cache object. 
- The memoized function first checks if the arguments passed to it exists in the relevant cache object. This is done by invoking `has`. If the `has` method returns `true`, the `hit` method is invoked which returns the value stored in the cache object. If the `has` method returns `false`, the underlying function (which has been memoized) is called. Then, the return value so computed is passed to the `miss` method along with the set of arguments, to store the same inside the cache object.

Clearly, the memoizing function can now be used on *any function with any number of arguments* and on *any cache implementing any cache eviction policy*. Essentially, we have abstracted out the implementation of the cache eviction policy from the rest of the memoization function.

The above technique can be used in a similar manner to implement other cache eviction policies as well. In the following example, the random replacement policy is implemented:

```js
class RRCache {
  constructor(n) {
    this.size = n;
    this.table = {};
  }
  has(key) {
    return this.table.hasOwnProperty(key);
  }
  hit(key) {
    return this.table[key];
  }
  miss(key, value) {
    if (Object.keys(this.table).length >= this.size) {
      let allKeys = Object.keys(this.table);
      let randomIndex = Math.floor(Math.random() * (allKeys.length - 1));
      let randomKey = allKeys[randomIndex];
      delete this.table[randomKey];
    }
  }
}

let fiboRandomCache = new RRCache(5);
function cacheThisFunction(cache, f) {
  return (...args) => {
    if (cache.has(args)) {
      return cache.hit(args);
    }
    let result = f(...args);
    cache.miss(args, value);
    return value;
  };
}

let memoizedFiboRandom = cacheThisFunction(fiboRandomCache, (n) => {
  if (n == 1 || n == 2) {
    return n;
  }
  return memoizedFiboRandom(n - 2) + memoizedFiboRandom(n - 1);
});
```

To see the implementation of the cache policies mentioned on this post (i.e., LIFO, FIFO, LRU, MRU, LFU, RR, TTL), please check out my [GitHub repository on cache replacement](https://github.com/oitee/cacheReplacement).