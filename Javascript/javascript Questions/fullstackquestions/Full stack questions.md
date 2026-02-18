# 100 Essential JavaScript Utility Functions - Questions Format

## Array Manipulation (1-40)

**1. How do you find and update an object in an array by ID?**
- **Use Case**: Update user email by ID in React state or API response
- **Input**: `users = [{ id: 1, name: "Alice", email: "a@ex.com" }, { id: 2, name: "Bob", email: "b@ex.com" }]`
- **Task**: Update user with id = 2 to have email = "bob@new.com"
- **Expected Output**: `[{ id: 1, name: "Alice", email: "a@ex.com" }, { id: 2, name: "Bob", email: "bob@new.com" }]`
  ```typescript
  interface IUser{
    id:number,
    name:string
}
const findUpdate=(users:IUser[],id:number,email:string)=>{
    const result=users.map((user:IUser)=>user.id===id ?({...user,email:email}:user )
    return result
}
const users=[{ id: 1, name: "Alice", email: "a@ex.com" }, { id: 2, name: "Bob", email: "b@ex.com" }]
const updateEmail="gautambinod629@gmail.com"
const id=2;
const result=findUpdate(users,id,updateEmail)
console.log(result)
  ```

**2. How do you remove an object from an array by ID?**
- **Use Case**: Delete a user from database response
- **Input**: `users = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]`
- **Task**: Remove user with id = 1
- **Expected Output**: `[{ id: 2, name: "Bob" }]`
```typescript
interface IUser{
    id:number,
    name:string
}
const findRemove=(users:IUser[],id:number)=>{
    return users.filter((user:IUser)=>user.id!==id)
}
const users=[{ id: 1, name: "Alice", email: "a@ex.com" }, { id: 2, name: "Bob", email: "b@ex.com" }]
const id=2;
const result=findRemove(users,id)
console.log(result)
```

**3. How do you merge two arrays of objects by a common key?**
- **Use Case**: Merge DB records with cached data
- **Input**: `arr1 = [{ id: 1, name: "Alice" }]`, `arr2 = [{ id: 1, email: "a@ex.com" }]`
- **Expected Output**: `[{ id: 1, name: "Alice", email: "a@ex.com" }]`
 ```typescript
interface IUser{
    id:number,
    name?:string,
    email?:string
}
const mergeByKey=(users1:IUser[],users2:IUser[])=>{
return users1.map((user1:IUser)=>{
    const matchObject=users2.find((user2)=>user2.id===user1.id)
    return matchObject?{...user1,...matchObject}:user1
})
}
const arr1:IUser[] = [{ id: 1, name: "Alice" },{ id: 6, name: "Alice" }]
const arr2:IUser[] = [{ id: 1, email: "a@ex.com" },{ id: 2, email: "a@ex.com" }]
const result=mergeByKey(arr1,arr2)
console.log(result)
 ```

**4. How do you remove duplicates from an array of objects based on a property?**
- **Use Case**: Filter duplicate users before sending to API
- **Array Methods**: `filter()`, `findIndex()`, `reduce()`
- **Input**: `[{ id: 1, email: "a@ex.com" }, { id: 2, email: "b@ex.com" }, { id: 3, email: "a@ex.com" }]`
- **Task**: Remove duplicates based on email
- **Expected Output**: `[{ id: 1, email: "a@ex.com" }, { id: 2, email: "b@ex.com" }]`

**5. How do you sort an array of objects by a property?**
- **Use Case**: Sort products by price before rendering
- **Input**: `[{ id: 1, price: 50 }, { id: 2, price: 30 }]`
- **Task**: Sort by price ascending
- **Expected Output**: `[{ id: 2, price: 30 }, { id: 1, price: 50 }]`  
```typescript
interface IProduct{
    id:number,
    price:number
}
enum SortByEnum{
    id='id',
    price='price'
}
const SortUtil=(products:IProduct[],sortBy:SortByEnum)=>{
    switch(sortBy){
       case SortByEnum.price:
    return products.sort((a:IProduct,b:IProduct)=>a.price-b.price)
    case SortByEnum.id:
         return products.sort((a:IProduct,b:IProduct)=>a.price-b.price)
    }
}
const products:IProduct[]=[{ id: 1, price: 50 }, { id: 2, price: 30 }]
const result=SortUtil(products,SortByEnum.price)
console.log(result)
```

**6. How do you sort an array of objects by a nested property?**
- **Use Case**: Sort users by address zip code
- **Array Methods**: `sort()`
- **Input**: `[{ name: "Alice", address: { zip: 2000 } }, { name: "Bob", address: { zip: 1000 } }]`
- **Task**: Sort by address.zip ascending
- **Expected Output**: `[{ name: "Bob", address: { zip: 1000 } }, { name: "Alice", address: { zip: 2000 } }]`

**7. How do you flatten a nested array?**
- **Use Case**: Merge multiple arrays of API data
- **Array Methods**: `flat()`, `concat()`, `reduce()`
- **Input**: `[[1, 2], [3, 4]]`
- **Expected Output**: `[1, 2, 3, 4]`

