
JavaScript provides several types of errors and ways to handle them using `throw`, `try`, and `catch`.

---
## 1. Types of Errors in JavaScript

Generally, there are 4 main types of errors:

### a. Syntax Error
Occurs when the code violates the syntax rules of JavaScript.

```javascript
let javaIsFun = true;
console.log(java is fun); // SyntaxError
```

### b. Reference Error
Occurs when trying to reference a variable that is not defined.

```javascript
console.log(value); // ReferenceError: value is not defined
```

### c. Type Error
Occurs when a value is not of the expected type.

```javascript
let javaIsFun = true;
console.log(javaIsFun.toUpperCase()); // TypeError: javaIsFun.toUpperCase is not a function
```

### d. Range Error
Occurs when a value is outside the allowed range.

```javascript
let myArray = [1, 2, 3];
console.log(myArray[3]); // RangeError: Index out of range
```

---

## 2. Error Handling using `throw`, `try`, and `catch`

### a. Using `throw`
You can manually throw an error using the `throw` statement. You can throw a string, an Error object, or a specific error type like `SyntaxError`.

```javascript
function div(a, b) {
  if (b === 0)
    throw new SyntaxError("Second parameter cannot be Zero");
  return a / b;
}

console.log(div(1, 0));
```

### b. Using `try` and `catch`
`try` allows you to test a block of code for errors, and `catch` lets you handle them gracefully.

```javascript
function div(a, b) {
  if (b === 0) throw new SyntaxError("Second parameter cannot be Zero");
  return a / b;
}

try {
  div(4, 0);
} catch (error) {
  console.log(error.message); // Second parameter cannot be Zero
  console.log(error.name);    // SyntaxError
}
```

---
## Summary

- **SyntaxError**: Incorrect code syntax.  
- **ReferenceError**: Using an undefined variable.  
- **TypeError**: Using a value in an inappropriate way.  
- **RangeError**: Value out of acceptable range.  
- **throw**: Used to manually raise an error.  
- **try/catch**: Used to handle errors safely without stopping program execution.
