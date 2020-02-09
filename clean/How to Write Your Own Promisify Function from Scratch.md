
# How to Write Your Own Promisify Function from Scratch

![Image](https://www.freecodecamp.org/news/content/images/size/w2000/2019/12/write.jpg)

### Introduction

In this article, you will learn how to write your own promisify function from scratch.

Promisification helps in dealing with callback-based APIs while keeping code consistent with promises.

We could just wrap any function with `new Promise()` and not worry about it at all. But doing that when we have many functions would be redundant.

If you understand promises and callbacks, then learning how to write promisify functions should be easy. So let's get started.

### But have you ever wondered how promisify works?

The important thing is not to stop questioning. Curiosity has its own reason for existence.

— Albert Einstein

Promises were introduced in the [ECMA-262 Standard, 6th Edition](http://www.ecma-international.org/ecma-262/6.0/) (ES6) that was published in June 2015.

It was quite an improvement over callbacks, as we all know how unreadable "callback hell" can be :)
![img](https://www.freecodecamp.org/news/content/images/2019/12/callback-1.gif)

As a Node.js developer, you should know [what a promise](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261) is and [how it work internally](https://101node.io/blog/how-promises-actually-work-inside-out/), which will also help you in JS interviews. Feel free to review them quickly before reading on.

### Why do we need to convert callbacks to promises?

1. With callbacks, if you want to do something sequentially you will have to specify an `err` argument in each callback, which is redundant. In promises or async-await, you can just add a `.catch` method or block which will catch any errors that occurred in the promise chain
2. With callbacks, you have no control over when it's called, under what context, or how many times it's being called, which can lead to memory leaks.
3. Using promises, we control these factors (especially error handling) so the code is more readable and maintainable.

### How to make callback-based functions return a promise

There are two ways to do it:

1. Wrap the function in another function which returns a promise. It then resolves or rejects based on callback arguments.
2. Promisification — We create a util/helper function `promisify` which will transform all error first callback-based APIs.
Example: there’s a callback-based API which provides the sum of two numbers. We want to promisify it so it returns a `thenable` promise.

```
const getSumAsync = (num1, num2, callback) => {

  if (!num1 || !num2) {
    return callback(new Error("Missing arguments"), null);
  }
  return callback(null, num1 + num2);
}
getSumAsync(1, 1, (err, result) => {
  if (err){
    doSomethingWithError(err)
  }else {
    console.log(result) // 2
  }
})
```

### Wrap into a promise

As you can see, `getSumPromise` delegates all the work to the original function `getSumAsync`, providing its own callback that translates to promise `resolve/reject`.

### Promisify

When we need to promisify many functions we can create a helper function `promisify`.

##What is Promisification?

Promisification means transformation. It’s a conversion of a function that accepts a callback into a function returning a promise.

Using Node.js's `util.promisify()`:

```
const { promisify } = require('util')
const getSumPromise = promisify(getSumAsync) // step 1
getSumPromise(1, 1) // step 2
.then(result => {
  console.log(result)
})
.catch(err =>{
  doSomethingWithError(err);
})
```

So it looks like a magic function which is transforming `getSumAsync` into `getSumPromise` which has `.then` and `.catch` methods

### Let’s write our own promisify function:

If you look at step 1 in the above code, the `promisify` function accepts a function as an argument, so the first thing we have to do write a function that can do the same:

```
const getSumPromise = myPromisify(getSumAsync)
const myPromisify = (fn) => {}
```

After that, `getSumPromise(1, 1)` is a function call. This means that our promisify should return another function which can be called with the same arguments of the original function:

```
const myPromisify = (fn) => {
 return (...args) => {
 }
}
```

In the above code you can see we are [spreading](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) arguments because we don’t know how many arguments the original function has. `args` will be an array containing all the arguments.

When you call `getSumPromise(1, 1)` you’re actually calling `(...args)=> {}`.
In the implementation above it returns a promise. That’s why you’re able to use `getSumPromise(1, 1).then(..).catch(..)`.

I hope you’ve gotten the hint that the wrapper function `(...args) => {}` should return a promise.

### Return a promise

```
const myPromisify = (fn) => {
  return (...args) => {
    return new Promise((resolve, reject) => {

    })
  }
}
```

Now the tricky part is how to decide when to `resolve or reject` a promise.
Actually, that will be decided by the original `getSumAsync` function implementation – it will call the original callback function and we just need to define it. Then based on`err` and `result` we will `reject` or `resolve` the promise.

```
const myPromisify = (fn) => {
  return (...args) => {
    return new Promise((resolve, reject) => {
      function customCallback(err, result) {
       if (err) {
         reject(err)
       }else {
         resolve(result);
        }
      }
   })
  }
}
```

Our `args[]` only consists of arguments passed by `getSumPromise(1, 1)` except the callback function. So you need to add `customCallback(err, result)` to the `args[]`which the original function `getSumAsync` will call accordingly as we are tracking the result in `customCallback`.

### Push customCallback to args[]

```
const myPromisify = (fn) => {
   return (...args) => {
     return new Promise((resolve, reject) => {
       function customCallback(err, result) {
         if (err) {
           reject(err)
         }else {
          resolve(result);
         }
        }
        args.push(customCallback)
        fn.call(this, ...args)
      })
  }
}
```

As you can see, we have added `fn.call(this, args)`, which will call the original function under the same context with the arguments `getSumAsync(1, 1, customCallback)`. Then our promisify function should be able to `resolve/reject` accordingly.

The above implementation will work when the original function expects a callback with two arguments, (err, result). That’s what we encounter most often. Then our custom callback is in exactly the right format and `promisify works great for such a case.

**But what if the original `fn` expects a callback with more arguments like `callback(err, result1, result2, ...)`?**

In order to make it compatible with that, we need to modify our `myPromisify` function which will be an advanced version.

```const myPromisify = (fn) => {
   return (...args) => {
     return new Promise((resolve, reject) => {
       function customCallback(err, ...results) {
         if (err) {
           return reject(err)
         }
         return resolve(results.length === 1 ? results[0] : results)
        }
        args.push(customCallback)
        fn.call(this, ...args)
      })
   }
}
```

Example:

```
const getSumAsync = (num1, num2, callback) => {

  if (!num1 || !num2) {
    return callback(new Error("Missing dependencies"), null);
  }

  const sum = num1 + num2;
  const message = `Sum is ${sum}`
  return callback(null, sum, message);
}
const getSumPromise = myPromisify(getSumAsync)
getSumPromise(2, 3).then(arrayOfResults) // [6, 'Sum is 6']
```

That’s all! Thank you for making it this far!

I hope you’re able to grasp the concept. Try to re-read it again. It’s a bit of code to wrap your head around, but not too complex. Let me know if it was helpful 🙂

Don’t forget to share it with your friends who are starting with Node.js or need to level up their Node.js skills.

References:

https://nodejs.org/dist/latest-v8.x/docs/api/util.html#util_util_promisify_original

https://github.com/digitaldesignlabs/es6-promisify