**8. How do you flatten a nested array of objects and extract specific values?**
- **Use Case**: Extract all IDs from nested menu tree
- **Array Methods**: `flatMap()`, `map()`, recursive approach
- **Input**: `[{ id: 1, children: [{ id: 2 }, { id: 3, children: [{ id: 4 }] }] }]`
- **Task**: Extract all id values recursively
- **Expected Output**: `[1, 2, 3, 4]`

**9. How do you chunk an array into smaller arrays of specified size?**
- **Use Case**: Paginate large API results
- **Array Methods**: `slice()`, `reduce()`, `push()`
- **Input**: `[1, 2, 3, 4, 5]`, `chunkSize = 2`
- **Expected Output**: `[[1, 2], [3, 4], [5]]`

**10. How do you find the difference between two arrays of objects?**
- **Use Case**: Detect new users not in cached data
- **Input**: `arr1 = [{ id: 1 }, { id: 2 }]`, `arr2 = [{ id: 2 }]`
- **Task**: Find objects in arr1 that are not in arr2
- **Expected Output**: `[{ id: 1 }]`
 ```typescript
interface Item{
    id:number
}
const difference=(arr1:Item[],arr2:Item[])=>{
    return arr1.filter((obj1:Item)=>!arr2.some((obj2:Item)=>obj2.id===obj1.id))
}
const arr1:Item[] = [{ id: 1 }, { id: 2 }]
const arr2:Item[] = [{ id: 2 }]
const result=difference(arr1,arr2)
console.log(result)
 ```

**11. How do you find the intersection between two arrays of objects?**
- **Use Case**: Find users present in both DB and cache
- **Input**: `arr1 = [{ id: 1 }, { id: 2 }]`, `arr2 = [{ id: 2 }, { id: 3 }]`
- **Task**: Find objects present in both arrays
- **Expected Output**: `[{ id: 2 }]`
 ```typescript
interface IUser{
    id:number,
}
const interSection=(users1:IUser[],users2:IUser[])=>{
  return users1.filter((users1:IUser)=>users2.some((user2)=>user2.id===users1.id));
}
const arr1:IUser[] = [{ id: 1 }, { id: 2 }]
 const arr2:IUser[] = [{ id: 2 }, { id: 3 }]
const result=interSection(arr1,arr2)
console.log(result)
 ```

**12. How do you create a lookup map from an array of objects?**
- **Use Case**: Quickly access user by ID
- **Input**: `[{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]`
- **Task**: Create map with id as key
- **Expected Output**: `{ 1: { id: 1, name: "Alice" }, 2: { id: 2, name: "Bob" } }`
 ```typescript
interface IUser{
   id:number,
   name:string
}
const loopUp=(users:IUser[])=>{
   // const result:Record<number,IUser>={}
   // for(const user of  users){
   //    result[user.id]=user
   // }
   // return result
   const result=users.reduce((acc:Record<number,IUser>,curr:IUser)=>{
                acc[curr.id]=curr
                return acc
   },{} as Record<number,IUser>)
   return result
}
const users:IUser[]=[{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]
const result=loopUp(users)
console.log(result)
 ```

**13. How do you count occurrences of values in an array?**
- **Use Case**: Count how many times a product was purchased
- **Input**: `["apple", "banana", "apple"]`
- **Expected Output**: `{ apple: 2, banana: 1 }`
```typescript
const countProduct=(products:string[]):Record<string,number>=>{
   //   const result:Record<string,number>={}
   //   for(const product of products){
   //      if(result[product]){
   //       result[product]++;
   //      }else{
   //       result[product]=1
   //      }
   //   }
   //   return result
  const result= products.reduce((acc:Record<string,number>,curr)=>{
          if(acc[curr]){
            acc[curr] ++
          }else{
            acc[curr]=1
          }
          return acc
   },{})
   return result;
}
const product:string[]=["apple", "banana", "apple"]
const result=countProduct(product)
console.log(result)
```
**14. How do you group an array of objects by a property?**
- **Use Case**: Group orders by userId
- **Input**: `[{ userId: 1, order: 10 }, { userId: 2, order: 20 }, { userId: 1, order: 30 }]`
- **Task**: Group by userId
- **Expected Output**: `{ 1: [{ userId: 1, order: 10 }, { userId: 1, order: 30 }], 2: [{ userId: 2, order: 20 }] }`
```typescript
interface IOrder {
  userId: number;
  order: number;
}
const groupById = (orders: IOrder[]) => {
  return orders.reduce<Record<number, IOrder[]>>((acc, curr) => {
    if (!acc[curr.userId]) {
      acc[curr.userId] = [];
    }
    acc[curr.userId].push(curr);
    return acc;
  }, {});
};

const orders: IOrder[] = [
  { userId: 1, order: 10 },
  { userId: 2, order: 20 },
  { userId: 1, order: 30 }
];
console.log(groupById(orders));
```

