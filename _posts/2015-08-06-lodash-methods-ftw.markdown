---
layout: post
title: "Lodash Methods FTW!"
date: 2015-08-06
---

I only recently realized the power of the [Lodash](https://lodash.com/)
library. Previously, I had only used the `_.extend` method (also called
`_.assign`) which can be used as follows:

```
var object1 = {
  prop1: 'hello',
  prop2: 'bye'
};

var object2 = {
  prop1: 'hey',
  prop3: 'what is up?'
};

_.extend(object1, object2);console.log(object1);
// ==> { prop1: 'hey', prop2: 'bye', prop3: 'what is up?' }
```

Essentially, `_.extend` takes an object as its first parameter, another object
as its second parameter, and adds the properties from the second object to the
first object as well as overwriting the properties on the first object if those
properties also exist on the second object.

My primary use case for `_.extend` is in PUT routes in my API back-end. That
data flow looks like this:

- User updates document using a form on the front-end
- Front-end code sends the updated document to the server using an AJAX PUT
  request
- PUT route on server queries the database for the document (this is the old
  version)
- PUT route on server uses `_.extend` to update the old version of the document
- PUT route on server saves the newly updated document and sends it back to the
  front-end

## `_.findIndex` & `_.findWhere`

This week I discovered `_.findIndex` and `_.findWhere`, and they are nice
additions to JavaScript's built-in array methods such as `.filter`, `.forEach`,
`.map`, and `.reduce`. This is how you can use `_.findIndex`:

```
var foods = [
  { id: 1, food: 'pizza' },
  { id: 2, food: 'ice cream' },
  { id: 3, food: 'ramen' },
  { id: 4, food: 'veggie burger' }
];

var ramenIndex = _.findIndex(arr, (item) => item.food === 'ramen');
console.log(ramenIndex);
// ==> 2
```

`_.findIndex` returns the index of the first element in the array that returns
`true` in its callback function. Clutch!

The second Lodash method I discovered this week is `_.findWhere`. I am a big
fan of the `.filter` in JavaScript. It is awesome because you can do this:

```
var animals = [
  { name: 'Carl'', type: 'bunny' },
  { name: 'Bob', type: 'bunny' },
  { name: 'Sue', type: 'cat' },
  { name: 'Sally', type: 'bunny' }
];

var bunnies = animals.filter((animal) => animal.type === 'bunny');
console.log(bunnies);
// ==> [
// { name: 'Carl', type: 'bunny' },
// { name: 'Bob', type: 'bunny' },
// { name: 'Sally', type: 'bunny' }
// ]
```

But, when you only expect/want one element out of that array, you would have to
do this:

```
var animals = [
  { name: 'Carl'', type: 'bunny' },
  { name: 'Bob', type: 'bunny' },
  { name: 'Sue', type: 'cat' },
  { name: 'Sally', type: 'bunny' }
];

var sue = animals.filter((animal) => animal.type === 'cat')[0];
console.log(sue);
// ==> { name: 'Sue', type: 'cat' }
```

Getting an array of only one element is not nice. That's why `_.findWhere` is
so awesome. It allows you to do this:

```
var animals = [
  { name: 'Carl'', type: 'bunny' },
  { name: 'Bob', type: 'bunny' },
  { name: 'Sue', type: 'cat' },
  { name: 'Sally', type: 'bunny' }
];

var sue = _.findWhere(animals, { type: 'cat' });
console.log(sue);
// ==> { name: 'Sue', type: 'cat' }
```

Same result, but much cleaner code. It searches the array and returns the first
element where all the properties match the object you've passed as the second
parameter. And that is why I love `_.findWhere`.

I will definitely be looking into utilizing more Lodash methods in my code, now
that I'm finally understanding how they can simplify my code and make it more
semantic. And I definitely suggest that you check out the
[Lodash](https://lodash.com/) library; because why write things that have
already been written for you?
