# Javascript Promises

## How to deal with long running operations

We have seen that long running processes can lock up an interface.

For example we may invoke a function that's supposed to retrieve data from a remote API. If the API is slow to return the data, we may be stuck in our application without being able to continue on our next task until all the data is received or an error is generated. This makes for a bad user experience.

One way to solve this problem is to use callback functions when we need to manage long running processes.
Another, more popular way, is to use *Promises*.

### Promises

A Javascript Promise is an object that performs a long running, asynchronous operation and returns the result of this operation if it was successful, or an error if it wasn't.

Let's look at the code below. We defined a function called ~ that sets up and returns a Promise object.
The Promise object takes an arrow function which in turn takes two arguments, `resolve` and `reject`.

Inside the Promise we check if the `isGood` parameter is true.
If it is, and the promise succeeds, resolve is called printing out a good message.
If `isGood` is not true, the promise fails, `reject` is called instead and the returned message is a bad one.

```
function makePromise(isGood) {
  return new Promise((resolve, reject) => {
    if (isGood) {
      resolve('all good');
    } else {
      reject('all bad');
    }
  });
}

let p = makePromise(true);

console.log(p); // all good
```

When we invoke makePromise(), we pass a true object. This resolves the promise and the string `'all good'` is returned.

If the value passed to `makePromise()` was `false`, the promise would not be resolved and the `'all bad'` message would be printed out.

Promises can be in a *pending* state if the Promise is neither resolved nor rejected.

### Pending promises

In the following code, we create a new Promise and we pass an empty anonymous function as an argument to it. Since this empty function has not called either `resolve` or `reject`, the Promise is now in a pending state.
We can see it's pending when we print it out on the console.

```
console.log(new Promise(() => {}));
// Promise { <pending> }
```

If the promise is not resolved yet, it sits there in a pending state. In the real world that could happen if you are making an external API call and the call takes a while to resolve.

### How to get values out of a Promise

We get values out of a promise with `.then()` and `.catch()`.
We attach these methods at the end of the Promise.
If the promise is resolved, the result will be available inside `.then()`. Otherwise, the result will be available on the `.catch()` method.

We simply concatenate these two methods one after the other and this lets us take care of both outputs. Here's an example:

```
p = makePromise(true);
console.log(p); // Promise { 'all good' }

p = makePromise(false);
console.log(p); // Promise { <rejected> 'all bad' }

p
  .then(goodValue => console.log(goodValue)) // all good
  .catch(badValue => console.log(badValue))  // all bad
  ```

When we write Promises it's helpful to separate `.then()` and `.catch()` on different lines for better readability.

If the result of the first `.then()` needs to be processed further we can also concatenate multiple `.then()` methods. The result of the first `.then()` will then be passed to the next `.then()` method.

We will see more Promises when we talk about retrieving data from external APIs in React.
