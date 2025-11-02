
## 1. this Keyword

The value of `this` depends on how a function is called.

```javascript
function info() {
  console.log(this);
}

info();       // window (or undefined in strict mode)
new info();   // new instance of info {}
```

---

## 2. bind()

`bind()` sets the value of `this` and returns a **new function** without calling it immediately.

```javascript
const user = {
  name: "Binod",
  printName(ar1, ar2) {
    console.log(`${this.name}`, ar1, ar2);
  },
};

const user1 = { name: "Gautam" };

const boundFunc = user.printName.bind(user1, "Hello", "World");
boundFunc(); // Output: Gautam Hello World
```

---

## 3. call()

`call()` sets `this` and **calls the function immediately**, passing arguments separately.

```javascript
const user = {
  name: "Binod",
  printName(ar1, ar2) {
    console.log(`${this.name}`, ar1, ar2);
  },
};

const user1 = { name: "Gautam" };

user.printName.call(user1, "arg1", "arg2");
// Output: Gautam arg1 arg2
```

---

## 4. apply()

`apply()` works like `call()` but passes arguments as an **array**.

```javascript
const user = {
  name: "Binod",
  printName(ar1, ar2) {
    console.log(`${this.name}`, ar1, ar2);
  },
};

const user1 = { name: "Gautam" };

user.printName.apply(user1, ["arg1", "arg2"]);
// Output: Gautam arg1 arg2
```

---

## Summary

|Method|Calls Immediately|Arguments|Description|
|---|---|---|---|
|`bind()`|❌ No|Separate|Returns new function|
|`call()`|✅ Yes|Separate|Calls immediately|
|`apply()`|✅ Yes|Array|Calls immediately|

### Quick Comparison

```javascript
const obj = { value: 42 };

function getValue(a, b) {
  return `${this.value}, ${a}, ${b}`;
}

// bind - returns new function
const boundFunc = getValue.bind(obj, 1, 2);
console.log(boundFunc()); // "42, 1, 2"

// call - executes immediately, separate arguments
console.log(getValue.call(obj, 1, 2)); // "42, 1, 2"

// apply - executes immediately, array arguments
console.log(getValue.apply(obj, [1, 2])); // "42, 1, 2"
```

---

## Common Use Cases

### Method Borrowing

```javascript
const person1 = {
  name: "Alice",
  greet: function() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

const person2 = { name: "Bob" };

person1.greet.call(person2); // "Hello, I'm Bob"
```

### Array Min/Max

```javascript
const numbers = [5, 6, 2, 3, 7];
const max = Math.max.apply(null, numbers);
console.log(max); // 7
```

### Event Handlers

```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    this.count++;
    console.log(`Count: ${this.count}`);
  }
  
  setupButton(button) {
    button.addEventListener('click', this.increment.bind(this));
  }
}
```

### Partial Application

```javascript
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2);
console.log(double(5)); // 10
```

---

## Key Takeaways

- **`bind()`**: Creates a new function with fixed `this`
- **`call()`**: Executes immediately with individual arguments
- **`apply()`**: Executes immediately with array arguments