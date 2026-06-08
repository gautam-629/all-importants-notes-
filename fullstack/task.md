
### Master Socket.io · Redis · BullMQ · Cron Jobs · S3 File Uploads · Microservices · SaaS · CI/CD

> **Project:** DevFeed — a developer content platform where devs post code snippets with images, follow each other, get real-time notifications, and (in Phase 3) subscribe to plans that unlock premium features.
> 
> **Goal of this document:** Teach you _why_ each technology exists before showing you _how_ to use it. Read every concept section before you implement it.
> 
> **Stack:** Express.js · PostgreSQL · Redis · Socket.io · BullMQ · AWS S3 · Docker · GitHub Actions · Stripe

---

## Table of Contents

1. [Concept Explanations — Why You Need Each Technology](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#1-concept-explanations)
    - 1.1 WebSockets & Socket.io
    - 1.2 Redis Caching
    - 1.3 Redis Queue (BullMQ)
    - 1.4 Redis Pub/Sub & Streams
    - 1.5 Cron Jobs
    - 1.6 File Uploads & AWS S3
    - 1.7 Microservice Architecture
    - 1.8 SaaS, Multi-Tenancy & Subscriptions
2. [Phase 1 — Monolithic App (Days 1–18)](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#2-phase-1--monolithic-app-days-118)
3. [Phase 2 — Microservice Architecture (Days 19–33)](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#3-phase-2--microservice-architecture-days-1933)
4. [Phase 3 — SaaS, Subscriptions & Multi-Tenancy (Days 34–45)](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#4-phase-3--saas-subscriptions--multi-tenancy-days-3445)
5. [CI/CD with GitHub Actions](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#5-cicd-with-github-actions)
6. [Database Schemas](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#6-database-schemas)
7. [Folder Structures](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#7-folder-structures)
8. [Environment Variables Reference](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#8-environment-variables-reference)
9. [Daily Schedule Template](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#9-daily-schedule-template)

---

## 1. Concept Explanations

> **Rule:** Read each section completely before you implement it. Knowing _why_ a tool exists removes 80% of confusion during implementation.

---

### 1.1 WebSockets & Socket.io

#### The Problem with Plain HTTP

HTTP follows a strict request → response cycle. The client asks a question, the server answers, and the connection closes. For most things (loading a page, fetching data) this works perfectly.

But for real-time features it fails completely. If someone likes your post, how does your browser know? With HTTP only, you have two bad choices:

- **Short polling:** browser asks the server every 3 seconds "anything new?" — wastes 99% of requests, burns server CPU
- **Long polling:** browser asks and server holds the connection open until something happens — hard to scale, messy to implement

#### The WebSocket Solution

A WebSocket is a persistent, bidirectional connection between client and server. Once opened, either side can send data at any time without waiting for the other to ask first. The connection stays alive until explicitly closed.

```
HTTP (request-response):
  Client ──── "give me the feed" ───► Server
  Client ◄─── "here is the feed" ──── Server
  [connection closes]
  Client ──── "anything new?" ───────► Server   ← wasteful polling
  Client ◄─── "nope" ─────────────── Server
  [connection closes]

WebSocket (persistent):
  Client ◄──────── open connection ──────────── Server
  Server ──── "new like on your post" ──────────► Client   ← instant push
  Server ──── "new follower" ───────────────────► Client
  Client ──── "user typing..." ─────────────────► Server
  [connection stays open]
```

#### Why Socket.io Instead of Raw WebSockets?

Raw WebSockets work, but you have to build everything yourself. Socket.io adds:

- **Automatic reconnection** — if the connection drops, it reconnects transparently
- **Fallback transports** — if WebSocket is blocked by a firewall, it falls back to HTTP long polling automatically
- **Rooms** — group connections by topic (`room: "user:42"`) and broadcast to everyone in a room
- **Namespaces** — separate channels on the same connection
- **Event-based API** — cleaner than raw `onmessage` handlers

#### Real-World Analogy

HTTP = sending letters by post (one question, one reply, conversation over).  
WebSocket = a phone call (either side speaks whenever they want, connection stays open).

#### In DevFeed you use it for

- Showing a notification badge the instant someone likes your post (no page refresh)
- Broadcasting real-time feed updates when someone you follow posts
- "User is typing" comment indicators

#### When NOT to use WebSockets

|Situation|Use instead|
|---|---|
|Loading a list of posts|REST GET endpoint|
|Background email sending|Queue (BullMQ)|
|Communication between backend services|Redis Pub/Sub or Streams|
|Scheduled tasks|Cron jobs|

---

### 1.2 Redis Caching

#### The Problem: Every Request Hits the Database

Every time a user loads the post feed, your app runs a PostgreSQL query. One query takes 20–200ms depending on data size and server load. That is fine for one user. But consider:

- 500 users load the feed simultaneously
- The feed query involves JOINs across posts, users, likes, and follows
- Every user gets the same result because the data hasn't changed in the last 30 seconds

You just ran 500 identical, expensive queries for no reason.

#### The Solution: Cache the Result

On the first request, run the database query and store the result in Redis with an expiry time (TTL — Time To Live). On every subsequent request within that TTL window, return the Redis result directly — no database involved. Redis reads from RAM, not disk, so it responds in under 1 millisecond.

```
Without cache:
  Request 1 ──► PostgreSQL (180ms) ──► Response
  Request 2 ──► PostgreSQL (175ms) ──► Response    ← same data, same cost
  Request 3 ──► PostgreSQL (190ms) ──► Response    ← same data, same cost

With cache (TTL = 60 seconds):
  Request 1 ──► Redis MISS ──► PostgreSQL (180ms) ──► store in Redis ──► Response
  Request 2 ──► Redis HIT  ──► Response (< 1ms)    ← 180x faster
  Request 3 ──► Redis HIT  ──► Response (< 1ms)    ← 180x faster
  [60 seconds pass, cache expires]
  Request 4 ──► Redis MISS ──► PostgreSQL (180ms) ──► store in Redis ──► Response
```

#### Key Concepts

|Concept|Meaning|
|---|---|
|Cache HIT|Key found in Redis — return immediately, skip DB|
|Cache MISS|Key not found — query DB, store result in Redis, return|
|TTL (Time To Live)|Seconds until a cache entry automatically expires|
|Cache invalidation|Manually deleting a cache key when the underlying data changes|
|Cache stampede|1,000 requests all miss cache at the same time and hammer the DB|

#### The Hard Problem: Cache Invalidation

When a user creates a new post, the cached feed is stale. You must delete the cache entry so the next request fetches fresh data. This sounds simple but gets complex fast:

- Which cache keys are affected? (User A's feed, User B's feed who follows User A...)
- What if you delete the wrong key?
- What if the DB write succeeds but the cache delete fails?

You will encounter and solve all of these in Phase 1.

#### Real-World Analogy

Without cache = every customer walks to the warehouse to collect their item.  
With cache = popular items are stocked at the front counter. Same item, instantly available, restocked when stock runs out (TTL expires).

#### In DevFeed you use it for

- Caching the post feed per user (TTL: 60 seconds)
- Caching user profiles (TTL: 5 minutes, invalidate on profile update)
- Caching post detail pages (TTL: 30 seconds, invalidate on like/edit)
- Rate limiting (store request counts per IP per minute)
- Session storage for quota tracking (Phase 3)

---

### 1.3 Redis Queue (BullMQ)

#### The Problem: Slow Tasks Block HTTP Responses

When a user registers, you want to:

1. Save their account to the database ✓ (fast, must be synchronous)
2. Send a welcome email (slow — 500ms–3 seconds)
3. Generate a default avatar (slow — image processing)
4. Notify analytics system (slow — external API call)

If you do all of this synchronously in your route handler, the HTTP response is blocked for 4+ seconds. If the email server is temporarily down, the entire registration fails. The user blames your app, not the email server.

#### The Solution: Offload to a Queue

Your route handler only does the fast, critical work (save to DB). Then it drops a message into a queue ("please send a welcome email to this user") and immediately returns a response. A separate worker process picks up that message and does the slow work independently.

```
Without queue:
  POST /auth/register
    ├── Save user to DB         (20ms)   ← must happen now
    ├── Send welcome email      (2000ms) ← why is the user waiting for this?
    ├── Generate avatar         (500ms)  ← and this?
    └── Notify analytics        (300ms)  ← and this?
  Total response time: 2820ms  ← terrible UX

With queue:
  POST /auth/register
    ├── Save user to DB         (20ms)   ← must happen now
    └── Add 3 jobs to queue     (2ms)    ← instant
  Total response time: 22ms    ← great UX

  [Worker process, running separately]
    ├── Job 1: Send welcome email      ← runs in background
    ├── Job 2: Generate avatar         ← runs in background
    └── Job 3: Notify analytics        ← runs in background
```

#### Why BullMQ Specifically?

BullMQ is built on Redis (which you already have) and gives you:

- **Automatic retries with backoff** — if the email server is down, the job retries after 5 seconds, then 30 seconds, then 5 minutes
- **Dead letter queue** — jobs that fail too many times are moved to a "failed" bucket for inspection
- **Concurrency control** — process 5 jobs simultaneously, not one at a time
- **Priority** — password reset emails skip ahead of digest emails
- **Scheduled/delayed jobs** — "send this email in 1 hour"
- **Bull Board dashboard** — visual UI to see all queues, jobs, and failures

#### Key Concepts

|Concept|Meaning|
|---|---|
|Producer|Code that adds jobs to the queue (your route handler)|
|Consumer / Worker|Separate process that runs jobs|
|Job|A unit of work: `{ name: 'welcome-email', data: { userId, email } }`|
|Queue|Named list of jobs waiting to be processed|
|Backoff|Wait longer between each retry (1s, 5s, 30s, 5m...)|
|Dead letter|Permanently failed jobs stored for manual inspection|
|Concurrency|How many jobs a worker runs in parallel|

#### Real-World Analogy

Without queue = a restaurant chef stops cooking to personally deliver every dish to every table.  
With queue = chef puts finished dishes on the pass. Waiters (workers) deliver them. Chef never stops cooking.

#### In DevFeed you use it for

- Welcome emails (triggered on registration)
- "Someone liked your post" email digests
- Image compression after S3 upload (Phase 1)
- Monthly invoice generation (Phase 3)
- Bulk notification sending

---

### 1.4 Redis Pub/Sub & Streams

#### The Problem: Tight Coupling Between Parts of Your App

In a monolith, when a user likes a post, the like handler might directly call `notificationService.send()`. This is fine until:

- The notification service has a bug and crashes — your like feature crashes too
- You want to add a third feature that also reacts to likes — now you have to edit the like handler again
- In microservices, "calling" another service means an HTTP request — if that service is down, your request fails

#### The Solution: Events (Publish/Subscribe)

Instead of calling code directly, publish an event ("something happened") and let interested parties subscribe to it. The publisher doesn't know or care who is listening. New subscribers can be added without touching the publisher.

```
Without Pub/Sub (tight coupling):
  Like Handler ──calls──► Notification Service   ← breaks if notification is down
  Like Handler ──calls──► Analytics Service
  Like Handler ──calls──► Feed Ranking Service   ← must edit Like Handler each time

With Pub/Sub (loose coupling):
  Like Handler ──publishes──► "post.liked" event ──► [Redis channel]
                                                          │
                              ┌───────────────────────────┤
                              ▼                           ▼                      ▼
                    Notification Service       Analytics Service        Feed Ranking Service
                    (subscribed to channel)    (subscribed to channel)  (subscribed to channel)
```

#### Redis Pub/Sub vs Redis Streams

Redis offers two messaging systems. Understanding the difference is important:

||Redis Pub/Sub|Redis Streams|
|---|---|---|
|Persistence|No — messages are lost if subscriber is offline|Yes — messages are stored and can be replayed|
|Delivery|At-most-once (fire and forget)|At-least-once (consumer groups acknowledge)|
|Consumer groups|No|Yes — multiple workers share the load|
|Use case|Real-time notifications where losing some is acceptable|Cross-service events where every event must be processed|
|Phase used|Phase 1 (monolith)|Phase 2 (microservices)|

#### Real-World Analogy

Pub/Sub = a radio station. Broadcasts on a frequency. Any radio tuned in receives the signal. If your radio is off, you miss the broadcast.  
Streams = a recorded podcast. Published to a feed. Listeners can tune in late and catch up from where they left off.

#### In DevFeed you use it for

- Phase 1: Backend publishes "post liked" → Redis Pub/Sub → Socket.io pushes to browser
- Phase 2: Post Service publishes to Redis Stream → Notification Service consumes → email sent
- Phase 2: Auth Service publishes "user banned" → all services invalidate that user's sessions

---

### 1.5 Cron Jobs

#### The Problem: Some Work Has No Trigger

Most of your app runs in response to a user action (HTTP request). But some work needs to happen automatically on a schedule:

- Every morning at 8 AM: send users a digest of activity they missed
- Every hour: delete expired refresh tokens from the database
- On the 1st of every month: generate invoices for Pro subscribers
- Every night at midnight: recalculate trending post scores

You cannot wait for a user to trigger these — they must run automatically on a timer.

#### The Solution: Cron Jobs

A cron job is a piece of code scheduled to run at specific times. The name comes from the Unix `cron` daemon. Schedules are defined using a 5-part expression:

```
┌─────────── minute        (0–59)
│  ┌──────── hour          (0–23)
│  │  ┌───── day of month  (1–31)
│  │  │  ┌── month         (1–12)
│  │  │  │  ┌─ day of week (0–7,  0 and 7 = Sunday)
│  │  │  │  │
*  *  *  *  *
```

#### Common Cron Expressions

|Expression|Meaning|
|---|---|
|`0 8 * * *`|Every day at 8:00 AM|
|`0 * * * *`|Every hour on the hour|
|`*/15 * * * *`|Every 15 minutes|
|`0 0 1 * *`|1st of every month at midnight|
|`0 9 * * 1`|Every Monday at 9:00 AM|
|`0 */6 * * *`|Every 6 hours|

#### node-cron vs BullMQ Repeatable Jobs

||node-cron|BullMQ Repeatable|
|---|---|---|
|Where schedule is stored|In-process memory|Redis (persisted)|
|Survives process restart|No|Yes|
|Works across multiple instances|No (runs on every instance)|Yes (only one instance runs it)|
|Use in|Phase 1 learning|Phase 3 production|

You will use `node-cron` in Phase 1 because it is simple and teaches the concept cleanly. In Phase 3 you upgrade to BullMQ repeatable jobs which are production-safe.

#### In DevFeed you use it for

- Daily activity digest emails at 8 AM
- Hourly cleanup of expired refresh tokens
- Nightly S3 cleanup of orphaned image files (uploaded but post was never saved)
- Monthly invoice generation for paid subscribers

---

### 1.6 File Uploads & AWS S3

#### The Problem: You Cannot Store Files on Your Server

When a user uploads a profile picture or an image to attach to their post, the naive approach is to save the file on your Express server's disk. This fails in several critical ways:

1. **No persistence across deployments** — every time you redeploy (even with Docker), the server's filesystem is wiped. All uploaded images are deleted.
2. **No horizontal scaling** — if you run 3 instances of your app for load balancing, a file uploaded to instance 1 is not accessible from instances 2 or 3.
3. **Disk space** — a server with 20GB of disk will fill up quickly with user images.
4. **CDN impossible** — you cannot put a CDN in front of files on your local disk efficiently.
5. **Security** — serving user-uploaded files from the same server as your application code is a security risk.

#### The Solution: AWS S3

Amazon S3 (Simple Storage Service) is an object storage service designed for exactly this purpose. You upload files to S3, and S3 gives you a permanent URL to access them. It stores unlimited files, has 99.999999999% durability, and integrates with CloudFront CDN.

```
Without S3 (bad):
  Browser ──── POST /upload ──────────────────────────► Express server
  Express ──── save to disk (/uploads/avatar-42.jpg) ──► Server disk
  Browser ──── GET /uploads/avatar-42.jpg ─────────────► Express (serves file)
  [Deploy new version]
  /uploads/avatar-42.jpg ← DELETED  ← data loss!

With S3 (correct):
  Browser ──── POST /upload ────────────────────────────► Express server
  Express ──── upload buffer to S3 ─────────────────────► AWS S3 bucket
  S3 returns ─── "https://bucket.s3.amazonaws.com/..." ─► Express returns URL to browser
  Browser ──── GET https://bucket.s3.amazonaws.com/... ──► S3 directly (no Express involved)
  [Deploy new version — S3 files untouched]
```

#### Two Upload Strategies

**Strategy A — Server-side upload (what you build in Phase 1):**

```
Browser ──► POST multipart/form-data ──► Express ──► multer ──► AWS SDK ──► S3
```

The file comes to your server first, then your server uploads it to S3. Simpler to implement and understand. Slightly more bandwidth usage on your server.

**Strategy B — Presigned URLs (production best practice, optional reading):**

```
Browser ──► POST /upload/presign ──► Express ──► S3 SDK generates URL ──► Browser
Browser ──► PUT directly to S3 using presigned URL (bypasses your server entirely)
```

The browser uploads directly to S3. Your server never sees the file bytes. Better for large files and scaling. You will understand why this matters after building Strategy A.

#### Key S3 Concepts

|Concept|Meaning|
|---|---|
|Bucket|Top-level container for files (like a drive)|
|Object / Key|A file stored in S3. The key is the full path: `avatars/user-42.jpg`|
|Region|Which AWS data centre stores your bucket (e.g. `ap-south-1` for India)|
|ACL|Access control: public-read means anyone with the URL can access it|
|Presigned URL|A temporary, signed URL that allows someone to upload or download a specific object|
|Multipart upload|For large files (> 5MB), S3 requires splitting them into parts|
|Lifecycle policy|Rules like "delete files older than 30 days" — useful for temp upload cleanup|

#### S3 + Image Processing Pipeline

Raw uploaded images are often too large for a web app. A best practice is to:

1. Accept the upload
2. Store the original in S3 at `originals/user-42/abc123.jpg`
3. Add a BullMQ job: "compress and resize this image"
4. Worker downloads from S3, processes with `sharp`, re-uploads to `thumbnails/user-42/abc123.jpg`
5. Store both URLs in your database

This is exactly what you will build. It combines S3 + BullMQ + cron (for cleanup) in one feature.

#### In DevFeed you use it for

- User profile picture uploads
- Images attached to posts (code screenshots, architecture diagrams)
- Cover images for profiles
- Phase 3: tenant logo uploads

---

### 1.7 Microservice Architecture

#### The Problem with a Growing Monolith

Your monolith works. But as it grows:

- **Scaling is inefficient** — if the notification service is slow, you must scale the entire app (including auth, posts, etc.) just to handle notification load
- **One failure can crash everything** — a bug in the image processing code throws an unhandled exception that crashes the entire process, taking down auth and posts with it
- **Deployment is risky** — changing one small feature requires redeploying the entire app
- **Team conflicts** — two developers editing the same `app.js` file constantly create merge conflicts

#### The Solution: Split into Services

Break the app into separate, independently deployable processes. Each service:

- Has a single responsibility (auth only, posts only, notifications only)
- Owns its own data (its own database tables, or even its own database)
- Deploys independently without touching other services
- Can be scaled independently (run 10 notification service instances, 2 auth instances)
- Fails independently (notification service crash doesn't affect login)

#### The Tradeoffs — Be Honest

|Aspect|Monolith|Microservices|
|---|---|---|
|Development complexity|Low|High|
|Operational complexity|Low|High (orchestration, service discovery)|
|Deployment|One thing to deploy|Many things to coordinate|
|Scaling|Scale entire app|Scale only what needs it|
|Fault isolation|One crash = everything down|Service crash = only that feature down|
|Testing|Easy (in-process)|Hard (requires running multiple services)|
|Network latency|Zero (function calls)|Added (HTTP / Redis between services)|
|Best for|Early stage, 1–3 developers|Growth stage, multiple teams|

**The most important lesson:** Start with a monolith. You can only split services correctly once you deeply understand how features connect to each other. This is why Phase 1 comes before Phase 2.

#### In DevFeed Phase 2 you split into

|Service|Port|Responsibility|
|---|---|---|
|API Gateway|3000|Single entry point, JWT validation, routing|
|Auth Service|3001|Register, login, token refresh, user profiles|
|Post Service|3002|Posts CRUD, likes, follows, image uploads to S3|
|Notification Service|3003|Real-time sockets, email queue, event consumption|

---

### 1.8 SaaS, Multi-Tenancy & Subscriptions

#### What is a Multi-Tenant SaaS?

A **SaaS** (Software as a Service) application sells access to software on a subscription basis. **Multi-tenancy** means multiple organisations (**tenants**) share the same application and infrastructure — but are completely isolated from each other's data.

Think Slack, Notion, or GitHub: one codebase, thousands of organisations, each organisation's data is invisible to others.

#### Multi-Tenancy Implementation Strategies

|Strategy|How it works|Isolation|Cost|Complexity|
|---|---|---|---|---|
|Database per tenant|Each org gets their own PostgreSQL database|Strongest|Highest|Very high|
|Schema per tenant|One DB, separate Postgres schema (`acme.posts`, `beta.posts`)|Strong|Medium|High|
|Row-level isolation|Shared tables, `tenant_id` column on every row|Good (requires discipline)|Lowest|Low|

**DevFeed uses row-level isolation** — it is the most practical for learning and the most common in real SaaS products. The critical rule: every single database query must filter by `tenant_id`. One forgotten `WHERE tenant_id = $1` exposes another organisation's data.

#### Subscription Plans & Feature Gating

Tenants pay for different tiers of access. Your code must enforce this:

```
Plan: Free
  → Can create up to 5 posts per day
  → Cannot access analytics
  → Cannot upload images > 2MB

Plan: Pro ($29/month)
  → Unlimited posts
  → Analytics dashboard
  → Images up to 20MB
  → S3 storage quota: 10GB per tenant

Plan: Enterprise ($99/month)
  → Everything in Pro
  → Custom S3 bucket (their own AWS account)
  → API access for programmatic posting
  → Unlimited S3 storage
```

#### Stripe — The Billing Standard

Stripe is the industry standard for subscription billing. It handles:

- Collecting payment details securely (you never touch card numbers)
- Recurring billing on a schedule
- Failed payment retries
- Proration when users upgrade/downgrade mid-cycle
- Generating and emailing invoices

Your app integrates with Stripe via **webhooks** — Stripe calls a URL on your server whenever something happens (payment succeeded, payment failed, subscription cancelled). You must handle these to keep your database in sync with the billing state.

---

## 2. Phase 1 — Monolithic App (Days 1–18)

### What You Are Building

A single Express.js application that handles everything: auth, posts, real-time notifications, background emails, cron jobs, and S3 image uploads. This is the foundation every subsequent phase builds on.

### Complete API Routes (Phase 1)

```
Auth
  POST   /auth/register          Create account, enqueue welcome email
  POST   /auth/login             Return JWT + refresh token
  POST   /auth/refresh           Exchange refresh token for new JWT
  POST   /auth/logout            Invalidate refresh token

Users
  GET    /users/:id              Public profile
  PUT    /users/me               Update own profile
  POST   /users/me/avatar        Upload profile picture → S3
  DELETE /users/me/avatar        Delete profile picture from S3
  POST   /users/:id/follow       Follow / unfollow toggle
  GET    /users/:id/followers    Paginated follower list
  GET    /users/:id/following    Paginated following list

Posts
  GET    /posts                  Paginated feed (cached 60s)
  GET    /posts/:id              Single post detail (cached 30s)
  POST   /posts                  Create post (optionally with image)
  PUT    /posts/:id              Update post (owner only)
  DELETE /posts/:id              Delete post + S3 image (owner only)
  POST   /posts/:id/like         Toggle like, emits socket event

Uploads
  POST   /upload/image           Upload image → S3, return URL
  DELETE /upload/image           Delete image from S3 by key

Notifications
  GET    /notifications          Paginated list for authenticated user
  PUT    /notifications/:id/read Mark notification as read
  PUT    /notifications/read-all Mark all as read

Admin (internal use)
  GET    /admin/queues           Bull Board dashboard
  GET    /health                 Health check endpoint
```

---

### Week 1 (Days 1–6) — Foundation & Core API

#### Day 1: Project Initialisation

Set up the project skeleton before writing any feature code.

```bash
mkdir devfeed && cd devfeed
npm init -y

# Core dependencies
npm install express pg dotenv bcrypt jsonwebtoken zod winston cors helmet express-rate-limit

# S3 and file upload
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner multer multer-s3 sharp

# Redis and queues
npm install ioredis bullmq @bull-board/express @bull-board/api

# Socket and scheduling
npm install socket.io node-cron

# Email
npm install nodemailer

# Dev dependencies
npm install -D eslint prettier jest supertest nodemon @types/node
```

Create `.env.example` with every variable you will need:

```env
# Server
NODE_ENV=development
PORT=3000

# PostgreSQL
DATABASE_URL=postgres://postgres:password@localhost:5432/devfeed

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d

# AWS S3
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
S3_BUCKET_NAME=devfeed-uploads
S3_BASE_URL=https://devfeed-uploads.s3.ap-south-1.amazonaws.com

# Email (use Mailtrap for development)
SMTP_HOST=sandbox.smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=your-mailtrap-user
SMTP_PASS=your-mailtrap-pass
EMAIL_FROM=noreply@devfeed.io
```

#### Day 2: Database Migrations

Create SQL migration files in `src/db/migrations/`. Run them manually with `psql` during development. In Phase 2 you add a migration runner.

See the complete schemas in [Section 6](https://claude.ai/chat/374f49df-1280-4061-bcf4-ccbd7192a8a5#6-database-schemas).

**Seed the database** with test data so you are not developing against an empty database:

```bash
node src/db/seed.js   # creates 5 users, 20 posts, some follows and likes
```

#### Day 3: Auth Routes

Build registration, login, refresh, and logout. Hash passwords with bcrypt. Sign JWTs with a 15-minute expiry. Store refresh tokens (hashed) in the database with a 7-day expiry.

```javascript
// src/controllers/auth.controller.js  — registration example
export const register = async (req, res, next) => {
  try {
    const { username, email, password } = req.body; // validated by Zod middleware first

    // Check uniqueness
    const existing = await db.query(
      'SELECT id FROM users WHERE email = $1 OR username = $2',
      [email, username]
    );
    if (existing.rows.length) {
      return res.status(409).json({ error: 'Email or username already taken' });
    }

    // Hash password
    const passwordHash = await bcrypt.hash(password, 12);

    // Save user
    const { rows } = await db.query(
      `INSERT INTO users (username, email, password_hash)
       VALUES ($1, $2, $3) RETURNING id, username, email, created_at`,
      [username, email, passwordHash]
    );
    const user = rows[0];

    // Issue tokens
    const accessToken = signJWT({ sub: user.id, username: user.username });
    const refreshToken = await createRefreshToken(user.id);

    // Enqueue welcome email (non-blocking)
    await emailQueue.add('welcome', { userId: user.id, email: user.email, username: user.username });

    res.status(201).json({ user, accessToken, refreshToken });
  } catch (err) {
    next(err); // global error handler
  }
};
```

#### Day 4: Posts CRUD

Build the posts API with proper ownership checks. The `GET /posts` endpoint is the most important — it is the one you will cache in Week 2.

```javascript
// src/controllers/post.controller.js — paginated feed
export const getFeed = async (req, res, next) => {
  try {
    const { page = 1, limit = 20 } = req.query;
    const offset = (page - 1) * limit;

    const { rows } = await db.query(
      `SELECT
         p.id, p.content, p.language, p.tags, p.likes_count,
         p.image_url, p.image_key,
         p.created_at, p.updated_at,
         json_build_object('id', u.id, 'username', u.username, 'avatar_url', u.avatar_url) AS author
       FROM posts p
       JOIN users u ON u.id = p.author_id
       ORDER BY p.created_at DESC
       LIMIT $1 OFFSET $2`,
      [limit, offset]
    );

    res.json({
      posts: rows,
      pagination: { page: Number(page), limit: Number(limit), hasMore: rows.length === limit }
    });
  } catch (err) {
    next(err);
  }
};
```

#### Day 5: User Routes + Follow System

Build `GET /users/:id`, follow/unfollow toggle, follower lists. The follow system affects the feed (Phase 2 will make the feed show posts from followed users only).

#### Day 6: Integration Tests

Write tests for every route you have built so far. Tests run in CI — if they pass in CI, your code is deployable.

```javascript
// tests/auth.test.js
describe('POST /auth/register', () => {
  it('creates a user and returns tokens', async () => {
    const res = await request(app)
      .post('/auth/register')
      .send({ username: 'testuser', email: 'test@example.com', password: 'Password123!' });

    expect(res.status).toBe(201);
    expect(res.body).toHaveProperty('accessToken');
    expect(res.body).toHaveProperty('refreshToken');
    expect(res.body.user.password_hash).toBeUndefined(); // never expose password hash
  });

  it('rejects duplicate email', async () => {
    await createUser({ email: 'dupe@example.com' });
    const res = await request(app)
      .post('/auth/register')
      .send({ username: 'other', email: 'dupe@example.com', password: 'Password123!' });

    expect(res.status).toBe(409);
  });
});
```

---

### Week 2 (Days 7–12) — Redis, Sockets, Queues, S3

#### Day 7: Redis Caching

Connect ioredis and write the caching middleware. Apply it to `GET /posts` and `GET /posts/:id`.

```javascript
// src/services/redis.service.js
import Redis from 'ioredis';

export const redis = new Redis(process.env.REDIS_URL, {
  maxRetriesPerRequest: 3,
  retryDelayOnFailover: 100,
  lazyConnect: false,
});

redis.on('connect', () => logger.info('Redis connected'));
redis.on('error', (err) => logger.error('Redis error', { err: err.message }));

export const cache = {
  get: async (key) => {
    const val = await redis.get(key);
    return val ? JSON.parse(val) : null;
  },
  set: async (key, value, ttlSeconds) => {
    await redis.setex(key, ttlSeconds, JSON.stringify(value));
  },
  del: async (...keys) => {
    if (keys.length) await redis.del(...keys);
  },
  // Delete all keys matching a pattern — use carefully, expensive on large datasets
  delPattern: async (pattern) => {
    const keys = await redis.keys(pattern);
    if (keys.length) await redis.del(...keys);
  },
};
```

```javascript
// src/middleware/cache.middleware.js
import { cache } from '../services/redis.service.js';
import { logger } from '../utils/logger.js';

export const cacheMiddleware = (ttlSeconds, keyFn) => async (req, res, next) => {
  // keyFn lets callers customise the cache key per route
  const key = keyFn ? keyFn(req) : `cache:${req.originalUrl}`;

  try {
    const cached = await cache.get(key);
    if (cached) {
      logger.debug('Cache HIT', { key });
      return res.json(cached);
    }
    logger.debug('Cache MISS', { key });
  } catch (err) {
    // If Redis is down, fall through to DB — never let cache failure break the app
    logger.warn('Cache read failed, falling through', { err: err.message });
  }

  // Intercept res.json to also store the response in cache
  const originalJson = res.json.bind(res);
  res.json = async (data) => {
    try {
      await cache.set(key, data, ttlSeconds);
    } catch (err) {
      logger.warn('Cache write failed', { err: err.message });
    }
    return originalJson(data);
  };

  next();
};

// Usage in routes:
// router.get('/', cacheMiddleware(60, (req) => `feed:${req.user.id}:${req.query.page}`), getFeed);
```

**Cache invalidation — invalidate on write:**

```javascript
// In post.controller.js — after creating a post
export const createPost = async (req, res, next) => {
  try {
    // ... save post to DB ...

    // Invalidate feed cache for this user AND all their followers
    await cache.del(`feed:${req.user.id}`);
    const followers = await getFollowerIds(req.user.id);
    for (const followerId of followers) {
      await cache.del(`feed:${followerId}`);
    }

    res.status(201).json({ post });
  } catch (err) {
    next(err);
  }
};
```

#### Day 8: AWS S3 Setup & Image Uploads

Set up S3 and build the upload endpoint. This is a key feature that combines multer (multipart parsing), sharp (image processing), and the AWS SDK.

**Step 1: Create S3 bucket**

1. Go to AWS Console → S3 → Create Bucket
2. Name: `devfeed-uploads` (must be globally unique — add your name)
3. Region: pick closest to your users
4. Block all public access: **OFF** (posts with images need to be publicly readable)
5. Enable versioning: OFF (keeps cost down while learning)
6. Create bucket

**Step 2: Create IAM user**

1. AWS Console → IAM → Users → Create User
2. Name: `devfeed-s3-user`
3. Attach policy: `AmazonS3FullAccess` (restrict to specific bucket in production)
4. Create access key → save `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

**Step 3: Implement the upload service**

```javascript
// src/services/s3.service.js
import {
  S3Client,
  PutObjectCommand,
  DeleteObjectCommand,
  GetObjectCommand,
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import sharp from 'sharp';
import { v4 as uuidv4 } from 'uuid';
import path from 'path';

const s3 = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  },
});

const BUCKET = process.env.S3_BUCKET_NAME;
const BASE_URL = process.env.S3_BASE_URL;

// Upload a file buffer to S3
export const uploadToS3 = async ({ buffer, mimetype, folder = 'posts' }) => {
  const ext = mimetype.split('/')[1] || 'jpg';
  const key = `${folder}/${uuidv4()}.${ext}`;

  await s3.send(new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    Body: buffer,
    ContentType: mimetype,
  }));

  return {
    key,
    url: `${BASE_URL}/${key}`,
  };
};

// Upload an image with automatic resizing
export const uploadImageToS3 = async ({ buffer, mimetype, folder = 'posts', maxWidth = 1200 }) => {
  // Resize image if wider than maxWidth, convert to WebP for smaller file size
  const processed = await sharp(buffer)
    .resize({ width: maxWidth, withoutEnlargement: true })
    .webp({ quality: 85 })
    .toBuffer();

  return uploadToS3({ buffer: processed, mimetype: 'image/webp', folder });
};

// Upload thumbnail version
export const uploadThumbnailToS3 = async ({ buffer, folder = 'thumbnails' }) => {
  const thumb = await sharp(buffer)
    .resize({ width: 400, height: 400, fit: 'cover' })
    .webp({ quality: 70 })
    .toBuffer();

  return uploadToS3({ buffer: thumb, mimetype: 'image/webp', folder });
};

// Delete a file from S3
export const deleteFromS3 = async (key) => {
  await s3.send(new DeleteObjectCommand({ Bucket: BUCKET, Key: key }));
};

// Generate a temporary presigned URL for private objects (Phase 3: Enterprise plan)
export const getPresignedDownloadUrl = async (key, expiresInSeconds = 3600) => {
  const command = new GetObjectCommand({ Bucket: BUCKET, Key: key });
  return getSignedUrl(s3, command, { expiresIn: expiresInSeconds });
};
```

**Step 4: Build the upload middleware and routes**

```javascript
// src/middleware/upload.middleware.js
import multer from 'multer';

// Store in memory (buffer) — we process with sharp before sending to S3
const storage = multer.memoryStorage();

const fileFilter = (req, file, cb) => {
  const allowed = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  if (allowed.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Only JPEG, PNG, GIF, and WebP images are allowed'), false);
  }
};

export const uploadImage = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 10 * 1024 * 1024,  // 10MB max (Phase 3: enforce per-plan limits here)
    files: 1,
  },
}).single('image');  // field name must be "image" in the form

// src/routes/upload.routes.js
import express from 'express';
import { authMiddleware } from '../middleware/auth.middleware.js';
import { uploadImage } from '../middleware/upload.middleware.js';
import { uploadImageToS3, uploadThumbnailToS3, deleteFromS3 } from '../services/s3.service.js';
import { imageQueue } from '../services/queue.service.js';

const router = express.Router();

// POST /upload/image — upload image, resize, send to S3
router.post('/image', authMiddleware, uploadImage, async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No image file provided' });
    }

    // Upload full-size version immediately (synchronous — user needs the URL now)
    const { key, url } = await uploadImageToS3({
      buffer: req.file.buffer,
      mimetype: req.file.mimetype,
      folder: 'posts',
    });

    // Enqueue thumbnail generation (async — happens in background)
    await imageQueue.add('generate-thumbnail', {
      key,
      buffer: req.file.buffer.toString('base64'),  // buffers must be serialised for queue
      uploadedBy: req.user.id,
    });

    res.json({
      key,
      url,
      thumbnailUrl: null,  // will be populated once the queue job completes
    });
  } catch (err) {
    next(err);
  }
});

// DELETE /upload/image — delete image from S3
router.delete('/image', authMiddleware, async (req, res, next) => {
  try {
    const { key } = req.body;
    if (!key) return res.status(400).json({ error: 'key is required' });

    // Security: verify the key belongs to this user (key starts with posts/{userId}/)
    // For simplicity in Phase 1, we trust the user owns the key they send
    // Phase 3: store key → userId mapping in DB and verify ownership

    await deleteFromS3(key);
    res.json({ deleted: true });
  } catch (err) {
    next(err);
  }
});
```

**Step 5: Wire uploads into the post creation flow**

```javascript
// POST /posts — create post with optional image
router.post('/', authMiddleware, uploadImage, async (req, res, next) => {
  try {
    const { content, language, tags } = req.body;
    let imageUrl = null;
    let imageKey = null;

    // If an image was attached, upload it to S3 first
    if (req.file) {
      const result = await uploadImageToS3({
        buffer: req.file.buffer,
        mimetype: req.file.mimetype,
        folder: `posts/${req.user.id}`,
      });
      imageUrl = result.url;
      imageKey = result.key;
    }

    const { rows } = await db.query(
      `INSERT INTO posts (author_id, content, language, tags, image_url, image_key)
       VALUES ($1, $2, $3, $4, $5, $6)
       RETURNING *`,
      [req.user.id, content, language, JSON.parse(tags || '[]'), imageUrl, imageKey]
    );

    // Invalidate feed cache
    await cache.delPattern(`feed:*`);

    // Notify followers via Pub/Sub → Socket.io
    await publisher.publish('post.events', JSON.stringify({
      type: 'post.created',
      postId: rows[0].id,
      authorId: req.user.id,
    }));

    res.status(201).json({ post: rows[0] });
  } catch (err) {
    // If post save failed but S3 upload succeeded, clean up S3
    if (imageKey) await deleteFromS3(imageKey).catch(() => {});
    next(err);
  }
});

// DELETE /posts/:id — delete post and its S3 image
router.delete('/:id', authMiddleware, async (req, res, next) => {
  try {
    const { rows } = await db.query(
      'SELECT * FROM posts WHERE id = $1 AND author_id = $2',
      [req.params.id, req.user.id]
    );
    if (!rows.length) return res.status(404).json({ error: 'Post not found or not yours' });

    const post = rows[0];

    // Delete from database first
    await db.query('DELETE FROM posts WHERE id = $1', [post.id]);

    // Delete S3 image if one exists (fire-and-forget — don't fail the request if S3 delete fails)
    if (post.image_key) {
      deleteFromS3(post.image_key).catch((err) =>
        logger.error('S3 cleanup failed after post delete', { key: post.image_key, err: err.message })
      );
    }

    // Invalidate cache
    await cache.del(`post:${post.id}`);
    await cache.delPattern(`feed:*`);

    res.json({ deleted: true });
  } catch (err) {
    next(err);
  }
});
```

#### Day 9: BullMQ Queues — Email & Image Processing

Set up two queues: `emails` and `images`. Wire up workers to process them.

```javascript
// src/services/queue.service.js
import { Queue } from 'bullmq';
import { redis } from './redis.service.js';

const connection = { host: redis.options.host, port: redis.options.port };

export const emailQueue = new Queue('emails', { connection });
export const imageQueue = new Queue('images', { connection });

// src/workers/email.worker.js
import { Worker } from 'bullmq';
import nodemailer from 'nodemailer';

const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: process.env.SMTP_PORT,
  auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS },
});

