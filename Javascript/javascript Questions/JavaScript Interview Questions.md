
##1. What is Hoisting?

**Simple Answer:** Hoisting moves variable and function declarations to the top of their scope before the code runs.

**Example:**

```javascript
// You can call the function before declaring it
sayHello(); // Works fine!

function sayHello() {
  console.log("Hello, world!");
}
```

**Why does this work?**

- JavaScript reads all declarations first (before executing code)
- Function declarations are fully loaded into memory
- Then the code executes line by line

**Variable Hoisting with `var`:**
```javascript
console.log(message); // undefined (not an error!)
var message = "Hi there!";
console.log(message); // "Hi there!"
```

The variable `message` is hoisted (moved to top), but its value is not. So it exists but is `undefined` until assigned.

---
## 2. What are Closures?

**Simple Answer:** A closure lets an inner function access variables from its outer function, even after the outer function has finished running.

**Real-Life Example: Private Bank Account**

```javascript
function createBankAccount() {
  let balance = 1000; // This is private!

  return {
    deposit(amount) {
      balance += amount;
      console.log(`Deposited: ${amount}`);
    },
    withdraw(amount) {
      if (amount <= balance) {
        balance -= amount;
        console.log(`Withdrew: ${amount}`);
      } else {
        console.log("Insufficient funds");
      }
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount();
account.deposit(500);        // Deposited: 500
console.log(account.getBalance()); // 1500
account.withdraw(2000);      // Insufficient funds
```

**Key Point:** The `balance` variable is hidden and can only be accessed through the methods we provide.

---
## 3. What are Debounce and Throttle?

**Simple Answer:** Both control how often a function runs during rapid events.

**Debounce:** Waits until the user stops doing something, then executes once.

- **Use case:** Search box (wait until user stops typing)

**Throttle:** Executes at most once per time interval.

- **Use case:** Scroll events (check position every 100ms, not 1000 times per second)

|Feature|Debounce|Throttle|
|---|---|---|
|When|After user stops|Once per interval|
|Use Case|Search, input, window resize|Scroll, drag, API limits|
|Goal|Reduce unnecessary calls|Limit call frequency|

---

## 4. What is the Event Loop?

**Simple Answer:** The event loop manages how JavaScript handles asynchronous code (like setTimeout, API calls, etc.).

**How it works:**

1. JavaScript runs code line by line (synchronous code runs first)
2. Asynchronous callbacks go to a queue
3. The event loop checks if the main code is done
4. If yes, it takes callbacks from the queue and runs them

This is why `setTimeout` doesn't block your code!

---
## 5. Shallow Copy vs Deep Copy

**Shallow Copy:** Copies the top level, but nested objects are still connected to the original.

```javascript
const original = { name: "Alice", address: { city: "NY" } };
const shallowCopy = { ...original };

shallowCopy.address.city = "LA";

console.log(original.address.city); // "LA" (changed!)
```

**Deep Copy:** Creates a completely independent copy.

```javascript
const original = { name: "Alice", address: { city: "NY" } };
const deepCopy = JSON.parse(JSON.stringify(original));

deepCopy.address.city = "LA";

console.log(original.address.city); // "NY" (unchanged!)
```

**Quick Summary:**

- **Shallow:** Changes to nested objects affect the original
- **Deep:** Changes don't affect the original at all

---
## 6. What is a Generator?

**Simple Answer:** A generator is a function that can pause and resume its execution.

**Example:**

```javascript
function* countToThree() {
  yield 1;
  yield 2;
  yield 3;
}

const counter = countToThree();
console.log(counter.next().value); // 1
console.log(counter.next().value); // 2
console.log(counter.next().value); // 3
```

**Key Points:**

- Use `function*` to create a generator
- Use `yield` to pause and return a value
- Call `.next()` to resume execution
- Great for handling sequences of values without loading everything into memory at once