**15. How do you aggregate sum of amounts per user?**
- **Use Case**: Calculate total spent per user
- **Array Methods**: `reduce()`, `groupBy()` + `reduce()`
- **Input**: `[{ userId: 1, amount: 50 }, { userId: 1, amount: 30 }, { userId: 2, amount: 20 }]`
- **Task**: Sum amounts by userId
- **Expected Output**: `{ 1: 80, 2: 20 }`

**16. How do you find max and min values in an array of objects?**
- **Use Case**: Determine highest and lowest order amount
- **Input**: `[{ order: 100 }, { order: 50 }, { order: 150 }]`
- **Task**: Find max and min order values
- **Expected Output**: `{ max: 150, min: 50 }`
```typescript
 interface Iproduct{
   order:number
 }
 const productOrder=(products:Iproduct[])=>{
   const order=products.map((p)=>p.order)
   const highest=Math.max(...order);
   const lowest=Math.min(...order
   return ({
      max:highest
      min:lowest
   })
 }
 const product:Iproduct[]=[{ order: 100 }, { order: 50 }, { order: 150 }]
 const result=productOrder(product)
 console.log(result)
```

**17. How do you convert an array of objects to a dictionary by name?**
- **Use Case**: Fast lookup of product by name
- **Array Methods**: `reduce()`, `Object.fromEntries()`, `map()`
- **Input**: `[{ name: "apple", price: 50 }, { name: "banana", price: 30 }]`
- **Task**: Create dictionary with name as key
- **Expected Output**: `{ apple: { name: "apple", price: 50 }, banana: { name: "banana", price: 30 } }`

**18. How do you normalize nested array data by flattening it?**
- **Use Case**: Flatten categories for dropdown
- **Array Methods**: `flatMap()`, `map()`, recursive approach
- **Input**: `[{ id: 1, children: [{ id: 2 }] }]`
- **Task**: Flatten structure and remove children property
- **Expected Output**: `[{ id: 1 }, { id: 2 }]`

**19. How do you transform a posts array to group by user?**
- **Use Case**: API response grouped by user
- **Array Methods**: `reduce()`, `map()`, `filter()`
- **Input**: `[{ id: 1, title: "Hi", user: { id: 1, name: "Alice" } }, { id: 2, title: "Hello", user: { id: 1, name: "Alice" } }]`
- **Task**: Group posts by user
- **Expected Output**: `{ user: { id: 1, name: "Alice" }, posts: [{ id: 1, title: "Hi" }, { id: 2, title: "Hello" }] }`

**20. How do you extract unique values from nested arrays?**
- **Use Case**: Get all unique skills of employees
- **Array Methods**: `flatMap()`, `reduce()`, `Set`, `filter()`
- **Input**: `[{ id: 1, skills: ["JS", "React"] }, { id: 2, skills: ["Node", "JS"] }]`
- **Task**: Extract all unique skills
- **Expected Output**: `["JS", "React", "Node"]`

**21. How do you partition an array based on a condition?**
- **Use Case**: Separate active and inactive users
- **Array Methods**: `reduce()`, `filter()`
- **Input**: `[{ id: 1, active: true }, { id: 2, active: false }, { id: 3, active: true }]`
- **Task**: Partition by active status
- **Expected Output**: `{ active: [{ id: 1, active: true }, { id: 3, active: true }], inactive: [{ id: 2, active: false }] }`

**22. How do you shuffle an array randomly?**
- **Use Case**: Randomize quiz questions or playlist order
- **Array Methods**: `sort()`, `slice()` with Fisher-Yates algorithm
- **Input**: `[1, 2, 3, 4, 5]`
- **Expected Output**: `[3, 1, 5, 2, 4]` (random order)

**23. How do you rotate an array left or right by n positions?**
- **Use Case**: Carousel image rotation or circular buffer
- **Array Methods**: `slice()`, `concat()`, `splice()`
- **Input**: `[1, 2, 3, 4, 5]`, `rotateRight = 2`
- **Expected Output**: `[4, 5, 1, 2, 3]`

**24. How do you find the most/least frequent element in an array?**
- **Use Case**: Find most popular product or least used feature
- **Array Methods**: `reduce()`, `entries()`, `sort()`, `map()`
- **Input**: `["apple", "banana", "apple", "orange", "banana", "apple"]`
- **Expected Output**: `{ most: "apple", least: "orange" }`

**25. How do you implement array pagination?**
- **Use Case**: Display paginated results in UI
- **Array Methods**: `slice()`, `Math.ceil()`
- **Input**: `data = [1,2,3,4,5,6,7,8,9,10]`, `page = 2`, `pageSize = 3`
- **Expected Output**: `{ data: [4, 5, 6], page: 2, totalPages: 4, hasNext: true, hasPrev: true }`

**26. How do you zip multiple arrays together?**
- **Use Case**: Combine separate arrays of names, ages, emails
- **Array Methods**: `map()`, `reduce()`, destructuring
- **Input**: `names = ["Alice", "Bob"]`, `ages = [25, 30]`, `emails = ["a@ex.com", "b@ex.com"]`
- **Expected Output**: `[{ name: "Alice", age: 25, email: "a@ex.com" }, { name: "Bob", age: 30, email: "b@ex.com" }]`

