This Node.js module (`nwglobal`) provides a workaround for [node-webkit](https://github.com/rogerwang/node-webkit/)'s issues [#702](https://github.com/rogerwang/node-webkit/issues/702), [#716](https://github.com/rogerwang/node-webkit/issues/716), [#832](https://github.com/rogerwang/node-webkit/issues/832).

These issues happen in node-webkit because, as the modules run in Node context, the constructors of their global objects (such as `Date` or `ArrayBuffer` or even `Array`) differ from WebKit's.

(An example below demonstrates that you may pass an array from some `<script>…</script>` to the [async](https://github.com/caolan/async/) module that you have previously `require`d, but that module cannot recognize such an array.)

To prevent the trouble, `nwglobal` exports Node's constructors. You may use them instead of the constructors available in WebKit's context, and then you may pass the resulting object instances to any Node code.

# Installation

[![(npm package version)](https://nodei.co/npm/nwglobal.png?compact=true)](https://npmjs.org/package/nwglobal)

* Latest packaged version: `npm install nwglobal`

* Latest githubbed version: `npm install https://github.com/Mithgol/nwglobal/tarball/master`

You may visit https://github.com/Mithgol/nwglobal#readme occasionally to read the latest `README` because the package's version is not planned to grow after changes when they happen in `README` only. (And `npm publish --force` is [forbidden](http://blog.npmjs.org/post/77758351673/no-more-npm-publish-f) nowadays.)

# Example

Classic async [waterfall example:](https://github.com/caolan/async/blob/b6a1336bcb0865d6d26224f9553b9e1886fe696e/README.md#waterfall)

```js
require('async').waterfall([
    function(callback){
        callback(null, 'one', 'two');
    },
    function(arg1, arg2, callback){
        callback(null, 'three');
    },
    function(arg1, callback){
        // arg1 now equals 'three'
        callback(null, 'done');
    }
], function (err, result) {
   // result now equals 'done'
   console.log(result);
});
```

does not report `'done'` in node-webkit ([issue #832](https://github.com/rogerwang/node-webkit/issues/832)), but can be fixed with the following changes:

```js
require('async').waterfall( require('nwglobal').Array(
    function(callback){
        callback(null, 'one', 'two');
    },
    function(arg1, arg2, callback){
        callback(null, 'three');
    },
    function(arg1, callback){
        // arg1 now equals 'three'
        callback(null, 'done');
    }
), function (err, result) {
   // result now equals 'done'
   console.log(result);
});
```

You may find another example in “[Differences of JavaScript contexts](https://github.com/rogerwang/node-webkit/wiki/Differences-of-JavaScript-contexts)”.

# Implementation details

The following Node.js globals are available as the exported fields of `require('nwglobal')`:

* **Standard object types:** `Array`, `Boolean`, `Date`, `Function`, `Number`, `Object`, `RegExp`, `String`.

* **Typed array types:** `ArrayBuffer`, `DataView`, `Float32Array`, `Float64Array`, `Int16Array`, `Int32Array`, `Int8Array`, `Uint16Array`, `Uint32Array`, `Uint8Array`.

* **Error types:** `Error`, `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, `URIError`.

* **Special value types:** `Infinity`, `NaN`, `undefined`, `null`.

However, the latter four (`Infinity`, `NaN`, `undefined`, `null`) are actually superglobal (i.e. they are the same in Node's and WebKit's contexts). You may use `nwglobal` to check it with the following four statements in node-webkit's “Developer Tools” console:

* `null === require('nwglobal').null`

* `typeof require('nwglobal').undefined === 'undefined'`

* `Infinity === require('nwglobal').Infinity`

* `isNaN( require('nwglobal').NaN )`

These statements are `true`. (Meaning that you won't need these four exported values IRL.)

# Limits

It is not possible to replace the default constructors of arrays and objects created by `[]` and `{}` initialisers.

([Standard ECMA-262 5.1 Edition](http://www.ecma-international.org/ecma-262/5.1/) very specifically defines [array initialiser](http://www.ecma-international.org/ecma-262/5.1/#sec-11.1.4) and [object initialiser](http://www.ecma-international.org/ecma-262/5.1/#sec-11.1.5) so that the corresponding **standard built-in constructor** is used for each.)

Therefore you have to use `Array()` and `Object()` constructors exported by `require('nwglobal')` in order to create arrays and objects in Node's context.

NB: you can cast browser's `[]` array to node.js array with `require('nwglobal').Array.prototype.slice.call(browserContextArray)`.

NB2: With this you can transform jquery set.

# License

MIT License, see the LICENSE file.
