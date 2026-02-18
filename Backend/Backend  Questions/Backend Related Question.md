# Mid-Level Backend Interview Question Set (Node.js + Express/Nest + DB + Redis + Queue + Socket.IO)

## 🟢 1️⃣ Node.js (Core – Must Be Strong)

---
1.[[Event Loop and core concept]]
2.[[Node js clustering]]
- [ ] **How do you handle CPU-heavy tasks?**
- [ ] **How do you implement graceful shutdown?**
- [ ] **What causes memory leaks in Node?**
- [ ] **How do you debug performance issues?**
- [ ] **How do you structure environment configs (dev/stage/prod)?**
## 🟢 2️⃣ Express / NestJS (Architecture Focused)

- [ ] **How does middleware chain work internally?**
- [ ] **How do you structure a scalable backend project?**
- [ ] **How do you implement global error handling?**
- [ ] **How do you validate request body properly?**
- [ ] **How do you implement authentication (JWT)?**
- [ ] **How do you implement refresh token securely?**
- [ ] **How do you implement RBAC?**
- [ ] **How do you handle file upload securely?**
- [ ] **What is the difference between Guards, Pipes, Interceptors (Nest)?**
- [ ] **How do you implement request logging in production?**

## 🟢 3️⃣ Database (SQL + MongoDB Practical)

- [ ] **Design a user + role + permission schema.**
- [ ] **Difference between `WHERE` and `HAVING`.**
- [ ] **What is indexing? When does it hurt performance?**
- [ ] **What is N+1 problem?**
- [ ] **How do you optimize slow queries?**
- [ ] **What is transaction? When should you use it?**
- [ ] **Explain isolation levels.**
- [ ] **What is soft delete and how to implement?**
- [ ] **How do you design product + category (hierarchy)?**
- [ ] **Write SQL for paginated filtered API endpoint.**

## 🟢 4️⃣ Redis (Very Important for Mid-Level)
  .[[Redis Complete Guide]]
1.**Why use Redis in backend?**
    Often used as:
    - Cache
    - Session store
    - Message broker
    - Real-time data store
2.**What caching strategy do you use?**
    I usually use the Cache-Aside strategy with Redis. The API checks the cache first; on a miss, it fetches from the database and stores the result with a TTL(cache-aside pattern.).
3.**What is TTL?**
    **TTL (Time To Live)** is the **duration for which data remains valid before it expires automatically**.
- [ ] **How to prevent cache stampede?**
- [ ] **Difference between Redis and DB indexing.**
- [ ] **What is Redis Pub/Sub?**
- [ ] **How do you scale Redis?**
- [ ] **When should you NOT use Redis?**

## 🟢 5️⃣ Queue System (Bull / BullMQ)
## 1. Why Do We Need Background Jobs?

Node.js runs on a **single thread**. Heavy tasks like:

- Sending emails
- Cleaning old data
- Generating reports

If executed **synchronously**, these block the main thread, preventing your server from handling other HTTP requests until the task completes.

**Solution**: Offload heavy tasks to background jobs so the main thread stays responsive.

---

## 2. What Happens If a Job Fails?

Background job libraries like **Bull** handle failures automatically with:

```javascript
await emailQueue.add(
  { 
    type: 'welcome', 
    data: { to: 'user@example.com', name: 'Alice' } 
  },
  { 
    attempts: 3,                              // Retry up to 3 times
    backoff: { 
      type: 'exponential', 
      delay: 5000                             // Wait 5s, 10s, 20s between retries
    },
    timeout: 60000                            // Fail if job exceeds 60 seconds
  }
);
```

---

## 3. What Is the Retry Mechanism?

When a job fails, the queue automatically retries it based on configured settings:

- **Attempts**: Number of retry attempts
- **Backoff**: Delay strategy between retries (exponential, fixed)
- **Timeout**: Maximum execution time before considering job failed

---

## 4. What Is a Delayed Job?