**27. How do you transpose a 2D array (matrix)?**
- **Use Case**: Flip table rows and columns for data visualization
- **Array Methods**: `map()`, array indexing
- **Input**: `[[1, 2, 3], [4, 5, 6]]`
- **Expected Output**: `[[1, 4], [2, 5], [3, 6]]`

**28. How do you find all permutations of an array?**
- **Use Case**: Generate all possible arrangements for scheduling
- **Array Methods**: `reduce()`, `map()`, `flatMap()`, recursion
- **Input**: `[1, 2, 3]`
- **Expected Output**: `[[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]`

**29. How do you implement binary search on a sorted array?**
- **Use Case**: Fast search in large sorted datasets
- **Array Methods**: `slice()`, array indexing, recursion/iteration
- **Input**: `sortedArray = [1, 3, 5, 7, 9, 11]`, `target = 7`
- **Expected Output**: `{ found: true, index: 3, iterations: 2 }`

**30. How do you merge multiple sorted arrays into one sorted array?**
- **Use Case**: Combine sorted results from different APIs
- **Array Methods**: `concat()`, `sort()`, or merge algorithm with `push()`, `shift()`
- **Input**: `arr1 = [1, 5, 9]`, `arr2 = [2, 6, 8]`, `arr3 = [3, 4, 7]`
- **Expected Output**: `[1, 2, 3, 4, 5, 6, 7, 8, 9]`

**31. How do you find the longest increasing subsequence in an array?**
- **Use Case**: Stock price analysis, performance metrics trending
- **Array Methods**: `reduce()`, `filter()`, dynamic programming approach
- **Input**: `[10, 22, 9, 33, 21, 50, 41, 60]`
- **Expected Output**: `[10, 22, 33, 50, 60]` (length: 5)

**32. How do you implement sliding window maximum?**
- **Use Case**: Moving averages, real-time analytics
- **Array Methods**: `slice()`, `reduce()`, `Math.max()`, deque approach
- **Input**: `array = [1, 3, -1, -3, 5, 3, 6, 7]`, `windowSize = 3`
- **Expected Output**: `[3, 3, 5, 5, 6, 7]`

**33. How do you find all subarrays that sum to a target value?**
- **Use Case**: Budget allocation, payment combinations
- **Array Methods**: `slice()`, `reduce()`, nested loops or hash map approach
- **Input**: `array = [1, 2, 3, 4]`, `targetSum = 5`
- **Expected Output**: `[[2, 3], [1, 4]]`

**34. How do you implement array-based stack operations?**
- **Use Case**: Undo/redo functionality, function call stack simulation
- **Array Methods**: `push()`, `pop()`, `length`, `at()`
- **Operations**: `push(5)`, `push(10)`, `pop()`, `peek()`, `isEmpty()`
- **Expected Output**: `{ stack: [5], popped: 10, top: 5, empty: false }`

**35. How do you implement array-based queue operations?**
- **Use Case**: Task processing, breadth-first search
- **Array Methods**: `push()`, `shift()`, `length`, `at()`
- **Operations**: `enqueue(1)`, `enqueue(2)`, `dequeue()`, `front()`, `isEmpty()`
- **Expected Output**: `{ queue: [2], dequeued: 1, front: 2, empty: false }`

**36. How do you find the kth largest/smallest element in an array?**
- **Use Case**: Top performers, percentile calculations
- **Array Methods**: `sort()`, `slice()`, or quickselect algorithm
- **Input**: `[3, 2, 1, 5, 6, 4]`, `k = 2` (2nd largest)
- **Expected Output**: `5`

**37. How do you implement array union, intersection, and difference operations?**
- **Use Case**: Set operations on user permissions, feature flags
- **Array Methods**: `filter()`, `includes()`, `concat()`, `Set` operations
- **Input**: `arr1 = [1, 2, 3, 4]`, `arr2 = [3, 4, 5, 6]`
- **Expected Output**: `{ union: [1,2,3,4,5,6], intersection: [3,4], difference: [1,2] }`

**38. How do you detect and remove cycles in array references?**
- **Use Case**: Prevent infinite loops in hierarchical data
- **Array Methods**: `map()`, `filter()`, `WeakSet` for tracking
- **Input**: `[{ id: 1, refs: [2] }, { id: 2, refs: [1, 3] }, { id: 3, refs: [] }]`
- **Task**: Detect circular reference between id 1 and 2
- **Expected Output**: `{ hasCycles: true, cycles: [[1, 2]] }`

**39. How do you implement array-based LRU (Least Recently Used) cache?**
- **Use Case**: Cache management with size limits
- **Array Methods**: `findIndex()`, `splice()`, `unshift()`, `pop()`
- **Input**: `capacity = 3`, operations: `set(1,"a")`, `set(2,"b")`, `set(3,"c")`, `get(1)`, `set(4,"d")`
- **Expected Output**: `{ cache: [1:"a", 4:"d", 3:"c"], evicted: 2:"b" }`

