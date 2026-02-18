
## Table of Contents

1. Introduction
2. Mutating Methods
3. Non-Mutating Methods
4. Iteration Methods
5. Search Methods
6. Transformation Methods
7. Utility Methods
8. ES6+ Methods
9. Method Chaining
10. Performance Tips
## Introduction

JavaScript arrays come with a rich set of built-in methods that allow you to manipulate, search, transform, and iterate over array elements. Understanding these methods is crucial for writing efficient and readable JavaScript code.

Arrays in JavaScript are objects with special behavior for numeric indexes and a `length` property. They provide methods that can be categorized into several groups based on their functionality.
```javascript
## Arrays can have arbitrary properties (like objects)
const arr = [1, 2, 3];
arr.foo = "bar";
console.log(arr.foo);
console.log(arr)
```

## Mutating Methods(8 methods)

These methods modify the original array.
### push()

Adds one or more elements to the end of an array.

```javascript
const fruits = ['apple', 'banana'];
fruits.push('orange', 'grape');
console.log(fruits); // ['apple', 'banana', 'orange', 'grape']
```

### pop()

Removes and returns the last element from an array.

```javascript
const fruits = ['apple', 'banana', 'orange'];
const lastFruit = fruits.pop();
console.log(lastFruit); // 'orange'
console.log(fruits); // ['apple', 'banana']
```

### unshift()

Adds one or more elements to the beginning of an array.

```javascript
const fruits = ['banana', 'orange'];
fruits.unshift('apple', 'grape');
console.log(fruits); // ['apple', 'grape', 'banana', 'orange']
```

### shift()

Removes and returns the first element from an array.

```javascript
const fruits = ['apple', 'banana', 'orange'];
const firstFruit = fruits.shift();
console.log(firstFruit); // 'apple'
console.log(fruits); // ['banana', 'orange']
```

### splice()

Changes the contents of an array by removing or replacing existing elements and/or adding new elements.

```javascript
const fruits = ['apple', 'banana', 'orange', 'grape'];
// Remove 2 elements starting at index 1, add 'kiwi', 'mango'
const removed = fruits.splice(1, 2, 'kiwi', 'mango');
console.log(fruits); // ['apple', 'kiwi', 'mango', 'grape']
console.log(removed); // ['banana', 'orange']
```

### sort()

Sorts the elements of an array in place and returns the sorted array.

```javascript
const numbers = [3, 1, 4, 1, 5, 9];
numbers.sort(); // Sorts as strings by default
console.log(numbers); // [1, 1, 3, 4, 5, 9]

// For numerical sorting
const nums = [10, 5, 40, 25, 1000, 1];
nums.sort((a, b) => a - b);
console.log(nums); // [1, 5, 10, 25, 40, 1000]
```

### reverse()

Reverses an array in place.

```javascript
const letters = ['a', 'b', 'c', 'd'];
letters.reverse();
console.log(letters); // ['d', 'c', 'b', 'a']
```

### fill()

Fills all or part of an array with a static value.

```javascript
const arr = new Array(5);
arr.fill(0);
console.log(arr); // [0, 0, 0, 0, 0]

const numbers = [1, 2, 3, 4, 5];
numbers.fill(9, 2, 4); // Fill with 9 from index 2 to 4
console.log(numbers); // [1, 2, 9, 9, 5]
```

## Non-Mutating Method(3 Method)

These methods return a new array without modifying the original.
### concat()

Merges two or more arrays.

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const arr3 = [7, 8, 9];
const combined = arr1.concat(arr2, arr3);
console.log(combined); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(arr1); // [1, 2, 3] (unchanged)
```

### slice()

Returns a shallow copy of a portion of an array.

```javascript
const fruits = ['apple', 'banana', 'orange', 'grape', 'kiwi'];
const citrus = fruits.slice(2, 4);
console.log(citrus); // ['orange', 'grape']
console.log(fruits); // Original array unchanged
```

### join()

Creates and returns a new string by concatenating all array elements.

```javascript
const words = ['Hello', 'world', 'from', 'JavaScript'];
const sentence = words.join(' ');
console.log(sentence); // 'Hello world from JavaScript'

