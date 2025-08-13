# JavaScript Data Types

JavaScript data types are mainly divided into **two categories**:  

---

## 1. Primitive Data Types
These are the most basic types of data. They are **immutable** (cannot be changed directly) and stored **by value**.

| Data Type     | Description                                              | Example                 |
| ------------- | -------------------------------------------------------- | ----------------------- |
| **String**    | Text data inside quotes.                                 | `"Hello"`, `'Binod'`    |
| **Number**    | Numeric values (both integers and decimals).             | `42`, `3.14`            |
| **Boolean**   | Logical values: `true` or `false`.                       | `true`, `false`         |
| **Undefined** | A variable declared but not assigned a value.            | `let x; // undefined`   |
| **Null**      | An intentional empty value.                              | `let y = null;`         |
| **Symbol**    | Unique and immutable value used as object property keys. | `Symbol("id")`          |
| **BigInt**    | For very large integers beyond `Number` limit.           | `12345678901234567890n` |

---

## 2. Non-Primitive (Reference) Data Types
These are more complex and **mutable** (can be changed). They are stored **by reference**.

| Data Type  | Description | Example |
|------------|-------------|---------|
| **Object** | Collection of key-value pairs. | `{ name: "Binod", age: 25 }` |
| **Array**  | Ordered list of values. | `[1, 2, 3]` |
| **Function** | Block of code that performs a task. | `function greet() { return "Hi"; }` |

---

## Key Differences Between Primitive & Non-Primitive Types

| Feature         | Primitive | Non-Primitive |
|-----------------|-----------|---------------|
| **Mutability**  | Immutable (value cannot be changed directly) | Mutable (values can be changed) |
| **Storage**     | Stored by value | Stored by reference |
| **Examples**    | String, Number, Boolean, Undefined, Null, Symbol, BigInt | Object, Array, Function |
| **Copy Behavior** | Copying creates a new independent value | Copying references the same memory location |
| **Size**        | Fixed size | Dynamic size |

---

## Example Showing the Difference

```javascript
// Primitive (by value)
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 (unchanged)

// Non-Primitive (by reference)
let obj1 = { name: "Binod" };
let obj2 = obj1;
obj2.name = "Gautam";
console.log(obj1.name); // "Gautam" (changed because they share reference)