**40. How do you generate all combinations of array elements?**
- **Use Case**: Feature testing combinations, meal planning
- **Array Methods**: `reduce()`, `flatMap()`, `map()`, recursion
- **Input**: `[["red", "blue"], ["small", "large"], ["cotton", "wool"]]`
- **Expected Output**: `[["red","small","cotton"], ["red","small","wool"], ["red","large","cotton"], ...]`

## String Manipulation (41-60)

**41. How do you convert a string to camelCase?**
- **Use Case**: Normalize API keys
- **Input**: `"user_name"`
- **Expected Output**: `"userName"`

**22. How do you convert a string to snake_case?**
- **Use Case**: Format object keys for backend
- **Input**: `"userName"`
- **Expected Output**: `"user_name"`

**23. How do you convert a string to kebab-case?**
- **Use Case**: URL slug generation
- **Input**: `"User Name"`
- **Expected Output**: `"user-name"`

**24. How do you capitalize the first letter of each word?**
- **Use Case**: Display proper names in UI
- **Input**: `"hello world"`
- **Expected Output**: `"Hello World"`

**25. How do you truncate a string with ellipsis?**
- **Use Case**: Preview long text
- **Input**: `"This is a long sentence"`, `maxLength = 10`
- **Expected Output**: `"This is a ..."`

**26. How do you validate an email with regex?**
- **Use Case**: Validate user input
- **Input**: `"test@example.com"`
- **Expected Output**: `true`

**27. How do you mask sensitive parts of a string?**
- **Use Case**: Hide credit card or password
- **Input**: `"1234567890123456"`
- **Task**: Mask all but last 4 characters
- **Expected Output**: `"************3456"`

**28. How do you extract numbers from a string?**
- **Use Case**: Parse price or IDs
- **Input**: `"Order123"`
- **Expected Output**: `123`

**29. How do you count word frequency in a string?**
- **Use Case**: Analyze text data
- **Input**: `"apple banana apple"`
- **Expected Output**: `{ apple: 2, banana: 1 }`

**30. How do you convert a query string to an object?**
- **Use Case**: Parse URL parameters
- **Input**: `"?a=1&b=2"`
- **Expected Output**: `{ a: "1", b: "2" }`

**31. How do you convert an object to a query string?**
- **Use Case**: Send GET requests
- **Input**: `{ a: 1, b: 2 }`
- **Expected Output**: `"a=1&b=2"`

**32. How do you format a number with commas?**
- **Use Case**: Display large numbers in UI
- **Input**: `1000000`
- **Expected Output**: `"1,000,000"`

**33. How do you convert object keys from snake_case to camelCase?**
- **Use Case**: Normalize API response
- **Input**: `{ user_name: "Alice", first_name: "Alice" }`
- **Expected Output**: `{ userName: "Alice", firstName: "Alice" }`

**34. How do you convert object keys from camelCase to snake_case?**
- **Use Case**: Send data to backend
- **Input**: `{ userName: "Alice", firstName: "Alice" }`
- **Expected Output**: `{ user_name: "Alice", first_name: "Alice" }`

**35. How do you remove HTML tags from a string?**
- **Use Case**: Sanitize user input
- **Input**: `"<p>Hello <strong>World</strong></p>"`
- **Expected Output**: `"Hello World"`

**36. How do you escape special HTML characters?**
- **Use Case**: Prevent XSS attacks
- **Input**: `"<script>alert('xss')</script>"`
- **Expected Output**: `"&lt;script&gt;alert('xss')&lt;/script&gt;"`

**37. How do you generate a random alphanumeric string?**
- **Use Case**: Temporary password or token generation
- **Input**: `length = 8`
- **Expected Output**: `"a1B2c3D4"` (example)

**38. How do you convert markdown to plain text?**
- **Use Case**: Display clean text preview
- **Input**: `"**bold** and *italic* text"`
- **Expected Output**: `"bold and italic text"`

**39. How do you reverse a string?**
- **Use Case**: UI text effects or algorithms
- **Input**: `"hello"`
- **Expected Output**: `"olleh"`

**60. How do you check if a string is a palindrome?**
- **Use Case**: Validate codes or IDs
- **Input**: `"madam"`
- **Expected Output**: `true`

## Object Manipulation (61-70)

**61. How do you create a deep clone of an object?**
- **Use Case**: Prevent mutation of state
- **Input**: `{ a: 1, b: { c: 2, d: [3, 4] } }`
- **Expected Output**: Complete deep copy with no shared references

**42. How do you create a shallow clone of an object?**
- **Use Case**: Quick copy of simple objects
- **Input**: `{ a: 1, b: 2, c: 3 }`
- **Expected Output**: `{ a: 1, b: 2, c: 3 }` (new object, same values)