const worker = new Worker('emails', async (job) => {
  logger.info('Processing email job', { jobName: job.name, jobId: job.id });

  switch (job.name) {
    case 'welcome':
      await transporter.sendMail({
        from: process.env.EMAIL_FROM,
        to: job.data.email,
        subject: 'Welcome to DevFeed!',
        html: `<h1>Hi ${job.data.username}!</h1><p>Welcome to DevFeed...</p>`,
      });
      break;

    case 'digest':
      const activity = await fetchUserDigest(job.data.userId);
      await transporter.sendMail({
        from: process.env.EMAIL_FROM,
        to: job.data.email,
        subject: 'Your DevFeed daily digest',
        html: renderDigestEmail(activity),
      });
      break;

    default:
      logger.warn('Unknown email job type', { jobName: job.name });
  }
}, {
  connection,
  concurrency: 5,
  // Retry failed jobs with exponential backoff
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 5000 },
  },
});

worker.on('completed', (job) => logger.info('Email job completed', { jobId: job.id }));
worker.on('failed', (job, err) => logger.error('Email job failed', { jobId: job.id, err: err.message }));
```

```javascript
// src/workers/image.worker.js
import { Worker } from 'bullmq';
import { uploadThumbnailToS3 } from '../services/s3.service.js';

const worker = new Worker('images', async (job) => {
  if (job.name === 'generate-thumbnail') {
    const buffer = Buffer.from(job.data.buffer, 'base64');
    const { key, url } = await uploadThumbnailToS3({ buffer });

    // Update the post record with thumbnail URL
    await db.query(
      'UPDATE posts SET thumbnail_url = $1 WHERE image_key = $2',
      [url, job.data.key]
    );

    logger.info('Thumbnail generated', { thumbnailKey: key });
  }
}, { connection, concurrency: 3 });
```

#### Day 10: Socket.io Real-Time Notifications

```javascript
// src/services/socket.service.js
import { Server } from 'socket.io';
import { verifyJWT } from '../utils/jwt.js';
import { subscriber } from './redis.service.js';  // dedicated subscriber connection
import { logger } from '../utils/logger.js';

