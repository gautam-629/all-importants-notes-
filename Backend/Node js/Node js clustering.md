# Clustering in Node.js

---
## What is Clustering?

**Clustering** is a technique that allows Node.js to create **multiple instances (processes)** of your application to utilize **multiple CPU cores**.

Since Node.js runs on a **single thread**, by default it uses only **one CPU core**.  
Clustering enables Node.js to handle **more traffic** by running multiple processes in parallel.

Each process is called a **worker**, and all workers share the **same server port**.

---
## Why Do We Need Clustering?

- Node.js is single-threaded.
- Modern servers have **multiple CPU cores**.
- Without clustering, Node.js only uses **one core**.
- With clustering, Node.js can use **all cores** → better performance.

---
## How Clustering Works

- A **master process** is created.
- The master forks multiple **worker processes**.
- Each worker runs independently.
- The operating system distributes incoming requests among workers.

---
## Basic Example

```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isMaster) {
  const cpuCount = os.cpus().length;

  console.log(`Master ${process.pid} is running`);

  // Create workers
  for (let i = 0; i < cpuCount; i++) {
    cluster.fork();
  }

} else {
  // Workers share the same port
  http.createServer((req, res) => {
    res.end(`Handled by worker ${process.pid}`);
  }).listen(3000);

  console.log(`Worker ${process.pid} started`);
}

```

## When to Use Clustering?

Use clustering when:
- Your application handles **high traffic**
- You want to use **multiple CPU cores**
- You are running a **production server**
- You need better **performance and scalability**

## When NOT to Use Clustering?
- For small applications or development
- When using external process managers like **PM2** (it already supports clustering)
- For CPU-heavy tasks (use **worker threads** instead)