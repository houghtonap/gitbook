---
description: >-
  A guide for using top-level await in Typescript in the Visual Studio Code
  environment.
---

# VSCode Guide for Using Top-level Await in Typescript

## Table of Contents

* [What is Top-level `await`](vscode-guide-for-using-top-level-await-in-typescript.md#what-is-top-level-await)
* [Setting Up the VSCode Environment](vscode-guide-for-using-top-level-await-in-typescript.md#setting-up-the-vscode-environment)
* [References](vscode-guide-for-using-top-level-await-in-typescript.md#references)

13 November 2020

### What is Top-level `await`

The [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator is a primary expression in JavaScript that is used for waiting until a JavaScript [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) object is settled and it can **only** be used inside an [`async function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) or inside an `async function*` generator. Here is a simple TypeScript example:

```typescript
import * as fs from 'fs' ;

type AsyncIterables < T > = AsyncIterable < T > | AsyncIterableIterator < T > ;

async function toString < T > ( iterable : AsyncIterables < T > )
{
  let text = '' ;
  for await ( const datum of iterable )
    text += < string > < unknown > datum ;
  return text ;
}

const document : string = './test.txt' ;
const options  : object = { encoding : 'ascii' } ;
const stream = fs.createReadStream( document, options ) ;

toString( stream ).then( console.log ) ;
```

This example demonstrates the implementation of the `toString` asynchronous function that reads from any asynchronous `Iterable` type and collects the contents into the `text` string variable until there are no more datum in the `Iterable`. The `toString` asynchronous function resolves the JavaScript [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) object it initially made when it returns the contents of the `text` string variable.

The Node.js [`fs.createReadStream`](https://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options) method creates an asynchronous read stream for the document `./test.txt` which the `toString` asynchronous function consumes until the document has been completely read. When the initial JavaScript [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) object returned by the `toString` asynchronous function is settled, then the JavaScript [`Promise.then`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) chaining method makes the result available to the Node.js [`console.log`](https://nodejs.org/api/console.html#console_console_log_data_args) method.

The above example is simplistic for this discussion however, realize:

1. Instead of reading text from a document, a real world example might be reading from an HTTP stream or a database.
2. Instead of simply displaying the results on the console, there could be business logic that needs to happen in the JavaScript [`Promise.then`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) chaining method which might initiate complex nesting that could ultimately lead to a [Pyramid of Doom](https://en.wikipedia.org/wiki/Pyramid_of_doom_%28programming%29).

Instead of waiting for a JavaScript [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) object to be settled and using the JavaScript [`Promise.then`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) chaining method to handle the result, what if we could rewrite the last line of the example more intuitively as:

```typescript
console.log( await toString( stream ) ) ;
```

Unfortunately, at the time of this article, you cannot do that since the JavaScript [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator can only be used within an [`async function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) or within an `async function*` generator. The focus of the [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/) is to allow the use of the JavaScript [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator outside of an [`async function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) or outside an `async function*` generator, _with some caveats_.

The [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/) has reached Stage 3 status and has been experimentally implemented in both [Node.js](https://nodejs.org/api/esm.html#esm_experimental_top_level_await) and [TypeScript](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#top-level-await).

### Setting Up the VSCode Environment

When I initially tried to setup my VSCode environment I kept receiving:

> Top-level 'await' expressions are only allowed when the 'module' option is set to 'esnext' or 'system', and the 'target' option is set to 'es2017' or higher.ts\(1378\)

The TypeScript error is informative however, after changing my `tsconfig.json` configuration options to be:

```javascript
{
  "compilerOptions": {
    "target": "es2020",  /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "module": "esnext",  /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
    // Lots of other options removed for clarity...
  }
}
```

TypeScript continued to give me error 1378. After some extensive Googling I was unable to find a resolution for this issue and decided to step back to reevaluate the situation.

It is important to _note the caveats_ in the [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/). You are **only** allowed to use the JavaScript [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator outside of an [`async function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and outside of an `async function*` generator, **when it is done inside an ES module**. This is what TypeScript error 1378 is telling you. So why did TypeScript still give me error 1378 after I changed my `tsconfig.json` configuration options?

The solution is simple, the answer is complicated, but it boils down to a lack of coordination between TypeScript and Node.js. TypeScript transpiles its `.ts` language files into `.js` JavaScript files which can then be consumed by the JavaScript machinery, e.g., Node.js. Both TypeScript and Node.js need to support the [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/), which they do. TypeScript has supported the proposal since its [v3.8](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#top-level-await) revision and Node.js has supported the proposal since its [v14.8.0](https://nodejs.org/en/blog/release/v14.8.0/) revision.

TypeScript error 1378 was explicit in how to correct the situation, but what about Node.js? There are two significant aspects about Node.js supporting the [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/):

1. Node.js "unflagged" its experimental command line flag `--harmony-top-level-await`.
2. Node.js, per the proposal, implemented the use the JavaScript [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator outside of an [`async function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and outside of an `async function*` generator, **when it was done inside an ES module**.

The "unflagging" of the experimental `--harmony-top-level-await` command line flag means that they turned the experimental feature on by default rather than requiring you to enable the feature. It also means that you do not need to change any VSCode build or debug tasks to include that command line flag when running or debugging the TypeScript transpiled files.

The other part is where the issue becomes problematic. Node.js correctly follows the [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/). ES modules **must** be placed in JavaScript files ending with a `.mjs` file extension. Unfortunately, TypeScript transpiles its language files to JavaScript files ending with a `.js` file extension and there is currently no way to allow TypeScript to generate ES modules with an `.mjs` file extension.

It might appear that we are at the end of the road without a resolution, but we need to dig deeper into Node.js and its implementation of [ES modules](https://nodejs.org/api/esm.html). In the [Enabling](https://nodejs.org/api/esm.html#esm_enabling) section of the documentation it says:

> Node.js treats JavaScript code as CommonJS modules by default. Authors can tell Node.js to treat JavaScript code as ECMAScript modules via the `.mjs` file extension, the `package.json` ["type"](https://nodejs.org/api/packages.html#packages_type) field, or the `--input-type` flag. See [Modules: Packages](https://nodejs.org/api/packages.html#packages_determining_module_system) for more details.

The solution to TypeScript error 1378 involves not only changing the appropriate options in the `tsconfig.json` configuration file, but also configuring the appropriate options in the `package.json` configuration file:

```javascript
{
  "type": "module",
  "engines" : { "node" : ">=14.8.0" },
}
```

The above snippet is **only showing** the appropriate options in the `package.json` configuration file for clarity. To change Node.js behavior when loading ES modules you **must change** the `type` key to have the value `module` so it will load ES modules from either JavaScript files with a `.js` or a `.mjs` file extension. The `engines` key is not strictly required, but since the [ECMAScript TC39 Top-level `await` proposal](https://github.com/tc39/proposal-top-level-await/) was _turned on by default_ in the [v14.8.0](https://nodejs.org/en/blog/release/v14.8.0/) revision, it is a dependency of the Node.js project and it should be made explicit.

Unfortunately, after I changed the `package.json` configuration options, TypeScript gave me:

> 'await' expressions are only allowed at the top level of a file when that file is a module, but this file has no imports or exports. Consider adding an empty 'export {}' to make this file a module.ts\(1375\)

Here is a simple TypeScript example demonstrating error 1375 :

```javascript
console.log( await Promise.resolve( 200 ) ) ;
```

TypeScript error 1375 is valid. This TypeScript file is not a valid ES module since it **does not have any** import or export statements. However, when TypeScript transpiles this file to JavaScript and the resulting JavaScript file is run by Node.js, Node.js produces the correct result! The reason is that we changed the appropriate options in the `package.json` configuration file and the Node.js behavior changed to treat JavaScript files ending in `.js` and `.mjs` as ES modules.

Some takeaways from this:

* For the initial TypeScript error 1378, TypeScript could have easily checked the project's `package.json` configuration file and seeing that the `type` key was either missing or not set appropriately could have also indicated that correction in its message.
* For TypeScript error 1375, TypeScript considers this to be an error however, it could have checked the project's `package.json` configuration file and seeing the `type` key was set to the value `module` then either reduced this situation to a warning or not given any error since the options in the `package.json` configuration file are set appropriately for Node.js to consider the transpiled JavaScript as a valid ES module.

To correct TypeScript 1375 error in the above one line example, simply change it to:

```javascript
export { } ;
console.log( await Promise.resolve( 200 ) ) ;
```

Now both TypeScript and Node.js are cooperating and living in harmony.

### References

* [ECMAScript 2020 Language Specification](http://www.ecma-international.org/ecma-262/11.0/index.html#title)
* [ECMAScript TC39 Top-level `await` Proposal](https://github.com/tc39/proposal-top-level-await/)
* [Mozilla Developer Network JavaScript Guide: Iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators)
* [Mozilla Developer Network JavaScript Guide: Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
* [Mozilla Developer Network JavaScript Reference: `async function` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* [Mozilla Developer Network JavaScript Reference: `await` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
* [Mozilla Developer Network JavaScript Reference: `Promise` object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [Node.js ECMAScript Modules Documentation](https://nodejs.org/api/esm.html#esm_experimental_top_level_await)
* [Node.js v14.8.0 Release Notes](https://nodejs.org/en/blog/release/v14.8.0/)
* [TypeScript v3.8 Release Notes](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#top-level-await)
* [Wikipedia Pyramid of Doom article](https://en.wikipedia.org/wiki/Pyramid_of_doom_%28programming%29)

