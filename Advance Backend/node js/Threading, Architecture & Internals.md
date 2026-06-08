# 🖥️ Node.js Complete Notes — Threading, Architecture & Internals

> **Course:** Backend / System Design **College:** Nagarjuna College of Information Technology **Topics Covered:** Apache → NGINX → Node.js → libuv → Thread Pool → Event Loop

---

## 📌 Background: Why Do We Study This?

When an app sends emails or processes heavy jobs (like building a report), it uses a **message / build queue** (e.g., RabbitMQ, BullMQ) so tasks run **asynchronously** — without blocking the main server.

```
 User Request
      │
      ▼
 ┌─────────┐      push job      ┌───────────────┐
 │  Server │ ────────────────► │  Message Queue │
 └─────────┘                   └───────┬───────┘
                                        │ worker picks up
                                        ▼
                               ┌────────────────┐
                               │  Worker Process │
                               │  (send email,  │
                               │   build PDF...) │
                               └────────────────┘
```

> To understand **how Node.js handles this efficiently**, we must first understand **how web servers evolved** — from Apache all the way to Node.js.

---

# PART 1 — Evolution of Web Servers

---

## 📚 Chapter 1: Apache Web Server

> **Why study Apache?** Apache was the dominant web server for decades. Its problems directly inspired NGINX and eventually Node.js architecture.

```
Client ──────► Server (Apache) ──────► Database
```

---

### 🔴 Phase 1 — One Process Per Request (Early Apache)

For **every single incoming request**, Apache would **fork a brand new process**.

```
 Incoming Requests          Apache (Early)
 ─────────────────          ──────────────
 Request 1 ──────────────► Process 1  (RAM: ~10MB each)
 Request 2 ──────────────► Process 2  (RAM: ~10MB each)
 Request 3 ──────────────► Process 3  (RAM: ~10MB each)
 Request 4 ──────────────► Process 4  (RAM: ~10MB each)
      ...                       ...
 Request 1000 ───────────► Process 1000 ← 💥 Server crashes!
```

> ❌ **Problem:** Forking a new process is very expensive (time + memory). 1000 requests = 1000 processes = server runs out of RAM.

---

### 🟡 Phase 2 — Multi-Threading (Improved Apache)

Apache improved by using **one process with multiple threads** (threads share memory, so they are lighter than full processes).

```
 Incoming Requests          Apache (Improved)
 ─────────────────          ─────────────────────────────────
 Request 1 ──────────────► ┌──────────────────────────────┐
 Request 2 ──────────────► │         Single Process        │
 Request 3 ──────────────► │  ├── Thread 1 (handling req1) │
 Request 4 ──────────────► │  ├── Thread 2 (handling req2) │
                            │  ├── Thread 3 (handling req3) │
                            │  └── Thread 4 (handling req4) │
                            └──────────────────────────────┘
```

> ⚠️ **Still a problem:** Each thread **blocks** while waiting for I/O.

```
Thread 1:  [waiting for DB............................................done] ← sitting idle!
Thread 2:  [waiting for file read.............................done]         ← sitting idle!
Thread 3:  [waiting for API response.........................................done]
```

> 1000 concurrent requests = 1000 threads sitting idle, each consuming ~1–2 MB RAM = **1–2 GB wasted RAM just for waiting!**

---

## 📚 Chapter 2: NGINX — The Next Step

> **Why was NGINX created?** Apache's thread-per-request model didn't scale well under high concurrency. NGINX introduced a completely different approach: **non-blocking, event-driven architecture**.

---

### 🟢 NGINX — Non-Blocking Event-Driven Model

Instead of one thread per request, NGINX uses a **small fixed number of worker processes** — each running a **single thread with an event loop**.

