# Functional Programming

In _funtional programming_, solutions to problems are based on isolated fns, without producing side effects outside their scope

Functional programming employs:

1. Isolation: Functions are isolated and are independent of state of rest of the program/ global variables
2. Pure functions: Give same output for same input, everytime
3. No side effects: changes, or mutations, to the state of the program outside the function are carefully controlled

___

- In js, `functions` that can be assigned to a `variable`, _passed to or returned_ from _another function_ are called __First class functions__
All js functions are first class functions
- Functions that _accept function args or return function values_ are called `Higher Order Functions`
- These function args or return values can be called `lambda`

___

## Principles, best practices

Follow these when writing functions

- Never mutate variables or objects directly
    Mutate copy of original if needed
- Always declare dependencies explicitly
    By explicitly accepting needed params in fn definition
-

___

## Implementation of FP

1. `Array.prototype.map`

    `map` is a pure function, and its output depends solely on its inputs. Plus, it takes another function as its argument

    ```js
    Array.prototype.myMap = function(callback) {
    
        const newArray = [];
        this.forEach( (el, i, arr) => newArray.push(callback(el, i, arr)) )

        return newArray;
    };

    [1,2,3].myMap()
    ```

2. `Array.prototype.filter`

    `filter` accepts a callback and returns an array of elements for which the callback evaluated to true

    ```js
    Array.prototype.myFilter = function(callback) {
        const newArray = [];
        this.forEach((e, i, arr) => {
            let res = callback(e, i, arr)
            if(Boolean(res)) newArray.push(e)
        })

        return newArray;
    };
    ```

3. `Array.prototype.reduce`

    `reduce` iterates over Array and returns one value\
    This value is the accumulation of result of evaluation of callback fn on every element

    In addition to the current element, index and original array, reduce callback takes a 4th param `accumulator`, and also initial value as second param

    `const sumOfAges = users.reduce((sum, user) => sum + user.age, 0);`

4. `slice`

    `slice` take two args, start index and end index, and returns copy of array elements from start index to end(non inclusive) index
    without mutating original array

    `slice` is preferred over `splice` as it will not mutate original array

5. `concat`

    `concat` is used to concatenate multiple arrays together. The array to concatenate is passed as arg, resultant array is returned without mutating original

    Use `concat` instead of `push` as push will mutate the array

6. `sort`

    `sort` accepts a callback function, usually called a compare function
    - If the compare fn evaluates to -1 for two elements _a and b_, a comes before b
    - If the compare fn evaluates to 0 for two elements _a and b_, both stay enchanged
    - If the compare fn evaluates to 1 for two elements _a and b_, b comes before a

    ```js
    function alphabeticalOrder(arr) {
        arr.sort((a, b) => {
            return a === b ? 0 : a < b ? -1 : 1;
        })

        return arr
    }

    alphabeticalOrder(["a", "d", "c", "a", "z", "g"]);
    ```

    One problem with _sort_ is that it _`mutates` original array_\
    Copy the original array before sorting to avoid mutation

    ```diff
    function alphabeticalOrder(arr) {
    -   arr.sort((a, b) => {
    -       ...
    -   }
    +   return [...arr].sort((a, b) => a === b ? 0 : a < b ? -1 : 1)
    }
    ```

7. `split`

    `split` function takes in a `string` or `regex` argument and uses this as a _delimiter to split string characters_ into array

    If space is used as a delimiter, we will get an array of words

    Because __strings are immutable__, using `split` makes it easy to work with strings

    ```js
    function splitify(str) {
        return str.split(/\W/)
    }
    // splits the string at all non word characters
    splitify("Hello World,I-am code");
    ```

8. `join`

    `join` will combine all aray elements into a single string\
    Accepts a delimiter arg to separate array elements in the string

    ```js
    function sentensify(str) {
        return str.split(/\W/).join(" ")
    }

    sentensify("May-the-force-be-with-you");
    ```

    > The `split` and `join` methods are often used together for powerful string manipulation\
ex. to generate URL slugs based on page title:\
`return title.toLowerCase().split(" ").filter(e => e).join("-")`

9. `every`

    returns `true` if _**every element** in array passes a test_,
    else returns `false`

10. `some`

    returns true if _**any element** passes a test_,
    else returns false

___

## Currying

The `arity` of a function is the _number of arguments it requires_\
_`Currying` a function_ means to __convert a _function of N arity_ into _N functions of arity 1_ .__\
i.e. it _restructures_ a function so it takes _one argument_\
then _`returns` **another function**_ that takes the _next argument_, and so on

```js
// currying `add(x, y, z)` with normal syntax
function add(x) {
  return function(y){
    return function(z){
      return x+y+z
    }
  }
}

add(10)(20)(30);

// currying with arrow function syntax
const added = x => y => z => x+y+z

console.log(added(2)(3)(2))
```

When _all function args can't be supplied_ at the same time,\
`currying` is useful to _store each function call_ in a __variable__.\
This way the __variable__ _holds reference to the next fn call_ which takes the _next arg when available_

Example: In the above code, if the second arg to add was not available when `add()` was called,
then we can just call add with first arg, and store the returned fn reference in a variable.
When the second arg is available, we can call this variable with it

```js
// the complete fn call is added(x)(y)(z)
// here we only have `x` available

const waitForY = added(10)
...
// now `y` is available but not `z`
const waitForZ = waitForY(20)
...
// finally `z` is available
let result = waitForZ(30)
```

## Partial Application

It is defined as

1. __calling a fn with some args__,\
see line 7 in below snippet
2. and return _another fn_ which can be __called with more args__\
see line 8

> `Function.prototype.bind` is used to borrow a method from another existing object.\
__`this` value__ _of the new returned function_ is set to _1st arg_ provided to bind\
any _further args provided to bind_, will be _prepended to the __new function args__ when it is **called**_

```js
1| function add(a, b, c) {
2|     return a + b + c
3| }
4| 
5| // use bind to set `this` value of newAdd = to `add` object
6| // the optional args will be used when calling the newAdd, we only need to provide partial/remaining args
7| const newAdd = add.bind(this, 1, 2)
8| newAdd(3)
9| // this will result in add(1, 2, 3) .... i.e. return value will be 5
```

___