export const initSocket = (httpServer) => {
  const io = new Server(httpServer, {
    cors: { origin: process.env.CLIENT_URL, credentials: true },
  });

  // Authenticate every socket connection using JWT
  io.use((socket, next) => {
    const token = socket.handshake.auth.token;
    if (!token) return next(new Error('Authentication required'));

    try {
      const payload = verifyJWT(token);
      socket.user = payload;
      next();
    } catch (err) {
      next(new Error('Invalid token'));
    }
  });

  io.on('connection', (socket) => {
    logger.info('Socket connected', { userId: socket.user.sub });

    // Each user joins their personal room so we can target notifications
    socket.join(`user:${socket.user.sub}`);

    socket.on('disconnect', () => {
      logger.info('Socket disconnected', { userId: socket.user.sub });
    });
  });

  // Subscribe to Redis Pub/Sub channel — when Post Service publishes events,
  // forward them to the correct socket room
  subscriber.subscribe('post.events', 'user.events');

  subscriber.on('message', (channel, message) => {
    try {
      const event = JSON.parse(message);

      switch (event.type) {
        case 'post.liked':
          io.to(`user:${event.postAuthorId}`).emit('notification', {
            type: 'post_liked',
            postId: event.postId,
            likedBy: event.likedByUsername,
            timestamp: new Date().toISOString(),
          });
          break;

        case 'user.followed':
          io.to(`user:${event.followedId}`).emit('notification', {
            type: 'new_follower',
            followedBy: event.followerUsername,
            timestamp: new Date().toISOString(),
          });
          break;

        case 'post.created':
          // Notify followers of new post (in a real app, query follower list and emit to each)
          // For simplicity, emit to a "following" room
          io.to(`following:${event.authorId}`).emit('new_post', {
            postId: event.postId,
            authorId: event.authorId,
          });
          break;
      }
    } catch (err) {
      logger.error('Failed to process socket event', { err: err.message, message });
    }
  });

  return io;
};
```

#### Day 11: Cron Jobs

```javascript
// src/jobs/digest.cron.js
import cron from 'node-cron';
import { emailQueue } from '../services/queue.service.js';
import { deleteFromS3 } from '../services/s3.service.js';
import { db } from '../db/index.js';
import { logger } from '../utils/logger.js';

