### 1.What is the Repository Pattern and when should you use it in Node.js?
Separating database logic from business logic for maintainability.
Example: UserRepository for all DB operations related to users.

### 2.How do you handle environment-specific configuration in Node.js (e.g., dev, staging, prod)?
  Use case: Using .env files and configuration files with dotenv and config packages.
``` javascript
import dotenv from "dotenv";
import path from "path";

const envFile = `.env.${process.env.NODE_ENV || "development"}`;
dotenv.config({ path: path.resolve(process.cwd(), envFile) });

console.log("Environment:", process.env.NODE_ENV);
console.log("DB Host:", process.env.DB_HOST);

//nest js
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // makes config available everywhere
      envFilePath: [`.env.${process.env.NODE_ENV || 'development'}`], // pick file dynamically
    }),
  ],
})
export class AppModule {}
```
### 3.Why use layered architecture (Controller → Service → Repository) in Node.js apps?
Use case: Enhances maintainability and testability.
Example: Add caching in service layer without changing controller or DB.

### 4.How does JWT authentication work, and what are the security caveats?
Tokens stored in `localStorage` can be stolen via JavaScript; using `HttpOnly` cookies is safer because JS cannot access them and they only travel over HTTPS.  
Cookie-based authentication works best when the frontend and backend share the same domain.

### 5.How do you implement a secure refresh token flow with JWTs?
 # 🔑 Authentication with Access & Refresh Tokens

### Process of Generating Access & Refresh Tokens
1. **Login** → User logs in with valid credentials.  
2. **Token Issue** → Server generates:
   - **Access Token** (short-lived, e.g., 15 min)  
   - **Refresh Token** (long-lived, stored in DB, e.g., 7 days)  
3. **Response** → Both tokens sent to client.  
   - Access Token → used for API requests.  
   - Refresh Token → kept safe (httpOnly cookie or secure storage).  

---

### 📌 Importance of Refresh Token
- **Access Token** is short-lived → reduces risk if stolen.  
- **Refresh Token** allows issuing a **new Access Token** without asking the user to log in again.  
- Server validates refresh token against DB → only one valid refresh token per user session.  

---

## #🚪 Logout Steps
1. Client sends **logout request**.  
2. Server **deletes refresh token** from DB.  
3. Server responds with:  
   ```json
   { "accessToken": null }
```

