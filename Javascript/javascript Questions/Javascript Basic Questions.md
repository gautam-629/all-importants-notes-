 1.Write a function to reverse a string without using built-in methods.
 ```javascript
 const reserString=(str:string)=>{
    return str.split('').reverse().join('')
}
const reserString=(str:string)=>{
    let rev:string="";
    for(let i=str.length-1;i>=0;i--){
         rev +=str[i]
    }
    return rev
}
 ```  
2.Write a function to find the factorial of a number.  
```javascript
const factorial=(n:number)=>{
   let result:number=1;
   for(let i=n;i>=1;i--){
        result=result*i
   }
   return result
}
```
3.How do you find the largest number in an array?
```javascript
const arr=[1,100,3,4,5,6,7,8,9]
Math.max(...arr)
const result=arr.reduce((acc,curr)=>curr>acc?curr:acc)
```
4.Write a function to remove duplicates from an array.  
```javascript
// ✅ Function to remove duplicates from an array of objects by 'id'
const removeDuplicateFromArrayOfObject = (arr: any[]) => {
  return arr.reduce((unique, current) => {
    // Check if an object with the same 'id' already exists in 'unique'
    const exists = unique.some(item => item.id === current.id);

    // If not exists, add it to the array
    if (!exists) {
      unique.push(current);
    }

    return unique;
  }, []);
};
const arr = [
  { id: 1, name: "Binod" },
  { id: 1, name: "Gautam" },
  { id: 2, name: "Hero" },
];

const result = removeDuplicateFromArrayOfObject(arr);

console.log(result, 'results');
// ✅ Output: [ { id: 1, name: "Binod" }, { id: 2, name: "Hero" } ]
const removeDublicate=(arr:any[])=>{
   const result= arr.filter((value,index,arr)=>
        return arr.indexOf(value)===index
})
return result
//  return [...new Set(arr)]
}
```
5.How do you check if a variable is an array

```javascript
const fruits = ["apple", "banana", "cherry"];
const name = "Binod";

console.log(Array.isArray(fruits)); 
console.log(Array.isArray(name));
```

6.Write a function to count the occurrences of each character in a string.  
```javascript
function countCharacters(str:string) {
  const counts:any = {};
  for (const char of str) {
    if (counts[char]) {
    counts[char] = counts[char] + 1;
  } else {
  counts[char] = 1;
}
  }
  return counts;
}
const result = countCharacters("hello world");
console.log(result);
```
7.Implement a function that flattens a nested array.  
```javascript
const flattedArray=(arr:any[])=>{
    let result:number[]=[];
    for(const item of arr){
        if(Array.isArray(item)){
            result=result.concat(item)
        }else{
            result.push(item)
        }
    }
    return result
}
const nestedArray=[1,2,3,4,5,[1,2,3,4]]
const result=flattedArray(nestedArray)
console.log(result)
// console.log(nestedArray.flat())
```
8.How do you swap two variables without using a third variable?  
```javascript
const swap=(a:number,b:number)=>{
      let temp:number;
    temp=a
     a=b
     b=temp
      console.log(a,b)
}
const a=50
const b=100
swap(a,b)
```
9.Write a function to find the sum of all numbers in an array.  
```javascript
const arr=[1,2,3,4,5,6,7,8]
let sum:number=0
arr.forEach(v=>sum += v)
console.log(sum)
```