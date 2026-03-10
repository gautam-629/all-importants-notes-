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
