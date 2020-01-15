# 6 Ways to Use Promises in Your Javascript Code

![1*HHP2OfDg_jZnfBO0jMOC_g.jpeg](https://miro.medium.com/max/1280/1*HHP2OfDg_jZnfBO0jMOC_g.jpeg)

Before creating this, I didn’t imagine there would be so many different ways to create a Promise.
I imagine that it’s likely that I may have also missed a few.

There’s a lot of redundancy here, but every method is worth going over in detail because of the nuances.




![1*c9ufOYOOGmdQ3JQHdH-Ghg.gif](https://miro.medium.com/max/1839/1*c9ufOYOOGmdQ3JQHdH-Ghg.gif)

### 1. Create a new Promise

This is the standard (or typical) way of creating a Promise. It offers the most flexibility, but it’s also the most verbose.

```
// (1) resolved
new Promise(resolve => {
  resolve(777)
})
// (2) rejected
new Promise((_, reject) => {
  reject(555)
})
// (3) rejected
new Promise(() => {
  throw 555
})
```

This method exposes `resolve` and `reject` functions that can be later called when it’s time to resolve the Promise.

```
new Promise(resolve => {
  setTimeout(() => resolve(777), 1000)
})
```

### 2. Static methods

Promise has a pair of static methods, `resolve` and `reject`, to quickly create a new Promise with either a resolved or rejected value.

```
// (4) resolved
Promise.resolve(777)
// (5) rejected
Promise.reject(555)
```

This is handy when a function needs a Promise as an argument or as a return value.

```
// example
const fetchJson  = (input) => {
  // always return a Promise
  if (!input) return Promise.reject('input cannot be empty')
  /* more code here */
}
```

### 3. Then and Catch

The `then` method on a resolved Promise will return a new Promise.
Likewise, the `catch` method of a rejected Promise will return a new Promise.

```
// (6) resolved -> resolved
Promise.resolve().then(() => 777)
// (7) rejected -> rejected
Promise.reject().catch(() => Promise.reject(555))
// (8) rejected -> rejected
Promise.reject().catch(() => {
  throw 555
})
```

The `then` method on a rejected Promise and the `catch` method on a resolved Promise also return a new Promise. But they don’t execute the callbacks and are equivalent to the original Promise.

```
// (9) stays resolved (777)
Promise.resolve(777).catch(() => 555)
// (10) stays rejected (555)
Promise.reject(555).then(() => 777)
```

Note: While the value stays the same, the method returns a new Promise (and they are *not* equal).

```
// resolved (777)
const p1 = Promise.resolve(777)
// resolved (777)
const p2 = p1.catch(() => {})
p1 == p2 //=> false
```

### 4. Resolved <-> Rejected

A rejected Promise doesn’t have to stay rejected, it can return a resolved Promise and vice-versa.
This is handy when a Promise is rejected, but you want to handle the error and return a new value.
Or if the value returned doesn’t pass additional validation, you can force a rejection.

```
// (11) rejected -> resolved
Promise.reject().catch(() => 777)
// (12) rejected -> resolved
Promise.reject().catch(() => Promise.resolve(777))
// (13) resolved -> rejected
Promise.resolve().then(() => Promise.reject(555))
// (14) resolved -> rejected
Promise.resolve().then(() => {
  throw 555
})
```

### 5. Async / Await

A function marked with `async` always returns a Promise.
A returned value will be wrapped in a Promise.
A Promise can also be returned. The value inside will be unwrapped and then rewrapped in a new Promise object.

```
// (15) function returns resolved (777)
async function func() {
  return 777
}
// (16) arrow returns resolved (777)
const func = async () => {
  return 777
}
// (17) return rejected (555)
const func = async () {
  return Promise.reject(555)
}
// (18) return rejected (555)
const func = async () {
  throw 555
}
```

### 6. Thenables

Any object that has a `then` method can be used in place of a Promise.

```
// (19) resolved (777)
Promise.resolve({ then: resolve => resolve(777) })
// (20) rejected (555)
Promise.resolve({ then: (_, reject) => reject(555) })
// (21) treated as a Promise, resolves to 777
await {
  then(resolve) {
    resolve(777)
 }
}
// (22) treated as a Promise, resolves to 777
await {
  then(_, reject) {
    reject(555)
 }
}
```

The use cases for this is pretty limited and usage is not encouraged, but it’s good to know that it is possible.
 If you know of a valid use case, to use this over a Promise, let me know in the comments.

### Fin

The Promise has a lot of flexibility, offering a handful of ways to be created and transformed.
Becoming familiar with all of these techniques will help you choose the right tool for the right situation.

### Conclusion

[How to Share React UI Components between Projects and Apps](https://blog.bitsrc.io/how-to-easily-share-react-components-between-projects-3dd42149c09)

[5 Tools for Faster Development in React](https://blog.bitsrc.io/5-tools-for-faster-development-in-react-676f134050f2)

[11 JavaScript Utility Libraries you Should Know in 2019](https://blog.bitsrc.io/11-javascript-utility-libraries-you-should-know-in-2018-3646fb31ade)