// Daily digest email — every day at 8:00 AM
cron.schedule('0 8 * * *', async () => {
  logger.info('[CRON] Starting daily digest job');
  try {
    const { rows: users } = await db.query(`
      SELECT id, email, username
      FROM users
      WHERE last_active_at > NOW() - INTERVAL '7 days'
    `);

    for (const user of users) {
      await emailQueue.add('digest', {
        userId: user.id,
        email: user.email,
        username: user.username,
      }, {
        // Spread jobs over 2 hours to avoid SMTP rate limits
        delay: Math.random() * 2 * 60 * 60 * 1000,
      });
    }
    logger.info(`[CRON] Digest: queued ${users.length} email jobs`);
  } catch (err) {
    logger.error('[CRON] Digest job failed', { err: err.message });
  }
});

// Hourly refresh token cleanup
cron.schedule('0 * * * *', async () => {
  try {
    const { rowCount } = await db.query(
      'DELETE FROM refresh_tokens WHERE expires_at < NOW()'
    );
    logger.info(`[CRON] Cleaned up ${rowCount} expired refresh tokens`);
  } catch (err) {
    logger.error('[CRON] Token cleanup failed', { err: err.message });
  }
});

// Nightly S3 orphan cleanup — delete images uploaded but never attached to a post
// Runs at 2:00 AM every night
cron.schedule('0 2 * * *', async () => {
  logger.info('[CRON] Starting S3 orphan cleanup');
  try {
    // Find S3 keys that exist in uploads table but have no matching post
    const { rows: orphans } = await db.query(`
      SELECT key FROM s3_uploads
      WHERE created_at < NOW() - INTERVAL '24 hours'
        AND post_id IS NULL
    `);

    for (const { key } of orphans) {
      await deleteFromS3(key).catch((err) =>
        logger.warn('S3 orphan delete failed', { key, err: err.message })
      );
    }

    await db.query(`
      DELETE FROM s3_uploads
      WHERE created_at < NOW() - INTERVAL '24 hours'
        AND post_id IS NULL
    `);

    logger.info(`[CRON] S3 orphan cleanup: removed ${orphans.length} files`);
  } catch (err) {
    logger.error('[CRON] S3 orphan cleanup failed', { err: err.message });
  }
});
```

#### Day 12: Test the Full Week 2 Stack

Write integration tests that cover:

- Cache HIT and MISS behaviour on `GET /posts`
- Cache invalidation when a new post is created
- Image upload returns a valid S3 URL
- BullMQ receives a job when a user registers
- Socket.io notification emitted when a post is liked
- Cron jobs produce correct queue jobs (mock the schedule, call the handler directly)

---

### Week 3 (Days 13–18) — Docker, CI/CD, Security

#### Day 13: Dockerise the Application

```dockerfile
# Dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./

