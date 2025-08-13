# Hoisting in JavaScript

**Hoisting** in JavaScript is a behavior where **variable and function declarations are moved to the top of their containing scope** during the compilation phase, before the code is executed.  
This allows you to use variables and functions **before they are declared** (with some caveats).

---

## Key Points About Hoisting

1. **Applies to both variables and functions**  
   - Function declarations are fully hoisted.  
   - `var` variables are hoisted **and initialized with `undefined`**.  
   - `let` and `const` are hoisted **but not initialized**, so accessing them before declaration causes a **ReferenceError** (known as the Temporal Dead Zone).

2. **Scope Matters**  
   - Hoisting respects **function scope** for `var` and **block scope** for `let` and `const`.

3. **Only declarations are hoisted, not initializations**  
   - The value assignment happens in the original place in the code, not at the top.

---

## Examples

### 1. Hoisting with `var`
```javascript
console.log(a); // undefined
var a = 10;
console.log(a); // 10
```
- `var a` is hoisted to the top of the scope.  
- Only the declaration is hoisted, not the value `10`.

---

### 2. Hoisting with `let`
```javascript
console.log(b); // ❌ ReferenceError
let b = 20;
```
- `b` is hoisted but **not initialized**, so you cannot access it before declaration.  
- This is part of the **Temporal Dead Zone (TDZ)**.

---

### 3. Hoisting with `const`
```javascript
console.log(c); // ❌ ReferenceError
const c = 30;
```
- Works the same way as `let`; the variable is hoisted but cannot be accessed before declaration.

---

### 4. Hoisting with Functions
```javascript
greet(); // "Hello!"

function greet() {
    console.log("Hello!");
}
```
- Function declarations are fully hoisted, so you can call the function before it appears in the code.

**Note:** Function expressions are not hoisted in the same way:
```javascript
hello(); // ❌ TypeError: hello is not a function

var hello = function() {
    console.log("Hi");
};
```
- Only the variable `hello` is hoisted as `undefined`, so calling it before assignment fails.

---

## Summary

- **Hoisting**: Moving declarations to the top of the scope.  
- **`var`**: Hoisted and initialized as `undefined`.  
- **`let` & `const`**: Hoisted but not initialized → TDZ.  
- **Functions**: Declarations are fully hoisted, expressions are not.