**43. How do you merge two objects deeply?**
- **Use Case**: Combine API defaults with user data
- **Input**: `obj1 = { a: 1, b: { c: 2 } }`, `obj2 = { b: { d: 3 }, e: 4 }`
- **Expected Output**: `{ a: 1, b: { c: 2, d: 3 }, e: 4 }`

**44. How do you remove specific keys from an object?**
- **Use Case**: Strip sensitive info before sending to API
- **Input**: `{ username: "Alice", password: "123", email: "alice@ex.com" }`, `keysToRemove = ["password"]`
- **Expected Output**: `{ username: "Alice", email: "alice@ex.com" }`

**45. How do you pick specific keys from an object?**
- **Use Case**: Limit data fields for frontend
- **Input**: `{ a: 1, b: 2, c: 3, d: 4 }`, `keysToPick = ["a", "c"]`
- **Expected Output**: `{ a: 1, c: 3 }`

**46. How do you omit specific keys from an object?**
- **Use Case**: Filter out private fields
- **Input**: `{ public: "data", private: "secret", internal: "info" }`, `keysToOmit = ["private", "internal"]`
- **Expected Output**: `{ public: "data" }`

**47. How do you freeze an object and check if it's frozen?**
- **Use Case**: Protect configuration objects from modification
- **Input**: `{ apiUrl: "https://api.com", timeout: 5000 }`
- **Task**: Freeze the object and verify immutability

**48. How do you compare two objects for deep equality?**
- **Use Case**: Check if state has changed
- **Input**: `obj1 = { a: 1, b: { c: 2 } }`, `obj2 = { a: 1, b: { c: 2 } }`
- **Expected Output**: `true`

**49. How do you safely access a nested property without errors?**
- **Use Case**: Avoid "Cannot read property of undefined" errors
- **Input**: `obj = { user: { profile: { name: "Alice" } } }`, `path = "user.profile.name"`
- **Expected Output**: `"Alice"`

**70. How do you set a nested property dynamically?**
- **Use Case**: Update nested state in Redux or React
- **Input**: `obj = {}`, `path = "user.profile.name"`, `value = "Alice"`
- **Expected Output**: `{ user: { profile: { name: "Alice" } } }`

## Async Operations (71-80)

**71. How do you implement a delay function with Promise?**
- **Use Case**: Wait before API retry
- **Input**: `delay = 1000` (milliseconds)
- **Expected Output**: Promise that resolves after 1 second

**52. How do you run multiple Promises in parallel?**
- **Use Case**: Fetch multiple APIs simultaneously
- **Input**: `[fetchUser(), fetchPosts(), fetchComments()]`
- **Expected Output**: Array of results `[userData, postsData, commentsData]`

**53. How do you run multiple Promises in sequence?**
- **Use Case**: Sequential API calls where each depends on the previous
- **Input**: `[() => fetchUser(), (user) => fetchUserPosts(user.id), (posts) => fetchComments(posts)]`
- **Expected Output**: Final result after all sequential operations

**54. How do you retry a Promise n times on failure?**
- **Use Case**: Resilient API requests with retry logic
- **Input**: `promiseFunction = fetchData`, `maxRetries = 3`
- **Expected Output**: Resolves on success or rejects after 3 failed attempts

**55. How do you add a timeout wrapper to a Promise?**
- **Use Case**: Prevent hanging API requests
- **Input**: `promise = fetch('/api/data')`, `timeout = 5000`
- **Expected Output**: Resolves with data or rejects with timeout error

**56. How do you fetch multiple APIs and combine their results?**
- **Use Case**: Aggregate data from different microservices
- **Input**: `urls = ['/api/users', '/api/orders', '/api/products']`
- **Expected Output**: Combined object with all API responses

**57. How do you throttle API calls to limit request rate?**
- **Use Case**: Respect API rate limits
- **Input**: `apiFunction`, `maxCallsPerSecond = 2`
- **Expected Output**: Function that limits execution to 2 calls per second

**58. How do you debounce a function that returns a Promise?**
- **Use Case**: Search input with API call
- **Input**: `searchFunction`, `delay = 300`
- **Expected Output**: Function that only executes after 300ms of inactivity

**59. How do you implement request cancellation for Promises?**
- **Use Case**: Cancel fetch request when user navigates away
- **Input**: `fetch(url, { signal: abortController.signal })`
- **Expected Output**: Cancelable request that can be aborted

**60. How do you queue async tasks with concurrency limits?**
- **Use Case**: Process large number of tasks without overwhelming the system
- **Input**: `tasks = [task1, task2, task3, task4, task5]`, `concurrency = 2`
- **Expected Output**: Execute max 2 tasks simultaneously until all complete

## Date & Time Utilities (61-70)

**61. How do you format a date to ISO string?**
- **Use Case**: Send dates to API in standard format
- **Input**: `new Date("2024-01-15 14:30:00")`
- **Expected Output**: `"2024-01-15T14:30:00.000Z"`

**62. How do you calculate days between two dates?**
- **Use Case**: Show subscription days remaining
- **Input**: `startDate = "2024-01-01"`, `endDate = "2024-01-15"`
- **Expected Output**: `14`

