---
layout: post
title: "Diwali Hack Week"
tags: project
image: /assets/images/diwali_hack_week.jpg
---

During this Diwali week, I am starting a new [project](https://github.com/oitee/twirl) and I intend to spend around 8 hours a day, hoping to finish this project within the week. 

This is partly to celebrate the joy of Diwali. But also to see if a full-time project can be completed in a week, as I would be expected to, in a software engineering job. Wish me luck!

## The Project Scope

1. Similar to [https://tinyurl.com/app](https://tinyurl.com/app)
2. Provide a way for users to sign up (verify the signup with [recaptcha](https://developers.google.com/recaptcha/docs/display))
3. Only logged in users can create short links
4. Users can have two roles
    1. Admin
    2. Non-admin
5. Admin users should be able to see the top 50 most used short links
6. Non-admin users should be able to see their top 50 most used short links
7. Non-admin users should also be able to disable some links

## Daily logs

### November 1, 2021:

1. Setting up the project structure.
2. Installing basic dependencies like, express, jest, nodemon, node-fetch etc
3. Understanding HTML forms
4. Understanding how cookies can be used to create user sessions
5. How to parse and write cookie headers in Express
6. Basic user sign-in and sign-up flow without any database. 
7. HTTP status codes, redirection.
8. Templating HTML using [mustache](https://www.npmjs.com/package/mustache)
9. Testing full integration of all the routes, re-directions, session management, by making HTTP requests using node-fetch
10. Testing functionality of sign-up, login flow
11. Adding [GitHub workflow](https://github.com/oitee/twirl/actions) to run integration tests on GitHub

### November 2, 2021:

1. Modularise the code base, by separating controllers, models and routers into their own directories
2. Using Express router for separating out routing from server start-up
3. Understanding how best match incoming routes using [path-to-regexp](https://www.npmjs.com/package/path-to-regexp) package
4. Adding an authentication middle-ware, that does all the re-directions for routes which expect session existence, so that routes don't have to manually check for session presence
5. Using signed cookies to ensure the cookie payload has not been tampered with (understanding digital signatures and how to have signed cookies in Express)
6. Using cookie encryption to obfuscate cookie contents so that the cookie cannot be readable by any system other than the server (understanding how to add encryption on the cookie payload in Express)
7. Setting Postgres using Docker
8. Creating the database schema for users and roles
9. Modifying the sign-up/login flow to use the database as a persistent store (as opposed to using epehemeral JS object in memory for storing users in the system)
10. Add installation of Postgres as a service on [GitHub workflow](https://github.com/oitee/twirl/actions) so that integration tests can continue to be run on GitHub.

### November 3, 2021:

1. Understand [how to store passwords in database](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html): plain hashing, cryptographic hashing, cryptographic salted hashing, peppering, time-insensitive equality check, multiple iterations on hash generation.
2. Using [pbkdf2](https://nodejs.org/api/crypto.html#cryptopbkdf2syncpassword-salt-iterations-keylen-digest) from NodeJs' Crypto module, to generate salted hash value before storing and for matching passwords
3. Understand the difference between encryption(AES), hashing(SHA-256) and encoding(base64)
4. Understand difference between hash functions and cryptographic hash functions and why collisions in cryptographic hash functions are detrimental. Eg, [SHA-1 collision found in 2017](https://security.googleblog.com/2017/02/announcing-first-sha1-collision.html) makes it a bad candidate for password hashing.
5. Add unit test for password matching (extract salt out, generate hash, compare hash)
6. Update authentication middle-ware to pass in full user object for authenticated routes, so that the route controllers can access user information without having to ask the database all the time
7. Understand implications of [SQL injection](https://owasp.org/www-community/attacks/SQL_Injection) and why we need to use `$1` and `$2` in SQL query parameters.
8. Add bug fix in cookie creation in sign-up flow, where instead of storing the user_id the username was being stored.
9. Create database schema for storing short-to-long link mapping and for usage counts
10. Ensure that the shortening algortihm does not have any collisions, by using incrementing counter in Postgres
11. Use NodeJs' [Crypto createHash](https://nodejs.org/api/crypto.html#cryptocreatehashalgorithm-options) function to generate base64 digest + unique counter to achive unique short link
12. Add link controller to ensure conversion between short and long links is possible per user (short links are maintained on a per-user basis along with usage analytics)
13. [Integration tests](https://github.com/oitee/twirl/actions) for multi-user multi-link shortening and reversing of links, plus usage analytics.
 

_This is just a daily log I am maintaining as I embark on this project. I will publish a detailed post on this project, including a README on my [GitHub repository](https://github.com/oitee/twirl), once it is completed. Happy Diwali!_