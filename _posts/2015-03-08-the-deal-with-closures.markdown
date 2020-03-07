---
layout: post
title: "The Deal With Closures"
date: 2015-03-08
---

Closures are functions that can refer to independent variables that existed
when the function was created. Many coders says that these functions "remember"
the environment and the variables in that environment when the functions were
being defined.

Here's an example of a closure:

```
var outerFunc = function() {
  var yup = "YES! I love it!";
  var innerFunc = function() {
    return yup;
  };
  return innerFunc;
};

var doYouLikePizza = outerFunc();
doYouLikePizza();
// "YES! I love it!"
```

In this example, we define an outer function that contains the variable `yup`
as well as the inner function. Then it returns the inner function. Below that,
we define the variable `doYouLikePizza` to the return of `outerFunc` (this
essentially means that we're defining `doYouLikePizza` to be `innerFunc`). When
we call `doYouLikePizza()`, `innerFunc` remembers the `yup` variable from when
it was created, and `doYouLikePizza()` returns our `yup` string.

Now let's make the example slightly more interesting:

```
var outerFunc = function() {
  var yup = "YES! I love it!";
  var nope = "No. Pizza sucks.";
  var count = 0;
  var innerFunc = function() {
    count++;
    if (count % 2 === 0) return nope;
    return yup;
  };
  return innerFunc;
};

var doYouLikePizza = outerFunc();
doYouLikePizza();
// "YES! I love it!"

doYouLikePizza();
// "No. Pizza sucks."

doYouLikePizza();
// "YES! I love it!"
```

Now, we're still defining `outerFunc` and `innerFunc` and setting
`doYouLikePizza` to the return of `outerFunc`. But inside `outerFunc`, we've
defined a `nope` variable as well as a `count` variable. So when we call
`doYouLikePizza()`, we increment `count` and use it to evaluate which string we
return.

Even though `doYouLikePizza` does not have direct access to the variables
defined inside of `outerFunc`, `innerFunc` (which was assigned to
`doYouLikePizza`) remembers these variables and can refer to them and write
over them.

Pretty nifty!