A **delayed job** runs after a specific time delay instead of immediately.

**Example**: Send a reminder email 10 minutes after user signup

```javascript
await queue.add(jobData, { 
  delay: 10 * 60 * 1000  // Delay by 10 minutes
});
```

---

## 5. What Is Rate Limiting in Queues?

**Rate limiting** controls how many jobs process **per unit of time**, preventing system overload.

```javascript
const queue = new Queue('email', {
  limiter: { 
    max: 100,           // Maximum 100 jobs
    duration: 60000     // Per 60 seconds (1 minute)
  }
});
```

**Use case**: Prevent hitting external API rate limits or overwhelming email servers.

---

## 6. How Do You Monitor Queues?

### Events

Listen to queue events for real-time monitoring:

```javascript
queue.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

queue.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed:`, err);
});

queue.on('stalled', (job) => {
  console.warn(`Job ${job.id} stalled`);
});
```

### Logging

- Store job status and errors in a database
- Use external logging services (e.g., Datadog, Sentry)

---

## 7. How Do You Handle High-Volume Jobs?

**Strategies:**

1. **Rate Limiting**: Control job processing speed
2. **Queue Partitioning**: Separate queues for different job types
    
    ```javascript
    const emailQueue = new Queue('email');const reportQueue = new Queue('reports');const cleanupQueue = new Queue('cleanup');
    ```
    
3. **Priority**: Assign priority levels to jobs
4. **Concurrency**: Control how many jobs run simultaneously
    
    ```javascript
    queue.process(5, async (job) => {  // Process up to 5 jobs concurrently});
    ```
    

---

## 8. What Is Idempotency in Job Processing?

**Idempotency** ensures running a job **multiple times produces the same result** without duplicate effects.

**Example**: Sending an email should happen once, even if the job retries.

**Implementation:**

```javascript
async function sendEmail(userId, emailType) {
  // Check if email already sent
  const sent = await db.checkEmailSent(userId, emailType);
  if (sent) return; // Skip if already sent
  
  await emailService.send(userId, emailType);
  await db.markEmailSent(userId, emailType); // Record sending
}
```

---

## 9. Event-Driven vs Queue-Driven Systems

|**Event-Driven**|**Queue-Driven**|
|---|---|
|**Immediate** execution when event occurs|**Deferred** execution (jobs can be delayed)|
|Uses EventEmitter or Pub/Sub|Uses job queues (Bull, BullMQ, Bee)|
|No built-in retry mechanism|Built-in retry, backoff, timeout|
|In-memory (no persistence)|Persisted in Redis/DB|
|Example: `eventEmitter.emit('userSignup')`|Example: `queue.add('sendEmail', data)`|
|Good for real-time, in-process tasks|Good for heavy, async, distributed tasks|

**When to use:**

- **Event-Driven**: Real-time notifications, internal app events
- **Queue-Driven**: Email sending, report generation, data processing

## 🟢 6️⃣ Socket.IO (Real-Time)

- [ ] **How does WebSocket work?**
- [ ] **Difference between WebSocket and HTTP?**
- [ ] **How to authenticate user in Socket.IO?**
- [ ] **How to emit event to specific user?**
- [ ] **What are rooms and namespaces?**
- [ ] **How to scale Socket.IO across servers?**
- [ ] **Why do we need Redis adapter?**
- [ ] **How to store online users?**
- [ ] **How to handle reconnection?**
- [ ] **Design simple chat backend architecture.**

## 🟢 7️⃣ Security & Production

- [x] **How to protect against SQL injection?**
- [x] **How to prevent XSS?**
- [ ] **How to protect against brute-force login?**
- [x] **How to implement rate limiter?**
- [ ] **How to secure environment variables?**
- [ ] **How to store password securely?**
- [x] **bcrypt vs argon2?**
- [x] **What is CORS?**
- [x] **What is horizontal vs vertical scaling?**
# Node.js Security & Scaling Guide with PostgreSQL
A comprehensive guide covering essential security practices and scaling strategies for Node.js applications with PostgreSQL.

---
## Table of Contents

- [SQL Injection Protection](https://claude.ai/chat/81385fe7-3aca-4a0d-a90a-df57379e538b#sql-injection-protection)
- [XSS Prevention](https://claude.ai/chat/81385fe7-3aca-4a0d-a90a-df57379e538b#xss-prevention)
- [Rate Limiter Implementation](https://claude.ai/chat/81385fe7-3aca-4a0d-a90a-df57379e538b#rate-limiter-implementation)
- [bcrypt vs Argon2](https://claude.ai/chat/81385fe7-3aca-4a0d-a90a-df57379e538b#bcrypt-vs-argon2)
- [CORS (Cross-Origin Resource Sharing)](https://claude.ai/chat/81385fe7-3aca-4a0d-a90a-df57379e538b#cors-cross-origin-resource-sharing)
- [Horizontal vs Vertical Scaling](https://claude.ai/chat/81385fe7-3aca-4a0d-a90a-df57379e538b#horizontal-vs-vertical-scaling)

---

## SQL Injection Protection

SQL injection is one of the most common and dangerous vulnerabilities. Always use parameterized queries to prevent attackers from injecting malicious SQL code.

### Vulnerable Code (Never Do This)

```javascript
// ❌ VULNERABLE - String concatenation
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;
const result = await pool.query(query);
```

### Safe Implementation

```javascript
// ✅ SAFE - Parameterized query with pg library
const query = 'SELECT * FROM users WHERE email = $1';
const result = await pool.query(query, [userEmail]);

// Multiple parameters
const query = 'SELECT * FROM users WHERE email = $1 AND status = $2';
const result = await pool.query(query, [userEmail, 'active']);
```

### Using an ORM (Recommended)

```javascript
// With Prisma - Parameterization is automatic
const user = await prisma.user.findUnique({ 
  where: { email: userEmail } 
});

// With Sequelize
const user = await User.findOne({ 
  where: { email: userEmail } 
});
```

**Key Principles:**

- Never concatenate user input into SQL queries
- Always use parameterized queries or prepared statements
- Use ORMs that handle parameterization automatically
- Validate and sanitize input as an additional layer

---

## XSS Prevention

Cross-Site Scripting (XSS) allows attackers to inject malicious scripts into web pages viewed by other users.

### Prevention Strategies

#### 1. Escape Output

Use templating engines that automatically escape HTML:

```javascript
// React automatically escapes
const UserProfile = ({ username }) => (
  <div>Welcome, {username}</div>
);

// Express with EJS (auto-escapes by default)
res.render('profile', { username: userInput });
```

#### 2. Content Security Policy (CSP)

```javascript
const helmet = require('helmet');

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"], // Avoid unsafe-inline in production
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", "data:", "https:"],
  }
}));
```

#### 3. Sanitize Rich Text Input

```javascript
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

