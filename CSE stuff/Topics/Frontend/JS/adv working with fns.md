
# Recursion

when a function is called, the execution context data sturcture contains all the details about the execution of the function (variables, `this` and other details)
if the fn further calls another function, then the current fns exec context is moved to `execution context stack`

when nested fn finishes, we pop the stack and get our exec context back.

# Rest params and spread syntax

usage
```jsx
function showName(firstName, lastName, ...titles) {
  alert( firstName + ' ' + lastName ); // Julius Caesar

  // the rest go into titles array
  // i.e. titles = ["Consul", "Imperator"]
  alert( titles[0] ); // Consul
  alert( titles[1] ); // Imperator
  alert( titles.length ); // 2
}

showName("Julius", "Caesar", "Consul", "Imperator");
```
rest params should be at the end
```jsx
function f(arg1, ...rest, arg2) { // arg2 after ...rest ?!
  // error
}
```

Arguments:
`arguments`  is what we used to have before. its array like and an iterable. which means it does not have methods of an array like `arguments.map() -> doesnt exist`
it contains all the arguments, we can capture them partially as we've attempted above.

Spread:
```jsx
let arr = [3, 5, 1];

alert( Math.max(...arr) ); // 5 (spread turns array into a list of arguments)


let str = "Hello";
// Array.from converts an iterable into an array
alert( Array.from(str) ); // H,e,l,l,o
```

can copy arr and obj w this
```jsx
let obj = { a: 1, b: 2, c: 3 };

let objCopy = { ...obj }; // spread the object into a list of parameters
                          // then return the result in a new object

// do the objects have the same contents?
alert(JSON.stringify(obj) === JSON.stringify(objCopy)); // true

// are the objects equal?
alert(obj === objCopy); // false (not same reference)

// modifying our initial object does not modify the copy:
obj.d = 4;
alert(JSON.stringify(obj)); // {"a":1,"b":2,"c":3,"d":4}
alert(JSON.stringify(objCopy)); // {"a":1,"b":2,"c":3}
```

```jsx
let arr = [1, 2, 3];

let arrCopy = [...arr]; // spread the array into a list of parameters
                        // then put the result into a new array

// do the arrays have the same contents?
alert(JSON.stringify(arr) === JSON.stringify(arrCopy)); // true

// are the arrays equal?
alert(arr === arrCopy); // false (not same reference)

// modifying our initial array does not modify the copy:
arr.push(4);
alert(arr); // 1, 2, 3, 4
alert(arrCopy); // 1, 2, 3
```

# Variable scope, closure

