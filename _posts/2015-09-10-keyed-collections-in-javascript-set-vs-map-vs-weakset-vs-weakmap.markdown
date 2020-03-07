---
layout: post
title: "Keyed Collections in JavaScript: Set vs Map vs WeakSet vs WeakMap"
date: 2015-09-10
---

In celebration of the [release of Node.js
4.0.0](https://nodejs.org/en/blog/release/v4.0.0/) this week, I'm going to
cover some more ES2015 features that we now have access to in Node.js by
default.

Until ES2015, JavaScript had only one type of collection for data: `Array`.
Arrays are collections that are indexed using numeric values. But now we have
four new types of collections: `Set`, `Map`, `WeakSet`, and `WeakMap`. Instead
of being indexed like arrays, these new collections are keyed.

This article will give an overview of each new collection type so that you can
begin using them in your code.

## Set

Sets are collections of values, either primitive or object references. These
values must be unique and are added using the `add` method. The `size` property
returns the number of values in the set. Here are a couple examples of creating
sets:

```
var set1 = new Set();
set1;
// ==> Set { }

set1.add('a');
// ==> Set { 'a' }

set1.add(1);
// ==> Set { 'a', 1 }

set1.add('1');
// ==> Set { 'a', 1, '1' }

// The value 1 has already been added.
set1.add(1);
// ==> Set { 'a', 1, '1' }

// Values can be object references.
var obj1 = { prop1: 'Hello!' };
set1.add(obj1);
// ==> Set { 'a', 1, '1', { prop1: 'Hello!' } }

// Sets can also be created using iterables, such as arrays.
var set2 = new Set(['a', 'b', 'c', 'c']);
set2;
// ==> Set { 'a', 'b', 'c' }

set2.size;
// ==> 3
```

There are many other useful methods, such as `delete`, `clear`, and `has`. They
function as you might expect:

```
var set3 = new Set([1, 2, 3, 4, 5]);
set3;
// ==> Set { 1, 2, 3, 4, 5 }

set3.has(2);
// ==> true

set3.delete(2);
set3;
// ==> Set { 1, 3, 4, 5 }

set3.has(2);
// ==> false

set3.clear();
set3;
// ==> Set { }
```

A cool thing about sets is that when iterating over them, the values are
iterated over in the order in which they were inserted. You can iterate over
them using the `forEach` method on `Set.prototype`:

```
var set4 = new Set();
set4.add(5);
set4.add(2);
set4.add(9);
set4.add(1);

// The first two params in the callback are the value.
// This is for consistency with the Array forEach method.

set4.forEach((val1, val2, set) => { console.log(val1 + 10); });
// ==> 15
// ==> 12
// ==> 19
// ==> 11
```

## Map

Maps are another new collection type in JavaScript. The data is stored as
key-value pairs that can be added using the `set` method. Keys and values can
both be of any type, primitive or object. The `size` property returns the
number of key-value pairs in the map.

Like sets, the keys are iterated over in the order in which they were inserted.
This is probably the biggest advantage of storing data in a map as opposed to a
JavaScript `Object` object.

Creating a map:

```
var map1 = new Map();
map1.set(1, 'hello');
// ==> Map { 1 => 'hello' }

map1.set('jimmy', {x: 'y'})
// ==> Map { 1 => 'hello', 'jimmy' => { lastName: 'farrell' } }

var arr1 = ['a', 'b'];
map1.set(arr1, [1, 2]);
// ==> Map { 1 => 'hello', 'jimmy' => { x: 'y' }, [ 'a', 'b' ] => [ 1, 2 ] }

// You can also pass an iterable to the constructor.
var map2 = new Map([['hello', 'world'], [1, 'one']]);
map2;
// ==> Map { 'hello' => 'world', 1 => 'one' }

map2.size;
// ==> 2
```

Maps have the same `delete`, `clear`, and `has` methods, but they also have a
`get` method for accessing the value of a specific key. You can use these
methods like such:

```
var map3 = new Map();
map3.set(1, 'a');
map3.set(2, 'b');
map3.set(3, 'c');
map3;
// ==> Map { 1 => 'a', 2 => 'b', 3 => 'c' }

map3.has(3);
// == true

map3.delete(3);
map3;
// ==> Map { 1 => 'a', 2 => 'b' }

map3.has(3);
// ==> false

map3.get(2);
// ==> 'b'

map3.clear();
map3;
// ==> Map { }
```

Maps also have a `forEach` method, similar to the one for sets.

```
var map4 = new Map();
map4.set('hello', 'world');
map4.set(10, 5);

// First param is the value
// Second param is the key

map4.forEach((val, key, map) => { console.log(key + val); });
// ==> 'helloworld'
// ==> 15
```

## WeakSet and WeakMap

As you can tell by their names, weakSets and weakMaps are similar to sets and
maps, but with some key differences.

Values in a weakSet can only be object references. Keys in a weakMap can only
be object references.

The values in weakSets and keys in weakMaps are weakly held object references.
This means that if that reference is the only reference to that object, the
object can be garbage collected and that entry (value in a weakSet, key-value
pair in a weakMap) will be removed from the collection.

Because they are comprised of weakly held object references, weakSets and
weakMaps cannot be iterated over and do not have a `forEach` method. The `clear`
method is also not available for weakSets and weakMaps.
