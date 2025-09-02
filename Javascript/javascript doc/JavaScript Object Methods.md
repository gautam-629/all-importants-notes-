
## 1. Object Creation and Basic Example
We can create objects using object literals.

```javascript
const foodMap = {
  Burger: 200,
  pizza: 500,
  joice: 300,
};
```

---

## 2. Deleting a Property
We can delete a property from an object using the `delete` keyword.

```javascript
delete foodMap.Burger;
```

**Result:**  
`foodMap` now has only `pizza` and `joice`.

---

## 3. Copying Objects with `Object.assign()`
`Object.assign()` is used to copy the values of all enumerable properties from one or more source objects to a target object.

```javascript
const newFoodMap = Object.assign({ id: 1 }, foodMap);
```

**Explanation:**  
- The target object `{ id: 1 }` receives all properties from `foodMap`.  
- `newFoodMap` will be: `{ id: 1, pizza: 500, joice: 300 }`

---

## 4. Sealing an Object with `Object.seal()`
`Object.seal()` prevents adding or deleting properties but allows modification of existing property values.

```javascript
Object.seal(foodMap);
```

**Explanation:**  
- You cannot add new properties or delete existing ones.  
- You can still change the value of existing properties.

---

## 5. Freezing an Object with `Object.freeze()`
`Object.freeze()` makes an object completely immutable. You cannot add, delete, or modify any properties.

```javascript
Object.freeze(foodMap);
```

**Explanation:**  
- After freezing, `foodMap` cannot be changed in any way.

---

## 6. Getting Keys, Values, and Entries
We can convert an object into arrays of keys, values, or key-value pairs.

```javascript
const keys = Object.keys(foodMap);      // ['pizza', 'joice']
const values = Object.values(foodMap);  // [500, 300]
const entries = Object.entries(foodMap);// [['pizza', 500], ['joice', 300]]
```

**Iterating entries:**

```javascript
for (let [key, value] of entries) {
  console.log(`${key}: ${value}`);
}

// Output:
// pizza: 500
// joice: 300
```

---

## 7. Freezing an Object Example
You can also freeze an object to prevent any modifications.

```javascript
let person = Object.freeze({
  name: "John",
  age: 30,
});
```

**Explanation:**  
- `person` object cannot be modified.  
- Trying to change `person.name` or `person.age` will have no effect.