```
                          NGINX Process Model
                          ───────────────────

             ┌─────────────────────────────────────┐
             │           Master Process             │
             │   (manages workers, reads config)    │
             └─────────────┬───────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │  Worker 1   │  │  Worker 2   │  │  Worker 3   │
    │ (1 thread)  │  │ (1 thread)  │  │ (1 thread)  │
    │ Event Loop  │  │ Event Loop  │  │ Event Loop  │
    └─────────────┘  └─────────────┘  └─────────────┘
         ↑ handles         ↑ handles        ↑ handles
       1000s of          1000s of         1000s of
       connections       connections      connections
```

### NGINX Scaling Rule:

```
 Number of CPU Cores  ──────►  Number of Worker Processes

       4 Cores        ──────►  4 Worker Processes
                               (each = 1 thread + event loop)
```

> ✅ No idle threads waiting. When waiting for I/O, the worker immediately picks up the **next request** from the event loop. Same thread, more work done.

---

### 🆚 Apache vs NGINX — Side-by-Side

```
                APACHE (Thread-per-Request)
 ┌──────────────────────────────────────────────────┐
 │  Request 1 → Thread 1 → [waiting DB..........] ← IDLE
 │  Request 2 → Thread 2 → [waiting File.......] ← IDLE
 │  Request 3 → Thread 3 → [waiting API........] ← IDLE
 │  (Each idle thread still uses 1–2 MB RAM)
 └──────────────────────────────────────────────────┘

                NGINX (Event-Driven)
 ┌──────────────────────────────────────────────────┐
 │  1 Worker Thread handles ALL requests:
 │
 │  [Req1 starts] → registers callback → moves on
 │  [Req2 starts] → registers callback → moves on
 │  [Req3 starts] → registers callback → moves on
 │  [Req1 I/O done] → callback fires → send response
 │  [Req2 I/O done] → callback fires → send response
 └──────────────────────────────────────────────────┘
```

---

# PART 2 — Node.js Architecture

---

## 📚 Chapter 3: High I/O vs High CPU Processing

> **Why this distinction matters?** Node.js is modelled after NGINX's event-driven approach. But there is one important category of tasks it handles differently — CPU-intensive work.

---

### 🔵 High I/O (Input / Output)

Tasks that involve **waiting** for an external resource:

```
 ┌────────────────────────────────────────────────┐
 │               HIGH I/O TASKS                   │
 │                                                 │
 │  Type              Example                      │
 │  ────────────────  ─────────────────────────   │
 │  File System    →  Reading / Writing files      │
 │  Network        →  HTTP/API calls               │
 │  Database       →  SELECT, INSERT queries       │
 │  Cache          →  Redis GET/SET                │
 └────────────────────────────────────────────────┘
```

> ✅ Node.js is **excellent** at I/O — it registers a callback and moves on. Zero idle waiting.

---
### 🔴 High CPU (Processing-Intensive)

Tasks that require **heavy computation** by the processor:

```
 ┌────────────────────────────────────────────────┐
 │            HIGH CPU TASKS                      │
 │                                                 │
 │  Type                Example                    │
 │  ──────────────────  ───────────────────────   │
 │  Data Compression →  Zipping / Unzipping        │
 │  Video Processing →  Encoding, Transcoding      │
 │  Cryptography     →  Encryption / Decryption    │
 │  Image Processing →  Resizing, Filters          │
 └────────────────────────────────────────────────┘
```

> ❌ Node.js **struggles** here — we'll understand exactly WHY in the next chapter.

---

## 📚 Chapter 4: Node.js — Is It Single or Multi-Threaded?

### ✅ YES — Single Threaded (for YOUR code)

### ⚠️ YES — Multi-Threaded internally (via libuv — NOT your code)

