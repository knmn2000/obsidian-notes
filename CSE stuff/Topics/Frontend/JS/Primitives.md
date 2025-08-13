# Chapter 1
primitives (strings, numbers etc) are not actually objects, even though we get to use methods with them (just like objects)

There are 7 primitive types: `string`, `number`, `bigint`, `boolean`, `symbol`, `null` and `undefined`.

diff between primitives and objects:

- Objects: capbale of storing multiple values as properties, and can be created with {}. there are other kind of objects like functions
- Primitives: they are a value of a primitive type (mentioned above)

many built in objects exist like dates, errors, html elements etc. they have diff properties and methods

Objects are “heavier” than primitives, hence use more machinery power.

if primitives are not objects, then how do they have methods.

when we try to access a method for a primitive, like str.toUpperCase(), at the point of invoking this method, a new special object is spawned, that has the value of this string, and has all the methods that a string may need, this method is called, the value is returned, and then the function is destroyed.

^^^ this is how primitives still have methods, but remain lightweight.

today, js engines optimise in fancy ways to avoid spawning objects to handle the above stuff

null and undefined dont have any methods associated with them.

---
---

# Chapter 2: Numbers

```javascript
let billion = 1000000000;
```

We also can use underscore `_` as the separator:

```javascript
let billion = 1_000_000_000;
```

```js
let billion = 1e9;  // 1 billion, literally: 1 and 9 zeroes

alert( 7.3e9 );  // 7.3 billions (same as 7300000000 or 7_300_000_000)

let mcs = 1e-6; // five zeroes to the left from 1
// -3 divides by 1 with 3 zeroes
1e-3 === 1 / 1000; // 0.001

// -6 divides by 1 with 6 zeroes
1.23e-6 === 1.23 / 1000000; // 0.00000123

// an example with a bigger number
1234e-2 === 1234 / 100; // 12.34, decimal point moves 2 times

alert( 0xff ); // 255
alert( 0xFF ); // 255 (the same, case doesn't matter)

let a = 0b11111111; // binary form of 255
let b = 0o377; // octal form of 255

alert( a == b ); // true, the same number 255 at both sides
```


### toString(base)

to convert an into to some other base

```js
let num = 255;

alert( num.toString(16) );  // ff
alert( num.toString(2) );   // 11111111

alert( 123456..toString(36) ); // 2n9c


// Please note that two dots in `123456..toString(36)` is not a typo. If we want to call a method directly on a number, like `toString` in the example above, then we need to place two dots `..` after it.


//If we placed a single dot: `123456.toString(36)`, then there would be an error, because JavaScript syntax implies the decimal part after the first dot. And if we place one more dot, then JavaScript knows that the decimal part is empty and now uses the method.

//Also could write `(123456).toString(36)`.
```

### Rounding

take care when rounding negative numbers

`Math.floor`

Rounds down: `3.1` becomes `3`, and `-1.1` becomes `-2`.

`Math.ceil`

Rounds up: `3.1` becomes `4`, and `-1.1` becomes `-1`.


### why isnt 0.1 + 0.2 === 0.3? why 0.30000004?

What is `0.1`? It is one divided by ten `1/10`, one-tenth. In the decimal numeral system, such numbers are easily representable. Compare it to one-third: `1/3`. It becomes an endless fraction `0.33333(3)`.

So, division by powers `10` is guaranteed to work well in the decimal system, but division by `3` is not. For the same reason, in the binary numeral system, the division by powers of `2` is guaranteed to work, but `1/10` becomes an endless binary fraction.

There’s just no way to store _exactly 0.1_ or _exactly 0.2_ using the binary system, just like there is no way to store one-third as a decimal fraction.

The numeric format IEEE-754 solves this by rounding to the nearest possible number. These rounding rules normally don’t allow us to see that “tiny precision loss”, but it exists.

Can we work around the problem? Sure, the most reliable method is to round the result with the help of a method toFixed(n):

```js
let sum = 0.1 + 0.2;
alert( sum.toFixed(2) ); // "0.30"
```

# Parsing

```js
alert( parseInt('100px') ); // 100
alert( parseFloat('12.5em') ); // 12.5

alert( parseInt('12.3') ); // 12, only the integer part is returned
alert( parseFloat('12.3.4') ); // 12.3, the second point stops the reading

alert( parseInt('a123') ); // NaN, the first symbol stops the process
```


---
---
# Chapter 3: Strings

Strings are immutable