# Production dependencies only
FROM base AS production
RUN npm ci --production
COPY . .
EXPOSE 3000
CMD ["node", "src/app.js"]

# Development with hot reload
FROM base AS development
RUN npm ci
COPY . .
CMD ["npm", "run", "dev"]
```

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    build:
      context: .
      target: development
    ports:
      - "3000:3000"
    env_file: .env
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: devfeed
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./src/db/migrations:/docker-entrypoint-initdb.d  # auto-runs migrations on first start
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
  redisdata:
```

#### Day 14: GitHub Actions — CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: ['*']
  pull_request:
    branches: [main]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: devfeed_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run database migrations
        run: node src/db/migrate.js
        env:
          DATABASE_URL: postgres://postgres:testpass@localhost:5432/devfeed_test

      - name: Run tests
        run: npm test
        env:
          NODE_ENV: test
          DATABASE_URL: postgres://postgres:testpass@localhost:5432/devfeed_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret-do-not-use-in-production
          # Use fake S3 credentials for tests — mock S3 calls in test setup
          AWS_REGION: us-east-1
          AWS_ACCESS_KEY_ID: test
          AWS_SECRET_ACCESS_KEY: test
          S3_BUCKET_NAME: test-bucket
          S3_BASE_URL: https://test-bucket.s3.amazonaws.com
```

#### Day 15: GitHub Actions — CD Pipeline

```yaml
# .github/workflows/cd.yml
name: CD

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: []   # run after CI passes (configure branch protection to require CI)

    steps:
      - uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          target: production
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Deploy to server via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /opt/devfeed
            docker pull ghcr.io/${{ github.repository }}:latest
            docker compose up -d --no-deps --force-recreate app
            docker image prune -f
```

#### Days 16–18: Logging, Validation, Security, Review

**Day 16 — Logging:**

- Winston structured logging (JSON output in production, pretty in development)
- Request logging middleware: log every request with method, path, status code, duration, user ID
- Log every queue job start, complete, and fail
- Log every S3 upload and delete

**Day 17 — Validation & Security:**

- Zod schemas for every request body — reject malformed input before it reaches controllers
- `helmet()` middleware — sets 14 security-related HTTP headers automatically
- `express-rate-limit` + Redis store — 100 requests per 15 minutes per IP on auth routes
- Review all SQL queries — parameterised queries only, zero string concatenation
- File upload security: validate MIME type (not just extension), enforce size limits

**Day 18 — Phase 1 Review:**

- Write `README.md` with: local setup instructions, environment variables table, architecture diagram, API reference
- Run the full test suite — must pass in CI
- Manual end-to-end test: register → upload avatar → create post with image → like post → confirm notification appears via socket

---

## 3. Phase 2 — Microservice Architecture (Days 19–33)

### Why You Are Refactoring

You have a working monolith. Now you split it into services. The purpose of this phase is not to make the app "better" — it will actually get more complex. The purpose is to learn:

- How to define the right service boundaries
- How services communicate safely without coupling
- How to deploy each service independently
- How S3 uploads work when they cross service boundaries

### Service Responsibilities

|Service|Port|Owns|Publishes Events|
|---|---|---|---|
|API Gateway|3000|Routing, rate limiting, JWT validation|Nothing|
|Auth Service|3001|users, refresh_tokens, passwords|`user.registered`, `user.banned`|
|Post Service|3002|posts, likes, follows, S3 uploads|`post.created`, `post.liked`, `user.followed`, `image.uploaded`|
|Notification Service|3003|notifications, sockets, email queue|Nothing|

**S3 in microservices:** The Post Service owns all S3 uploads for posts. The Auth Service owns avatar uploads. Each service uses the same S3 bucket but with different key prefixes (`posts/`, `avatars/`) to separate concerns. They each have their own copy of `s3.service.js` using shared AWS credentials.

---

### Week 4 (Days 19–24) — Decomposition

#### Day 19: Monorepo Structure

```
devfeed-microservices/
├── gateway/
├── auth-service/
├── post-service/
├── notification-service/
├── shared/              # npm workspace — shared utilities
│   ├── package.json
│   ├── jwt.js
│   ├── events.js        # event type constants
│   ├── s3.service.js    # shared S3 utilities
│   └── logger.js
├── package.json         # workspace root
└── docker-compose.yml
```

Use npm workspaces so services can import from `shared/`:

```json
// package.json (root)
{
  "workspaces": ["gateway", "auth-service", "post-service", "notification-service", "shared"],
  "scripts": {
    "dev": "docker compose up",
    "test": "npm test --workspaces"
  }
}
```

#### Day 20: Auth Service

Identical logic to Phase 1 auth, but now it is a standalone Express app. Additionally, it exposes one new internal endpoint:

```javascript
// GET /internal/verify — validate a JWT, return user payload
// Used by the Gateway to verify tokens before forwarding
router.get('/internal/verify', async (req, res) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    const payload = verifyJWT(token);
    res.json({ valid: true, user: payload });
  } catch (err) {
    res.status(401).json({ valid: false, error: err.message });
  }
});
```

#### Day 21: Post Service with S3

The Post Service takes over all post-related routes AND image uploads. The key difference from Phase 1: instead of directly emitting socket events, it publishes to a Redis Stream.

```javascript
// post-service/src/events/publisher.js
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export const publishEvent = async (streamName, event) => {
  await redis.xadd(
    streamName,
    '*',           // auto-generate message ID
    'type', event.type,
    'payload', JSON.stringify(event)
  );
};

// Usage in post controller:
await publishEvent('post-events', {
  type: 'post.liked',
  postId: post.id,
  postAuthorId: post.authorId,
  likedBy: req.user.id,
  likedByUsername: req.user.username,
});
```

#### Day 22: Notification Service

Consumes events from Redis Streams and handles all real-time communication:

```javascript
// notification-service/src/consumers/event.consumer.js
import { Redis } from 'ioredis';
import { io } from '../socket/socket.service.js';
import { emailQueue } from '../services/queue.service.js';
import { db } from '../db/index.js';

const redis = new Redis(process.env.REDIS_URL);
const STREAM = 'post-events';
const GROUP = 'notification-consumers';
const CONSUMER = `worker-${process.env.HOSTNAME || 'local'}`;

const startConsumer = async () => {
  // Create consumer group if it doesn't exist
  try {
    await redis.xgroup('CREATE', STREAM, GROUP, '$', 'MKSTREAM');
  } catch (err) {
    if (!err.message.includes('BUSYGROUP')) throw err;
    // Group already exists — that's fine
  }

  logger.info('Notification consumer started', { stream: STREAM, group: GROUP });

  while (true) {  // Long-running consumer loop
    try {
      const results = await redis.xreadgroup(
        'GROUP', GROUP, CONSUMER,
        'COUNT', 10,
        'BLOCK', 2000,  // block for 2 seconds if no messages
        'STREAMS', STREAM, '>'
      );

      if (!results) continue;  // No messages — loop again

      for (const [, messages] of results) {
        for (const [messageId, fields] of messages) {
          try {
            await processEvent(fields);
            // Acknowledge message so it isn't redelivered
            await redis.xack(STREAM, GROUP, messageId);
          } catch (err) {
            logger.error('Failed to process event', { messageId, err: err.message });
            // Do NOT ack — message will be redelivered for retry
          }
        }
      }
    } catch (err) {
      logger.error('Consumer loop error', { err: err.message });
      await sleep(1000);  // Back off before retrying
    }
  }
};

const processEvent = async (fields) => {
  const type = fields[fields.indexOf('type') + 1];
  const payload = JSON.parse(fields[fields.indexOf('payload') + 1]);

  switch (type) {
    case 'post.liked':
      // 1. Save to notifications table
      await db.query(
        `INSERT INTO notifications (recipient_id, sender_id, type, payload)
         VALUES ($1, $2, 'post_liked', $3)`,
        [payload.postAuthorId, payload.likedBy, JSON.stringify({ postId: payload.postId })]
      );
      // 2. Push via socket
      io.to(`user:${payload.postAuthorId}`).emit('notification', {
        type: 'post_liked',
        postId: payload.postId,
        likedByUsername: payload.likedByUsername,
      });
      // 3. Optionally enqueue email notification
      await emailQueue.add('post-liked-notification', payload, {
        delay: 5 * 60 * 1000,  // 5 minute delay (batch notifications)
      });
      break;

    case 'image.uploaded':
      // Post Service uploaded an image — generate thumbnail in notification service's worker
      await imageQueue.add('generate-thumbnail', payload);
      break;
  }
};

