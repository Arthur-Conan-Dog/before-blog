# Module System

https://medium.com/sungthecoder/javascript-module-module-loader-module-bundler-es6-module-confused-yet-6343510e7bde

## Inline JS

Inline script is when you add JavaScript code between <script></script> tag.

Cons:

* Lack of Code Reusability

* Lack of Dependency Resolution

* Pollution of global namespace

## <script> tag

Cons:

* Lack of Dependency Resolution

* Pollution of global namespace

## Module Object and IIFE(Module Pattern)

In this approach, we expose only one object to global scope. The single object contains all the methods and values we need in our application. And for all the other files, they are wrapped into IIFE format. By wrapping each file into IIFE, all the local variables stay within the the scope of the function.

Cons:

* Lack of Dependency Resolution

* Pollution of global namespace: reduced, but still is not zero.

## CommonJS

In 2009, there were discussions about bringing JavaScript to server side. Thus ServerJS was born. ServerJS later changed its name to CommonJS.

CommonJS is not a JavaScript library. It is a standardization organization.

The goal of CommonJS is to define common APIs for web server, desktop and command line applications.

```js
// something.js
module.export = {};

// other.js
const something = require('./something');
```

If you have written code on NodeJS, this syntax may look familiar. That’s because NodeJS implemented CommonJS style module API.

Cons:

* Synchronous: the system will be on halt until the module is ready. This mean this line of code will freeze the browser while all the modules are being loaded.

### AMD: Asynchronous Module Definition

To transfer module syntax from server usage to browser usage, CommonJS proposed several module formats. One of the proposals, “Module/Transfer/C” later become Asynchronous Module Definition (AMD).

```js
define([‘add’, ‘reduce’], function(add, reduce){
  return function(){...};
});
```

CommonJS and AMD solves the two remaining problems with module pattern: dependency resolution and pollution of global scope.

### RequireJS

RequireJS is a JavaScript module loader. With RequireJS, you can write AMD style module for the browser side code.

### Cons

* AMD syntax is too verbose.

* It can be hard to maintain the order of the dependencies in the `define` function.

* With current browsers(HTTP 1.1), loading many small files can degrade the performance.

## Browserify

Because of these reasons, some people want to use CommonJS syntax, instead.

Browserify is a module bundler. Browserify traverse the dependency tree of your codes and bundle them up in a single file.

## UMD

What if you are writing a module and deploy to the wild internet? Which module style do you need to write?

Universal Module Definition(UMD) is to solve this particular problem. Under the hood, UMD is a series of if/else statements to identify module style that current environment supports.

## ES6 module syntax

JavaScript didn’t have module system built into the language. That’s the reason we have so many different way of importing and exporting modules.

ES6 uses ‘import’ and ‘export’ keywords to import and export modules.

Cons:

Browsers are not ready for this new syntax.

## Webpack

Webpack is a module bundler. Just like Browserify, it traverses dependency tree and bundles up into a single or more files.

If it is the same as Browserify, why would we need yet another module bundler? Webpack can handle CommonJS, AMD and ES6 modules. And Webpack comes with more flexibility and cool features.

## Questions

1. module.exports vs exports?

Both of them reference to the same empty object at the beginning, but only module.exports will be returned.

https://stackoverflow.com/a/26451885

