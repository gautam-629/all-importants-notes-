## 1.Generics and constraints
TypeScript generics let you write reusable, type-safe code that works across multiple types while still preserving type information. Constraints refine generics by restricting what types are allowed.
1.Basic Generics
```typescript
//example 1
function identity<T>(arg:T):T{
 return arg
}
const num=identity<number>(4)
const str=identity("Hello")
//example 2
interface PaginatedResponse<T>{
    data:T[],
    total:number,
    page:number,
    pageSize:number,
    totalPage:number
}
interface User {
  id: number;
  name: string;
  email: string;
}
const userResponse:PaginatedResponse<User>={
    data:[
         { id: 1, name: "Alice", email: "alice@test.com" },
    { id: 2, name: "Bob", email: "bob@test.com" }
    ],
    page:1,
    pageSize:3,
    total:10,
    totalPage:2
}
```
2.Generic Constraints (`extends`)
```typescript
//T must have a .length
function getLength<T extends {length:number}>(item:T):number{
   return item.length;
}
getLength<string []>(["Binod"])
getLength<string>("hello world")
getLength<number>(123)
```
3.Using `keyof` with Constraints
```typescript
function getProperty<T,K extends keyof T>(obj:T,key:K){
   return obj[key]
}
const user=
    name:"Binod",
    address:"ktm"
}
const result=getProperty(user,"name")
console.log(result)
```
4.Generic Class
```typescript
class Person<T,K> {
  constructor(
    public name: T,
    private age: K
  ) {}
}
const binod=new Person<string,number>("Binod",3)
console.log(binod)
```

## 2. Utility types (Partial, Pick, Omit, Record, ReturnType)
#### 1.Partial:Makes all properties optional.
```typescript
type User = {
  name: string;
  age: number;
};
type PartialUser = Partial<User>;
//Result
{
  name?: string;
  age?: number;
}
```
#### 2.Pick: Select only specific properties from a type.
```typescript
type User = {
  name: string;
  age: number;
  email: string;
};
type UserPreview = Pick<User, "name" | "email">;
//Result
{
  name: string;
  email: string;
}
```
#### 3. Omit Remove specific properties from a type.
```typescript
type User = {
  name: string;
  age: number;
  email: string;
};
type UserWithoutEmail = Omit<User, "email">;
//Result
{
  name: string;
  age: number;
}
```
4.Record:Creates an object type with keys `K` and values `T`.
```typescript
type Roles = "admin" | "user"; //Types can define things interfaces cannot:
const rolePermissions: Record<Roles, number> = {
  admin: 1,
  user: 2
};
//Result
{
  admin: number;
  user: number;
}
```