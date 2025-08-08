> source: javascript.info

---
# Objects intro

Non primitive because can contain multiple types of [[data structures]]

question -> what can we pass to the object [[constructor]]?
```js
let user = new Object() // what can we pass to the constructor??
let user2 = {};
```

```js
let user = {
"multiple word thing": 123, // this is possible
}

let fruit = "something blah blah, can be calculated at runtime also"
let user = {
[fruit]: 213; // computed property
}
```

we can use reserved keywords for object keys
```js

let obj = {
for: 1,
let: 2,
return: 5,
}

```

`in` keyword
```js
let user = { name: "John", age: 30 };

alert( "age" in user ); // true, user.age exists
alert( "blabla" in user );
```

but why do we need `in` keyword, cant we use obj.someKey === undefined to figure if key is there?
answer: 
```js
let obj = {
  test: undefined
};

alert( obj.test ); // it's undefined, so - no such property?

alert( "test" in obj ); // true, the property does exist!
```

this is possible:
```js
let user = {
  name: "John",
  age: 30,
  isAdmin: true
};

for (let key in user) {
  // keys
  alert( key );  // name, age, isAdmin
  // values for the keys
  alert( user[key] ); // John, 30, true
}
```

--- 
#note 
### OBJECTS ARE SPECIALLY ORDERED 

if keys are ints, they will be sorted.
### 🤔 Why this behavior?

This was specified in the ECMAScript specification to align with how JavaScript engines optimize property access, especially for arrays or array-like objects (where integer keys are common).

- **Arrays** in JS are just objects with integer-like keys.
    
- Sorting integer keys helps with performance and consistency.
Initially, the ECMAScript specification did not define a strict order for property enumeration in objects. However, as JavaScript evolved, **most engines adopted a consistent pattern**:

1. **Integer-like keys**: Enumerated in ascending numeric order.
    
2. **String keys**: Enumerated in the order they were added.
    
3. **Symbol keys**: Enumerated in the order they were added.


```js
let codes = {
  "49": "Germany",
  "41": "Switzerland",
  "44": "Great Britain",
  // ..,
  "1": "USA"
};

for (let code in codes) {
  alert(code); // 1, 41, 44, 49
}
```

SORTING OF THE OBJECT IS DEFFERED UNTIL ENUMERATION. 
THAT MEANS SORTED OBJECTS KA ENUMERATION IS NLOGN TIME

INTEGER KEYS APPEAR FIRST ALWAYS, EVEN IF A SINGLE INTEGER IS PRESENT

if non ints, then they will be fifo
```js
let user = {
  name: "John",
  surname: "Smith"
};
user.age = 25; // add one more

// non-integer properties are listed in the creation order
for (let prop in user) {
  alert( prop ); // name, surname, age
}
```

---
---
# Chapter 2: obj refs and copying

objs are copied by reference

objA === objB only if they are literally the same obj (same address in memory)
```js
let a = {}
let b = a;

a === b is true

let a = {}
let b = {}

a !== b
```


Object.assign exists
```js
let user = { name: "John" };

let permissions1 = { canView: true };
let permissions2 = { canEdit: true };

// copies all properties from permissions1 and permissions2 into user
Object.assign(user, permissions1, permissions2);

// now user = { name: "John", canView: true, canEdit: true }
alert(user.name); // John
alert(user.canView); // true
alert(user.canEdit); // true
```
it overwrites over existing keys.

NOTE: it does not perform a [[deepcopy]]
```js
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = Object.assign({}, user);

alert( user.sizes === clone.sizes ); // true, same object

// user and clone share sizes
user.sizes.width = 60;    // change a property from one place
alert(clone.sizes.width); // 60, get the result from the other one
```

### structuredClone
performs a deepcopy

```js
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = structuredClone(user);

alert( user.sizes === clone.sizes ); // false, different objects

// user and clone are totally unrelated now
user.sizes.width = 60;    // change a property from one place
alert(clone.sizes.width); // 50, not related
```

