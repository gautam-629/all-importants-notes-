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

#### 🟢 3️⃣ Database (SQL)
1. Design a **User + Role + Permission Schema** → [[E-Commerce API Database Design & Full-Stack Scenarios]]
2. Difference between `WHERE` and `HAVING` → [[WHERE and HAVING SQL]]
3. What is **Indexing? When does it hurt performance?** → [[Query Optimize]]
4. What is **N+1 Problem?** → [[N+1 problem]]
5. What is **Transaction? When should you use it?** → [[Transaction]]
6. What is **Soft Delete and how to implement?** → [[soft delete]]
7. 🗑 **Soft Delete – Notes** → [[soft delete]]
8. Write SQL for **Paginated Filtered API Endpoint** → [[Integrate pagination In Sql]]
## 🟢 4️⃣ Redis (Very Important for Mid-Level)
  .[[Redis Complete Guide]]
  [Redis example ](git@github.com:gautam-629/Redis-Demo.git)
- [ ] **How to prevent cache stampede?**
- [ ] **Difference between Redis and DB indexing.**
- [ ] **What is Redis Pub/Sub?**
- [ ] **How do you scale Redis?**
- [ ] **When should you NOT use Redis?**

## 🟢 5️⃣ Queue System (Bull / BullMQ)
1.[[Queue System]]
2.[Queue Example](git@github.com:gautam-629/Redis-Demo.git)
## 🟢 6️⃣ Socket.IO (Real-Time)
 [web chat demo](https://github.com/gautam-629/web-chat-individual_demo)
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
[[Security & Production]]
- [x] **How to protect against SQL injection?**
- [x] **How to prevent XSS?**
- [ ] **How to protect against brute-force login?**
- [x] **How to implement rate limiter?**
- [ ] **How to secure environment variables?**
- [ ] **How to store password securely?**
- [x] **bcrypt vs argon2?**
- [x] **What is CORS?**
- [x] **What is horizontal vs vertical scaling?**

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