**63. How do you add or subtract days from a date?**
- **Use Case**: Calculate due dates or expiry dates
- **Input**: `date = "2024-01-01"`, `daysToAdd = 7`
- **Expected Output**: `"2024-01-08"`

**64. How do you format a date to relative time?**
- **Use Case**: Show "2 hours ago" in social media feeds
- **Input**: `date = new Date(Date.now() - 2 * 60 * 60 * 1000)` (2 hours ago)
- **Expected Output**: `"2 hours ago"`

**65. How do you check if a date falls on a weekend?**
- **Use Case**: Business day calculations
- **Input**: `new Date("2024-01-13")` (Saturday)
- **Expected Output**: `true`

**66. How do you get the start and end of a week/month/year?**
- **Use Case**: Generate analytics date ranges
- **Input**: `date = "2024-01-15"`, `period = "month"`
- **Expected Output**: `{ start: "2024-01-01", end: "2024-01-31" }`

**67. How do you parse multiple date formats into a standard format?**
- **Use Case**: Handle various user date inputs
- **Input**: `["01/15/2024", "2024-01-15", "15 Jan 2024", "January 15, 2024"]`
- **Expected Output**: All converted to consistent Date objects

**68. How do you convert a date between timezones?**
- **Use Case**: Display user's local time
- **Input**: `date = "2024-01-15T15:00:00Z"`, `fromTZ = "UTC"`, `toTZ = "America/New_York"`
- **Expected Output**: `"2024-01-15T10:00:00-05:00"`

**69. How do you validate if a date string is valid?**
- **Use Case**: Validate user date input in forms
- **Input**: `"2024-02-30"` (invalid date)
- **Expected Output**: `false`

**70. How do you format duration in human-readable format?**
- **Use Case**: Show video length or elapsed time
- **Input**: `seconds = 3661` (1 hour, 1 minute, 1 second)
- **Expected Output**: `"1h 1m 1s"`

## Validation & Type Checking (71-80)

**71. How do you check if a value is empty (null, undefined, empty string, empty array, empty object)?**
- **Use Case**: Form validation and data cleaning
- **Input**: `["", null, undefined, [], {}, "   ", 0, false]`
- **Expected Output**: `[true, true, true, true, true, true, false, false]`

**72. How do you validate a phone number format?**
- **Use Case**: Contact form validation
- **Input**: `["+1-555-123-4567", "555.123.4567", "(555) 123-4567", "invalid"]`
- **Expected Output**: `[true, true, true, false]`

**73. How do you check if an object has a specific nested property?**
- **Use Case**: Safe property access before using values
- **Input**: `obj = { user: { profile: { name: "Alice" } } }`, `path = "user.profile.age"`
- **Expected Output**: `false`

**74. How do you validate URL format?**
- **Use Case**: Link validation in forms and content
- **Input**: `["https://example.com", "http://test.org", "invalid-url", "ftp://files.com"]`
- **Expected Output**: `[true, true, false, true]`

**75. How do you implement custom type validation at runtime?**
- **Use Case**: API response validation and type safety
- **Input**: `value = "123"`, `expectedType = "number"`, `shouldCoerce = true`
- **Expected Output**: `{ isValid: true, value: 123, originalType: "string" }`

**76. How do you validate a credit card number using Luhn algorithm?**
- **Use Case**: Payment form validation
- **Input**: `"4532015112830366"` (valid Visa number)
- **Expected Output**: `{ isValid: true, type: "Visa" }`

**77. How do you check if a string contains only letters?**
- **Use Case**: Name field validation
- **Input**: `["Alice", "John123", "Mary-Jane", "José"]`
- **Expected Output**: `[true, false, false, true]`

**78. How do you validate password strength?**
- **Use Case**: User registration security requirements
- **Input**: `"MyPassword123!"`
- **Expected Output**: `{ score: 4, strength: "Strong", feedback: ["Good length", "Contains numbers", "Contains symbols"] }`

**79. How do you check if an array contains only unique values?**
- **Use Case**: Data integrity validation
- **Input**: `[[1, 2, 3], [1, 2, 2], ["a", "b", "c"], ["a", "a"]]`
- **Expected Output**: `[true, false, true, false]`

**80. How do you validate and parse JSON strings safely?**
- **Use Case**: API payload validation
- **Input**: `['{"name": "John"}', '{"invalid": json}', 'not json at all']`
- **Expected Output**: `[{name: "John"}, null, null]`

## Performance & Caching (81-90)

**81. How do you implement memoization for expensive function calls?**
- **Use Case**: Cache results of costly calculations like Fibonacci
- **Input**: `expensiveFunction = fibonacci`, `input = 40`
- **Expected Output**: First call computes, subsequent calls return cached result

**82. How do you create an LRU (Least Recently Used) cache?**
- **Use Case**: Limit memory usage while maintaining frequently accessed data
- **Input**: `maxSize = 3`, operations: `set("a", 1), set("b", 2), set("c", 3), set("d", 4)`
- **Expected Output**: Cache contains `{b: 2, c: 3, d: 4}` (evicted "a")