```js
let str = 'Hi';

str[0] = 'h'; // error
alert( str[0] ); // doesn't work
```

workaround is to actually create a new string

```js
let str = 'Hi';

str = 'h' + str[1]; // replace the string

alert( str ); // hi
```


note:
```js
alert( 'Österreich' > 'Zealand' ); // true
```
special characters are out of order.
so what we need to do is
```js
alert( 'Österreich'.localeCompare('Zealand') ); // -1
```

---
---

# Chapter 4: Arrays

`arr.at(i)`:
- is exactly the same as `arr[i]`, if `i >= 0`.
- for negative values of `i`, it steps back from the end of the array.

```js
let fruits = ["Apple", "Orange", "Pear"];

alert( fruits.shift() ); // remove Apple and alert it

alert( fruits ); // Orange, Pear

//---
let fruits = ["Apple", "Orange"];

fruits.push("Pear");

alert( fruits ); // Apple, Orange, Pear

//---
let fruits = ["Apple", "Orange", "Pear"];

alert( fruits.pop() ); // remove "Pear" and alert it

alert( fruits ); // Apple, Orange

//===
let fruits = ["Orange", "Pear"];

fruits.unshift('Apple');

alert( fruits ); // Apple, Orange, Pear

//---
let fruits = ["Apple"];

fruits.push("Orange", "Peach");
fruits.unshift("Pineapple", "Lemon");

// ["Pineapple", "Lemon", "Apple", "Orange", "Peach"]
alert( fruits );
```


### Difference between array an objects

 what makes arrays really special is their internal representation. The engine tries to store its elements in the contiguous memory area, one after another, just as depicted on the illustrations in this chapter, and there are other optimizations as well, to make arrays work really fast.

But they all break if we quit working with an array as with an “ordered collection” and start working with it as if it were a regular object.


```js
let fruits = []; // make an array

fruits[99999] = 5; // assign a property with the index far greater than its length

fruits.age = 25; // create a property with an arbitrary name
```

***here we started treating array as an object, so array specific optimizations will be turned off.***

Please think of arrays as special structures to work with the _ordered data_. They provide special methods for that. Arrays are carefully tuned inside JavaScript engines to work with contiguous ordered data, please use them this way. And if you need arbitrary keys, chances are high that you actually require a regular object `{}`.

### Performance: Methods `push/pop` run fast, while `shift/unshift` are slow.


The `shift` operation must do 3 things:

1. Remove the element with the index `0`.
2. Move all elements to the left, renumber them from the index `1` to `0`, from `2` to `1` and so on.
3. Update the `length` property.
4. 
**The more elements in the array, the more time to move them, more in-memory operations.**

The similar thing happens with `unshift`: to add an element to the beginning of the array, we need first to move existing elements to the right, increasing their indexes.

And what’s with `push/pop`? They do not need to move anything. To extract an element from the end, the `pop` method cleans the index and shortens `length`.
**The `pop` method does not need to move anything, because other elements keep their indexes. That’s why it’s blazingly fast.**

---

what if we use `for...in` syntax for iterating over arrays?
we will start iterating over all the objects of the array
thats why we use `for...of` syntax in arrays

---

```js
let arr = [1, 2, 3, 4, 5];

arr.length = 2; // truncate to 2 elements
alert( arr ); // [1, 2]

arr.length = 5; // return length back
alert( arr[3] ); // undefined: the values do not return -> IRRERVERSIBLE
```

---

splice alters the original array
slice does not , it creates a copy

---
---

# Chapter 5: Iterables

if an object is not technically an array, but we want to perform iterations over it, then it would be amazing if we could use the `for...of` syntax

that where we use iterables
they are the generalization of arrays, allowing us to use for...of loop over any object.

```js
let range = {
  from: 1,
  to: 5
};

// We want the for..of to work:
// for(let num of range) ... num=1,2,3,4,5
```

To make the `range` object iterable (and thus let `for..of` work) we need to add a method to the object named `Symbol.iterator` (a special built-in symbol just for that).

1. When `for..of` starts, it calls that method once (or errors if not found). The method must return an _iterator_ – an object with the method `next`.
2. Onward, `for..of` works _only with that returned object_.
3. When `for..of` wants the next value, it calls `next()` on that object.
4. The result of `next()` must have the form `{done: Boolean, value: any}`, where `done=true` means that the loop is finished, otherwise `value` is the next value.