startConsumer().catch((err) => {
  logger.error('Consumer failed to start', { err });
  process.exit(1);
});
```

#### Day 23: API Gateway with S3-Aware Routing

The gateway routes upload requests to the correct service:

```javascript
// gateway/src/app.js
import express from 'express';
import { createProxyMiddleware } from 'http-proxy-middleware';
import { authMiddleware } from './middleware/auth.middleware.js';
import { rateLimiter } from './middleware/rateLimit.middleware.js';

const app = express();

// Public routes — no auth required
app.use('/auth', rateLimiter, createProxyMiddleware({
  target: 'http://auth-service:3001',
  changeOrigin: true,
}));

// Protected routes — auth validated at gateway, user info forwarded to service
const protectedProxy = (target) => [
  authMiddleware,
  createProxyMiddleware({
    target,
    changeOrigin: true,
    on: {
      proxyReq: (proxyReq, req) => {
        // Forward verified user identity — services trust these headers from gateway
        proxyReq.setHeader('X-User-Id', req.user.sub);
        proxyReq.setHeader('X-User-Username', req.user.username);
        proxyReq.setHeader('X-Internal-Key', process.env.INTERNAL_SECRET);
      },
    },
  }),
];

// File uploads go to post-service (handles S3 directly)
// Note: must disable body parsing for multipart — proxy passes the raw stream
app.use('/upload', ...protectedProxy('http://post-service:3002'));

app.use('/posts', ...protectedProxy('http://post-service:3002'));
app.use('/users', ...protectedProxy('http://auth-service:3001'));
app.use('/notifications', ...protectedProxy('http://notification-service:3003'));
```

**Important: Multipart file uploads through a proxy**

Proxying multipart file uploads (for S3) through the gateway requires careful handling. The gateway must NOT parse the body — it must pass the raw multipart stream directly to the post-service. The `http-proxy-middleware` handles this transparently as long as you do not add `express.json()` middleware before the `/upload` route.

#### Day 24: Docker Compose for All Services

```yaml
# docker-compose.yml
version: '3.9'

services:
  gateway:
    build: ./gateway
    ports: ["3000:3000"]
    environment:
      AUTH_SERVICE_URL: http://auth-service:3001
      REDIS_URL: redis://redis:6379
      INTERNAL_SECRET: ${INTERNAL_SECRET}
    depends_on: [auth-service, post-service, notification-service]

  auth-service:
    build: ./auth-service
    environment:
      DATABASE_URL: postgres://postgres:password@postgres:5432/devfeed_auth
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET}
      AWS_REGION: ${AWS_REGION}
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      S3_BUCKET_NAME: ${S3_BUCKET_NAME}

  post-service:
    build: ./post-service
    environment:
      DATABASE_URL: postgres://postgres:password@postgres:5432/devfeed_posts
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET}
      INTERNAL_SECRET: ${INTERNAL_SECRET}
      AWS_REGION: ${AWS_REGION}
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      S3_BUCKET_NAME: ${S3_BUCKET_NAME}
      S3_BASE_URL: ${S3_BASE_URL}

  notification-service:
    build: ./notification-service
    environment:
      DATABASE_URL: postgres://postgres:password@postgres:5432/devfeed_notifications
      REDIS_URL: redis://redis:6379
      SMTP_HOST: ${SMTP_HOST}
      SMTP_USER: ${SMTP_USER}
      SMTP_PASS: ${SMTP_PASS}

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: password
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    volumes: [redisdata:/data]

volumes:
  pgdata:
  redisdata:
```

---

### Week 5 (Days 25–30) — Resilience, Observability

#### Day 25: Circuit Breaker for S3 Calls

In a microservice environment, S3 can become temporarily unavailable. Without a circuit breaker, all upload requests hang for 30 seconds before timing out. With a circuit breaker, after 5 failures the circuit opens and all requests fail fast with a clear error:

```javascript
// post-service/src/services/s3.resilient.js
import CircuitBreaker from 'opossum';
import { uploadImageToS3 as rawUpload, deleteFromS3 as rawDelete } from './s3.service.js';

const breakerOptions = {
  timeout: 10000,        // fail if function takes longer than 10 seconds
  errorThresholdPercentage: 50,  // open circuit if 50% of requests fail
  resetTimeout: 30000,   // try again after 30 seconds
};

const uploadBreaker = new CircuitBreaker(rawUpload, breakerOptions);
const deleteBreaker = new CircuitBreaker(rawDelete, breakerOptions);

uploadBreaker.on('open', () => logger.warn('S3 upload circuit OPEN — failing fast'));
uploadBreaker.on('halfOpen', () => logger.info('S3 upload circuit HALF-OPEN — testing'));
uploadBreaker.on('close', () => logger.info('S3 upload circuit CLOSED — normal'));

export const uploadImageToS3 = (args) => uploadBreaker.fire(args);
export const deleteFromS3 = (key) => deleteBreaker.fire(key);
```

#### Days 26–30: Logging, Health Checks, Service Auth, Integration Testing

Follow the same approach as Phase 1 but applied per-service. Key additions:

- **Correlation IDs:** Every request gets a `X-Request-Id` UUID that flows through all services, making it possible to trace a single user request through gateway → post-service → notification-service in logs
- **Health checks:** `GET /health` on each service returns `{ status, service, version, db: 'ok', redis: 'ok', s3: 'ok' }`
- **S3 health check:** ping S3 with a `HeadBucketCommand` — if it fails, the service is degraded

---

### Week 6 (Days 31–33) — Per-Service CI/CD

#### Day 31: Path-Based GitHub Actions

```yaml
# .github/workflows/post-service.yml
name: Post Service

on:
  push:
    paths:
      - 'post-service/**'
      - 'shared/**'
      - '.github/workflows/post-service.yml'
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: post-service
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: devfeed_posts_test
        options: >- --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
        ports: [5432:5432]
      redis:
        image: redis:7
        options: >- --health-cmd "redis-cli ping" --health-interval 10s --health-timeout 5s --health-retries 5
        ports: [6379:6379]
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
        env:
          DATABASE_URL: postgres://postgres:testpass@localhost:5432/devfeed_posts_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret
          # Mock S3 — use aws-sdk-client-mock in tests
          AWS_REGION: us-east-1
          AWS_ACCESS_KEY_ID: test
          AWS_SECRET_ACCESS_KEY: test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: ./post-service
          push: true
          tags: ghcr.io/${{ github.repository }}/post-service:${{ github.sha }}
      - name: Deploy post-service only
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /opt/devfeed
            docker pull ghcr.io/${{ github.repository }}/post-service:${{ github.sha }}
            docker compose up -d --no-deps --force-recreate post-service
```

---

## 4. Phase 3 — SaaS, Subscriptions & Multi-Tenancy (Days 34–45)

### New Subscription Plans with S3 Quotas

|Feature|Free|Pro ($29/mo)|Enterprise ($99/mo)|
|---|---|---|---|
|Posts per day|5|Unlimited|Unlimited|
|Image uploads|2MB max per file|20MB max per file|100MB max per file|
|S3 storage quota|500MB|10GB|Unlimited|
|Team members|1|10|Unlimited|
|Analytics|No|Yes|Yes|
|API access|No|Yes|Yes|
|Custom S3 bucket|No|No|Yes|

### Day-by-Day Plan

#### Days 34–36: Tenant Data Model & Middleware

Add multi-tenancy tables (see Section 6). Every request passes through the tenant middleware which resolves the tenant from the JWT and attaches `req.tenant` to the request.

#### Day 37: Feature Gating Middleware

```javascript
// src/middleware/plan.middleware.js
const PLAN_LIMITS = {
  free:       { image_max_bytes: 2 * 1024 * 1024,  storage_quota_bytes: 500 * 1024 * 1024 },
  pro:        { image_max_bytes: 20 * 1024 * 1024, storage_quota_bytes: 10 * 1024 * 1024 * 1024 },
  enterprise: { image_max_bytes: 100 * 1024 * 1024, storage_quota_bytes: Infinity },
};

const PLAN_FEATURES = {
  analytics:          ['pro', 'enterprise'],
  api_access:         ['pro', 'enterprise'],
  custom_s3_bucket:   ['enterprise'],
};

export const requireFeature = (feature) => (req, res, next) => {
  if (!PLAN_FEATURES[feature]?.includes(req.tenant.plan)) {
    return res.status(403).json({
      error: 'This feature requires a higher plan',
      feature,
      currentPlan: req.tenant.plan,
      requiredPlan: PLAN_FEATURES[feature][0],
      upgradeUrl: `https://devfeed.io/billing/upgrade`,
    });
  }
  next();
};

export const enforceImageSizeLimit = (req, res, next) => {
  const limit = PLAN_LIMITS[req.tenant.plan]?.image_max_bytes;
  if (req.file && req.file.size > limit) {
    return res.status(413).json({
      error: 'Image exceeds plan size limit',
      limitMB: limit / (1024 * 1024),
      plan: req.tenant.plan,
    });
  }
  next();
};
```

#### Day 38: S3 Storage Quota Enforcement

Track how much S3 storage each tenant uses. Enforce limits before upload.

```javascript
// S3 key pattern for tenants: tenants/{tenantId}/posts/{filename}
// This makes it trivial to calculate per-tenant storage usage

export const getTenantStorageUsed = async (tenantId) => {
  const { rows } = await db.query(
    'SELECT COALESCE(SUM(file_size_bytes), 0) AS total FROM s3_uploads WHERE tenant_id = $1',
    [tenantId]
  );
  return Number(rows[0].total);
};

export const checkStorageQuota = async (req, res, next) => {
  const limit = PLAN_LIMITS[req.tenant.plan].storage_quota_bytes;
  if (limit === Infinity) return next();

  const used = await getTenantStorageUsed(req.tenant.id);
  const fileSize = req.file?.size || 0;

  if (used + fileSize > limit) {
    return res.status(413).json({
      error: 'Storage quota exceeded',
      usedGB: (used / (1024 ** 3)).toFixed(2),
      limitGB: (limit / (1024 ** 3)).toFixed(2),
      plan: req.tenant.plan,
    });
  }
  next();
};