const clean = DOMPurify.sanitize(dirtyHTML);
```

#### 4. Secure Cookies

```javascript
// Set HTTP-only, secure cookies
res.cookie('session', token, { 
  httpOnly: true,      // Prevents JavaScript access
  secure: true,        // HTTPS only
  sameSite: 'strict',  // CSRF protection
  maxAge: 3600000      // 1 hour
});
```

#### 5. Use Security Headers

```javascript
const helmet = require('helmet');
app.use(helmet());

// Manually set headers
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  next();
});
```

---

## Rate Limiter Implementation

Rate limiting prevents abuse by limiting the number of requests from a single IP or user.

### Basic Implementation with express-rate-limit

```javascript
const rateLimit = require('express-rate-limit');

// General API rate limiter
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                  // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.',
  standardHeaders: true,     // Return rate limit info in `RateLimit-*` headers
  legacyHeaders: false,      // Disable `X-RateLimit-*` headers
});

app.use('/api/', apiLimiter);
```

### Stricter Limits for Sensitive Endpoints

```javascript
// Login rate limiter (prevent brute force)
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true, // Don't count successful logins
  message: 'Too many login attempts, please try again after 15 minutes.'
});

app.post('/api/login', loginLimiter, async (req, res) => {
  // Login logic
});
```

### Distributed Rate Limiting with Redis

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

const redisClient = new Redis({
  host: 'localhost',
  port: 6379,
});

const limiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate-limit:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 100,
});

app.use('/api/', limiter);
```