```js
let range = {
  from: 1,
  to: 5
};

// 1. call to for..of initially calls this
range[Symbol.iterator] = function() {

  // ...it returns the iterator object:
  // 2. Onward, for..of works only with the iterator object below, asking it for next values
  return {
    current: this.from,
    last: this.to,

    // 3. next() is called on each iteration by the for..of loop
    next() {
      // 4. it should return the value as an object {done:.., value :...}
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};

// now it works!
for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```
Please note the core feature of iterables: separation of concerns.

- The `range` itself does not have the `next()` method.
- Instead, another object, a so-called “iterator” is created by the call to `range[Symbol.iterator]()`, and its `next()` generates values for the iteration.

so this means that the iterator object is separate from the object itself, so we can do something like

```js
let range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    this.current = this.from;
    return this;
  },

  next() {
    if (this.current <= this.to) {
      return { done: false, value: this.current++ };
    } else {
      return { done: true };
    }
  }
};

for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```

now we've made the iterator part of the object.
drawback is that we cant run 2 iterations in parallel over this object, because both iterations that were earlier separate, are now the same.

---
### Strings are iterable

```js
for (let char of "test") {
  // triggers 4 times: once for each character
  alert( char ); // t, then e, then s, then t
}
```

---

### Calling iterators explicitly 

```js
let str = "Hello";

// does the same as
// for (let char of str) alert(char);

let iterator = str[Symbol.iterator]();

while (true) {
  let result = iterator.next();
  if (result.done) break;
  alert(result.value); // outputs characters one by one
}
```

we have control over calling .next(). more control than the for...of loop

---

### Array.from()

turning array-like things into arrays 

```js
let arr = {
0: 'c',
1: 'a',
2: 'b',
length: 3
}

// this is array like, we can convert to array with 
Array.from(arrayLike);
```


----
----

# Chapter 6: Map and Set

Map vs Object?
Map allows keys of any type

