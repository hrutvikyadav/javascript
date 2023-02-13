# OOP

## Create new instances of a Class

- With `new ClassName()` syntax
  > has some disadvantages
- Better approach is to use `Object.create(ClassName.prototype)`
  This creates a new object without above disadvantages and sets the given param as its prototype

## Constructor property

- Is a _special property_ of objects that _references the `constructor` which `instantiated` the object_
- ex

    ```js
    mustangGT.constructor === Mustang
    // mustangGT is object of Class Mustang
    ```

## Own property

- check if an _instance_ has certain `own property` with `.hasOwnProperty()`
- these are the _props `defined` on the `constructor`_
- ex. `carObj.hasOwnProperty(wheels)`

## Prototype property

- when multiple objects have duplicate varibles, use `Class.prototype` to declare those `prototype properties`
- _properties in prototypes_ are __shared among all instances__ of that `class`
- `prototype` is part of the constructor, usually all objects in `js` have _prototype prop which is part of that objects' constructor_
- ex.

    ```js
    // declare a prop for doors for a class Mustang
    // because all models of Mustang have 2 doors, i.e. duplicate values
    Mustang.prototype.numDoors = 2
    ```

> Instead of adding properties one by one like `Mustang.prototype.numDoors = 2; Mustang.prototype.anotherProp = "xyz";`\
 we can simply assign an object containing all prototype properties to `Mustang.prototype` as in:
>
>    ```js
>    Mustang.prototype = {
>        contructor: Mustang,
>        numDoors: 2,
>        anotherProp: "xyz",
>        abc: { a: 1, b: "abc2", lm: ["mnf", 3, 10.1] }
>    }
>    ```
>
> One _side effect_ to keep in mind when _assigning a prototype object_ like this is that __constructor property is overriden__ and behaves incorrectly\
the __solution is to define _`constructor` property_ explicitly__ as done above

- _Prototype is **inherited**_ from `constructor` function of the respective `object`
  It is possible to verify this relationship with `Mustang.prototype.isPrototypeOf(mustangGT);`
  This confirms if `mustangGT` object inherits prototype from `Mustang` class

- `prototype` is itself an `object`[^protobj], so any `prototype` can have it's own `prototype` too
- Here, `Object.prototype` is prototype of `Mustang.prototype`[^verify]
- this is useful because due to this __prototype chain__, _methods defined in `Object.prototype`_ can be _used by all objects instantiated with Object constructor_, and other _objects instantiated further_ down. An example is `hasOwnProperty()` mehtod, __defined in Object.prototype, and accessible to all other objects__

## Inheritance

1. create instance of Parent class, which the child will inherit from
2. set the prototype of Child class to be instance created in above step

    ```js
    Bird.prototype = Object.create(Animal.prototype)
    ```

    > Here `Bird` inherites `Animals` prototype along with it's constructor, due to this, `.constructor` method behaves incorrectly\
    > Set the `constructor` _`prototype` property_ explicitly
    >
    > ```diff
    > + Bird.prototype.constructor = Bird;
    > ```

3. Add other props as needed

    ```diff
    + Bird.prototype.fly = function() { console.log("fly") };
    ```

4. Override props inheritted from __parent class__\
   _reassign new value_ to inheritted property on _child class_ 

   ```js
   Penguin.prototype.fly = function() { console.log("cant fly!")}
   ```

## Mixins

For sharing behaviour between unrelated classes, use `Mixins` over `Inherittance`
`Mixins` take an `object` and _add a property to object_

```js
let flyMixin = (object) => {
  object.fly = () => console.log("I can fly now!")
}
```

## Closure

Use `Closure` to _**avoid modification** of object properties from outside_
Declare a _prop/variable inside constructor_ and _define a method_ to _access/modify_ that value\
This will change the scope of the variable and make it private

```diff
function Bird() {
- this.weight = 15;
+ let weight = 15;

  this.getWeight = () => weight
}
```

## Immediately Invoked Function Expression (IIFE)

```js
(function () {
  console.log("Chirp, chirp!");
})();
```

Above anonymous function will execute right away and log "Chirp, chirp!"\
This way, _we dont need to store the fn in a variable OR name it, in order __to call it later__._\
Instead it is _Invoked immediately because of the `()` at the end_

> `IIFE` is often used to group related functionality together into a single object as a `module`

## Modules

Combine multiple related functions together by creating an object with _function names as `keys`_ and _definitions as `values`_\
Then return this object using an `IIFE`

```js
let motionModule = (function () {
  return {
    glideMixin: function(obj) {
      obj.glide = function() {
        console.log("Gliding on the water");
      };
    },
    flyMixin: function(obj) {
      obj.fly = function() {
        console.log("Flying, wooosh!");
      };
    }
  }
})();
```

Here, the object returned by the IIFE contains all the related functionality as properties
> Advantage of the `module pattern` is that _all related behaviors can be packaged into a single object_ that can then be _used by other parts of your code_

This can be used now as below

```js
motionModule.glideMixin(duck);
duck.glide(); // logs "Gliding on the water"
```

---

[^protobj]: see `typeof Mustang.prototype`
[^verify]: verify with `Object.prototype.isPrototypeOf(Mustang.prototype);`