```
 ┌─────────────────────────────────────────────────────────────┐
 │                    NODE.js PROCESS                          │
 │                                                             │
 │  ┌───────────────────────────────────────────────────────┐ │
 │  │           YOUR JAVASCRIPT (V8 Engine)                  │ │
 │  │                                                         │ │
 │  │   const data = await fs.readFile('file.txt')           │ │
 │  │   const result = bcrypt.hash(password, 10)             │ │
 │  │                                                         │ │
 │  │              👆 SINGLE THREAD ONLY                     │ │
 │  └──────────────────────┬────────────────────────────────┘ │
 │                         │ calls Node.js APIs               │
 │  ┌──────────────────────▼────────────────────────────────┐ │
 │  │                    libuv (C Library)                   │ │
 │  │                                                         │ │
 │  │   ┌─────────────┐    ┌──────────────────────────────┐ │ │
 │  │   │ Event Loop  │    │      Thread Pool              │ │ │
 │  │   │  (1 thread) │    │  T1 │ T2 │ T3 │ T4           │ │ │
 │  │   └─────────────┘    └──────────────────────────────┘ │ │
 │  │                                                         │ │
 │  │   ┌──────────────────────────────────────────────────┐ │ │
 │  │   │   OS Async I/O  (epoll / kqueue / IOCP)          │ │ │
 │  │   └──────────────────────────────────────────────────┘ │ │
 │  └───────────────────────────────────────────────────────┘ │
 └─────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapter 5: What is libuv?

> **libuv** is a **C library** that powers Node.js under the hood. It provides three major components:

```
 libuv
  │
  ├── 1. Event Loop        ← The coordinator (runs on main thread)
  │
  ├── 2. Thread Pool       ← Background workers (4 threads default)
  │
  └── 3. OS-level I/O      ← epoll (Linux) / kqueue (Mac) / IOCP (Win)
                              Handles network sockets natively
```

---

### 🧵 The Thread Pool — What Is It?

The **Thread Pool** is a group of **background threads** kept ready inside libuv to handle tasks that cannot be done at the OS level asynchronously.

```
 Default Thread Pool Size: 4 threads
 (Can be changed: UV_THREADPOOL_SIZE=8 node app.js)

 ┌────────────────────────────────────────────────────┐
 │                THREAD POOL (libuv)                  │
 │                                                      │
 │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────┐ │
 │   │ Thread 1 │  │ Thread 2 │  │ Thread 3 │  │ T4 │ │
 │   │          │  │          │  │          │  │    │ │
 │   │ fs.read  │  │ crypto   │  │  zlib    │  │dns │ │
 │   └──────────┘  └──────────┘  └──────────┘  └────┘ │
 └────────────────────────────────────────────────────┘
```

---

### 📋 What Goes INTO the Thread Pool?

|Task|Why thread pool?|
|---|---|
|`fs.readFile()` / `fs.writeFile()`|File system — OS can't do it natively async|
|`crypto.pbkdf2()` / `crypto.scrypt()`|CPU-heavy hashing algorithms|
|`zlib.gzip()` / `zlib.deflate()`|Compression|
|`dns.lookup()`|DNS resolution|

---

### 📋 What Does NOT Use Thread Pool?

|Task|Why NOT?|
|---|---|
|`http.get()` / `fetch()`|OS handles network sockets natively (epoll/kqueue)|
|`net.createServer()`|OS-level async sockets — no thread needed|
|`setTimeout()` / `setInterval()`|Handled directly by Event Loop timers|
|`setImmediate()`|Handled directly by Event Loop|

> ⚠️ **Important:** Network I/O (the most common async task in Node.js!) does NOT use the thread pool — the OS handles it natively and asynchronously.

---

### 🔁 How Thread Pool Works with Your Code

```
 Your JS calls fs.readFile('data.json', callback)
       │
       ▼
 Event Loop sees it: "This is a file read — offload it"
       │
       ▼
 ┌─────────────────────────────────────┐
 │  Thread Pool                         │
 │  Thread 1: reading data.json...      │ ← background
 └─────────────────────────────────────┘
       │
       │  Meanwhile — Main Thread is FREE ✅
       │  [handles Request 2]
       │  [handles Request 3]
       │  [runs other callbacks]
       │
 Thread 1 finishes reading
       │
       ▼
 Event Loop: "Thread 1 is done, queue the callback"
       │
       ▼
 Your callback(data) runs on Main Thread
