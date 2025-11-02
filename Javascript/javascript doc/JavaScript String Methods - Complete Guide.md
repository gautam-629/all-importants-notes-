
## Table of Contents

1. [Introduction](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#introduction)
2. [Case Conversion Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#case-conversion-methods)
3. [Search and Match Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#search-and-match-methods)
4. [Extraction Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#extraction-methods)
5. [Modification Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#modification-methods)
6. [Padding Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#padding-methods)
7. [Trimming Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#trimming-methods)
8. [Comparison Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#comparison-methods)
9. [Split and Join Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#split-and-join-methods)
10. [Character Access Methods](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#character-access-methods)
11. [Template Literals](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#template-literals)
12. [Method Chaining](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#method-chaining)
13. [Performance Tips](https://claude.ai/chat/35c34bc9-b22f-4040-a0a0-e28110664c15#performance-tips)

## Introduction

JavaScript strings are immutable sequences of characters. Unlike arrays, string methods never modify the original string but instead return a new string. This guide covers all essential string methods with practical examples.

Strings in JavaScript can be created using single quotes, double quotes, or backticks (template literals). They are primitive values but JavaScript automatically wraps them with String objects when you call methods on them.

## Case Conversion Methods

These methods convert the case of string characters.

### toUpperCase()

Converts all characters to uppercase.

```javascript
const text = 'Hello World';
console.log(text.toUpperCase()); // 'HELLO WORLD'
console.log(text); // 'Hello World' (original unchanged)
```

### toLowerCase()

Converts all characters to lowercase.

```javascript
const text = 'Hello World';
console.log(text.toLowerCase()); // 'hello world'
```

### toLocaleUpperCase()

Converts to uppercase according to locale-specific case mappings.

```javascript
const text = 'istanbul';
console.log(text.toLocaleUpperCase('tr-TR')); // 'İSTANBUL'
console.log(text.toUpperCase()); // 'ISTANBUL'
```

### toLocaleLowerCase()

Converts to lowercase according to locale-specific case mappings.

```javascript
const text = 'ISTANBUL';
console.log(text.toLocaleLowerCase('tr-TR')); // 'istanbul'
```

## Search and Match Methods

These methods help you find and match patterns within strings.

### indexOf()

Returns the index of the first occurrence of a substring.

```javascript
const text = 'Hello World, Hello Universe';
console.log(text.indexOf('Hello')); // 0
console.log(text.indexOf('World')); // 6
console.log(text.indexOf('hello')); // -1 (case-sensitive)
console.log(text.indexOf('Hello', 5)); // 13 (search from index 5)
```

### lastIndexOf()

Returns the index of the last occurrence of a substring.

```javascript
const text = 'Hello World, Hello Universe';
console.log(text.lastIndexOf('Hello')); // 13
console.log(text.lastIndexOf('o')); // 17
```

### search()

Searches for a match using a regular expression and returns the index.

```javascript
const text = 'Hello World 123';
console.log(text.search(/\d+/)); // 12 (finds numbers)
console.log(text.search(/world/i)); // 6 (case-insensitive)
```

### match()

Retrieves the result of matching a string against a regular expression.

```javascript
const text = 'The rain in Spain stays mainly in the plain';
console.log(text.match(/ain/g)); // ['ain', 'ain', 'ain', 'ain']

const email = 'Contact: john@example.com';
const emailMatch = email.match(/[\w.]+@[\w.]+/);
console.log(emailMatch[0]); // 'john@example.com'
```

### matchAll()

Returns an iterator of all matches including capturing groups.

```javascript
const text = 'test1 test2 test3';
const regex = /test(\d)/g;
const matches = [...text.matchAll(regex)];

matches.forEach(match => {
    console.log(`Found ${match[0]}, number: ${match[1]}`);
});
// Output: Found test1, number: 1
//         Found test2, number: 2
//         Found test3, number: 3
```

### includes()

Checks if a string contains a specified substring.

```javascript
const text = 'Hello World';
console.log(text.includes('World')); // true
console.log(text.includes('world')); // false
console.log(text.includes('Hello', 1)); // false (search from index 1)
```

### startsWith()

Checks if a string starts with specified characters.

```javascript
const text = 'Hello World';
console.log(text.startsWith('Hello')); // true
console.log(text.startsWith('World')); // false
console.log(text.startsWith('World', 6)); // true (check from index 6)
```

### endsWith()

Checks if a string ends with specified characters.

```javascript
const text = 'Hello World';
console.log(text.endsWith('World')); // true
console.log(text.endsWith('Hello')); // false
console.log(text.endsWith('Hello', 5)); // true (check first 5 characters)
```

## Extraction Methods

These methods extract parts of a string.

### slice()

Extracts a section of a string and returns it as a new string.

```javascript
const text = 'Hello World';
console.log(text.slice(0, 5)); // 'Hello'
console.log(text.slice(6)); // 'World'
console.log(text.slice(-5)); // 'World' (negative index from end)
console.log(text.slice(-5, -1)); // 'Worl'
```

### substring()

Similar to slice but doesn't accept negative indexes.

```javascript
const text = 'Hello World';
console.log(text.substring(0, 5)); // 'Hello'
console.log(text.substring(6)); // 'World'
console.log(text.substring(6, 11)); // 'World'
```

### substr() (Deprecated)

Extracts a specified number of characters from a start position.

```javascript
const text = 'Hello World';
console.log(text.substr(0, 5)); // 'Hello'
console.log(text.substr(6, 5)); // 'World'
// Note: Use slice() or substring() instead
```

## Modification Methods

These methods modify or transform strings.

### replace()

Replaces the first occurrence of a specified value.

```javascript
const text = 'Hello World, Hello Universe';
console.log(text.replace('Hello', 'Hi')); // 'Hi World, Hello Universe'
console.log(text.replace(/Hello/g, 'Hi')); // 'Hi World, Hi Universe'

// With callback function
const result = 'I have 2 apples and 3 oranges'.replace(/\d+/g, match => {
    return parseInt(match) * 2;
});
console.log(result); // 'I have 4 apples and 6 oranges'
```

### replaceAll()

Replaces all occurrences of a specified value.

```javascript
const text = 'Hello World, Hello Universe';
console.log(text.replaceAll('Hello', 'Hi')); // 'Hi World, Hi Universe'
console.log(text.replaceAll(/o/g, '0')); // 'Hell0 W0rld, Hell0 Universe'
```

### concat()

Joins two or more strings.

```javascript
const str1 = 'Hello';
const str2 = 'World';
console.log(str1.concat(' ', str2)); // 'Hello World'
console.log(str1.concat(' ', str2, '!')); // 'Hello World!'

// Template literals are often preferred
console.log(`${str1} ${str2}`); // 'Hello World'
```

### repeat()

Returns a new string with a specified number of copies.

```javascript
const text = 'Hello ';
console.log(text.repeat(3)); // 'Hello Hello Hello '

const dash = '-';
console.log(dash.repeat(10)); // '----------'
```

## Padding Methods

These methods add padding to strings.

### padStart()

Pads the current string from the start.

```javascript
const num = '5';
console.log(num.padStart(3, '0')); // '005'

const text = 'Hello';
console.log(text.padStart(10, '*')); // '*****Hello'
console.log(text.padStart(10)); // '     Hello' (default space)
```

### padEnd()

Pads the current string from the end.

```javascript
const num = '5';
console.log(num.padEnd(3, '0')); // '500'

const text = 'Hello';
console.log(text.padEnd(10, '*')); // 'Hello*****'
```

## Trimming Methods

These methods remove whitespace from strings.

### trim()

Removes whitespace from both ends.

```javascript
const text = '   Hello World   ';
console.log(text.trim()); // 'Hello World'
console.log(text.trim().length); // 11
```

### trimStart() / trimLeft()

Removes whitespace from the beginning.

```javascript
const text = '   Hello World   ';
console.log(text.trimStart()); // 'Hello World   '
console.log(text.trimLeft()); // 'Hello World   ' (alias)
```

### trimEnd() / trimRight()

Removes whitespace from the end.

```javascript
const text = '   Hello World   ';
console.log(text.trimEnd()); // '   Hello World'
console.log(text.trimRight()); // '   Hello World' (alias)
```

## Comparison Methods

These methods compare strings.

### localeCompare()

Compares two strings in the current locale.

```javascript
const a = 'apple';
const b = 'banana';
console.log(a.localeCompare(b)); // -1 (a comes before b)
console.log(b.localeCompare(a)); // 1 (b comes after a)
console.log(a.localeCompare('apple')); // 0 (equal)

// With locale
const german = 'ä';
console.log(german.localeCompare('z', 'de')); // -1
console.log(german.localeCompare('z', 'sv')); // 1
```

## Split and Join Methods

### split()

Splits a string into an array of substrings.

```javascript
const text = 'Hello World';
console.log(text.split(' ')); // ['Hello', 'World']

const csv = 'apple,banana,orange';
console.log(csv.split(',')); // ['apple', 'banana', 'orange']

// With limit
const sentence = 'one two three four five';
console.log(sentence.split(' ', 3)); // ['one', 'two', 'three']

// Split every character
const word = 'Hello';
console.log(word.split('')); // ['H', 'e', 'l', 'l', 'o']

// With regex
const text2 = 'Hello   World  Test';
console.log(text2.split(/\s+/)); // ['Hello', 'World', 'Test']
```

## Character Access Methods

These methods access individual characters.

### charAt()

Returns the character at a specified index.

```javascript
const text = 'Hello';
console.log(text.charAt(0)); // 'H'
console.log(text.charAt(4)); // 'o'
console.log(text.charAt(10)); // '' (empty string for out of bounds)
```

### charCodeAt()

Returns the Unicode value of the character at a specified index.

```javascript
const text = 'Hello';
console.log(text.charCodeAt(0)); // 72 (H)
console.log(text.charCodeAt(1)); // 101 (e)
```

### codePointAt()

Returns the Unicode code point value at a given position.

```javascript
const text = '😀Hello';
console.log(text.codePointAt(0)); // 128512 (emoji)
console.log(text.codePointAt(2)); // 72 (H)
```

### at()

Returns the character at a specified index (supports negative indexing).

```javascript
const text = 'Hello';
console.log(text.at(0)); // 'H'
console.log(text.at(-1)); // 'o' (last character)
console.log(text.at(-2)); // 'l'
```

### String.fromCharCode()

Static method that creates a string from Unicode values.

```javascript
console.log(String.fromCharCode(72, 101, 108, 108, 111)); // 'Hello'
console.log(String.fromCharCode(65, 66, 67)); // 'ABC'
```

### String.fromCodePoint()

Static method that creates a string from code points.

```javascript
console.log(String.fromCodePoint(128512)); // '😀'
console.log(String.fromCodePoint(72, 101, 108, 108, 111)); // 'Hello'
```

## Template Literals

ES6 template literals provide powerful string features.

### Basic Usage

```javascript
const name = 'John';
const age = 30;
const message = `Hello, my name is ${name} and I am ${age} years old.`;
console.log(message); // 'Hello, my name is John and I am 30 years old.'
```

### Multi-line Strings

```javascript
const multiLine = `
    This is a
    multi-line
    string
`;
console.log(multiLine);
```

### Expression Evaluation

```javascript
const a = 5;
const b = 10;
console.log(`Sum: ${a + b}`); // 'Sum: 15'
console.log(`Product: ${a * b}`); // 'Product: 50'

const user = { name: 'John', age: 30 };
console.log(`${user.name} is ${user.age} years old`);
```

### Tagged Templates

```javascript
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => {
        return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
    }, '');
}

const name = 'John';
const age = 30;
const html = highlight`Name: ${name}, Age: ${age}`;
console.log(html); // 'Name: <mark>John</mark>, Age: <mark>30</mark>'
```

## Method Chaining

String methods can be chained together for complex transformations.

```javascript
const text = '  Hello World  ';

// Clean and transform text
const result = text
    .trim()
    .toLowerCase()
    .replace('world', 'universe')
    .split(' ')
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');

console.log(result); // 'Hello Universe'

// URL slug creation
const title = '  JavaScript String Methods Guide!  ';
const slug = title
    .trim()
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/\s+/g, '-');

console.log(slug); // 'javascript-string-methods-guide'

// Email validation and formatting
const email = '  JoHn.DoE@Example.COM  ';
const formattedEmail = email
    .trim()
    .toLowerCase()
    .replace(/\s+/g, '');

console.log(formattedEmail); // 'john.doe@example.com'
```

## Performance Tips

### 1. String Concatenation

```javascript
// Less efficient: multiple concatenations
let result = '';
for (let i = 0; i < 1000; i++) {
    result += 'text';
}

// More efficient: use array join
const parts = [];
for (let i = 0; i < 1000; i++) {
    parts.push('text');
}
const result = parts.join('');

// Most efficient: template literals for simple cases
const result = `${'text'.repeat(1000)}`;
```

### 2. Choose the Right Method

```javascript
// Use includes() for simple checks
text.includes('hello'); // Good
text.indexOf('hello') !== -1; // Less readable

// Use startsWith/endsWith instead of slice comparisons
text.startsWith('Hello'); // Good
text.slice(0, 5) === 'Hello'; // Less efficient

// Use trim() instead of regex when possible
text.trim(); // Good
text.replace(/^\s+|\s+$/g, ''); // Unnecessary complexity
```

### 3. Avoid Unnecessary Conversions

```javascript
// Bad: multiple conversions
const result = text.toLowerCase().toUpperCase().toLowerCase();

// Good: single conversion
const result = text.toLowerCase();
```

### 4. Regular Expression Caching

```javascript
// Less efficient: creates new regex each time
function validate(text) {
    return text.match(/^\d+$/);
}

// More efficient: cache regex
const numRegex = /^\d+$/;
function validate(text) {
    return text.match(numRegex);
}
```

### 5. String Search Optimization

```javascript
// For single character search
text.indexOf('x'); // Faster than regex

// For pattern matching
text.match(/pattern/); // Use regex when needed

// Early termination
if (text.startsWith('prefix')) {
    // More efficient than checking the entire string
}
```

## Common Use Cases

### URL Parameter Parsing

```javascript
const url = 'https://example.com?name=John&age=30&city=NYC';
const params = url
    .split('?')[1]
    .split('&')
    .reduce((acc, param) => {
        const [key, value] = param.split('=');
        acc[key] = decodeURIComponent(value);
        return acc;
    }, {});

console.log(params); // { name: 'John', age: '30', city: 'NYC' }
```

### Title Case Conversion

```javascript
function toTitleCase(str) {
    return str
        .toLowerCase()
        .split(' ')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(' ');
}

console.log(toTitleCase('hello world from javascript'));
// 'Hello World From Javascript'
```

### String Truncation

```javascript
function truncate(str, maxLength) {
    if (str.length <= maxLength) return str;
    return str.slice(0, maxLength - 3) + '...';
}

console.log(truncate('This is a very long text', 15));
// 'This is a ve...'
```

### Word Count

```javascript
function wordCount(str) {
    return str.trim().split(/\s+/).filter(word => word.length > 0).length;
}

console.log(wordCount('Hello World  from   JavaScript'));
// 4
```

### Reverse String

```javascript
function reverseString(str) {
    return str.split('').reverse().join('');
}

console.log(reverseString('Hello')); // 'olleH'
```

## Conclusion

JavaScript string methods provide comprehensive tools for text manipulation. Understanding these methods will help you write cleaner, more efficient code for text processing tasks. Remember that strings are immutable, so all methods return new strings rather than modifying the original.

Key takeaways:

- Strings are immutable - methods always return new strings
- Regular expressions provide powerful pattern matching capabilities
- Template literals offer modern, readable string interpolation
- Method chaining enables elegant string transformations
- Choose the right method for performance optimization