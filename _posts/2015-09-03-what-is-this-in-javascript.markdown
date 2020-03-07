---
layout: post
title: "What Is 'this' in JavaScript?"
date: 2015-09-03
---

Today I had a phone interview for a software engineering position. I felt
fairly prepared for the interview, simply because I've been
living/breathing/eating/sweating/crying code for 6 months straight. But a very
simple JavaScript question tripped me up a little bit.

_"What is `this` in JavaScript?"_

Now, I know a lot about JavaScript, and I've thought about `this` very often in
the past 6 months. But this very general, very broad question got me a little
frazzled. I didn't know where to start my answer because I had never tried to
answer this question before. And it really does cover a lot of things.

My approach, and I don't think it was a _super_ terrible one, was to explain as
much information about `this` as I could muster. I won't go into specifics, but
I mentioned the methods `call`, `apply`, and `bind` as well as gave a very
indirect and haphazard explanation of the call site. Not the best response, but
still a way to show that I know a lot about `this` even if I couldn't answer
the question succinctly and elegantly.

However, if I had a chance to respond to that question again, this is what I
would say:

> `this` is a binding that is defined in a function's scope at the time that
> the function is called. It's value is an object that is determined by the
> call site of the function. There are four rules that govern the value of
> `this`. In increasing precedence, these rules are: default binding, implicit
> binding, explicit binding, and `new` binding.

From there, the interviewer might ask for clarification on what the "call site"
of a function refers to. And this would be an appropriate response:

> The call site of a function is the location in which the function is invoked.
> It is easiest to think about the call site by referencing the call stack when
> the function is invoked. For example, if function `a` calls function `b`,
> then the call stack when `b` is invoked is simply `a`. So, to determine the
> value of this during this particular invocation of function `b`, we must look
> at where function `b` is invoked inside of function `a`.

Then, the interviewer might ask about the four rules mentioned previously, and
an answer could be:

1. **Default binding** is the rule with least precedence. If none of the other
   three rules apply, then the value of `this` will be the global object, which
   varies depending on the runtime environment. In the browser, the global
   object is usually `window`. In Node.js, it is `global`.
2. **Implicit binding** refers to the context of the function call. For
   example, if the function is accessed as a method of an object, such as
   `obj.func`, then this inside `func` will refer to `obj`.
3. **Explicit binding** takes precedence over the previous two rules and
   requires use of either the `call`, `apply`, or `bind` function methods.
4. **`new` binding** is set when the `new` keyword is used in a function call.
   The newly constructed object that is created by the `new` keyword will be
   the value of `this`.

In my opinion, this would've been a home run answer to this very common
JavaScript interview question. Don't fumble around and give a correct, but
incomplete, response like I did. And if you're interested in diving a little
bit deeper into `this`, I highly recommend the [_You Don't Know
JS_](https://github.com/getify/You-Dont-Know-JS/) book series by Kyle Simpson.
He has the entire series available for free on GitHub, and they are fantastic
books!
