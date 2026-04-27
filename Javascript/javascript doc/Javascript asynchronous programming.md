
JavaScript runs on a **single thread**. Without async programming, every slow task (network request, file read, timer) would freeze the entire page until it finishes.

Asynchronous programming lets slow work happen **in the background** while JavaScript keeps executing other code.

There are three patterns, each building on the previous:

1. **Callbacks** — original pattern
2. **Promises** — cleaner chaining
3. **async / await** — modern standard

---

## 1. Sync vs Async

### Synchronous — blocks everything

```javascript
console.log("1. Start");
const user = getUser();   // 😴 page is frozen for 3 seconds
console.log("2. Got user");
console.log("3. End");

// Output order: 1 → 2 → 3 (but page is frozen!)
```

### Asynchronous — non-blocking

```javascript
console.log("1. Start");

setTimeout(() => {
  console.log("2. This ran after 2s");
}, 2000);

console.log("3. End");  // runs BEFORE step 2!

// Output order: 1 → 3 → 2
```

> **Key insight:** `setTimeout` doesn't block. JavaScript registers the callback and moves on immediately.

---

## 2. Callbacks

A **callback** is a function passed as an argument to be called when async work finishes.

### Basic example

```javascript
function fetchUser(id, callback) {
  setTimeout(() => {
    const user = { id, name: "Binod", role: "admin" };
    callback(null, user);  // null = no error
  }, 1000);
}

fetchUser("1", (err, user) => {
  if (err) return console.log(err);
  console.log(user);
});
```

### TypeScript typed callbacks

```typescript
interface IUser   { id: string; name: string; role: string }
interface IOrder  { id: string; customerId: string; amount: number }
interface IInvoice { id: string; orderId: string; totalAmount: number; customerId: string }

function fetchUser(
  id: string,
  callback: (err: Error | null, user: IUser) => void
) {
  setTimeout(() => {
    const user: IUser = { id, name: "Binod", role: "admin" };
    callback(null, user);
  }, 1000);
}

function fetchOrders(
  id: string,
  callback: (err: Error | null, order: IOrder) => void
) {
  setTimeout(() => {
    const order: IOrder = { id, amount: 23, customerId: "1" };
    callback(null, order);
  }, 1000);
}

function fetchInvoice(
  id: string,
  callback: (err: Error | null, invoice: IInvoice) => void
) {
  setTimeout(() => {
    const invoice: IInvoice = { id, customerId: "1", orderId: "1", totalAmount: 343 };
    callback(null, invoice);
  }, 1000);
}
```

### ⚠️ Callback Hell — the main problem

Nesting multiple callbacks creates deeply indented, hard-to-read code:

```javascript
fetchUser("1", (err, user) => {
  if (err) return console.log(err);
  fetchOrders(user.id, (err, order) => {
    if (err) return console.log(err);
    fetchInvoice(order.customerId, (err, invoice) => {
      if (err) return console.log(err);
      // deeply nested — hard to read and maintain 😬
    });
  });
});
```

**Problems with callback hell:**

- Hard to read and reason about
- Error handling must be repeated at every level
- Very difficult to debug

---

## 3. Promises

A **Promise** represents a value that will be available in the future. It has three possible states:

|State|Description|
|---|---|
|`Pending`|Initial state — work is in progress|
|`Fulfilled`|Completed successfully — value available|
|`Rejected`|Failed — error available|

```
Pending  →  resolve()  →  Fulfilled
Pending  →  reject()   →  Rejected
```

### Creating a Promise

```typescript
function fetchUser(id: number): Promise<IUser> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        const user: IUser = { id, name: "Binod", role: "admin" };
        resolve(user);               // success
      } else {
        reject(new Error("Invalid ID"));  // failure
      }
    }, 1000);
  });
}

function fetchOrders(id: number): Promise<IOrder> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, amount: 23, customerId: 1 });
      } else {
        reject(new Error("Invalid ID"));
      }
    }, 1000);
  });
}

function fetchInvoice(id: number): Promise<IInvoice> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, customerId: 1, orderId: 1, totalAmount: 343 });
      } else {
        reject(new Error("Invalid ID"));
      }
    }, 1000);
  });
}
```

### Chaining with `.then()` and `.catch()`

```javascript
fetchUser(1)
  .then(user => {
    console.log(user);
    return fetchOrders(user.id);   // return the next Promise to chain
  })
  .then(order => {
    console.log(order);
    return fetchInvoice(order.id);
  })
  .then(invoice => {
    console.log(invoice);
  })
  .catch(err => console.error(err));  // ONE catch handles ALL errors above
```

> **Key advantage:** Flat structure, single error handler — much better than callback hell.

---

## 4. async / await