### Custom Rate Limiter with PostgreSQL

```javascript
const checkRateLimit = async (req, res, next) => {
  const ip = req.ip;
  const now = Date.now();
  const windowMs = 15 * 60 * 1000; // 15 minutes
  const maxRequests = 100;

  try {
    // Clean old entries
    await pool.query(
      'DELETE FROM rate_limits WHERE timestamp < $1',
      [now - windowMs]
    );

    // Count requests
    const result = await pool.query(
      'SELECT COUNT(*) FROM rate_limits WHERE ip = $1 AND timestamp > $2',
      [ip, now - windowMs]
    );

    if (parseInt(result.rows[0].count) >= maxRequests) {
      return res.status(429).json({ error: 'Too many requests' });
    }

    // Log request
    await pool.query(
      'INSERT INTO rate_limits (ip, timestamp) VALUES ($1, $2)',
      [ip, now]
    );

    next();
  } catch (error) {
    console.error('Rate limit error:', error);
    next();
  }
};

app.use('/api/', checkRateLimit);
```

---

## bcrypt vs Argon2

Both are secure password hashing algorithms, but they have different strengths.

### Comparison Table

|Feature|bcrypt|Argon2|
|---|---|---|
|Release Year|1999|2015|
|Algorithm Type|Key derivation function|Password hashing competition winner|
|Memory Hardness|Low|High (configurable)|
|GPU Resistance|Moderate|Excellent|
|Variants|One|Three (Argon2i, Argon2d, Argon2id)|
|Maturity|Very mature, battle-tested|Newer, gaining adoption|
|Configuration|Simple (cost factor)|More options (memory, parallelism, iterations)|

### bcrypt Implementation

```javascript
const bcrypt = require('bcrypt');

// Hash password
const hashPassword = async (password) => {
  const saltRounds = 12; // Recommended: 10-12
  const hash = await bcrypt.hash(password, saltRounds);
  return hash;
};

// Verify password
const verifyPassword = async (password, hash) => {
  const isValid = await bcrypt.compare(password, hash);
  return isValid;
};

// Usage
const hashedPassword = await hashPassword('mySecurePassword123');
const isValid = await verifyPassword('mySecurePassword123', hashedPassword);
```

### Argon2 Implementation

```javascript
const argon2 = require('argon2');

// Hash password (using Argon2id - recommended variant)
const hashPassword = async (password) => {
  const hash = await argon2.hash(password, {
    type: argon2.argon2id,     // Argon2id (hybrid mode)
    memoryCost: 65536,         // 64 MB
    timeCost: 3,               // Number of iterations
    parallelism: 4,            // Number of threads
  });
  return hash;
};

// Verify password
const verifyPassword = async (password, hash) => {
  const isValid = await argon2.verify(hash, password);
  return isValid;
};

// Usage
const hashedPassword = await hashPassword('mySecurePassword123');
const isValid = await verifyPassword('mySecurePassword123', hashedPassword);
```

### Which Should You Use?

**Choose Argon2id if:**

- Starting a new project
- Need maximum security against GPU/ASIC attacks
- Have control over server resources
- Want the latest cryptographic standards

**Choose bcrypt if:**

- Already implemented and working well
- Need maximum compatibility
- Prefer simpler configuration
- Have legacy systems to maintain