```

---

## 📚 Chapter 6: The BIG Question — Why Does CPU Work Block Node.js?

> **If libuv has threads internally, why does heavy CPU work still block Node.js?**

### 🔑 The Answer — Two Critical Rules:

```
 ┌───────────────────────────────────────────────────────────────┐
 │                                                               │
 │  RULE 1: libuv threads are ONLY for I/O tasks (file, crypto, │
 │           zlib, dns) — chosen by Node.js core developers.     │
 │                                                               │
 │  RULE 2: YOUR JavaScript always executes on the MAIN THREAD   │
 │           (V8 engine). libuv threads never run your JS code.  │
 │                                                               │
 └───────────────────────────────────────────────────────────────┘
```

---

### 📊 Visual Proof — I/O Task (Non-Blocking ✅)

```
 Main Thread (V8):
 ──────────────────────────────────────────────────────────▶ time
   [start req] [handle req2] [handle req3] [run callback!]
        │                                        ▲
        │  offloads to thread pool               │ result returned
        ▼                                        │
 Thread Pool:
              [reading file.......................done] ──────┘
```

> Main thread is **completely free** while the file is being read ✅

---

### 📊 Visual Proof — CPU Task (BLOCKING ❌)

```
 Main Thread (V8):
 ──────────────────────────────────────────────────────────▶ time
   [start]  [encrypting 1 million passwords..............done]
                ↑
                MAIN THREAD IS STUCK HERE
                Event Loop is FROZEN
                No new requests handled
                No callbacks run
                Server appears unresponsive ❌
```

> The JS `for` loop / manual computation runs **directly on the main thread** — libuv has no way to intercept plain JavaScript execution.

---

### 🆚 I/O Task vs CPU Task — Full Comparison

```
 I/O Task (fs.readFile)              CPU Task (manual for-loop)
 ──────────────────────              ──────────────────────────

 JS calls fs.readFile()              JS runs: for(let i=0; i<1e9; i++)
         │                                      │
         ▼                                      ▼
 libuv intercepts → offloads         V8 executes directly on main thread
 to Thread Pool                                 │
         │                                      ▼
 Main thread = FREE ✅               Main thread = BUSY ❌
 Handles other requests              Event Loop FROZEN
         │                           No other requests handled
         ▼                                      │
 Thread done → callback queued                  ▼
         │                           Work finishes → resumes
         ▼
 Callback runs on main thread
```

---

# PART 3 — The Event Loop in Depth

---

## 📚 Chapter 7: What is the Event Loop?

> **Why does Node.js need an Event Loop?** Because it is single-threaded. Without an event loop, the main thread would block and wait for every I/O operation to complete before moving on.

The Event Loop is the **coordinator** that continuously checks:

- Has any async operation finished?
- Are there any callbacks to run?
- Any timers ready to fire?

```
 ┌──────────────────────────────────────────────────────────────┐
 │                      EVENT LOOP                              │
 │                                                              │
 │   ┌───────────┐                                              │
 │   │  1.TIMERS │ ← setTimeout() / setInterval() callbacks     │
 │   └─────┬─────┘                                              │
 │         │                                                     │
 │   ┌─────▼──────────────┐                                     │
 │   │  2.PENDING CALLBACKS│ ← deferred I/O error callbacks     │
 │   └─────┬──────────────┘                                     │
 │         │                                                     │
 │   ┌─────▼──────────────┐                                     │
 │   │  3.IDLE / PREPARE  │ ← internal use only                 │
 │   └─────┬──────────────┘                                     │
 │         │                                                     │
 │   ┌─────▼──────────────┐                                     │
 │   │  4.POLL            │ ← Wait & receive new I/O events     │
 │   │                    │   Run I/O callbacks                  │
 │   └─────┬──────────────┘                                     │
 │         │                                                     │
 │   ┌─────▼──────────────┐                                     │
 │   │  5.CHECK           │ ← setImmediate() callbacks          │
 │   └─────┬──────────────┘                                     │
 │         │                                                     │
 │   ┌─────▼──────────────┐                                     │
 │   │  6.CLOSE CALLBACKS │ ← socket.on('close') etc.           │
 │   └─────┬──────────────┘                                     │
 │         │                                                     │
 │         └──────────────────────► loop repeats ◄──────────────┘
 └──────────────────────────────────────────────────────────────┘