const numbers = [1, 2, 3, 4, 5];
const csv = numbers.join(',');
console.log(csv); // '1,2,3,4,5'
```

## Iteration Methods(4 method)

These methods execute a function for each array element.

### forEach()

Executes a provided function once for each array element.

```javascript
const numbers = [1, 2, 3, 4, 5];
numbers.forEach((num, index) => {
    console.log(`Index ${index}: ${num * 2}`);
});
// Output: Index 0: 2, Index 1: 4, Index 2: 6, Index 3: 8, Index 4: 10
```
usecase: Iterating or updating external state
```javascript
let total=0;
const prices=[3,13,3]
prices.forEach(price=>{
 total+=price
})
console.log(total)
```
### map()

Creates a new array with the results of calling a provided function on every element.

```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

const users = [
    { name: 'John', age: 30 },
    { name: 'Jane', age: 25 },
    { name: 'Bob', age: 35 }
];
const names = users.map(user => user.name);
console.log(names); // ['John', 'Jane', 'Bob']
```

### filter()

Creates a new array with all elements that pass the test implemented by the provided function.

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const evenNumbers = numbers.filter(num => num % 2 === 0);
console.log(evenNumbers); // [2, 4, 6, 8, 10]

const users = [
    { name: 'John', age: 30, active: true },
    { name: 'Jane', age: 25, active: false },
    { name: 'Bob', age: 35, active: true }
];
const activeUsers = users.filter(user => user.active);
console.log(activeUsers); // [{ name: 'John', age: 30, active: true }, { name: 'Bob', age: 35, active: true }]
```

### reduce()

Executes a reducer function on each element, resulting in a single output value.

```javascript
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum); // 15

const words = ['Hello', 'world', 'from', 'JavaScript'];
const sentence = words.reduce((acc, word) => acc + ' ' + word);
console.log(sentence); // 'Hello world from JavaScript'

// Finding maximum value
const max = numbers.reduce((acc, num) => num > acc ? num : acc);
console.log(max); // 5
```

### reduceRight()

Same as reduce, but processes the array from right to left.

```javascript
const letters = ['a', 'b', 'c', 'd'];
const reversed = letters.reduceRight((acc, letter) => acc + letter, '');
console.log(reversed); // 'dcba'
```

## Search Methods(5 methods)

These methods help you find elements in arrays.

### find()

Returns the first element that satisfies the provided testing function.

```javascript
const users = [
    { id: 1, name: 'John', age: 30 },
    { id: 2, name: 'Jane', age: 25 },
    { id: 3, name: 'Bob', age: 35 }
];
const user = users.find(u => u.age > 30);
console.log(user); // { id: 3, name: 'Bob', age: 35 }
```

### findIndex()

Returns the index of the first element that satisfies the provided testing function.

```javascript
const numbers = [10, 20, 30, 40, 50];
const index = numbers.findIndex(num => num > 25);
console.log(index); // 2
```

### indexOf()

Returns the first index at which a given element can be found.

```javascript
const fruits = ['apple', 'banana', 'orange', 'banana'];
const index = fruits.indexOf('banana');
console.log(index); // 1

const notFound = fruits.indexOf('grape');
console.log(notFound); // -1
```

### lastIndexOf()

Returns the last index at which a given element can be found.

```javascript
const fruits = ['apple', 'banana', 'orange', 'banana'];
const lastIndex = fruits.lastIndexOf('banana');
console.log(lastIndex); // 3
```

### includes()

Determines whether an array includes a certain value.

```javascript
const fruits = ['apple', 'banana', 'orange'];
console.log(fruits.includes('banana')); // true
console.log(fruits.includes('grape')); // false
```

## Transformation Methods(2 Method)

### flat()

Creates a new array with all sub-array elements concatenated recursively up to specified depth.

```javascript
const nested = [1, 2, [3, 4], [5, [6, 7]]];
const flattened = nested.flat();
console.log(flattened); // [1, 2, 3, 4, 5, [6, 7]]

const deepFlattened = nested.flat(2);
console.log(deepFlattened); // [1, 2, 3, 4, 5, 6, 7]
```

### flatMap()

First maps each element using a mapping function, then flattens the result.

```javascript
const sentences = ['Hello world', 'How are you'];
const words = sentences.flatMap(sentence => sentence.split(' '));
console.log(words); // ['Hello', 'world', 'How', 'are', 'you']
```

## Utility Methods(5 Method)

### every()

Tests whether all elements pass the test implemented by the provided function.

```javascript
const numbers = [2, 4, 6, 8, 10];
const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven); // true

const mixed = [2, 4, 5, 8, 10];
const allEvenMixed = mixed.every(num => num % 2 === 0);
console.log(allEvenMixed); // false
```

### some()

Tests whether at least one element passes the test implemented by the provided function.

