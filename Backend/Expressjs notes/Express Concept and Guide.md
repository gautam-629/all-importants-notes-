# Middleware in Express.js

**Middleware functions** are functions that have access to:  

- The **request object** (`req`)  
- The **response object** (`res`)  
- The **next function** (`next`) in the application’s request-response cycle  

They can perform the following tasks:

1. Execute any code.
2. Make changes to the request and response objects.
3. End the request-response cycle.
4. Call the next middleware in the stack.

> **Note:** To load a middleware function, use `app.use()`.

---

## Types of Middleware

### 1. Application-level Middleware
- Bound to the `app` object using `app.use()` or similar methods.
- Executed for **every request** to the app.

**Example:**
```javascript
const express = require('express');
const app = express();

// Application-level middleware
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next(); // Pass to the next middleware
});

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000);
```

---

### 2. Router-level Middleware
- Bound to instances of `express.Router()`.
- Only invoked for routes within that router.

**Example:**
```javascript
const express = require('express');
const router = express.Router();

// Router-level middleware
router.use((req, res, next) => {
  console.log('Request URL:', req.originalUrl);
  next();
});

router.get('/users', (req, res) => {
  res.send('User List');
});

app.use('/api', router); // Routes prefixed with /api
```

---

### 3. Error-handling Middleware
- Defined with **four parameters**: `(err, req, res, next)`.
- Used to handle errors in the application.

**Example:**
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

---

### 4. Built-in Middleware
#### 4.1 express.static
- Serves **static assets** like HTML files, images, etc.

```javascript
app.use(express.static('public')); // Serves files from the 'public' folder
```

#### 4.2 express.json
- Parses incoming requests with **JSON payloads**.

```javascript
app.use(express.json()); // Available in Express 4.16.0+
```

#### 4.3 express.urlencoded
- Parses incoming requests with **URL-encoded payloads**.

```javascript
app.use(express.urlencoded({ extended: true })); // Available in Express 4.16.0+
```

---

### 5. Third-party Middleware
- Example: `cookie-parser` for parsing cookies.

```javascript
const cookieParser = require('cookie-parser');
app.use(cookieParser());

app.get('/', (req, res) => {
  console.log('Cookies: ', req.cookies);
  res.send('Cookies parsed!');
});
```