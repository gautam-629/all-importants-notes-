# JavaScript: Difference Between `let`, `var`, and `const` & Scope

---

## 1. Overview

In JavaScript, `let`, `var`, and `const` are used to declare variables.  
They differ in **scope**, **re-declaration**, **re-assignment**, and **hoisting**.

---

## 2. Key Differences

| Feature           | `var` | `let` | `const` |
|-------------------|-------|-------|---------|
| **Scope**         | Function scope | Block scope | Block scope |
| **Re-declaration**| ✅ Allowed | ❌ Not allowed | ❌ Not allowed |
| **Re-assignment** | ✅ Allowed | ✅ Allowed | ❌ Not allowed |
| **Hoisting**      | ✅ Hoisted (initialized as `undefined`) | ✅ Hoisted (but in Temporal Dead Zone) | ✅ Hoisted (but in Temporal Dead Zone) |
| **Initialization**| Optional | Optional | Required |

---

## 3. Scope Explanation

- **Global Scope**: Variables accessible from anywhere in the code.
- **Function Scope**: Variables declared inside a function are only available inside that function.
- **Block Scope**: Variables declared inside `{ ... }` (like in loops, if statements) are only accessible inside that block.

---

## 4. Examples

### Example 2: `let` (Block Scope)
```javascript
function testLet() {
    if (true) {
        let y = 20;
    }
    console.log(y); // ❌ ReferenceError: y is not defined
}
testLet();
```

---

### Example 3: `const` (Block Scope + No Re-assignment)
```javascript
function testConst() {
    const z = 30;
    z = 40; // ❌ TypeError: Assignment to constant variable
}
testConst();
```

---

### Example 4: `var` Re-declaration
```javascript
var a = 1;
var a = 2; // ✅ allowed
console.log(a); // 2
```

---

### Example 5: `let` Re-declaration
```javascript
let b = 1;
// let b = 2; // ❌ SyntaxError: Identifier 'b' has already been declared
b = 2; // ✅ allowed
console.log(b); // 2
```

---

### Example 6: `const` Re-declaration
```javascript
const c = 1;
// const c = 2; // ❌ SyntaxError: Identifier 'c' has already been declared
// c = 2; // ❌ TypeError: Assignment to constant variable
console.log(c); // 1
```

---

### Example 7: Hoisting with `var`
```javascript
console.log(a); // undefined (hoisted but uninitialized)
var a = 5;
```

---

### Example 8: Hoisting with `let`
```javascript
// console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 5;
console.log(b); // 5
```

---

### Example 9: Hoisting with `const`
```javascript
// console.log(c); // ❌ ReferenceError: Cannot access 'c' before initialization
const c = 5;
console.log(c); // 5
```

---

## 5. Quick Summary

- Use **`let`** when you expect the value to change.
- Use **`const`** for values that should never change.
- Avoid **`var`** in modern JavaScript to prevent scope-related bugs.
