# Module System

ref: [Brief history of JavaScript Modules](https://medium.com/sungthecoder/javascript-module-module-loader-module-bundler-es6-module-confused-yet-6343510e7bde)

## Inline JS

Inline script is when you add JavaScript code between `<script></script>` tag.

Cons:

- Lack of Code Reusability

- Lack of Dependency Resolution

- Pollution of global namespace

## `<script></script>` tag

Divide the big chunk of inline JS into smaller pieces and load them using `<script>` tags.

Cons:

- Lack of Dependency Resolution

- Pollution of global namespace

## Module Object and IIFE(Module Pattern)

Expose only one object to global scope. The single object contains all the methods and values we need in our application. And for all the other files, they are wrapped into IIFE format.

By wrapping each file into IIFE, all the local variables stay within the the scope of the function.

eg. jQuery.

Cons:

- Lack of Dependency Resolution

- Pollution of global namespace: reduced to 1, but still is not 0.

## CommonJS & AMD

Background: In 2009, discussions about bringing JavaScript to server side. Thus ServerJS was born and later changed its name to CommonJS.

CommonJS is not a JavaScript library. It is a standardization organization.

The goal of CommonJS is to define common APIs for web server, desktop and command line applications.

```js
// something.js
module.export = {};

// other.js
const something = require("./something");
```

eg. Node.js.

Cons:

- Synchronous: the system will be on halt until the required module is ready which might freeze the browser.

### AMD: Asynchronous Module Definition

To transfer module syntax from server usage to browser usage, CommonJS proposed several module formats. One of the proposals, “Module/Transfer/C” later become Asynchronous Module Definition (AMD).

```js
define([‘add’, ‘reduce’], function(add, reduce){
  return function(){...};
});
```

CommonJS and AMD solves the two remaining problems with module pattern: dependency resolution and pollution of global scope.

### RequireJS

RequireJS is a JavaScript **module loader**.

With RequireJS, you can write AMD style module for the browser side code by provide only one script: `<script data-main=”main” src=”require.js”></script>`. `data-main` attribute tells RequireJS to where is the starting point of your application.

### Cons

- AMD syntax is too verbose.

- It can be hard to maintain the order of the dependencies in the `define` function.

- With current browsers(HTTP 1.1), loading many small files can degrade the performance.

## Browserify

Because of above reasons, some people want to use CommonJS syntax instead.

Browserify is a **module bundler**.

It traverse the dependency tree of your codes and bundle them up in a single file.

## UMD

Problem: what if you are writing a module and deploy to the internet? which module style do you need to write?

Universal Module Definition(UMD) is a series of if/else statements to identify module style that current environment supports.

## ES6 module syntax

JavaScript didn’t have module system built into the language. That’s the reason we have so many different way of importing and exporting modules.

ES6 uses ‘import’ and ‘export’ keywords to import and export modules.

Cons:

- Browsers are not ready for this new syntax.

## Webpack

Webpack is a __module bundler__, Just like Browserify, it traverses dependency tree and bundles up into a single or more files.

If it is the same as Browserify, why would we need yet another module bundler?

Webpack can handle CommonJS, AMD and ES6 modules at the same time. It also comes with more flexibility and cool features like:

- code split: reuse shared modules;

- loader: can load any type of files;

- plugin: manipulate bundles before it is written into files.

## Questions

1. [module.exports vs exports?](https://stackoverflow.com/a/26451885)

Both of them reference to the same empty object at the beginning, but only module.exports will be returned.
