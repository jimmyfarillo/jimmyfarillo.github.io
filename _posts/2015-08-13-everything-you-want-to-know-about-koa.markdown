---
layout: post
title: "Everything You Want to Know About Koa"
date: 2015-08-13
---

_Update (8/20/15): I recently [gave a talk about
Koa](http://www.nycnode.com/videos/jimmy-farrell-downstream-and-upstream-request-flow-with-the-koa-web-framework)
at the NYC Node.js meetup. The [GitHub
repository](https://github.com/jimmyfarrell/intro-to-koa) for that talk
provides some live code examples of Koa._

## The Basics

If you're a [Node.js](https://nodejs.org/) developer, you most likely use
[Express](http://expressjs.com/) as your web application framework. I believe
it has become the default technology. But that doesn't mean it is the only
option you have. In fact, the team behind Express also created another web
application framework called [Koa](http://koajs.com/). This post will serve as
a basic introduction so that you can begin using Koa for your next web app.

The most unique aspect of Koa, especially if you're coming from Express, is
that Koa leverages ES6 generator functions in its architecture. This blog post
will not go in-depth on generators, so if you are unfamiliar with generators,
you can learn more
[here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
or [here](http://davidwalsh.name/es6-generators).

In Express you would define middleware like this:

```
var express = require('express');
var app = express();

app.use(function(req, res, next) {
  console.log('Do something');
  next();
});

app.listen(8000);
```

In Koa, that same middleware would be defined like this:

```
var koa = require('koa');
var app = koa();

app.use(function*(next) {
  console.log('Do something');
  yield next;
});

app.listen(8000);
```

These trivial middleware functions both do the same thing: print "Do something"
and then allow the request to continue downstream along to the next piece in
the pipeline.

## Bidirectional Request Flow

Fairly similar so far. But Express is unidirectional, whereas Koa allows for
downstream **AND** upstream request flow. Here's an example of downstream and
upstream flow:

```
var koa = require('koa');
var app = koa();

app.use(function*(next) {
  console.log('A');
  yield next;
  console.log('B');
});

app.use(function*(next) {
  console.log('C');
  yield next;
  console.log('D');
});

app.use(function*(next) {
  console.log('E');
  yield next;
});

app.listen(8000);
```

You might've guessed, but the order that these letters prints out is: A, C, E,
D, then B. The request flows through the first function where it prints A, then
it yields to the next function in the middleware pipeline, that function prints
C and then yields to the next function, that function prints E then yields to
the next function (which in this case, since there is no more middleware, is an
internal Koa generator). Since the request reached the "bottom" of the pipeline
and the previous two generators have not completed, the request begins flowing
upstream. The second generator function resumes where it left off and then
prints D, then the request continues flowing upstream, and the first generator
function resumes where it left off and prints B.

This can be a bit trippy to understand at first. The [Koa
Guide](https://github.com/koajs/koa/blob/master/docs/guide.md) calls the code
before the `yield next` the "capture phase" and the code after the `yield next`
the "bubble phase". I just call it awesome.

It might be difficult to think of times when you would want to utilize this
upstream request flow, but there are use cases. With upstream request flow, you
can attach properties to the server response that may be dependent on other
downstream middleware. But really, it's more of a design choice than a feature
that expands your server's functionality in a mind-blowing manner.

## Asynchronous Operations

In my opinion, the coolest thing about Koa is how you are able to handle async
code. With Node.js, we are familiar with callbacks and promises. Let's show a
simple GET route that makes a database query.

First with a callback using Express:

```
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  User.find({}, function(err, users) {
    res.json(users);
  });
});

app.listen(8000);
```

Now with a promise in Express:

```
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  User.find()
    .then(function(users) {
      res.json(users);
    });
});

app.listen(8000);
```

But with Koa, you can forget about callbacks and promises, because there is an
even simpler way to handle async operations using generators.

```
var koa = require('koa');
var app = koa();
var router = require('koa-router');

router.get('/', function*() {
  var users = yield User.find();
  this.body = users;
});

app.use(router.routes());app.listen(8000);
```

A few things are happening in this small block of code, so let's go through it.
The first 2 lines are requiring in Koa and creating an `app` instance. The
third line is requiring in a third-party router that can be used with Koa since
Koa does not come with its own router. We use the router by passing it to our
`app`, like in the second to last line of this code block. `router.routes()`
returns a generator function that Koa can use. This router allows us to create
routes like our `router.get`.

The `router.get` defines the URL for this route as well as a generator function
to be used. The first line of this generator is declaring a variable `users`
and then yielding to the database query `User.find()`. Since this is an
asynchronous operation, this generator will pause and wait for the database
query to finish. When the database query finishes, the generator resumes and
the result of the query is stored in the `users` variable.

On the last line of the generator, we set `this.body` to our `users` variable.
In Koa, `this` is the Koa context, which is essentially a wrapper for the
Node.js request and response. By setting `this.body`, we are setting the
server's response body that will get sent back to the client.

The last line simply starts our server and has it listening on port 8000.

As you can see, the `yield` keyword lets us write extremely readable code that
appears synchronous. No need for annoying callbacks or mysterious promises
(TBH, I LOVE promises!). Just super simple code.

## Error Handling

Okay, but there is no error handling in that code. True, let's talk about that.

Error handling in Koa is pretty straightforward. Again, I'm going to compare
with callbacks and promises in Express.

In Express, you define error handling middleware by creating a middleware with
a specific signature, namely, a function with 4 parameters.

Error handling with callbacks in Express:

```
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  User.find({}, function(err, users) {
    if (err) next(err);
    res.json(users);
  });
});

app.use(function(err, req, res, next) {
  res.status = err.status || 500;
  res.send(err.message || 'Internal server error.');
});

app.listen(8000);
```

Error handling with promises in Express:

```
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  User.find()
    .then(function(users) {
      res.json(users);
    })
    .then(null, next);
});

app.use(function(err, req, res, next) {
  res.status = err.status || 500;
  res.send(err.message || 'Internal server error.');
});

app.listen(8000);
```

In Koa, you also define error handling middleware at the top of the pipeline
and give it a signature of:

```
try {
  yield next;
}
```

Here is an example of error handling in Koa:

```
var koa = require('koa');
var app = koa();
var router = require('koa-router');

app.use(function*(next) {
  try {
    yield next;
  } catch (err) {
    this.status = err.status || 500;
    this.body = err.message || 'Internal server error';
    this.app.emit('error', err, this);
  }
});

router.get('/', function*() {
  try {
    var users = yield User.find();
    this.body = users;
  } catch (err) {
    this.throw(err, 500);
  }
});

app.use(router.routes());app.listen(8000);
```

As our first middleware function, we define a generator that tries to `yield
next` and catches an error if there is one. Koa will use this middleware if we
throw an error somewhere in our code, like we are doing in our GET route.

In the GET route, we are attempting to retrieve the users from the database and
set them to `this.body`. If an error occurs, we move into the `catch` block
where we throw an error using the Koa context's `throw` method. This causes the
request to flow to our error handling middleware, where set the response status
and body as well as emit an error to our Koa app. The `this.app.emit` line is
recommended by the Koa creators to allow your Koa app's default error handling
behavior to execute.

If you made it this far, you're definitely ready to start implementing your own
back-end using Koa. May the generators guide you well!