circular reference is cloned correctly as well
```js
let user = {};
// let's create a circular reference:
// user.me references the user itself
user.me = user;

let clone = structuredClone(user);
alert(clone.me === clone); // true

```

functional properties fail

```js
// error
structuredClone({
  f: function() {}
});

```

---
---
# Chapter 3: Garbage collection

happens automatically.

[[Reachability]]

Roots -> things that cannot be deleted for obvious reasons like
- currently executing fn and its context
- chain of nested calls and their contexts
- global vars
^ these are inherently reachable always

```js
// user has a reference to the object
let user = {
  name: "John"
};

user = null; 
// now the object that was created is garbage'd

BUT

// user has a reference to the object
let user = {
  name: "John"
};

let admin = user;

user = null;

// the object wont be garbage'd because admin still exists
```

#note outgoing references do not matter in reachability.
if the root is deleted, then no matter how big the tree is, all of it is garbage'd

the algo is called mark-and-sweep
```
The following “garbage collection” steps are regularly performed:

- The garbage collector takes roots and “marks” (remembers) them.
- Then it visits and “marks” all references from them.
- Then it visits marked objects and marks _their_ references. All visited objects are remembered, so as not to visit the same object twice in the future.
- …And so on until every reachable (from the roots) references are visited.
- All objects except marked ones are removed.
```

optimisations to garbage collection:

- objects are broken into new and old. usually objects are born, do their job and die. all objects are considered as ***new*** , but if the object survives long enough, then it comes under the **old** category, and is examined less often
- there could be a huge graph to traverse for the algo, so we do it in smaller parts. and parts are cleared incrementally. so we have many little garbage collections. this introduces many tiny delays instead of single big one.
- algo is run during CPU IDLE time

--- 
---

# Chapter 4: Obj methods and "this"

#todo PRACTICE THE QUESTIONS: 

A function that is a property of an object is called its _method_.

methods use the `this` keyword to access properties of the object

^ notice that all of these terms are very OOP like

```js
let user = {
  name: "John",
  age: 30,

  sayHi() {
    // "this" is the "current object"
    alert(this.name);
  }

};

user.sayHi(); // John
```


value of `this` is eval'd during runtime depending on context.
the concept of `this` is different in other langs.

```js
let user = { name: "John" };
let admin = { name: "Admin" };

function sayHi() {
  alert( this.name );
}

// use the same function in two objects
user.f = sayHi;
admin.f = sayHi;

// these calls have different this
// "this" inside the function is the object "before the dot"
user.f(); // John  (this == user)
admin.f(); // Admin  (this == admin)

admin['f'](); // Admin (dot or square brackets access the method – doesn't matter)
```

`this` is undefined outside of an object.

arrow fns have no `this`

---
---

# Chapter 5 : constructor and "new" operator

constructor functions are regular functions, 2 conventions
- start with capital letter
- executed with the `new` operator

```js
function User(name) {
// this = {} -> IMPLICIT
  this.name = name;
  this.isAdmin = false;
  // NOTICE THAT THERE IS NO RETURN. BUT WE ARE ACTUALLY IMPLICITY RETURNING THIS
  // return this; -> implicitly happens
}

let user = new User("Jack");

alert(user.name); // Jack
alert(user.isAdmin); // false
```

When a function is executed with `new`, it does the following steps:

1. A new empty object is created and assigned to `this`.
2. The function body executes. Usually it modifies `this`, adds new properties to it.
3. The value of `this` is returned.

---
---
# Chapter 6: optional chaining '?.'

```js
let user = null;

alert( user?.address ); // undefined
alert( user?.address.street ); // undefined
```

```js
let user = null;
let x = 0;

user?.sayHi(x++); // no "user", so the execution doesn't reach sayHi call and x++

alert(x); // 0, value not incremented
```

```js
let userAdmin = {
  admin() {
    alert("I am admin");
  }
};

let userGuest = {};

userAdmin.admin?.(); // I am admin

userGuest.admin?.(); // nothing happens (no such method)
```

```js
delete user?.name; // delete user.name if user exists
```