```

---

### ⚡ Microtask Queue — Highest Priority

Between **every phase**, the **microtask queue** runs first (before moving to the next phase):

```
 Phase N finishes
       │
       ▼
 ┌─────────────────────────────────────────────┐
 │            MICROTASK QUEUE                  │
 │                                             │
 │  1. process.nextTick() calls run FIRST      │
 │  2. Promise .then() / .catch() run SECOND   │
 └─────────────────────────────────────────────┘
       │
       ▼
 Phase N+1 begins
```

> `process.nextTick()` has the **highest priority** of all async mechanisms in Node.js.

---

### 📋 Event Loop Phase Summary

|Phase|What Runs|Examples|
|---|---|---|
|1. Timers|Timer callbacks|`setTimeout`, `setInterval`|
|2. Pending CB|OS-level errors|TCP errors|
|3. Idle/Prepare|Internal only|—|
|4. Poll|I/O events|File read done, HTTP response|
|5. Check|Immediate callbacks|`setImmediate()`|
|6. Close|Cleanup callbacks|`socket.on('close')`|
|_Between phases_|Microtasks|`process.nextTick()`, Promises|

---

## 📚 Chapter 8: Complete Node.js Architecture — Everything Together

```
 ┌─────────────────────────────────────────────────────────────────────┐
 │                         NODE.js PROCESS                             │
 │                                                                     │
 │   YOUR JS CODE (runs here)                                          │
 │   ┌─────────────────────────────────────────────────────────────┐  │
 │   │  const data = await fs.readFile('file.txt')  ← async I/O   │  │
 │   │  for(let i=0; i<1e9; i++) { }               ← CPU block ❌ │  │
 │   │  await fetch('https://api.com')              ← async net   │  │
 │   │                V8 Engine — SINGLE THREAD                    │  │
 │   └──────────────────────────┬──────────────────────────────────┘  │
 │                              │                                      │
 │   ┌──────────────────────────▼──────────────────────────────────┐  │
 │   │                        libuv                                 │  │
 │   │                                                              │  │
 │   │  ┌──────────────────────────────────────────────────────┐  │  │
 │   │  │                   EVENT LOOP                          │  │  │
 │   │  │  Timers → Pending → Poll → Check → Close → repeat    │  │  │
 │   │  └──────────────────────┬───────────────────────────────┘  │  │
 │   │                         │                                    │  │
 │   │  ┌──────────────────────▼───────────────────────────────┐  │  │
 │   │  │              THREAD POOL (4 threads)                  │  │  │
 │   │  │   [Thread 1]   [Thread 2]   [Thread 3]   [Thread 4]  │  │  │
 │   │  │   fs read/write  crypto      zlib          dns        │  │  │
 │   │  └──────────────────────────────────────────────────────┘  │  │
 │   │                                                              │  │
 │   │  ┌──────────────────────────────────────────────────────┐  │  │
 │   │  │           OS ASYNC I/O                                │  │  │
 │   │  │   epoll (Linux) / kqueue (Mac) / IOCP (Windows)      │  │  │
 │   │  │   Handles: HTTP, TCP, UDP, Sockets (no thread needed) │  │  │
 │   │  └──────────────────────────────────────────────────────┘  │  │
 │   └──────────────────────────────────────────────────────────────┘  │
 └─────────────────────────────────────────────────────────────────────┘
```

---

# PART 4 — Fixing CPU Blocking

---

## 📚 Chapter 9: How to Fix CPU Blocking in Node.js

### Option 1 — Worker Threads (Recommended)

```javascript
// main.js
const { Worker } = require('worker_threads');

// Offload heavy CPU work to a separate thread
const worker = new Worker('./heavy-task.js');