**83. How do you debounce expensive operations?**
- **Use Case**: Prevent excessive API calls during user input
- **Input**: `searchFunction`, `delay = 300ms`, rapid inputs: "a", "ap", "app", "appl", "apple"
- **Expected Output**: Only final "apple" search executes after 300ms pause

**84. How do you throttle high-frequency events?**
- **Use Case**: Limit scroll/resize event handlers for performance
- **Input**: `scrollHandler`, `interval = 100ms`, continuous scroll events
- **Expected Output**: Handler executes maximum once every 100ms

**85. How do you batch multiple operations for efficiency?**
- **Use Case**: Combine multiple API requests into fewer calls
- **Input**: `requests = [req1, req2, req3, req4, req5]`, `batchSize = 2`
- **Expected Output**: `[[req1, req2], [req3, req4], [req5]]`

**86. How do you implement lazy loading for function execution?**
- **Use Case**: Defer expensive computations until actually needed
- **Input**: `expensiveFunction = () => heavyCalculation()`
- **Expected Output**: Function wrapper that only executes on first access

**87. How do you create an observable pattern for reactive programming?**
- **Use Case**: React to data changes across multiple components
- **Input**: `observable.subscribe(callback)`, `observable.next(newValue)`
- **Expected Output**: All subscribers receive notification when value changes

**88. How do you implement a simple event emitter?**
- **Use Case**: Custom event system for decoupled communication
- **Input**: `emitter.on("userLogin", handler)`, `emitter.emit("userLogin", userData)`
- **Expected Output**: Handler called with userData when event is emitted

**89. How do you create a worker pool for CPU-intensive tasks?**
- **Use Case**: Offload heavy computations to prevent UI blocking
- **Input**: `tasks = [task1, task2, task3, task4]`, `workerCount = 2`
- **Expected Output**: Tasks distributed across 2 web workers for parallel processing

**90. How do you monitor and report memory usage?**
- **Use Case**: Track application performance and detect memory leaks
- **Expected Output**: `{ usedJSHeapSize: "45MB", totalJSHeapSize: "67MB", jsHeapSizeLimit: "2GB" }`

## Browser & DOM Utilities (91-100)

**91. How do you detect browser type and version?**
- **Use Case**: Implement browser-specific features or polyfills
- **Input**: `navigator.userAgent`
- **Expected Output**: `{ browser: "Chrome", version: "120.0.6099.109", mobile: false }`

**92. How do you copy text to clipboard programmatically?**
- **Use Case**: Copy share links, code snippets, or user-generated content
- **Input**: `text = "Hello, World!"`
- **Expected Output**: Text copied to clipboard, return success/failure status

**93. How do you trigger file download with custom data?**
- **Use Case**: Export user data, generate reports, download configurations
- **Input**: `data = "Name,Email\nJohn,john@ex.com"`, `filename = "users.csv"`, `mimeType = "text/csv"`
- **Expected Output**: Browser downloads file with specified content

**94. How do you detect device type and screen orientation?**
- **Use Case**: Responsive design decisions and mobile-specific features
- **Expected Output**: `{ deviceType: "mobile", orientation: "portrait", screenSize: "small" }`

**95. How do you validate file uploads (type, size, count)?**
- **Use Case**: Secure file upload with client-side validation
- **Input**: `files = [file1, file2]`, `maxSize = "5MB"`, `allowedTypes = ["image/*", "application/pdf"]`
- **Expected Output**: `{ valid: [true, false], errors: [null, "File too large"], totalSize: "3.2MB" }`

**96. How do you convert uploaded images to base64 for preview?**
- **Use Case**: Show image preview before upload to server
- **Input**: `imageFile = File object from input[type="file"]`
- **Expected Output**: `"data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ..."`

**97. How do you implement smooth scrolling to elements?**
- **Use Case**: Navigation within single-page applications
- **Input**: `targetElement = "#section2"`, `behavior = "smooth"`, `offset = 80`
- **Expected Output**: Page smoothly scrolls to element with 80px offset

**98. How do you create localStorage with automatic expiry?**
- **Use Case**: Temporary data caching with automatic cleanup
- **Input**: `key = "userSession"`, `data = {token: "abc123"}`, `expiryHours = 24`
- **Expected Output**: Data stored and automatically removed after 24 hours

**99. How do you get element dimensions and viewport position?**
- **Use Case**: Dynamic positioning for tooltips, dropdowns, modals
- **Input**: `element = document.querySelector(".tooltip-trigger")`
- **Expected Output**: `{ width: 150, height: 30, top: 200, left: 100, inViewport: true }`

**100. How do you implement intersection observer for lazy loading?**
- **Use Case**: Performance optimization by loading content when visible
- **Input**: `elements = [img1, img2, img3]`, `threshold = 0.1`, `rootMargin = "50px"`
- **Expected Output**: Elements load when 10% visible with 50px margin

---