`async/await` is **syntactic sugar over Promises**. It makes async code read like synchronous code.

- Mark a function with `async` → it always returns a Promise
- Use `await` before a Promise inside that function → pauses execution at that line until the Promise settles (without blocking the thread)

### Basic usage

```typescript
async function loadDashboard(userId: number) {
  try {
    const user    = await fetchUser(userId);    // waits ~1s
    console.log(user);

    const orders  = await fetchOrders(user.id); // waits ~1s
    console.log(orders);

    const invoice = await fetchInvoice(orders.id); // waits ~1s
    console.log(invoice);

    // Total time: ~3 seconds (runs sequentially)

  } catch (err) {
    console.error("Dashboard failed:", err);  // catches any error above
  }
}

loadDashboard(42);  // call normally — no await needed here
```

> **Note:** An `async` function always returns a Promise, even if you return a plain value inside it.

---

## 5. Parallel Requests with `Promise.all()`

The `await`-one-by-one pattern is **sequential** — each call waits for the previous to finish.

If calls are **independent** (don't depend on each other's results), run them in parallel with `Promise.all()`.

### Sequential vs Parallel — comparison

```typescript
// ❌ Sequential — total ~3 seconds (each waits for the previous)
const user    = await fetchUser(1);
const order   = await fetchOrders(1);
const invoice = await fetchInvoice(1);

// ✅ Parallel — total ~1 second (all run at the same time)
const [user, order, invoice] = await Promise.all([
  fetchUser(1),
  fetchOrders(1),
  fetchInvoice(1),
]);

console.log(user, order, invoice);
```

### Full example

```typescript
const fetchData = async () => {
  try {
    const [user, order, invoice] = await Promise.all([
      fetchUser(1),
      fetchOrders(1),
      fetchInvoice(1),
    ]);
    console.log(user, order, invoice);
  } catch (error) {
    console.log(error);
  }
};

fetchData();
```

> ⚠️ **Fast-fail behavior:** If _any_ Promise in `Promise.all()` rejects, the entire call rejects immediately — even if others succeed. Use `Promise.allSettled()` if you want all results regardless of failures.

---

## 6. All Promise Combinators

|Method|Behavior|
|---|---|
|`Promise.all(promises)`|Waits for all. Rejects immediately if any fail (fast-fail).|
|`Promise.allSettled(promises)`|Waits for all. Returns every result — fulfilled or rejected. No fast-fail.|
|`Promise.race(promises)`|Resolves/rejects as soon as the _first_ Promise settles.|
|`Promise.any(promises)`|Resolves when the _first_ Promise fulfills. Rejects only if **all** fail.|

---

## 7. Common Mistakes

### ❌ Forgetting `await`

```javascript
// Wrong — user is a Promise object, not the value
const user = fetchUser(1);
console.log(user.name);  // undefined!

// Correct
const user = await fetchUser(1);
console.log(user.name);  // "Binod"
```

### ❌ Using `await` outside an `async` function

```javascript
// Wrong — SyntaxError
const data = await fetchUser(1);

// Correct — wrap in async function
async function main() {
  const data = await fetchUser(1);
}

// Or use top-level await (only in ES modules)
const data = await fetchUser(1);  // works in .mjs files
```

### ❌ No error handling

```javascript
// Wrong — unhandled rejections can crash silently
fetchUser(-1).then(u => console.log(u));

// Correct — always handle errors
fetchUser(-1)
  .then(u => console.log(u))
  .catch(err => console.error(err));

// Or with async/await
async function run() {
  try {
    const user = await fetchUser(-1);
  } catch (err) {
    console.error(err);
  }
}
```

### ❌ Making `Promise.all()` sequential by mistake

```javascript
// Wrong — awaiting before passing to Promise.all() makes it sequential
const result = await Promise.all([
  await fetchUser(1),    // ← await here runs first!
  await fetchOrders(1),  // ← then this
]);

// Correct — pass the Promises, not the resolved values
const result = await Promise.all([
  fetchUser(1),    // passes the Promise directly
  fetchOrders(1),
]);
```

---

## 8. Quick Summary

|Pattern|Syntax|Use When|
|---|---|---|
|**Callback**|`fn(arg, (err, result) => {})`|Simple single async op; legacy code|
|**Promise**|`fn().then().catch()`|Chaining multiple async ops|
|**async/await**|`const x = await fn()`|Modern code; most readable|
|**Promise.all**|`await Promise.all([...])`|Multiple independent async ops|

### The evolution at a glance

```
Callbacks    →  deep nesting,   repeated error handling
Promises     →  flat .then() chain, single .catch()
async/await  →  reads like sync code, try/catch for errors
Promise.all  →  parallel execution, much faster
```