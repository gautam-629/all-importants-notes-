## 1.Explain the Event Loop with real example
## 🔹 1. What is a Process?

A **process** is basically a **program in execution**.

**Example:** When you open Chrome, each tab runs as a separate process.
### A process has:
- Its own memory
- Resources
- Execution context (instructions being executed)
### Key points:
- Multiple processes can run independently
- Processes don't share memory by default
## 🔹 2. What is a Thread?
A **thread** is the **smallest unit of execution** inside a process.

- A process can have **one or more threads**
- Threads inside the same process **share memory and resources**, but each thread has its own **stack**
### Analogy:

- **Process** = Factory
- **Thread** = Worker inside the factory
- **Factory resources** = Shared memory, tools

---
## 🔹 3. Single Thread

**Single-threaded** means the program has only **one thread of execution**.

- It can execute **one task at a time**
- **Node.js** uses single-threaded event loop for JavaScript execution
### Example:

```javascript
console.log("Task 1");
console.log("Task 2");
console.log("Task 3");
```

### Output:

```
Task 1
Task 2
Task 3
```

Tasks are executed **line by line**.

### ⚠️ Problem with single thread:

If one task is slow/blocking, everything else waits.

---
## 🔹 4. Multithread

**Multithreaded** programs can run **multiple threads simultaneously**.

- Threads can perform different tasks at the same time
- Each thread shares process memory but can run independently
### Example (Real-life analogy):

Downloading a file while listening to music and typing a document at the same time.

- Each activity = **separate thread**
### Code example in other languages (Node.js is mostly single-threaded for JS):

```java
Thread t1 = new Thread(() -> System.out.println("Thread 1"));
Thread t2 = new Thread(() -> System.out.println("Thread 2"));
t1.start();
t2.start();
```

Threads can run **in parallel** (if CPU has multiple cores).

---
## 🔹 5. How Node.js Fits In

- **Node.js** runs JavaScript on a **single thread** (main thread)
- But Node.js uses **libuv library** to handle asynchronous operations on **worker threads** in the background
- This allows Node.js to be **non-blocking** and **highly scalable**, even with a single JS thread
### Analogy:

- **JS code** = Single chef
- **Background I/O tasks** (file, DB, network) = Helpers in the kitchen
- **Event Loop** = Manager scheduling tasks

---
## Summary

|Concept|Description|
|---|---|
|**Process**|Program in execution with its own memory and resources|
|**Thread**|Smallest unit of execution inside a process|
|**Single Thread**|One thread executing tasks sequentially|
|**Multithread**|Multiple threads executing tasks concurrently|
|**Node.js**|Single-threaded JS execution + multi-threaded I/O operations via libuv|

---
## 🔹 6. The Event Loop Explained

The **Event Loop** is the heart of Node.js's asynchronous architecture. It allows Node.js to perform non-blocking I/O operations despite JavaScript being single-threaded.
### How Does the Event Loop Work?

The Event Loop continuously checks if there are tasks to execute and processes them in a specific order.
### Key Components:

1. **Call Stack** - Where synchronous code executes
2. **Callback Queue** - Holds callbacks from async operations
3. **Event Loop** - Monitors Call Stack and Callback Queue
4. **Web APIs / libuv** - Handles async operations (timers, I/O, etc.)

---
## 2. What happens when a blocking operation runs in Node?

When a **blocking operation** runs in Node.js, it **stops the single JS thread**, so:

- No other code executes until it finishes.  
- Async tasks (like callbacks, timers) are delayed.  
- The server can appear **frozen** for other users.  

**Solution:** Use **non-blocking (asynchronous) operations** or offload heavy tasks to **worker threads**.
## 3. How does Node achieve concurrency?

Node.js is **single-threaded for JavaScript**, but it achieves concurrency using:

1. **Event Loop** – Continuously monitors tasks and executes callbacks when async operations are ready.  
2. **Non-blocking I/O** – Operations like file reads, network requests, and database queries run **in the background** via libuv.  
3. **Worker Threads (optional)** – For CPU-intensive tasks, Node can create threads to run heavy tasks without blocking the main thread.