```javascript
const numbers = [1, 3, 5, 8, 9];
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true
```

### isArray()

Static method that determines whether the passed value is an array.

```javascript
console.log(Array.isArray([1, 2, 3])); // true
console.log(Array.isArray('hello')); // false
console.log(Array.isArray({ length: 3 })); // false
```

### from()

Static method that creates a new array from an array-like or iterable object.

```javascript
// From string
const letters = Array.from('hello');
console.log(letters); // ['h', 'e', 'l', 'l', 'o']

// From Set
const uniqueNumbers = Array.from(new Set([1, 2, 2, 3, 3, 4]));
console.log(uniqueNumbers); // [1, 2, 3, 4]

// With mapping function
const doubled = Array.from([1, 2, 3], x => x * 2);
console.log(doubled); // [2, 4, 6]
```

### of()

Static method that creates a new array with a variable number of arguments.

```javascript
const arr1 = Array.of(7); // [7]
const arr2 = new Array(7); // [empty × 7]
const arr3 = Array.of(1, 2, 3); // [1, 2, 3]
```
## ES6+ Methods(3 Method)

### entries()

Returns an array iterator object with key/value pairs.

```javascript
const fruits = ['apple', 'banana', 'orange'];
for (const [index, fruit] of fruits.entries()) {
    console.log(`${index}: ${fruit}`);
}
// Output: 0: apple, 1: banana, 2: orange
```
### keys()
Returns an array iterator that contains the keys for each index.

```javascript
const fruits = ['apple', 'banana', 'orange'];
const keys = Array.from(fruits.keys());
console.log(keys); // [0, 1, 2]
```
### values()
Returns an array iterator that contains the values for each index.

```javascript
const fruits = ['apple', 'banana', 'orange'];
for (const fruit of fruits.values()) {
    console.log(fruit);
}
// Output: apple, banana, orange
```
### copyWithin()

Shallow copies part of an array to another location in the same array.

```javascript
const arr = [1, 2, 3, 4, 5];
arr.copyWithin(0, 3); // Copy elements from index 3 to index 0
console.log(arr); // [4, 5, 3, 4, 5]
```
## Method Chaining
One of the powerful features of array methods is the ability to chain them together:

```javascript
const users = [
    { name: 'John', age: 30, salary: 50000 },
    { name: 'Jane', age: 25, salary: 60000 },
    { name: 'Bob', age: 35, salary: 75000 },
    { name: 'Alice', age: 28, salary: 55000 }
];

const result = users
    .filter(user => user.age > 27) // Filter users older than 27
    .map(user => ({ ...user, bonus: user.salary * 0.1 })) // Add bonus
    .sort((a, b) => b.salary - a.salary) // Sort by salary descending
    .map(user => user.name); // Get names only

console.log(result); // ['Bob', 'Alice', 'John']
```

## Performance Tips

### 1. Choose the Right Method

- Use `find()` instead of `filter()[0]` when you need only the first match
- Use `some()` instead of `filter().length > 0` to check existence
- Use `includes()` for simple value checks instead of `indexOf() !== -1`

### 2. Early Termination

Methods like `find()`, `some()`, and `every()` stop execution once the condition is met:

```javascript
// Good: stops at first match
const user = users.find(u => u.id === 123);

// Less efficient: checks all elements
const user = users.filter(u => u.id === 123)[0];
```

### 3. Avoid Unnecessary Iterations

```javascript
// Less efficient: multiple iterations
const processedUsers = users
    .filter(user => user.active)
    .map(user => ({ ...user, fullName: `${user.firstName} ${user.lastName}` }))
    .filter(user => user.age > 18);

// More efficient: combine operations where possible
const processedUsers = users.reduce((acc, user) => {
    if (user.active && user.age > 18) {
        acc.push({ ...user, fullName: `${user.firstName} ${user.lastName}` });
    }
    return acc;
}, []);
```

### 4. Memory Considerations

- `map()`, `filter()`, and `slice()` create new arrays
- `forEach()` doesn't create new arrays
- Use `for...of` or traditional `for` loops for simple iterations when performance is critical

## Conclusion

JavaScript array methods provide powerful tools for data manipulation and functional programming patterns. Understanding when and how to use each method will help you write more efficient, readable, and maintainable code. Practice combining these methods through chaining to create elegant solutions for complex data transformations.

Remember that some methods mutate the original array while others return new arrays. Always consider whether you need to preserve the original data when choosing your approach.