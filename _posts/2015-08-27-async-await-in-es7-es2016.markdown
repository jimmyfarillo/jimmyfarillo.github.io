---
layout: post
title: "Async/Await in ES7 (ES2016)"
date: 2015-08-27
---

I've recently been
[learning a bit about Koa]({% post_url 2015-08-13-everything-you-want-to-know-about-koa %})
, which uses ES6 (ES2015) generator functions in its design and allows us to
treat our asynchronous operations differently than using callbacks or promises.

After [I presented about Koa at the NYC Node.js
meetup](http://www.nycnode.com/videos/jimmy-farrell-downstream-and-upstream-request-flow-with-the-koa-web-framework),
fellow presenter [Eddie Zaneski](https://twitter.com/eddiezane) showed me
something even more awesome than generators: `async` / `await`.

Here's an example of `async` / `await` in action:

```
async function doSomeStuff() {
  try {
    let results1 = await returnsAPromise();
    let results2 = await alsoReturnsAPromise();
    return { results1, results2 };
  } catch (err) {
    throw new Error(err);
  }
}
```

Our function is defined as an `async function`, which gives us access to a new
keyword, `await`. This keyword does exactly what it says; it waits for an
asynchronous operation to complete (either resolve or reject) before continuing
on with the rest of the function's code.

The asynchronous operation that is being waited for must return a promise. If
that promise resolves successfully, then the resolved value is returned from
`await`. It the promise is rejected, the execution of the code moves in the
`catch` block.

This code structure may look familiar if you read my post on Koa. That's
because Koa and `async` / `await` are both built using coroutines. However, the
internals of how each implements coroutines is quite different. In short, Koa
uses generator functions and a library called co, while `async` / `await` uses
a library called node-fibers. More on this here.

Back to the code.

Dealing with asynchronicity using `async` / `await` looks a lot like
synchronous code, and that's the whole point. No need for messy callbacks or
promises to handle control flow.

And though the code may look synchronous, `async` / `await` does not block the
JavaScript event loop. This means that other operations can be performed while
our function is awaiting the promise to be resolved/rejected.

Concurrency with `async` / `await` is also quite simple by wrapping the
asynchronous operations that are to be performed in parallel in a
`Promise.all`. Like so:

```
async function manyThings() {
  try {
    let promises = [thing1(), thing2(), thing3()];
    await Promise.all(promises);
  } catch (err) {
    throw new Error(err);
  }
}
```

I've also seen that `await*` is proposed syntactic sugar for `Promise.all`.
Meaning that we could achieve the same thing as above by doing:

```
async function manyThings() {
  try {
    let promises = [thing1(), thing2(), thing3()];
    await* promises;
  } catch (err) {
    throw new Error(err);
  }
}
```

This code is slightly nicer, since we don't have to directly interact with the
Promise API. But, alas, we may or may not get this syntax when the ES7 specs
are finalized. Either way, I'm definitely looking forward to `async` / `await`!
