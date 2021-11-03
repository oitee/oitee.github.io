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
11. Adding GitHub workflow to run integration tests on GitHub

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
10. Add installation of Postgres as a service on GitHub workflow so that integration tests can continue to be run on GitHub.

### 

_This is just a daily log I am maintaining as I embark on this project. I will publish a detailed post on this project, including a README on my [GitHub repository](https://github.com/oitee/twirl), once it is completed. Happy Diwali!_