# 🖥️ Processes & Threads — Core Concepts

> **Course:** AlgoCamp Backend Cohort | **Topic:** OS Internals for Server-Side Dev  
> **Subject:** Operating Systems | **Relevance:** Node.js & Backend Server Architecture

---
## 📋 Table of Contents

|#|Topic|Category|
|---|---|---|
|1|[What is a Process?](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#1-what-is-a-process)|OS Fundamentals|
|2|[CPU Basics](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#2-cpu-basics)|Hardware|
|3|[Single-Core vs Multi-Core](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#3-single-core-vs-multi-core)|Hardware|
|4|[Context Switching](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#4-context-switching-)|OS Scheduling|
|5|[Concurrency vs Parallelism](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#5-concurrency-vs-parallelism)|Core Concept|
|6|[Threads — Lightweight Processes](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#6-threads--lightweight-processes-)|OS Fundamentals|
|7|[File Descriptors](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#7-file-descriptors-)|OS Internals|
|8|[Hardware vs Software Threads](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#8-hardware-vs-software-threads)|Hardware + Software|
|9|[When is Multithreading Useful?](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#9-when-is-multithreading-useful)|Applied Concepts|
|10|[Go Routines (Bonus)](https://claude.ai/chat/12b80c72-3f53-49fe-b809-f4678f283303#10-go-routines-bonus)|Language Feature|

---
## 1. What is a Process?

> A **program under execution** is called a **process**. It lives in RAM.
### Components of a Process

|Component|Description|
|---|---|
|**PID**|Process Identifier — unique ID assigned by OS|
|**Stack Memory**|Stores function calls, local variables|
|**Heap Memory**|Dynamic memory allocation at runtime|
|**Text Area**|The actual compiled program code|
|**Program Counter**|Points to the next instruction to execute|

💡 _Why it matters:_ When you run a Node.js server, the OS creates a process with all of the above. Understanding this helps debug memory issues and understand server behavior.

---
## 2. CPU Basics

> **CPU = Central Processing Unit** — the brain that executes instructions.

- A modern CPU executes **10⁸ – 10⁹ instructions/second**
- **1 GHz = 10⁹ cycles/second** (clock speed)
- **1 Core = 1 CPU unit** → an 8-core machine has 8 CPU units
### Complexity in Context

```
n       → 10⁶ operations
n log n → 10⁶ × log(10⁶) ≈ 20 × 10⁶ = 2 × 10⁷ operations
```

Both fit comfortably within a CPU's per-second capacity.

---
## 3. Single-Core vs Multi-Core

### Single-Core

- Can execute instructions of **only one process at a time**
- Needs **Context Switching** to handle multiple processes

### Multi-Core

- Each core independently executes its own process
- True **parallelism** — multiple processes run _simultaneously_

---
## 4. Context Switching ⚡

> The OS rapidly switches between processes on a single core, giving the _illusion_ of parallel execution.

**Two strategies:**

1. **Run to completion** → one process finishes before the next starts → risk of **starvation**
2. **Time-sliced (concurrent)** → CPU cycles are divided among processes → fair, but non-deterministic

💡 _Why it matters:_ Context switching is the foundation of multitasking. Every OS scheduler (Linux, Windows, macOS) is built around this.

---
## 5. Concurrency vs Parallelism

|Aspect|Concurrency|Parallelism|
|---|---|---|
|**Definition**|Tasks _appear_ to run simultaneously (rapid switching)|Tasks _actually_ run simultaneously|
|**Hardware**|Works on single-core|Requires multi-core|
|**Determinism**|Non-deterministic|More deterministic|
|**Example**|Node.js event loop|GPU rendering, multi-threaded servers|

> **GPUs** have thousands of small cores → built for parallelism (ML, video rendering)  
> **CPUs** have few powerful cores → built for general-purpose + concurrency

---
## 6. Threads — Lightweight Processes 🧵

> **Threads** are lightweight processes that live _inside_ a process.

### What Threads Share vs Own

|Shared (by all threads)|Individually Owned|
|---|---|
|Heap memory|Call stack|
|Text (code) area|Program counter|

### Why Use Threads?

- **Cheaper** to create than spawning a new process
- Share memory → less overhead
- Great for handling **concurrent requests** on a server
### Example

```
Process (your server)
├── Thread T1 → handles request from Client 1  (own stack, own PC)
└── Thread T2 → handles request from Client 2  (own stack, own PC)
    both share → heap, code
```

💡 _Why it matters:_ A web server receiving 3 simultaneous requests can spin up 3 threads (P1, P2, P3) instead of 3 full processes — far more efficient.

---
## 7. File Descriptors 📁

> A **file descriptor** is a unique integer identifier the OS assigns to track open resources.

**Resources tracked by file descriptors:**

- Open files
- Network connections (sockets)
- Pipes, stdin/stdout/stderr

When a thread in a server process handles a client request, the OS assigns it a **file descriptor** to manage that connection.

💡 _Why it matters:_ Node.js is built on file descriptors under the hood. Understanding them is key to understanding I/O, streams, and socket connections.

---
## 8. Hardware vs Software Threads

|Aspect|Hardware Threads|Software Threads|
|---|---|---|
|**What they are**|Physical core count (shown in CPU specs)|Threads created by your program/OS|
|**Intel term**|"Threads" = total logical cores|Lightweight processes in code|
|**Performance cores**|~2× a normal core|—|
|**Efficient cores**|~1× a normal core|—|

---
## 9. When is Multithreading Useful?

✅ **Good for:**

- Handling many concurrent server requests
- Tasks that can be parallelized (e.g., image processing)
- I/O-bound work (waiting on DB, file, network)

❌ **Not helpful for:**

- Purely sequential tasks (e.g., bubble sort — step N depends on step N-1)

---
## 10. Go Routines (Bonus)

> Go's **goroutines** are even lighter than OS threads — minimal memory footprint, managed by Go's own scheduler. Ideal for massive concurrency.

---
## 📚 Recommended Reading

- **Operating System Concepts** — Galvin _(deep dive on context switching & scheduling)_
- **Introduction to Algorithms** — Cormen (CLRS) _(algorithm complexity)_

---
## 🔁 Quick Recap

```
Program → loaded into RAM → becomes a Process (PID, stack, heap, text, PC)
CPU executes it → single core = one at a time → Context Switching for multitasking
Multi-core = true parallelism
Threads = lightweight, share heap/text, own stack/PC
File Descriptors = OS handles for open resources
All of this → foundation of Node.js internals
```