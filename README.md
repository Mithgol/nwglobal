This Node.js module provides a workaround for [node-webkit](https://github.com/rogerwang/node-webkit/)'s issues [#702](https://github.com/rogerwang/node-webkit/issues/702), [#716](https://github.com/rogerwang/node-webkit/issues/716), [#832](https://github.com/rogerwang/node-webkit/issues/832).

These issues happen in node-webkit because, as the modules run in Node context, the constructors of their global objects (such as `Date` or `ArrayBuffer` or even `Array`) differ from WebKit's.

(For example, you may pass an array to the [async](https://github.com/caolan/async/) module that you have previously `require`d, but the module cannot recognize that.)

# Installation

* Latest packaged version: `npm install nwglobal`

* Latest githubbed version: `npm install https://github.com/Mithgol/nwglobal/tarball/master`

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

does not report `'done'` in node-webkit, but can be fixed with the following changes:

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

# License

MIT License, see the LICENSE file.