**Bottom Line:** Both are secure. Argon2id is technically superior and recommended for new projects, but bcrypt is perfectly acceptable and battle-tested.

---

## CORS (Cross-Origin Resource Sharing)

CORS is a security mechanism that controls which origins can access your server's resources. Browsers enforce the Same-Origin Policy, and CORS provides a way to relax these restrictions safely.

### Understanding CORS

```
Browser (https://frontend.com) → Server (https://api.backend.com)
                                    ↓
                        Checks CORS headers
                                    ↓
                        Allows or blocks request
```

### Basic Setup with Express

```javascript
const cors = require('cors');

// Allow all origins (DEVELOPMENT ONLY - not secure for production)
app.use(cors());
```

### Production Configuration

```javascript
const cors = require('cors');

// Whitelist specific origins
const corsOptions = {
  origin: ['https://yourfrontend.com', 'https://www.yourfrontend.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,              // Allow cookies
  optionsSuccessStatus: 200,      // Legacy browser support
  maxAge: 86400,                  // Cache preflight for 24 hours
};

app.use(cors(corsOptions));
```

### Dynamic Origin Validation

```javascript
const allowedOrigins = [
  'https://yourfrontend.com',
  'https://app.yourfrontend.com',
  'http://localhost:3000', // Development
];

const corsOptions = {
  origin: (origin, callback) => {
    // Allow requests with no origin (mobile apps, Postman)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
};

app.use(cors(corsOptions));
```

### Manual CORS Headers

```javascript
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://yourfrontend.com');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.header('Access-Control-Allow-Credentials', 'true');
  
  // Handle preflight
  if (req.method === 'OPTIONS') {
    return res.sendStatus(200);
  }
  
  next();
});
```

### Common CORS Issues

**Problem:** "No 'Access-Control-Allow-Origin' header is present"

- **Solution:** Enable CORS on your server or add the origin to whitelist

**Problem:** Credentials not being sent

- **Solution:** Set `credentials: true` in CORS config AND use `credentials: 'include'` in fetch requests

**Problem:** Custom headers blocked

- **Solution:** Add headers to `allowedHeaders` in CORS config

---

## Horizontal vs Vertical Scaling

Scaling is how your application handles increased load. There are two primary approaches.

### Vertical Scaling (Scale Up)

Increase the resources of a single server.

**Pros:**

- Simple to implement
- No architectural changes needed
- No data consistency issues
- Lower licensing costs

**Cons:**

- Hardware limits (can't scale infinitely)
- Single point of failure
- Downtime during upgrades
- More expensive at higher tiers

**Example:**

```
Before: 2 CPU cores, 4GB RAM
After:  8 CPU cores, 32GB RAM
```

### Horizontal Scaling (Scale Out)

Add more servers/instances to distribute load.

**Pros:**

- Theoretically unlimited scaling
- Better fault tolerance
- No downtime during scaling
- Cost-effective with cloud providers

**Cons:**

- Complex architecture
- Requires load balancing
- Data consistency challenges
- Session management complexity

**Example:**

```
Before: 1 server
After:  5 servers behind load balancer
```

### Node.js Horizontal Scaling with Cluster Module

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;
const express = require('express');

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    // Restart worker
    cluster.fork();
  });
} else {
  // Workers share the same TCP connection
  const app = express();
  
  app.get('/', (req, res) => {
    res.send(`Handled by worker ${process.pid}`);
  });

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
}
```

---

## 🟢 8️⃣ System Design (Mid-Level Expectation)

- [ ] **Design notification system.**
- [ ] **Design order processing system.**
- [ ] **Design scalable file upload service.**
- [ ] **Design chat system (basic).**
- [ ] **How would you handle 100k concurrent users?**
- [ ] **How to handle API rate limiting?**
- [ ] **How to design audit logging system?**
- [ ] **How to design multi-tenant SaaS?**
- [ ] **REST vs GraphQL?**
- [ ] **How to design microservice communication?**
