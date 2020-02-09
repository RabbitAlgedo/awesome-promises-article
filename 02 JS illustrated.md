# JS illustrated: Promises

This is the second **JS illustrated** article I've wrote. The first one was about the event


ES6 (ECMAScript 2015) has introduced a new feature called Promise. There are numerous excelent articles and books that explain the way that Promises work. In this article, we are going to try to provide a simple and understandable description of how Promises work, without digging into much detail.

Before we start explaining what a promise is and how it works, we need to take a look at the reason of its existence, in order to understand it correctly. In other words, we have to identify the problem that this new feature is trying to solve.


===


### Callbacks

Promises are inextricably linked to asynchrony. Before Promises, developers were able to write asynchronous code using callbacks. A callback is a function that is provided as parameter to another function, in order to be called, at some point in the future, by the latter function.

Lets take a look at the following code

```
ajaxCall("http://url-to-api", myCallback);
function myCallback(response) {
  // Handle response
}
```

We are calling ajaxCall function passing a url path as first argument and a callback function as the second argument. The ajaxCall function is supposed to execute a request to the provided url and call the callback function when the response is ready. In the meanwhile, the program continues its execution (the ajaxCall does not block the execution). That's an asynchronous piece of code.

This works great! But there are some problems that might arise, like the following (Kyle Simpson, 2015, You don't know JS: Async & Performance, 42):

* The callback function never gets called
* The callback function gets called too early
* The callback function gets called too late
* The callback function gets called more than once

These problems might be more difficult to be solved if the calling function (`ajaxCall`) is an external tool that we are not able to fix or even debug.



===

Note:[put some bridge here]

It seems that a serious problem with callbacks is that they give the control of our program execution to the calling function, a state known as *inversion of control* (IoC).

The following illustration shows the program flow of a callback based asynchronous task. We assume that we call a third party async function passing a callback as one of its parameters. The red areas indicate that we do not have the control of our program flow in these areas. We do not have access to the third party utility, so the right part of the illustration is red. The red part in the left side of the illustration indicates that we do not have the control of our program until the third party utility calls the callback function we provided.

![1](https://res.cloudinary.com/practicaldev/image/fetch/s--w3FxvWjN--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/rqchyt5pvcnd1cl057el.png)

Note: [Explain IoC]

But wait, there's something else, except from the IoC issue, that makes difficult to write asynchronous code with callbacks. It is known as the callback hell and describes the state of multiple nested callbacks, as shown in the following snippet.

```
function root(url) {
  ajax(url, resp1 => {
    ajax(resp1, resp2 => {
      ajax(resp2, resp3 => {
        ajax(resp3, resp4 => {
          ajax(resp4, resp5 => {
            // To be continued...
          });
        });
      });
    });
  });
}
```

As we can see, multiple nested callbacks makes our code unreadable and difficult to debug.

So, in order to recap, the main problems that arise from the use of callbacks are:

* Losing the control of our program execution (Inversion of Control)
* Unreadable code, especially when using multiple nested callbacks



===



### Promises

Now lets see what Promises are and how they can help us to overcome the problems of callbacks.

According to MDN

> The Promise object represents the eventual completion (or failure) of an asynchronous operation, and its resulting value.

and

> A Promise is a proxy for a value not necessarily known when the promise is created

What's new here is that asynchronous methods can be called and return something immediately, in contrast to callbacks where you had to pass a callback function and hope that the async function will call it some time in the future.

**But what's that it gets returned?**

It is a promise that some time in the future you will get an actual value.

For now, you can continue your execution using this promise
as a **placeholder of the future value.**


===

Note: [put bridge here]

Lets take a look at the constructor

```
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(); // Or reject
  }, 100);
});
```

We create a Promise with the `new Promise()` statement,
passing a function, called the **executor**. The executor gets called immediately at the time we create the promise, passing two functions as the
first two arguments, the **resolve** and the **reject**
functions respectively. The executor usually starts
the asynchronous operation (the `setTimeout()` function in our example).

The resolve function is called when the asynchronous task has been successfully completed its work. We then say that the promise has been **resolved**.
Optionally yet very often, we provide the result of the asynchronous task to the resolve function as the first argument.

In the same way, in case where the asynchronous task
has failed to execute its assigned task, the reject
function gets called passing the error message as the
first argument and now we say that the promise has been **rejected**.


===

Note: [put a bridge here]

The next illustration presents the way that promises work.
We see that, even if we use a third party utility,
we still have the control of our program flow because we,
immediately, get back a promise, a placeolder
that we can use in place of the actual future value.

![2](https://res.cloudinary.com/practicaldev/image/fetch/s--J1OuwpqD--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/4p38iy08e1zxli9umt1c.png)



===



According to Promises/A+ specification

> A promise must be in one of three states: pending,
fulfilled, or rejected

When a promise is in *pending* state, it can either
transition to the *fullfilled* (resolved) or the *rejected* state.

![3](https://res.cloudinary.com/practicaldev/image/fetch/s--z01-U2gm--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/s7eyl759whe2d15iea81.png)

What's very important here is that, if a promise
gets one of the fulfiled or rejected state, **it cannot change its state and value**. This is called **immutable identity** and protects us from unwanted changes in the state that would lead to undescoverable bugs in our code.



===



### Getting control back

As we saw earlier, when we use callbacks we rely on another piece of code, often writen by a third party, in order to trigger our callback function and continue the execution of the program.

With promises we do not rely on anyone in order to continue our program execution. We have a promise in our hands that we will get an actual value at some point in the future. For now, we can use this promise as a placeholder of our actual value and continue our program execution just as we would do in synchronous programming.


===


### Readable async code

Promises make our code more readable compared to callbacks (remember the callback hell?). Check out the following snippet:

```
function callAjax(url) {
  return new Promise((resolve, reject) => {
    ajax(url, (resp) => {
      resolve(resp);
    },
    err => {
      reject(err);
    });
  });  
}

callAjax("https://url.com")
  .then((resp1) => {
    return callAjax(resp1);
  })
  .then((resp2) => {
    return callAjax(resp2);
  })
  .then((resp3) => {
    return callAjax(resp3);
  })
  .then((resp4) => {
    return callAjax(resp4);
  });
  ```

We can chain multiple promises in a sequential manner and make our code look like synchronous code, avoiding nesting multiple callbacks one inside another.


===


### Promise API

The `Promise` object exposes a set of static methods that can be called in order to execute specific tasks. We are going to briefly present each on of them with some simple illustrations whenever possible.

**Promise.reject(reason)**
`Promise.reject()` creates an immediately rejected promise and it is a shorthand of the following code:

 ```
const p1 = new Promise((resolve, reject) => {
  reject("Error");
});
 ```

 The next snippet shows that `Promise.reject()` returns the same rejected promise with a traditionally constructed promise (`new Promise()`) that gets immediately rejected with the same reason.

```
 new Promise((resolve, reject) => {
  reject("Error");
}).catch(err => {
  console.log(err); // Logs "Error"
});

Promise.reject("Error").catch(err => {
  console.log(err); // Logs "Error"
});
```

**Promise.resolve(value)**
`Promise.resolve()` creates an immediately resolved promise with the given value. It is a shorthand of the following code:

```
const p1 = new Promise((resolve, reject) => {
  resolve(1);
});
```


===


Comparing a promise constructed with the `new` keyword and then, immediately resolved with value `1`, to a promise constructed by `Promise.resolve()` with the same value, we see that both of them return identical results.

```
new Promise((resolve, reject) => {
  resolve(1);
}).then(resp => {
  console.log(resp); // Logs 1
});

Promise.resolve(1).then(resp => {
  console.log(resp); // Logs 1
});
```

## **Thenables**


According to Promises/A+ specification

> *thenable* is an object or function that defines a then method

Lets see a *thenable* in action in the following snippet. We declare the thenable object that has a then method which immediately calls the second function with the `"Rejected"` value as argument. As we can see, we can call the `then` method of `thenable` object passing two functions the second of which get called with the `"Rejected"` value as the first argument, just like a promise.

```
const thenable = {
  then: (resolve, reject) => {
    reject("Rejected");
  }
};

thenable.then(resp => {
  console.log(resp); // Not executed
}, err => {
  console.log(err); // Logs "Rejected"
});
```



===



But what if we want to use the catch method as we do with promises?

```
thenable.catch(err => {
  console.log(err); // "TypeError: thenable.catch is not a function
});
```

Oops! En error indicating that the thenable object does not have a catch method available occurs! That's normat because that's the case. We have declared a plain object with only one method, then, that *happens* to conform, in some degree, to the promises api behaviour.

In any case, *it doesn't mean that an object which exposes a then method, is a promise object*.

But how can `Promise.resolve()` help with this situation?

`Promise.resolve()` can accept a *thenable* as its argument and then return a promise object. Lets treat our thenable object as a promise object.

```
Promise.resolve(thenable).catch(err => {
  console.log(err); // Logs "Rejected"
});
```

`Promise.resolve()` can be used as a tool of converting objects to promises.


===


**Promise.all(iterable)**
`Promise.all()` waits for all promises in the provided iterable to be resolved and, then, returns an array of the values from the resolved promises **in the order they were specified in the iterable**.

In the following example, we declare 3 promises, `p1`, `p2` and `p3` which they all get resolved after a specific amount of time. We intentionaly resolve `p2` before p1 to demonstrate that the order of the resolved values that get returned, is the order that the promises were declared in the array passed to `Promise.all()`, and not the order that these promises were resolved.

```
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);
  }, 200);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2);    
  }, 100);
});
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(3);
  }, 300);
});

Promise.all([p1, p2, p3]).then((resp) => {
  console.log(resp); // Logs [1,2,3]
}, (err) => {
  console.log(err); // Not executed
});
```

In the upcoming illustrations, the green circles indicate that the specific promise has been resolved and the red circles, that the specific promise has been rejected.

![5aum0ohxf7djz97sekez.png](https://res.cloudinary.com/practicaldev/image/fetch/s--uYJ-XKE_--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/5aum0ohxf7djz97sekez.png)

But what happens if one or more promises get rejected? The promise returned by `Promise.all()` gets rejected with the value of the first promise that got rejected among the promises contained in the iterable.


===


Even if more than one promises get rejected, the final result is a rejected promise with the value of the **first promise which was rejected**, and not an array of rejection messages.

```
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("Error p1");
  }, 200);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2);    
  }, 100);
});
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(3);
  }, 300);
});

Promise.all([p1, p2, p3]).then((resp) => {
  console.log(resp); // Not executed
}, (err) => {
  console.log(err); // Logs "Error p1"
});
```

![26ipydz90e2qtquwvh6o.png](https://res.cloudinary.com/practicaldev/image/fetch/s--Xn0_kuRh--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/26ipydz90e2qtquwvh6o.png)

## **Promise.allSettled(iterable)**

``Promise.allSettled()`` behaves like ``Promise.all()`` in the sence that it waits far all promises to be fullfiled. The difference is in the outcome.

```
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);    
  }, 200);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {    
    reject("Error p2");
  }, 100);
});
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(3);
  }, 300);
});

Promise.allSettled([p1, p2, p3]).then((resp) => {
  console.log(resp);
});

// Outcome
[{
  status: "fulfilled",
  value: 1
}, {
  reason: "Error p2",
  status: "rejected"
}, {
  status: "fulfilled",
  value: 3
}]
```

As you can see in the above snippet, the promise returned by the `Promise.allSettled()` gets resolved with an array of objects describing the status of the promises that were passed.


===


**Promise.race(iterable)**

`Promise.race()` waits for the first promise to be resolved or rejected and resolves, or rejects, respectively, the promise returned by `Promise.race()` with the value of that promise.

In the following example, `p2` promise resolved before `p1` got rejected.

```
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {    
    reject("Error p1");
  }, 200);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2);    
  }, 100);
});

Promise.race([p1, p2]).then((resp) => {
  console.log(resp); // Logs 2
}, (err) => {
  console.log(err); // Not executed
});
```


===

Note: [put bridge here]

![7xdypmymigrdmhzyooem.png](https://res.cloudinary.com/practicaldev/image/fetch/s--t38-GZbz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/7xdypmymigrdmhzyooem.png)

If we change the delays, and set `p1` to be rejected at 100ms, before `p2` gets resolved, the final promise will be rejected with the respecive message, as shown in the following illustration.

```
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {    
    reject("Error p1");
  }, 100);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2);    
  }, 200);
});

Promise.race([p1, p2]).then((resp) => {
  console.log(resp); // Not executed
}, (err) => {
  console.log(err); // Logs "Error p1"
});
```

![qxqjylr65q2alw939jmz.png](https://res.cloudinary.com/practicaldev/image/fetch/s--vqIbI-mb--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/qxqjylr65q2alw939jmz.png)


===


### Promise.prototype methods

We are now going to take a look at some methods exposed by the promise's prototype object. We have already mentioned some of them previously, and now, we are going to take a look at each one of them in more detail.

**Promise.prototype.then()**

We have already used `then()` many times in the previous examples. `then()` is used to handle the settled state of promises. It accepts a resolution handler function as its first parameter and a rejection handler function as its second parameter, and returns a promise.

```
const p1 = Promise.resolve(1);

p1.then(res => {
  console.log(res); // Logs 1  
}, err => {
  console.log(err); // Not executed
});
```


===

Note:[put a bridge here]

The next two illustrations present the way that a `then()` call operates.

![ot07iq679691vqrwtdxf.png](https://res.cloudinary.com/practicaldev/image/fetch/s--bG_77mFY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/ot07iq679691vqrwtdxf.png)

If the resolution handler of a `then()` call of a resolved promise is not a function, then no error is thrown, instead, the promise returned by `then()` carries the resolution value of the previous state.

In the following snippet, `p1` is resolved with value `1`. Calling `then()` with no arguments will return a new promise with p1 resolved state. Calling `then()` with an `undefined` resolution handler and a valid rejection handler will do the same. Finally, calling `then()` with a valid resolution handler will return the promise's value.

```
const p1 = Promise.resolve(1);

p1.then()
  .then(undefined, err => {
    console.log(err); // Not executed
  })
  .then(res => {
    console.log(res); // Logs 1  
  });
```

The same will happen in case that we pass an invalid rejection handler to a `then()` call of a rejected promise.


===

Note: [put a bridge here]

Lets see the following illustrations that present the flow of promises resolution or rejection using `then()`, assuming that `p1` is a resolved promise with value `1` and `p2` is a rejected promise with reason `"Error"`.

```
const p1 = Promise.resolve(1);
const p2 = Promise.reject("Error");
```

![ny5qz8z5csi1kniwhkfi.png](https://res.cloudinary.com/practicaldev/image/fetch/s--8RBU2PEG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/ny5qz8z5csi1kniwhkfi.png)

We see that if we don't pass any arguments or if we pass non-function objects as parametes to `then()`, the returned promise keeps the state (`resolved` / `rejected`) and the value of the initial state without throwing any error.

But what happens if we pass a function that does not return anything? The following illustration shows that in such case, the returned promise gets resolved or rejected with the `undefined` value.

![gskdsc6e77mpkfp0l1my.png](https://res.cloudinary.com/practicaldev/image/fetch/s--clgqn6TV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/gskdsc6e77mpkfp0l1my.png)


===


## **Promise.prototype.catch()**

We call `catch()` when we want to handle rejected cases only. `catch()` accepts a rejection handler as a parameter and returns another promise so it can be chained. It is the same as calling `then()`, providing an `undefined` or `null` resolution handler as the first parameter. Lets see the following snippet.

```
const p1 = Promise.reject("Error");

p1.then(undefined, err => {
  console.log(err); // Logs "Error"
});

p1.catch(err => {
  console.log(err); // Logs "Error"
});
```

In the next illustration we can see the way that `catch()` operates. Notice the second flow where we throw an error inside the resolution handler of the `then()` function and **it never gets caught**. That happens because this is an asynchronous operation and this error wouldn't have been caught even if we had executed this flow inside a `try...catch` block.

On the other hand, the last illustration shows the same case, with an additional `catch()` at the end of the flow, that, actually, catches the error.

![a1o496gxd82cfkkalllj.png](https://res.cloudinary.com/practicaldev/image/fetch/s--fV9R2DE8--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/a1o496gxd82cfkkalllj.png)


===


## **Promise.prototype.finally()**
`finally()` can be used when we do not care wheather the promise has been resolved or rejected, just if the promise has been settled. `finally()` accepts a function as its first parameter and returns another promise.

```
const p1 = Promise.resolve(1);

p1.finally(() => {
  console.log("Promise settled"); // Logs "Promise settled"
});
```

The promise which is returned by the `finally()` call is resolved with the resolution value of the initial promise.

```
const p1 = Promise.resolve(1);

p1.finally(() => {
  console.log("Promise settled"); // Logs "Promise settled"
}).then(res => {
  console.log(res); // Logs 1
});
```
### Conclusion

Promises is a wide topic that cannot be fully covered here. I've tried to present some simple illustrations that will help the reader to get an idea of the way that promises work in Javascript.


### References

- *[MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- *[Promises/A+](https://promisesaplus.com/)
- *[developers.google](https://developers.google.com/web/fundamentals/primers/promises)
- *[Kyle Simpson, 2015, You don't know JS: Async & Performance, 29-119](https://www.amazon.com/You-Dont-Know-JS-Performance/dp/1491904224)
