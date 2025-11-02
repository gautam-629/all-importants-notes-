**1. How do you find and update an object in an array by ID?**
- **Use Case**: Update user email by ID in React state or API response
- **Array Methods**: `map()`, `findIndex()`
- **Input**: `users = [{ id: 1, name: "Alice", email: "a@ex.com" }, { id: 2, name: "Bob", email: "b@ex.com" }]`
- **Task**: Update user with id = 2 to have email = "bob@new.com"
- **Expected Output**: `[{ id: 1, name: "Alice", email: "a@ex.com" }, { id: 2, name: "Bob", email: "bob@new.com" }]`
```javascript
function updateUser(data: any[], userId: number, updateEmail: string) {
    return data.map((u)=>u.id==userId?{...u,email:updateEmail}:u)
}
const users = [
  { id: 1, name: "Alice", email: "a@ex.com" },
  { id: 2, name: "Bob", email: "b@ex.com" }
];
const updatedUserData = updateUser(users, 1, "gautambinod629@gmail.com");
console.log(updatedUserData);
```
using findIndex()(modify original array as welll)
```javascript
function updateUser(data: any[], userId: number, updateEmail: string) {
    const index=data.findIndex((u)=>u.id===userId)
    if(index!=-1){
        data[index]={...data[index],email:updateEmail}
    }
    return data;
}
const users = [
  { id: 1, name: "Alice", email: "a@ex.com" },
  { id: 2, name: "Bob", email: "b@ex.com" }
];
const updatedUserData = updateUser(users, 1, "gautambinod629@gmail.com");
console.log(updatedUserData);
console.log(users);
```
**2. How do you remove an object from an array by ID?**
- **Use Case**: Delete a user from database response
- **Array Methods**: `filter()`, `splice()`
- **Input**: `users = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]`
- **Task**: Remove user with id = 1
- **Expected Output**: `[{ id: 2, name: "Bob" }]`
```javascript
function removeuserById(data: any[], userId: number) {
    return data.filter((u)=>u.id!==userId)
}
const users = [
  { id: 1, name: "Alice", email: "a@ex.com" },

  { id: 2, name: "Bob", email: "b@ex.com" }
];
const updatedUserData = removeuserById(users, 1);
console.log(updatedUserData,'updatedUserData');
console.log(users);
```
using splice() and findIndex()
```javascript
function removeuserById(data: any[], userId: number) {
    const index=data.findIndex((u)=>u.id===userId)
    if(index!==-1){
        data.splice(index,1)
    }
    return data;
}
const users = [
  { id: 1, name: "Alice", email: "a@ex.com" },
  { id: 2, name: "Bob", email: "b@ex.com" }
];
const updatedUserData = removeuserById(users, 1);
console.log(updatedUserData,'updatedUserData');
```
**3. How do you merge two arrays of objects by a common key?**
- **Use Case**: Merge DB records with cached data
- **Array Methods**: `map()`, `find()`,
- **Input**: `arr1 = [{ id: 1, name: "Alice" }]`, `arr2 = [{ id: 1, email: "a@ex.com" }]`
- **Expected Output**: `[{ id: 1, name: "Alice", email: "a@ex.com" }]`
```javascript
function mergeArray(arr1:any[],arr2:any[]){
  const merge= arr1.map((obj1)=>{
        const obj2=arr2.find((o)=>o.id===obj1.id || {})
        return {...obj1,...obj2}
     })
     return merge
}
const arr1 = [{ id: 1, name: "Alice" }];
const arr2 = [{ id: 1, email: "afgfhtex.com" },{ id: 1, email: "dghdg.com" }];

const resuult=mergeArray(arr1,arr2)
console.log(resuult)
```
**4. How do you remove duplicates from an array of objects based on a property?**
- **Use Case**: Filter duplicate users before sending to API
- **Array Methods**: `filter()`, `findIndex()`, `reduce()`
- **Input**: `[{ id: 1, email: "a@ex.com" }, { id: 2, email: "b@ex.com" }, { id: 3, email: "a@ex.com" }]`
- **Task**: Remove duplicates based on email
- **Expected Output**: `[{ id: 1, email: "a@ex.com" }, { id: 2, email: "b@ex.com" }]`

```javascript
const removeDeblicateBasedOnEmail=(arr1:any[])=>{
  return arr1.reduce((acc,obj)=>{
       if(!acc.some(item=>item.email===obj.email)){
           acc.push(obj)
       }
        return acc
   },[])
}
const arr1=[{ id: 1, email: "a@ex.com" }, { id: 2, email: "b@ex.com" }, { id: 3, email: "a@ex.com" }]
const result=removeDeblicateBasedOnEmail(arr1)
console.log(result)
```
