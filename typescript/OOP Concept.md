Object-Oriented Programming (OOP) in TypeScript is about structuring code using classes and objects, with concepts like encapsulation, inheritance, polymorphism, and abstraction.
- **Encapsulation:** Wrapping data and methods together in a class while restricting direct access to some parts of the object.
- **Inheritance:** Allowing one class to reuse properties and methods of another class.
- **Polymorphism:** Enabling one method or interface to behave differently depending on the object that uses it.
- **Abstraction:** Hiding complex implementation details and showing only the essential features of an object.
### 1.Classes & Properties
```typescript
class Animal{
    public name:string; // public by default
    private age:number; // only accessible within Animal class
    protected sound:string //only accessible in Animal & subClass
    constructor(name:string,age:number,sound:string){
            this.name=name;
            this.age=age;
            this.sound=sound
    }  
    getAge():Number{
      return this.age //private access via method
    }
}
```
Note: TypeScript adds **access modifiers** (`public`, `private`, `protected`) to control what's visible from outside a class.
### 2.Inheritance & Method Overriding
```typescript
class Animal{
    public name:string; // public by default
    private age:number; // only accessible within Animal class
    protected sound:string //only accessible in Animal & subClass
    constructor(name:string,age:number,sound:string){
            this.name=name;
            this.age=age;
            this.sound=sound
    }  
    getAge():Number{
      return this.age
    }
}

class Dog extends Animal{
    breed:string;
    constructor(name: string, age: number, breed: string,sound:string){
        super(name,age,sound)  // must call parent constructor
        this.breed=breed
    }
    speek():string{            // override parent method
      return `${this.name} says Woof!`
    }
}
const dog=new Dog("Rex",5,"labrador","Woof")
dog.speek()
dog.getAge()  // inherited from Animal
```
### 3. Interfaces & Abstract Classes
```typescript
// interface-defines shape,no implementation
interface Flyable{
    fly():void;
    altitude:number
}
// can have partial implementation
 abstract class Vehicle{
  abstract move():void  //muest be implemented by subclasses
  describe():string{ // concrete method-shared by all
     return `I am a ${this.constructor.name}`
  }
 }
class Plane extends Vehicle implements Flyable{
    altitude: number=10;
    move(): void {
        console.log("Taxing")
    }
    fly(): void {
         console.log(`Flying at${this.altitude}ft.`)
    }
}
const s3e=new Plane();
```
**Key difference:** interfaces are purely structural (erased at runtime), while abstract classes can carry real logic and state.
### 4. Access Modifier Shorthand
Instead of declaring and assigning properties manually, TypeScript lets you do it in the constructor signature:
```typescript
// Verbose way
class Person {
  name: string;
  private age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
// Shorthand — identical result, much cleaner
class Person {
  constructor(
    public name: string,
    private age: number
  ) {}
}
```

### 5.Static Members & Readonly
```typescript
class Counter {
  static count = 0;             // shared across all instances
  readonly id: number;          // set once, never changed
  constructor() {
    Counter.count++;
    this.id = Counter.count;
  }
}
const a = new Counter(); // a.id = 1, Counter.count = 1
const b = new Counter(); // b.id = 2, Counter.count = 2
a.id = 99;               // ❌ Error — readonly
```
### Quick Reference

|Concept|Keyword|Purpose|
|---|---|---|
|Inheritance|`extends`|Reuse & specialize a class|
|Interface|`interface`|Define a structural contract|
|Abstract class|`abstract`|Partial base implementation|
|Generics|`<T>`|Type-safe reusable code|
|Encapsulation|`private` / `protected`|Control access to members|
|Shared state|`static`|Belongs to the class, not an instance|
|Immutability|`readonly`|Prevent reassignment after construction|
