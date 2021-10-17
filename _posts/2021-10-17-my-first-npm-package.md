---
layout: post
title: "My First npm Package"
tags: programming
---

In this post, I describe how to publish an npm package, based on my experience of publishing my first npm package, [`@otee/toolbox`](https://www.npmjs.com/package/@otee/toolbox). 

## Create an Account on npm

An npm user-account (`otee`) needs to be created. To do this, one needs to [sign up](https://www.npmjs.com/signup) on npm using an email address, a unique username and a password.

## Create a Git Repository

A repository may be created, using git. This can be done using [GitHub's web UI](https://github.com/new). While creating a git repository, one can (optionally) add a ReadMe file, create a `.gitignore` folder and choose a licence. Once the repository is created, it needs to be cloned to the local system. This can be done by using the `git clone` keywords in the requisite root directory of the local system, followed by the remote URL of the git repository (generated from GitHub's web UI).

## Scoping Packages

Once a user account is created on npm, one can use the username as a scope for publishing packages. This scope can be used as name-spaces for packages published by that user. Scopes allow users to [publish new packages sharing the same name](https://docs.npmjs.com/about-scopes) as that of an existing package on npm.  

Scoped packages start with a `@` followed by the scope (username) and a `/`:

```bash
@otee/toolbox
```

## Initialising npm

In the repository where the git repository was cloned, the command `npm init` should be executed along with the `scope` flag followed by the name of the scope (username):

```bash
npm init --scope=@otee
```

This will create the npm package. Before doing this (or thereafter), the contents of the package may be added. Wherever necessary, the command `npm install` will need to be executed, to install dependencies relied by the package.

## Adding User

Before publishing the package, one needs to verify the credentials of the npm account. To do this, the following command needs to be executed: `npm adduser`. This will require the user to type in the username and password of the npm account

## Publishing

The package can be [publicly published](https://docs.npmjs.com/creating-and-publishing-scoped-public-packages) by using the following command: `npm publish --access public`. 

Subsequent changes made to the package will need to be published using the same command. Note that, before publishing new changes, the version of the package on `package.json` needs to be updated. Otherwise, an error message will be displayed, indicating a conflict with an existing version. 

## Deprecating Versions

To decommission earlier versions which are no longer supported (or containing bugs), one can either use the command `npm deprecate` to indicate to users importing those versions of the package, that they are not supported any more. 

```js
npm deprecate otee@toolbox"< .3" "critical bug fixed in v1.0.3"
```

## package.json

The `package.json` file contains important meta-data about the project. While publishing a package on npm, the following fields are most significant:

- `name`: This field should contain the name of the package. This should contain the scoped name-space as well (as shown above).
- `version`: This should mention the current version of the package. This should be updated every time any change(s) is introduced to the package.
- `main`: This should mention the location of the main file (often `index.js`) which will be accessible from third-party dependency imports. Note that, no other file or directory in the package will be accessible to third-party imports, other than the file mentioned in `main`.

## Project Structure

Here's the structure of the project we want to publish as a package:

```bash
.
├── index.js
├── LICENSE
├── package.json
├── package-lock.json
├── README.md
├── src
│   ├── array_utils.js
│   └── string_utils.js
└── test
    ├── test_array_utils.js
    └── test_string_utils.js
```

Note that, `array_utils.js` and `string_utils.js` contains certain functions that will be exported by the package. One of them is `keep`:

```js
/**
 * This works like map but removes all the non-truthy values from the resulting array
 * @param {[any]} list 
 * @param {function} mapper 
 * @returns {[any]}
 */
export function keep(list, mapper) {
  let reducer = (acc, entry) => {
    let result = mapper(entry);
    if (result) {
      acc.push(result);
    }
    return acc;
  };
  return list.reduce(reducer, []);
}
```

Since this function is placed in a file other than `index.js`, we need to make it accessible from `index.js`. To do this, the corresponding file `array_utils.js` needs to be exported from `index.js`. This will mean that all the exports of the `array_utils` module will be exported from `index.js` optionally by using an alias (to avoid name conflicts):

```js
export * as array_utils from "./src/array_utils.js";
export * as string_utils from "./src/string_utils.js";
```

To use `keep` externally, we can import the package `@otee/toolbox` and access `keep` by referring to `array_utils`:

```js
import * as toolbox from "@otee/toolbox";
console.log(toolbox.array_utils.keep([1,2,3], a => a));
```

For every new module in this package, we need to export it from `index.js` file.