// Upload route with quota enforcement:
router.post(
  '/image',
  authMiddleware,
  tenantMiddleware,
  uploadImage,                   // multer — parse multipart
  enforceImageSizeLimit,         // check file size against plan limit
  checkStorageQuota,             // check total storage against plan quota
  async (req, res, next) => {
    // Use tenant-scoped S3 key
    const folder = `tenants/${req.tenant.id}/posts`;
    const { key, url } = await uploadImageToS3({
      buffer: req.file.buffer,
      mimetype: req.file.mimetype,
      folder,
    });

    // Record the upload for quota tracking
    await db.query(
      `INSERT INTO s3_uploads (tenant_id, uploaded_by, key, file_size_bytes)
       VALUES ($1, $2, $3, $4)`,
      [req.tenant.id, req.user.sub, key, req.file.size]
    );

    res.json({ key, url });
  }
);
```

#### Day 39: Stripe Integration

```javascript
// billing-service/src/services/stripe.service.js
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export const createCheckoutSession = async ({ tenantId, plan, returnUrl }) => {
  const tenant = await getTenant(tenantId);
  const priceId = PLAN_PRICE_IDS[plan];

  const session = await stripe.checkout.sessions.create({
    customer: tenant.stripe_customer_id,
    mode: 'subscription',
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${returnUrl}?success=true&session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${returnUrl}?cancelled=true`,
    metadata: { tenantId },
  });

  return session;
};
```

#### Days 40–45: Webhooks, Billing Service, Invoices, Final CI/CD

**Stripe webhook handler — keep your DB in sync with billing state:**

```javascript
router.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),  // CRITICAL: raw body for signature verification
  async (req, res) => {
    let event;
    try {
      event = stripe.webhooks.constructEvent(
        req.body,
        req.headers['stripe-signature'],
        process.env.STRIPE_WEBHOOK_SECRET
      );
    } catch (err) {
      logger.error('Stripe webhook signature verification failed', { err: err.message });
      return res.status(400).send(`Webhook Error: ${err.message}`);
    }

    switch (event.type) {
      case 'checkout.session.completed':
        const { tenantId } = event.data.object.metadata;
        await upgradeTenantPlan(tenantId, event.data.object.subscription);
        await clearTenantCache(tenantId);
        break;

      case 'invoice.payment_succeeded':
        await recordSuccessfulPayment(event.data.object);
        break;

      case 'invoice.payment_failed':
        await handleFailedPayment(event.data.object);
        // Email the tenant admin to update payment method
        await emailQueue.add('payment-failed', { customerId: event.data.object.customer });
        break;

      case 'customer.subscription.deleted':
        // Subscription cancelled — downgrade to free
        await downgradeToFree(event.data.object.customer);
        // Delete S3 files exceeding free tier quota (or keep but block new uploads)
        await enforceFreeTierStorageLimit(event.data.object.customer);
        break;
    }

    res.json({ received: true });
  }
);
```

**Monthly invoice cron job (BullMQ repeatable — survives restarts):**

```javascript
// billing-service/src/jobs/invoice.job.js
import { Queue, Worker } from 'bullmq';

const invoiceQueue = new Queue('invoices', { connection });

// Register repeatable job on service startup
await invoiceQueue.add('monthly-invoices', {}, {
  repeat: { pattern: '0 0 1 * *' },  // 1st of every month at midnight
  jobId: 'monthly-invoices',  // stable ID prevents duplicate registrations on restart
});

const worker = new Worker('invoices', async (job) => {
  const { rows: tenants } = await db.query(
    `SELECT t.*, u.email FROM tenants t
     JOIN tenant_members tm ON tm.tenant_id = t.id AND tm.role = 'owner'
     JOIN users u ON u.id = tm.user_id
     WHERE t.plan != 'free'`
  );

  for (const tenant of tenants) {
    const storageUsed = await getTenantStorageUsed(tenant.id);
    const postsThisMonth = await getMonthlyPostCount(tenant.id);

    await emailQueue.add('monthly-invoice', {
      tenantId: tenant.id,
      ownerEmail: tenant.email,
      plan: tenant.plan,
      usageSummary: { storageUsedGB: storageUsed / (1024 ** 3), postsThisMonth },
    });
  }
}, { connection });
```

**S3 cleanup on tenant downgrade to free:**

```javascript
const enforceFreeTierStorageLimit = async (stripeCustomerId) => {
  const tenant = await getTenantByStripeId(stripeCustomerId);
  const FREE_LIMIT = 500 * 1024 * 1024; // 500MB

  const { rows: uploads } = await db.query(
    `SELECT key, file_size_bytes FROM s3_uploads
     WHERE tenant_id = $1
     ORDER BY created_at DESC`,
    [tenant.id]
  );

  let totalSize = 0;
  const toDelete = [];

  for (const upload of uploads) {
    totalSize += upload.file_size_bytes;
    if (totalSize > FREE_LIMIT) {
      toDelete.push(upload.key);
    }
  }

  // Enqueue deletion (not synchronous — could be thousands of files)
  if (toDelete.length) {
    await imageQueue.add('bulk-s3-delete', { keys: toDelete, tenantId: tenant.id });
  }

  logger.info('Free tier enforcement queued', {
    tenantId: tenant.id,
    filesToDelete: toDelete.length,
  });
};
```

---

## 5. CI/CD with GitHub Actions

### Phase 1 — Monolith CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: ['*']
  pull_request:
    branches: [main]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: testpass, POSTGRES_DB: devfeed_test }
        options: >- --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
        ports: [5432:5432]
      redis:
        image: redis:7
        options: >- --health-cmd "redis-cli ping" --health-interval 10s --health-timeout 5s --health-retries 5
        ports: [6379:6379]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: node src/db/migrate.js
        env:
          DATABASE_URL: postgres://postgres:testpass@localhost:5432/devfeed_test
      - run: npm test
        env:
          NODE_ENV: test
          DATABASE_URL: postgres://postgres:testpass@localhost:5432/devfeed_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret
          AWS_REGION: us-east-1
          AWS_ACCESS_KEY_ID: test
          AWS_SECRET_ACCESS_KEY: test
          S3_BUCKET_NAME: test-bucket
          S3_BASE_URL: https://test-bucket.s3.amazonaws.com
```

### Phase 1 — Monolith CD

```yaml
# .github/workflows/cd.yml
name: CD
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          target: production
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /opt/devfeed
            echo "IMAGE=ghcr.io/${{ github.repository }}:${{ github.sha }}" >> .env
            docker compose pull app
            docker compose up -d --no-deps --force-recreate app
            docker image prune -f
```

### Phase 3 — Staging + Production with Approval

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: type=sha
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.devfeed.io
    steps:
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /opt/devfeed-staging
            docker compose pull
            docker compose up -d --force-recreate

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production     # GitHub will require manual approval before this runs
      url: https://devfeed.io
    steps:
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /opt/devfeed
            docker compose pull
            docker compose up -d --force-recreate
```

---

## 6. Database Schemas

### Phase 1 — Full Schema

```sql
-- 001_create_users.sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username        VARCHAR(50)  UNIQUE NOT NULL,
  email           VARCHAR(255) UNIQUE NOT NULL,
  password_hash   TEXT NOT NULL,
  bio             TEXT,
  avatar_url      TEXT,          -- S3 URL to profile picture
  avatar_key      TEXT,          -- S3 key for deletion
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  last_active_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_email    ON users(email);
CREATE INDEX idx_users_username ON users(username);


-- 002_create_posts.sql
CREATE TABLE posts (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  author_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  content       TEXT NOT NULL CHECK (char_length(content) BETWEEN 1 AND 5000),
  language      VARCHAR(50),            -- programming language tag
  tags          TEXT[] DEFAULT '{}',
  likes_count   INT DEFAULT 0,
  image_url     TEXT,                   -- S3 URL of attached image
  image_key     TEXT,                   -- S3 key for deletion
  thumbnail_url TEXT,                   -- S3 URL of compressed thumbnail
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_posts_author  ON posts(author_id);
CREATE INDEX idx_posts_created ON posts(created_at DESC);

CREATE TABLE likes (
  post_id    UUID REFERENCES posts(id) ON DELETE CASCADE,
  user_id    UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (post_id, user_id)
);


-- 003_create_follows.sql
CREATE TABLE follows (
  follower_id  UUID REFERENCES users(id) ON DELETE CASCADE,
  following_id UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at   TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (follower_id, following_id),
  CHECK (follower_id != following_id)
);


-- 004_create_notifications.sql
CREATE TYPE notification_type AS ENUM ('post_liked', 'new_follower', 'post_mentioned');

CREATE TABLE notifications (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipient_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  sender_id    UUID REFERENCES users(id) ON DELETE SET NULL,
  type         notification_type NOT NULL,
  payload      JSONB NOT NULL DEFAULT '{}',
  read         BOOLEAN DEFAULT FALSE,
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notifications_recipient ON notifications(recipient_id, read);
CREATE INDEX idx_notifications_created   ON notifications(created_at DESC);


-- 005_create_refresh_tokens.sql
CREATE TABLE refresh_tokens (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token_hash  TEXT NOT NULL UNIQUE,
  expires_at  TIMESTAMPTZ NOT NULL,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_refresh_tokens_user    ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_expires ON refresh_tokens(expires_at);


-- 006_create_s3_uploads.sql
-- Tracks every file uploaded to S3 for quota enforcement and orphan cleanup
CREATE TABLE s3_uploads (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  uploaded_by     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  key             TEXT NOT NULL UNIQUE,   -- S3 object key
  file_size_bytes BIGINT NOT NULL,
  post_id         UUID REFERENCES posts(id) ON DELETE SET NULL,  -- NULL = orphaned
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_s3_uploads_user    ON s3_uploads(uploaded_by);
CREATE INDEX idx_s3_uploads_post    ON s3_uploads(post_id);
CREATE INDEX idx_s3_uploads_orphan  ON s3_uploads(created_at) WHERE post_id IS NULL;
```

### Phase 3 — SaaS Schema Additions

```sql
-- 007_create_tenants.sql
CREATE TABLE tenants (
  id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name                    VARCHAR(255) NOT NULL,
  slug                    VARCHAR(100) UNIQUE NOT NULL,
  plan                    VARCHAR(50) DEFAULT 'free',
  stripe_customer_id      VARCHAR(255) UNIQUE,
  stripe_subscription_id  VARCHAR(255) UNIQUE,
  custom_s3_bucket        TEXT,          -- Enterprise: their own S3 bucket name
  custom_s3_region        TEXT,          -- Enterprise: their own S3 region
  trial_ends_at           TIMESTAMPTZ,
  created_at              TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE tenant_members (
  tenant_id  UUID REFERENCES tenants(id) ON DELETE CASCADE,
  user_id    UUID REFERENCES users(id)   ON DELETE CASCADE,
  role       VARCHAR(50) DEFAULT 'member',
  joined_at  TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (tenant_id, user_id)
);

CREATE TABLE plans (
  id                    VARCHAR(50) PRIMARY KEY,
  name                  VARCHAR(100) NOT NULL,
  stripe_price_id       VARCHAR(255),
  posts_per_day         INT,           -- NULL = unlimited
  max_members           INT,           -- NULL = unlimited
  image_max_bytes       BIGINT,        -- per-file upload limit
  storage_quota_bytes   BIGINT,        -- total S3 quota, NULL = unlimited
  features              TEXT[] DEFAULT '{}',
  price_monthly_cents   INT DEFAULT 0
);

INSERT INTO plans VALUES
  ('free',       'Free',       NULL,            5,    1,  2097152,       524288000,    '{}',                                        0),
  ('pro',        'Pro',        'price_pro_id',  NULL, 10, 20971520,      10737418240,  '{analytics,api_access}',                    2900),
  ('enterprise', 'Enterprise', 'price_ent_id',  NULL, NULL, 104857600,   NULL,         '{analytics,api_access,custom_s3_bucket}',   9900);

-- Add tenant_id to all data tables
ALTER TABLE posts         ADD COLUMN tenant_id UUID REFERENCES tenants(id);
ALTER TABLE notifications ADD COLUMN tenant_id UUID REFERENCES tenants(id);
ALTER TABLE s3_uploads    ADD COLUMN tenant_id UUID REFERENCES tenants(id);

CREATE INDEX idx_posts_tenant      ON posts(tenant_id);
CREATE INDEX idx_s3_uploads_tenant ON s3_uploads(tenant_id);

-- 008_create_usage_events.sql
CREATE TABLE usage_events (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID NOT NULL REFERENCES tenants(id),
  event_type  VARCHAR(100) NOT NULL,   -- 'post.created', 'image.uploaded', 'api.call'
  metadata    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_usage_tenant_type ON usage_events(tenant_id, event_type, created_at);
```

---

## 7. Folder Structures

### Phase 1 — Monolith

```
devfeed/
├── src/
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── post.routes.js
│   │   ├── user.routes.js
│   │   ├── upload.routes.js        ← S3 upload endpoints
│   │   ├── notification.routes.js
│   │   └── admin.routes.js         ← Bull Board + health check
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── post.controller.js
│   │   ├── user.controller.js
│   │   └── notification.controller.js
│   ├── middleware/
│   │   ├── auth.middleware.js       ← JWT verification
│   │   ├── cache.middleware.js      ← Redis caching
│   │   ├── upload.middleware.js     ← multer configuration
│   │   ├── validate.middleware.js   ← Zod validation
│   │   ├── rateLimit.middleware.js
│   │   └── error.middleware.js
│   ├── services/
│   │   ├── redis.service.js         ← Redis connection + cache helpers
│   │   ├── s3.service.js            ← AWS S3 operations
│   │   ├── queue.service.js         ← BullMQ queue definitions
│   │   ├── socket.service.js        ← Socket.io setup + rooms
│   │   └── email.service.js         ← nodemailer
│   ├── workers/
│   │   ├── email.worker.js          ← BullMQ email job consumer
│   │   └── image.worker.js          ← BullMQ image processing consumer
│   ├── jobs/
│   │   └── digest.cron.js           ← node-cron scheduled jobs
│   ├── db/
│   │   ├── index.js                 ← pg Pool instance
│   │   ├── migrate.js               ← runs migration files in order
│   │   ├── seed.js                  ← development seed data
│   │   └── migrations/
│   │       ├── 001_create_users.sql
│   │       ├── 002_create_posts.sql
│   │       ├── 003_create_follows.sql
│   │       ├── 004_create_notifications.sql
│   │       ├── 005_create_refresh_tokens.sql
│   │       └── 006_create_s3_uploads.sql
│   ├── schemas/
│   │   ├── auth.schema.js
│   │   ├── post.schema.js
│   │   └── upload.schema.js
│   ├── utils/
│   │   ├── logger.js                ← Winston instance
│   │   └── jwt.js                   ← sign/verify helpers
│   └── app.js                       ← Express setup + middleware chain
├── tests/
│   ├── setup.js                     ← test DB setup, S3 mock
│   ├── auth.test.js
│   ├── posts.test.js
│   ├── upload.test.js               ← S3 upload tests
│   └── cache.test.js
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── package.json
```

### Phase 2 — Microservices

```
devfeed-microservices/
├── gateway/
│   ├── src/
│   │   ├── app.js
│   │   └── middleware/
│   │       ├── auth.middleware.js
│   │       └── rateLimit.middleware.js
│   ├── Dockerfile
│   └── package.json
├── auth-service/
│   ├── src/
│   │   ├── app.js
│   │   ├── routes/auth.routes.js
│   │   ├── routes/user.routes.js
│   │   ├── controllers/
│   │   ├── services/
│   │   │   ├── s3.service.js        ← avatar uploads
│   │   │   └── token.service.js
│   │   └── db/
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
├── post-service/
│   ├── src/
│   │   ├── app.js
│   │   ├── routes/
│   │   │   ├── post.routes.js
│   │   │   └── upload.routes.js     ← post image uploads
│   │   ├── controllers/
│   │   ├── services/
│   │   │   └── s3.service.js        ← post image S3 operations
│   │   ├── events/
│   │   │   └── publisher.js         ← Redis Stream publisher
│   │   └── db/
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
├── notification-service/
│   ├── src/
│   │   ├── app.js
│   │   ├── socket/socket.service.js
│   │   ├── consumers/event.consumer.js
│   │   ├── workers/
│   │   │   ├── email.worker.js
│   │   │   └── image.worker.js      ← thumbnail generation
│   │   └── db/
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
├── shared/
│   ├── jwt.js
│   ├── events.js                    ← event type string constants
│   ├── s3.service.js                ← shared S3 utilities
│   ├── logger.js
│   └── package.json
├── docker-compose.yml
└── .github/workflows/
    ├── auth-service.yml
    ├── post-service.yml
    ├── notification-service.yml
    └── gateway.yml
```

### Phase 3 — SaaS (additions)

```
devfeed-saas/
├── billing-service/
│   ├── src/
│   │   ├── app.js
│   │   ├── routes/
│   │   │   ├── billing.routes.js    ← checkout, plan info
│   │   │   ├── webhook.routes.js    ← Stripe webhooks
│   │   │   └── invoice.routes.js    ← invoice list, usage
│   │   ├── services/
│   │   │   └── stripe.service.js
│   │   └── jobs/
│   │       └── invoice.job.js       ← BullMQ repeatable invoice job
│   ├── Dockerfile
│   └── package.json
├── post-service/ (updated)
│   └── src/
│       └── middleware/
│           ├── tenant.middleware.js
│           ├── quota.middleware.js
│           ├── plan.middleware.js
│           └── s3quota.middleware.js  ← S3 storage quota check
├── shared/ (updated)
│   └── plans.js                      ← plan limits and feature definitions
```

---

## 8. Environment Variables Reference

|Variable|Used in|Description|
|---|---|---|
|`NODE_ENV`|All|`development`, `test`, or `production`|
|`PORT`|All|HTTP port (default 3000)|
|`DATABASE_URL`|All|PostgreSQL connection string|
|`REDIS_URL`|All|Redis connection string|
|`JWT_SECRET`|Gateway, Auth, Post|Secret for signing JWTs — must be identical across services|
|`JWT_EXPIRES_IN`|Auth|Access token TTL (e.g. `15m`)|
|`REFRESH_TOKEN_EXPIRES_IN`|Auth|Refresh token TTL (e.g. `7d`)|
|`INTERNAL_SECRET`|Gateway, all services|Shared secret for service-to-service trust|
|`AWS_REGION`|Auth, Post|AWS region of S3 bucket|
|`AWS_ACCESS_KEY_ID`|Auth, Post|IAM user access key|
|`AWS_SECRET_ACCESS_KEY`|Auth, Post|IAM user secret key|
|`S3_BUCKET_NAME`|Auth, Post|S3 bucket name|
|`S3_BASE_URL`|Auth, Post|Public base URL: `https://bucket.s3.region.amazonaws.com`|
|`SMTP_HOST`|Notification|SMTP server host|
|`SMTP_PORT`|Notification|SMTP server port|
|`SMTP_USER`|Notification|SMTP username|
|`SMTP_PASS`|Notification|SMTP password|
|`EMAIL_FROM`|Notification|From address on all emails|
|`STRIPE_SECRET_KEY`|Billing|Stripe secret API key|
|`STRIPE_WEBHOOK_SECRET`|Billing|Stripe webhook signing secret|
|`CLIENT_URL`|All|Frontend URL for CORS|

---

## 9. Daily Schedule Template

Use this template every day to keep learning momentum consistent.

```
Morning Block (2 hours) — Understand
  [ ] Read today's concept section in this document
  [ ] Look up the official docs for today's library
  [ ] Sketch the data flow on paper before writing code

Afternoon Block (3 hours) — Build
  [ ] Implement the feature step by step
  [ ] Write at least one test that proves it works
  [ ] Commit with a meaningful message:
      "feat: add S3 image upload to POST /posts"
      "fix: cache invalidation not firing on post delete"

Evening Block (1 hour) — Consolidate
  [ ] Push to GitHub, verify CI passes
  [ ] Write 3–5 sentences in your learning log
  [ ] Read tomorrow's concept section so it is already in your head overnight
```

### Weekly Learning Log Template

Keep one file per week: `logs/week-1.md`, `logs/week-2.md`, etc.

````markdown
## Week N — Day X — YYYY-MM-DD

### What I built today


### The concept that clicked today


### What confused me


### How I eventually figured it out


### Code / command I want to remember
```javascript


````

### One thing I would do differently

### Goal for tomorrow

```

---

## Technology Stack Summary

| Layer | Technology | Why this one |
|---|---|---|
| Runtime | Node.js 20 (LTS) | Non-blocking I/O ideal for I/O-bound work; huge npm ecosystem |
| Framework | Express.js | Minimal, unopinionated, industry standard for learning APIs |
| Database | PostgreSQL 16 | ACID compliant, excellent JSON support, industry standard |
| Cache + Pub/Sub | Redis 7 | Sub-millisecond reads from RAM; built-in pub/sub; powers BullMQ |
| Queue | BullMQ | TypeScript-first; built on Redis; retries, dashboards, repeatable jobs |
| File Storage | AWS S3 | Unlimited, durable, cheap, CDN-ready; industry standard |
| Image Processing | sharp | Fastest Node.js image library; resizes, converts, compresses |
| File Upload Parsing | multer | De facto standard Express middleware for multipart forms |
| Real-time | Socket.io | Handles reconnection and fallbacks; room-based targeting |
| Scheduler | node-cron → BullMQ repeatable | Learn simple, then graduate to production-safe |
| Containerisation | Docker + Docker Compose | Consistent environments across dev, CI, and production |
| CI/CD | GitHub Actions | Free, tightly integrated with GitHub, excellent ecosystem |
| Billing | Stripe | Industry standard; excellent test mode; robust webhook system |
| Validation | Zod | Schema-first; TypeScript-native; composable; great error messages |
| Logging | Winston | Structured JSON logging; multiple transports; log levels |
| Security | Helmet + express-rate-limit | Security headers and rate limiting with minimal configuration |

---

## Five Principles to Remember

**1. Build → Break → Debug → Understand → Move On**  
Do not read documentation for days before coding. Build something, watch it fail, understand why it failed, fix it, move on. The fastest learning comes from real errors, not theoretical understanding.

**2. Commit after every working feature**  
Every commit is a save point. If you break something, you can always revert. Write descriptive commit messages — they become your learning log.

**3. S3 errors are usually permissions errors**  
If an S3 call fails with `AccessDenied`, check the IAM policy. If it fails with `NoSuchBucket`, check the bucket name and region. Read the full AWS error message — it always tells you what is wrong.

**4. Draw the system before you code it**  
For every new feature or service, draw the data flow on paper first. Where does the request come from? What reads and writes happen? Where could it fail? Drawing reveals design problems that code hides.

**5. The monolith is not a mistake — it is the prerequisite**  
Microservices only make sense after you have deeply understood how features connect and depend on each other. Your Phase 1 monolith is not throwaway code — it is the knowledge you need to split services correctly in Phase 2.

---

*45 days. One commit per day. Every concept explained before implemented. Go build.*
```