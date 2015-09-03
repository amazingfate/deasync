DeAsync.js
=======
[![NPM version](http://img.shields.io/npm/v/deasync.svg)](https://www.npmjs.org/package/deasync)

Deasync is a utility module which provides utilities to wrap asynchronous functions into synchronous equivalent functions. While designed with [Node.js](http://nodejs.org) in mind and installable via `npm install deasync`, the core of deasync is writen in C++.


## Installation
Prerequisites

1. Node v0.11 or higher
2. Except on a few [platform and Node version combinations](https://github.com/abbr/deasync-bin) where binary distribution is included, deasync uses node-gyp to compile C++ source code so you may need the compilers listed in [node-gyp](https://github.com/TooTallNate/node-gyp). You may also need to [update npm's bundled node gyp](https://github.com/TooTallNate/node-gyp/wiki/Updating-npm's-bundled-node-gyp).

To install, run 
```npm install deasync```



## API Documentation

<a name="sleep" />
### sleep(timeout, callback)

Run synchronously wrapped version of the `setTimeout()` native Javascript function. The [callback] function will be called after the amount of time (in milliseconds) represented by `timeout` has passed. 


**Note:** when using `sleep()`, it is important to note that it should be taken care that your code does not block the entire thread AND does not incur a busy wait that pegs the CPU to 100%.

__Arguments__

* `timeout` - An integer that represents time to wait in miliseconds. 
* `callback()` - A callback to run once the timeout has completed

__Example__

```js
function SyncFunction(){
  var ret;
  setTimeout(function(){
      ret = "hello";
  },3000);
  while(ret === undefined) {
    require('deasync').sleep(100);
  }
  // returns hello with sleep; undefined without
  return ret;    
}
```

---------------------------------------

<a name="loopUntil" />
### loopUntil(predicate)

This will continously run in a blocking fashion until `predicate()` returns a `truthy` value. The predicate function will be called every `process.nextTick()`. Code below a `loopUntil` will not be executed while predicate() returns a falsey value;


**Note:** when using `sleep()`, it is important to note that it should be taken care that your code does not block the entire thread AND does not incur a busy wait that pegs the CPU to 100%.

__Arguments__

* `predicate` - A callback function that **must** return a boolean value. Blocking will continue until predicate() returns a truthy value.

__Example__

```js
var done = false,
    data,
    deasync = require('deasync');
    
asyncFunction(p1,function cb(res){
  data = res;
  done = true;
});
deasync.loopUntil(function(){return !done;});
// data is now populated
```

## Usages
* Generic wrapper of async function with standard API signature `function(p1,...pn,function cb(err,res){})`

```
var deasync = require('deasync');
var cp = require('child_process');
var exec = deasync(cp.exec);
// output result of ls -la
try{
  console.log(exec('ls -la'));
}
catch(err){
  console.log(err);
}
// done is printed last, as supposed, with cp.exec wrapped in deasync; first without.
console.log('done');
```

* For async function with non-standard API, for instance `function asyncFunction(p1,function cb(res){})`, use `loopWhile(predicateFunc)` where `predicateFunc` is a function that returns boolean loop condition

```
var done = false;
var data;
asyncFunction(p1,function cb(res){
  data = res;
  done = true;
});
require('deasync').loopWhile(function(){return !done;});
// data is now populated
```

* Sleep (a wrapper of setTimeout)

```
function SyncFunction(){
  var ret;
  setTimeout(function(){
      ret = "hello";
  },3000);
  while(ret === undefined) {
    require('deasync').sleep(100);
  }
  // returns hello with sleep; undefined without
  return ret;    
}
```


### Motivation
Suppose you maintain a library that exposes a function <code>getData</code>. Your users call it to get actual data:   
<code>var output = getData();</code>  
Under the hood data is saved in a file so you implemented <code>getData</code> using Node.js built-in <code>fs.readFileSync</code>. It's obvious both <code>getData</code> and <code>fs.readFileSync</code> are sync functions. One day you were told to switch the underlying data source to a repo such as MongoDB which can only be accessed asynchronously. You were also told to avoid pissing off your users, <code>getData</code> API cannot be changed to return merely a promise or demand a callback parameter. How do you meet both requirements?

You may tempted to use [node-fibers](https://github.com/laverdet/node-fibers) or a module derived from it, but node fibers can only wrap async function call into a sync function inside a fiber. In the case above you cannot assume all  callers are inside fibers. On the other hand, if you start a fiber in `getData` then `getData` itself will still return immediately without waiting for the async call result. For similar reason ES6 generators introduced in Node v11 won't work either. 

What really needed is a way to block subsequent JavaScript from running without blocking entire thread by yielding to allow other events in the event loop to be handled. Ideally the blockage is removed as soon as the result of async function is available. A less ideal but often acceptable alternative is a `sleep` function which you can use to implement the blockage like ```while(!done) sleep(100);```. It is less ideal because sleep duration has to be guessed. It is important the `sleep` function not only shouldn't block entire thread, but also shouldn't incur busy wait that pegs the CPU to 100%. 

DeAsync supports both alternatives.

## License

The MIT License (MIT)

Copyright (c) 2015

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
