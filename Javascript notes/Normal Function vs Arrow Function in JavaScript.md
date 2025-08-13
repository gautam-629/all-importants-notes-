
JavaScript provides two main ways to define functions: **normal functions** and **arrow functions**. They differ in behavior with `arguments` and `this`, among other things.

---

## 1. Arguments Object

- **Normal functions** have access to the `arguments` object.
- **Arrow functions** do not have their own `arguments` object.

```javascript
// Normal function
function display1() {
  console.log(arguments);
}
display1(1, 2, 3, 4, 5);
// Output: [1, 2, 3, 4, 5]

// Arrow function
let display2 = () => {
  console.log(arguments);
};
display2(2, 4, 5, 3, 7);
// Output: ReferenceError or undefined in strict mode
```

---

## 2. `this` Keyword

- **Normal functions** have their own `this` context depending on how they are called.
- **Arrow functions** do not have their own `this`; they use `this` from the enclosing lexical scope.

```javascript
let obj = {
  name: "Binod",

  // Normal function
  sayName() {
    console.log(this.name);
  },

  // Arrow function
  sayNameArrow: () => {
    console.log(this.name); // Arrow function does not have its own this
  },
};

obj.sayName();      // Output: Binod
obj.sayNameArrow(); // Output: undefined
```

---

## 3. Implicit Return

- Arrow functions allow a concise syntax with implicit return for single expressions.

```javascript
// Normal function
function sum(a, b) {
  return a + b;
}

// Arrow function with implicit return
const sumArrow = (a, b) => a + b;

console.log(sum(5, 3));      // Output: 8
console.log(sumArrow(5, 3)); // Output: 8
```

**Summary:**

| Feature                 | Normal Function       | Arrow Function                |
|-------------------------|--------------------|-------------------------------|
| Arguments Object        | Available          | Not Available                 |
| `this` Keyword          | Own `this`         | Lexical `this`               |
| Syntax                  | Longer             | Shorter, concise             |
| Implicit Return         | No                 | Yes, for single expression   |
