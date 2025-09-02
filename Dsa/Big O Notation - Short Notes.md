
### Why Important
- Common in interviews
- Shows code efficiency

### What is Big O
- Mathematical way to compare code efficiency
- Focus on:
  - **Time Complexity** → speed (operations, not seconds)
  - **Space Complexity** → memory usage
### Time Complexity
- Count operations, not execution time
- Independent of computer speed
- Example: Code 1 fewer operations → faster
### Space Complexity
- Measures memory usage
- Example: Code 1 = faster but more memory, Code 2 = slower but less memory
## Interview Tip
- Optimize for **time complexity** first
- Be ready to discuss **space complexity** if asked
## Big O, Omega, and Theta - Short Notes
### Notations
- **Ω (Omega):** Best case  
- **Θ (Theta):** Average case  
- **O (Big O):** Worst case  
### Example
- Searching in an array with a loop:
  - Find at first element → **Best case (Ω)**
  - Find in middle → **Average case (Θ)**
  - Find at last or not found → **Worst case (O)**
### Key Point
- **Big O always represents the worst case**
- No "best case Big O" or "average case Big O" → those are **Ω** and **Θ**
## Big O Notation: O(n)

### Concept
- **O(n):** Linear time complexity
- Number of operations grows **proportionally** with input size `n`
### Example
```js
function logItems(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
} 
```
## Big O Simplification: Drop Constants

### Rule
- **Ignore constants** when writing Big O.
- O(2n), O(3n), O(100n) → all simplified to **O(n)**.

## Example
```js
function logItemsTwice(n) {
  for (let i = 0; i < n; i++) console.log(i);
  for (let j = 0; j < n; j++) console.log(j);
}
```
## Big O: O(n²)
### Concept
- **O(n²):** Nested loops → operations = n × n
- Example: double nested loop (runs n² times)

```js
function logItems(n) {
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      console.log(i, j);
    }
  }
}
```