- [`new Map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Map) – creates the map.
- [`map.set(key, value)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/set) – stores the value by the key.
- [`map.get(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/get) – returns the value by the key, `undefined` if `key` doesn’t exist in map.
- [`map.has(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/has) – returns `true` if the `key` exists, `false` otherwise.
- [`map.delete(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/delete) – removes the element (the key/value pair) by the key.
- [`map.clear()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/clear) – removes everything from the map.
 - [`map.size`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/size) – returns the current element count.

```js

let map = new Map();

map.set('1', 'str1');   // a string key
map.set(1, 'num1');     // a numeric key
map.set(true, 'bool1'); // a boolean key

// remember the regular Object? it would convert keys to string
// Map keeps the type, so these two are different:
alert( map.get(1)   ); // 'num1'
alert( map.get('1') ); // 'str1'

alert( map.size ); // 3

```

note
if we do `map[key]` , then the map will be treated like an object and the optimizations etc made for map will go away. like `map[int]` will now be treated as `map[string]` which is not how maps usually work because you can have int and string keys 

### -> we can use objects as keys in Map. cant do so in a standard Object

```js
// we can chain the map method calls
map.set('1', 'str1')
  .set(1, 'num1')
  .set(true, 'bool1');
```

```js
// iterating through map, multiple options

map.keys() – returns an iterable for keys,
map.values() – returns an iterable for values,
map.entries() – returns an iterable for entries [key, value], it’s used by default in for..of.

let recipeMap = new Map([
  ['cucumber', 500],
  ['tomatoes', 350],
  ['onion',    50]
]);

// iterate over keys (vegetables)
for (let vegetable of recipeMap.keys()) {
  alert(vegetable); // cucumber, tomatoes, onion
}

// iterate over values (amounts)
for (let amount of recipeMap.values()) {
  alert(amount); // 500, 350, 50
}

// iterate over [key, value] entries
for (let entry of recipeMap) { // the same as of recipeMap.entries()
  alert(entry); // cucumber,500 (and so on)
}

// ALSO

recipeMap.forEach( (value, key, map) => {
  alert(`${key}: ${value}`); // cucumber: 500 etc
});

let map = new Map([
  ['1',  'str1'],
  [1,    'num1'],
  [true, 'bool1']
]);

let obj = {
  name: "John",
  age: 30
};

let map = new Map(Object.entries(obj));

alert( map.get('name') ); // John
```

### fromEntries
create object from map

```js
let prices = Object.fromEntries([
  ['banana', 1],
  ['orange', 2],
  ['meat', 4]
]);

// now prices = { banana: 1, orange: 2, meat: 4 }

alert(prices.orange); // 2
```

## Set

each key occurs once only
```js
let set = new Set();

let john = { name: "John" };
let pete = { name: "Pete" };
let mary = { name: "Mary" };

// visits, some users come multiple times
set.add(john);
set.add(pete);
set.add(mary);
set.add(john);
set.add(mary);

// set keeps only unique values
alert( set.size ); // 3

for (let user of set) {
  alert(user.name); // John (then Pete and Mary)
}


let set = new Set(["oranges", "apples", "bananas"]);

for (let value of set) alert(value);

// the same with forEach:
set.forEach((value, valueAgain, set) => {
  alert(value);
});

```

```js

Map – is a collection of keyed values.

Methods and properties:

new Map([iterable]) – creates the map, with optional iterable (e.g. array) of [key,value] pairs for initialization.
map.set(key, value) – stores the value by the key, returns the map itself.
map.get(key) – returns the value by the key, undefined if key doesn’t exist in map.
map.has(key) – returns true if the key exists, false otherwise.
map.delete(key) – removes the element by the key, returns true if key existed at the moment of the call, otherwise false.
map.clear() – removes everything from the map.
map.size – returns the current element count.
The differences from a regular Object:

Any keys, objects can be keys.
Additional convenient methods, the size property.
Set – is a collection of unique values.

Methods and properties:

new Set([iterable]) – creates the set, with optional iterable (e.g. array) of values for initialization.
set.add(value) – adds a value (does nothing if value exists), returns the set itself.
set.delete(value) – removes the value, returns true if value existed at the moment of the call, otherwise false.
set.has(value) – returns true if the value exists in the set, otherwise false.
set.clear() – removes everything from the set.
set.size – is the elements count.
Iteration over Map and Set is always in the insertion order, so we can’t say that these collections are unordered, but we can’t reorder elements or directly get an element by its number
```

---
---
# Chapter 7 : WeakMap and WeakSet

in weakmaps, your keys have to be objects.
why are they weak? because when those objects you used as keys die, then the values will be eliminated as well

```js
//MAP

let john = { name: "John" };

let map = new Map();
map.set(john, "...");

john = null; // overwrite the reference

// john is stored inside the map,
// we can get it by using map.keys()


// weak map

let john = { name: "John" };

let weakMap = new WeakMap();
weakMap.set(john, "...");

john = null; // overwrite the reference

// john is removed from memory!
```
can only use objects as keys
```js
weakMap.set("test", "Whoops"); // Error, because "test" is not an object
```

`WeakMap` does not support iteration and methods `keys()`, `values()`, `entries()`, so there’s no way to get all keys or values from it.

`WeakMap` has only the following methods:

- [`weakMap.set(key, value)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/set)
- [`weakMap.get(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/get)
- [`weakMap.delete(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/delete)
- [`weakMap.has(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/has)

why cant we get all the keys and values?
because when an object is cleared up from the memory, and hence from the weakmap, we dont exactly know when the cleanup from the weakmap happens. the JS engine will take care of that, but we dont know exactly when it happens. so if an object key is deleted, we dont know the count of key values in the weakmap

### usages
```js
weakMap.set(john, "secret documents");
// if john dies, secret documents will be destroyed automatically
```

```js

USING MAP
//counting visits
let visitsCountMap = new Map(); // map: user => visits count

// increase the visits count
function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}

// calling the func
let john = { name: "John" };

countUser(john); // count his visits

// later john leaves us
john = null; -> THE THINGS INSIDE MAP ARENT GARBAGE COLLECTED. THEY ARE USING MEMEORY
```

```js
USING WEAKMAP
// 📁 visitsCount.js
let visitsCountMap = new WeakMap(); // weakmap: user => visits count

// increase the visits count
function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}

// calling the func
let john = { name: "John" };

countUser(john); // count his visits

// later john leaves us
john = null; -> NOW GARBAGE COLLECTION WILL BE HANDLED, THE visitsCountMap WONT CONTAIN DEAD DATA
```

### WeakSet

similar to Set, just weak. if object keys are removed, then things are cleaned up


```js

let visitedSet = new WeakSet();

let john = { name: "John" };
let pete = { name: "Pete" };
let mary = { name: "Mary" };

visitedSet.add(john); // John visited us
visitedSet.add(pete); // Then Pete
visitedSet.add(john); // John again

// visitedSet has 2 users now

// check if John visited?
alert(visitedSet.has(john)); // true

// check if Mary visited?
alert(visitedSet.has(mary)); // false

john = null;

// visitedSet will be cleaned automatically
```