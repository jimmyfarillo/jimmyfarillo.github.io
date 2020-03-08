---
layout: post
title: "Iffy About IIFEs?"
date: 2015-03-03
---

Typically, when writing code, we build out functions and then write out some
other code that calls those functions. Something like this:

```
function myFunc() {
  return "I swear it's not me!";
}

var whatsThatSmell = myFunc();
```

However, there's a cool thing in JavaScript called an IIFE (immediately-invoked
function expression). IIFEs define functions and then run them right after
being created. Like this:

```
(function myFunc2() {
  return "I'm not the one who smells funny!"
}());
```

The parentheses after the function's closing brace immediately calls that
function.

## But why would we want to use an IIFE?

One reason is that an IIFE allows you to create a private scope. Scope is a
term to describe what parts of the code have access to other parts of the code.
So having a private scope for your function means that the data, such as
variables, inside the function are fully contained in the function and not
accessible from other parts of the code. For cases where using an IIFE is
appropriate, not using an IIFE would likely require you to define global
variables. Beautiful code tries to limit superfluous global variables, as
having too many global variables is considered to be "polluting" the namespace
— so don't do that.

You would not want to use an IIFE for functions that you need to run multiple
times. But for code that you just need to run once, and that you want to be
fully contained within a private scope, then an IIFE might be what you need.

A couple more cool things about IIFEs: 1) they don't need to be named and 2)
you can use crazy simple variables inside of them. Like this:

```
(function() {
  var x = 50;
  return x;
}());
```

Defining a variable named `x` in your code is usually a terrible idea because
it is both nondescript and likely to interfere with any other variables named
`x` that may exist in the environment where your code is being executed. But in
an IIFE, that `x` isn't touching anything else and nothing else can touch it.
You still probably wouldn't want to name a variable `x` in your IIFE because it
is extremely nondescript, but it's cool that you could do it without the
possibility of breaking your code.