worker.on('message', (result) => {
  console.log('Result:', result);  // Main thread still free!
});
```

```
 Main Thread:
 ────────────────────────────────────────────────────▶ time
   [handle req1]  [handle req2]  [callback: done!]
                                        ▲
 Worker Thread:                         │
   [encrypting million passwords........done] ─────────┘
```

> ✅ Main thread stays free. Worker thread does the heavy lifting.

---

### Option 2 — Child Processes

```javascript
const { fork } = require('child_process');
const child = fork('./heavy-task.js');
child.on('message', (result) => console.log(result));
child.send({ data: 'process this' });
```

### Option 3 — Use libuv's Thread Pool via Native Modules

Some Node.js built-in functions already use the thread pool for CPU work:

```javascript
// This ALREADY uses libuv thread pool — does NOT block main thread ✅
crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', (err, key) => {
  console.log('Hashing done!');
});
```

---

# PART 5 — Grand Summary

---

## 🗂️ Full Comparison Table

|Feature|Apache (Old)|Apache (New)|NGINX|Node.js|
|---|---|---|---|---|
|Model|1 process/request|1 thread/request|Event-driven workers|Event-driven + libuv|
|Concurrency|Very Low|Medium|Very High|Very High|
|High I/O|❌ Blocks|⚠️ Partial|✅ Excellent|✅ Excellent|
|High CPU|✅ OK|✅ OK|⚠️ Limited|❌ Blocks (needs Worker)|
|RAM Usage|🔴 Very High|🟡 High|🟢 Low|🟢 Low|
|Threads|Many processes|Many threads|X cores = X workers|1 main + libuv pool|

---

## 🔑 Key Takeaways — In Order

```
 1. APACHE (Early)
    └── 1 process per request → huge RAM waste → doesn't scale

 2. APACHE (Improved)
    └── Multi-threaded → better, but threads block on I/O → still wastes RAM

 3. NGINX
    └── Event-driven, non-blocking → X cores = X single-threaded workers
        Each worker handles thousands of connections without blocking

 4. NODE.js
    └── Same model as NGINX
        Uses libuv → Event Loop + Thread Pool + OS Async I/O
        Your JS = single thread (V8)
        libuv handles async I/O in background threads

 5. WHY CPU BLOCKS NODE.js
    └── Your JS code always runs on the main thread
        libuv threads only handle specific I/O tasks
        Heavy JS computation = main thread stuck = event loop frozen

 6. FIX FOR CPU BLOCKING
    └── Worker Threads → runs JS on a separate thread
        Child Processes → separate OS process
        Native crypto/zlib → already use libuv thread pool
```

---

## 🍽️ Final Analogy — The Restaurant

> Imagine Node.js as a **restaurant**:

```
 🧑‍🍳  Chef            = Main Thread (V8)     — takes orders, cooks
 🚗  Delivery Drivers = Thread Pool (libuv)  — fetch ingredients (I/O)
 📋  Order Board      = Event Loop           — tracks what's pending
 🏪  Suppliers        = OS Async I/O         — network / sockets
```

```
 SCENARIO A — I/O Task (Non-Blocking ✅)
 ─────────────────────────────────────────
 Chef gets order → sends driver to fetch vegetables (file I/O)
 Chef is FREE → cooks other dishes while driver is out
 Driver returns → Chef uses vegetables → serves dish ✅

 SCENARIO B — CPU Task (Blocking ❌)
 ─────────────────────────────────────────
 Chef must personally grind 10kg of spices (CPU work)
 Chef is STUCK at the grinder
 All other orders pile up, no cooking, customers wait ❌

 SOLUTION — Worker Threads
 ─────────────────────────────────────────
 Hire a SECOND CHEF to do the grinding
 Original chef stays free to handle all other orders ✅
```

---

> 📝 _Nagarjuna College of Information Technology — Backend / System Design Notes_ 🗓️ _Complete Reference: Apache → NGINX → Node.js → libuv → Thread Pool → Event Loop_