
### A Complete Guide for Node.js + React + SQL Developers
---
## Table of Contents

1. [What is Docker & Why Should You Care?](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#1-what-is-docker--why-should-you-care)
2. [Core Concepts](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#2-core-concepts)
3. [Installation](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#3-installation)
4. [Your First Docker Container](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#4-your-first-docker-container)
5. [Dockerfile — Packaging Your App](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#5-dockerfile--packaging-your-app)
6. [Dockerizing a Node.js Backend](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#6-dockerizing-a-nodejs-backend)
7. [Dockerizing a React Frontend](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#7-dockerizing-a-react-frontend)
8. [Dockerizing a Relational Database (PostgreSQL)](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#8-dockerizing-a-relational-database-postgresql)
9. [Docker Compose — Running Everything Together](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#9-docker-compose--running-everything-together)
10. [Volumes — Persisting Data](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#10-volumes--persisting-data)
11. [Networking in Docker](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#11-networking-in-docker)
12. [Environment Variables & .env Files](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#12-environment-variables--env-files)
13. [Multi-Stage Builds — Optimized Production Images](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#13-multi-stage-builds--optimized-production-images)
14. [Essential Docker Commands Cheat Sheet](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#14-essential-docker-commands-cheat-sheet)
15. [Docker in CI/CD (GitHub Actions)](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#15-docker-in-cicd-github-actions)
16. [Best Practices](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#16-best-practices)
17. [Common Mistakes & How to Fix Them](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#17-common-mistakes--how-to-fix-them)
18. [Docker Hub — Push, Pull & Share Your Images](https://claude.ai/chat/f97d88b0-7149-424b-abbf-7883e36db5b4#18-docker-hub--push-pull--share-your-images)

---

## 1. What is Docker & Why Should You Care?

### The Classic Problem

You build a Node.js app on your Mac. It works perfectly.  
Your colleague pulls the code on Windows. It breaks.  
You deploy to Linux. It breaks again.

> **"It works on my machine"** — every developer, ever.

### What Docker Does

Docker packages your app **along with everything it needs** (Node version, OS libraries, config) into a single portable unit called a **container**. That container runs identically on any machine.

```
┌──────────────────────────────────────────┐
│              Your Machine                │
│  ┌────────────┐  ┌────────────┐          │
│  │ Container 1│  │ Container 2│          │
│  │  Node App  │  │ PostgreSQL │          │
│  │  Node 20   │  │  PG 16     │          │
│  └────────────┘  └────────────┘          │
│         Docker Engine                    │
│         Linux Kernel                     │
└──────────────────────────────────────────┘
```

### Docker vs Virtual Machines

|Feature|Virtual Machine|Docker Container|
|---|---|---|
|Boot time|Minutes|Seconds|
|Size|GBs|MBs|
|OS included|Full OS|Shares host OS|
|Isolation|Full|Process-level|
|Use case|Full environments|App packaging|

---

## 2. Core Concepts

Before diving in, understand these 5 terms:

### 🖼️ Image

A **read-only blueprint** for creating containers. Think of it like a class in OOP.  
Built from a `Dockerfile`. Can be shared on Docker Hub.

### 📦 Container

A **running instance** of an image. Think of it like an object (instance of a class).  
You can run many containers from the same image.

### 📄 Dockerfile

A text file with **step-by-step instructions** to build a Docker image.

### 🗂️ Volume

A way to **persist data** outside a container. Essential for databases.

### 🌐 Network

Allows containers to **talk to each other** by name.

### 🔧 Docker Compose

A tool to define and run **multi-container apps** using a single YAML file.

```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
```

---

## 3. Installation

### Install Docker Desktop

Download from: https://www.docker.com/products/docker-desktop/

Available for:

- ✅ macOS (Intel & Apple Silicon)
- ✅ Windows (with WSL2)
- ✅ Linux

### Verify Installation

```bash
docker --version
# Docker version 25.x.x

docker compose version
# Docker Compose version v2.x.x

docker run hello-world
# Should print: Hello from Docker!
```

---

## 4. Your First Docker Container

### Run a container from an existing image

```bash
# Pull and run an nginx web server
docker run -p 8080:80 nginx
```

Now open http://localhost:8080 — you see the nginx welcome page!

**What happened?**

- `docker run` — create and start a container
- `-p 8080:80` — map port 8080 on your machine → port 80 inside container
- `nginx` — use the official nginx image from Docker Hub

### Useful flags

```bash
# Run in background (detached)
docker run -d -p 8080:80 nginx

# Give the container a name
docker run -d -p 8080:80 --name my-nginx nginx

# Run interactively (get a terminal inside)
docker run -it ubuntu bash
```

### Managing containers

```bash
# List running containers
docker ps

# List ALL containers (including stopped)
docker ps -a

# Stop a container
docker stop my-nginx

# Start a stopped container
docker start my-nginx

# Remove a container
docker rm my-nginx

# Remove a running container (force)
docker rm -f my-nginx

# View container logs
docker logs my-nginx

# Follow logs in real-time
docker logs -f my-nginx

# Execute a command inside a running container
docker exec -it my-nginx bash
```

---

## 5. Dockerfile — Packaging Your App

A `Dockerfile` is how you create your own custom image.

### Dockerfile Instructions

|Instruction|Purpose|
|---|---|
|`FROM`|Base image to start from|
|`WORKDIR`|Set working directory inside container|
|`COPY`|Copy files from host to container|
|`RUN`|Execute a command during build|
|`ENV`|Set environment variables|
|`EXPOSE`|Document which port the app uses|
|`CMD`|Default command when container starts|
|`ENTRYPOINT`|Like CMD but harder to override|

### Build an image from a Dockerfile

```bash
# In the directory containing your Dockerfile
docker build -t my-app:v1 .

# -t = tag (name:version)
# .  = build context (current directory)
```

---

## 6. Dockerizing a Node.js Backend

### Project structure

```
backend/
├── src/
│   └── index.js
├── package.json
├── package-lock.json
└── Dockerfile
```

### `src/index.js`

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'ok', message: 'API is running!' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### `Dockerfile` for Node.js

```dockerfile
# Step 1: Choose base image
# Use the official Node.js image (Alpine = lightweight Linux)
FROM node:20-alpine

# Step 2: Set working directory inside the container
WORKDIR /app

# Step 3: Copy package files FIRST (for better caching)
COPY package*.json ./

# Step 4: Install dependencies
RUN npm install

# Step 5: Copy the rest of your source code
COPY . .

# Step 6: Document the port your app listens on
EXPOSE 3000

# Step 7: Command to run when container starts
CMD ["node", "src/index.js"]
```

> ⚠️ **Why copy `package.json` before `COPY . .`?**  
> Docker caches each layer. If you copy package.json first and it hasn't changed, Docker reuses the cached `npm install` layer — making builds **much faster**.

### `.dockerignore` — Like `.gitignore` for Docker

```
node_modules
.git
.env
*.log
dist
build
```

Always create this! It prevents copying `node_modules` (hundreds of MBs) into the image.

### Build and run

```bash
cd backend

# Build the image
docker build -t my-backend:v1 .

# Run a container
docker run -d \
  -p 3000:3000 \
  --name backend \
  my-backend:v1

# Test it
curl http://localhost:3000/health
```

---

## 7. Dockerizing a React Frontend

### `Dockerfile` for React (Development)

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host"]
```

> `--host` flag is needed so Vite (or CRA) listens on `0.0.0.0` instead of just `localhost` inside the container.

### `Dockerfile` for React (Production with Nginx)

```dockerfile
# Stage 1: Build the React app
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve with nginx
FROM nginx:alpine

# Copy built files to nginx's public directory
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy custom nginx config (optional)
# COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 8. Dockerizing a Relational Database (PostgreSQL)

The good news: **you don't need a Dockerfile for PostgreSQL**. The official image is ready to use.

```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:16-alpine
```

### Connect to it

```bash
# Open psql inside the container
docker exec -it postgres-db psql -U myuser -d myapp

# Or connect from your local machine (if you have psql installed)
psql -h localhost -p 5432 -U myuser -d myapp
```

### For MySQL

```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=myapp \
  -e MYSQL_USER=myuser \
  -e MYSQL_PASSWORD=mypassword \
  -p 3306:3306 \
  -v mysql-data:/var/lib/mysql \
  mysql:8
```

---

## 9. Docker Compose — Running Everything Together

Running multiple `docker run` commands manually is messy. Docker Compose lets you define your entire stack in one file.

### Full Stack Example: Node + React + PostgreSQL

```
project/
├── backend/
│   ├── src/index.js
│   ├── package.json
│   └── Dockerfile
├── frontend/
│   ├── src/
│   ├── package.json
│   └── Dockerfile
├── .env
└── docker-compose.yml
```

### `docker-compose.yml`

```yaml
version: '3.9'

services:

  # ─── PostgreSQL Database ───────────────────────────────
  db:
    image: postgres:16-alpine
    container_name: myapp-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - myapp-network

  # ─── Node.js Backend ───────────────────────────────────
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: myapp-backend
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      PORT: 3000
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      NODE_ENV: development
    volumes:
      - ./backend:/app          # Mount source for hot reload
      - /app/node_modules       # Keep container's node_modules
    depends_on:
      - db
    networks:
      - myapp-network

  # ─── React Frontend ────────────────────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: myapp-frontend
    restart: unless-stopped
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app         # Mount source for hot reload
      - /app/node_modules       # Keep container's node_modules
    depends_on:
      - backend
    networks:
      - myapp-network

# ─── Named Volumes ──────────────────────────────────────
volumes:
  postgres-data:

# ─── Networks ───────────────────────────────────────────
networks:
  myapp-network:
    driver: bridge
```

> 🔑 **Key insight**: Inside the Docker network, containers talk to each other by **service name**. So your backend connects to the database using host `db` (not `localhost`). `DATABASE_URL: postgresql://user:pass@db:5432/myapp`

### Docker Compose Commands

```bash
# Start all services (build if needed)
docker compose up

# Start in background
docker compose up -d

# Force rebuild images
docker compose up --build

# Stop all services
docker compose down

# Stop and remove volumes (⚠️ deletes DB data!)
docker compose down -v

# View logs
docker compose logs

# View logs for specific service
docker compose logs backend

# Follow logs in real-time
docker compose logs -f backend

# Run a command in a service
docker compose exec backend sh
docker compose exec db psql -U myuser -d myapp

# List running services
docker compose ps

# Restart a single service
docker compose restart backend

# Scale a service (run 3 backend instances)
docker compose up --scale backend=3
```

---

## 10. Volumes — Persisting Data

Containers are **ephemeral** — when you delete a container, all data inside is gone.  
Volumes solve this problem.
### Types of Storage
```
┌─────────────────────────────────────────────────────┐
│  1. Named Volume  (managed by Docker)               │
│     docker run -v postgres-data:/var/lib/pg/data    │
│     Best for: databases, persistent app data        │
├─────────────────────────────────────────────────────┤
│  2. Bind Mount  (maps your host directory)          │
│     docker run -v ./src:/app/src                    │
│     Best for: development hot-reload                │
├─────────────────────────────────────────────────────┤
│  3. tmpfs  (memory only, not persisted)             │
│     Best for: secrets, temp files                   │
└─────────────────────────────────────────────────────┘
```
### Managing Volumes
```bash
# List all volumes
docker volume ls

# Inspect a volume (see where data is stored)
docker volume inspect postgres-data

# Remove a volume
docker volume rm postgres-data

# Remove ALL unused volumes (dangerous!)
docker volume prune
```
### Bind Mounts for Hot Reload in Development
```yaml
# docker-compose.yml — backend service
volumes:
  - ./backend:/app        # Your code → container (changes reflect immediately)
  - /app/node_modules     # This line prevents host node_modules from overwriting container's
```

Why the second line? Without it, your Mac's `node_modules` (built for macOS) would overwrite the container's `node_modules` (built for Linux) — causing crashes.

---
## 11. Networking in Docker
### Default Behavior

- Each Docker Compose project gets its own isolated network
- Containers within the same Compose file can reach each other by **service name**
- Containers on different networks cannot talk to each other

```
docker-compose.yml network: "myapp-network"

  [frontend] ──→ http://backend:3000/api  ✅
  [backend]  ──→ postgresql://db:5432     ✅
  [frontend] ──→ postgresql://db:5432     ✅ (but shouldn't in practice)
```
### From your host machine

- You access containers via `localhost` + the published port
- `localhost:3000` → backend container
- `localhost:5432` → postgres container
### Network Commands
```bash
# List networks
docker network ls

# Inspect a network
docker network inspect myapp-network

# Create a network manually
docker network create my-network
```
---
## 12. Environment Variables & .env Files

**Never hardcode secrets in your Dockerfile or docker-compose.yml.**
### `.env` file
```bash
# .env — add this to .gitignore!
DB_USER=myuser
DB_PASSWORD=supersecretpassword
DB_NAME=myapp
JWT_SECRET=anothersecret
NODE_ENV=development
```
### Using in docker-compose.yml

Docker Compose automatically reads `.env` from the same directory:
```yaml
services:
  db:
    environment:
      POSTGRES_USER: ${DB_USER}           # From .env
      POSTGRES_PASSWORD: ${DB_PASSWORD}   # From .env
      POSTGRES_DB: ${DB_NAME}             # From .env
```
### Multiple environments
```
.env                  # Default (development)
.env.production       # Production overrides
.env.test             # Test environment
```
```bash
# Use a specific env file
docker compose --env-file .env.production up
```
### Accessing in Node.js
```javascript
// Your existing code works as-is!
const dbUrl = process.env.DATABASE_URL;
const jwtSecret = process.env.JWT_SECRET;
```
---
## 13. Multi-Stage Builds — Optimized Production Images

Multi-stage builds let you use one Docker image to build and another to run — resulting in **tiny, secure production images**.
### Node.js Production Dockerfile
```dockerfile
# ─── Stage 1: Dependencies ───────────────────────────
FROM node:20-alpine AS deps

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production     # Install only prod dependencies


# ─── Stage 2: Builder ────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci                       # Install all deps (including devDeps)
COPY . .
RUN npm run build                # If you have a build step (TypeScript, etc.)


# ─── Stage 3: Production Runner ──────────────────────
FROM node:20-alpine AS runner

# Create non-root user for security
RUN addgroup -g 1001 nodejs && \
    adduser -S -u 1001 -G nodejs nodeuser

WORKDIR /app

# Copy only what's needed to run the app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package.json ./

USER nodeuser

EXPOSE 3000

CMD ["node", "dist/index.js"]
```
### Size comparison
```
node:20 (full)         →  1.1 GB
node:20-alpine         →  170 MB
Multi-stage alpine     →  ~60 MB   ✅
```
### Build for production

```bash
# Build production image
docker build -t my-backend:prod --target runner .

# Run production container
docker run -d \
  -p 3000:3000 \
  --env-file .env.production \
  my-backend:prod
```
---
## 14. Essential Docker Commands Cheat Sheet
### Images
```bash
docker images                          # List all local images
docker pull node:20-alpine             # Download an image
docker build -t myapp:v1 .             # Build image from Dockerfile
docker push myapp:v1                   # Push to Docker Hub / registry
docker rmi myapp:v1                    # Remove an image
docker image prune                     # Remove unused images
docker tag myapp:v1 myapp:latest       # Tag an image
```
### Containers
```bash
docker run -d -p 3000:3000 myapp:v1   # Run detached with port mapping
docker ps                              # List running containers
docker ps -a                           # List all containers
docker stop <name/id>                  # Gracefully stop
docker kill <name/id>                  # Force stop
docker rm <name/id>                    # Remove container
docker rm -f <name/id>                 # Force remove running container
docker logs -f <name/id>               # Follow logs
docker exec -it <name/id> sh           # Shell inside container
docker inspect <name/id>               # Full container details
docker stats                           # Live CPU/memory usage
```
### System
```bash
docker system df                       # Disk usage
docker system prune                    # Remove all unused resources
docker system prune -a                 # Remove everything not running
```
### Docker Compose
```bash
docker compose up -d                   # Start all services (background)
docker compose up --build              # Rebuild and start
docker compose down                    # Stop and remove containers
docker compose down -v                 # Also remove volumes
docker compose ps                      # Status of services
docker compose logs -f                 # Follow all logs
docker compose logs -f backend         # Follow specific service
docker compose exec backend sh         # Shell in service
docker compose restart backend         # Restart one service
docker compose pull                    # Pull latest images
docker compose config                  # Validate compose file
```
---
## 15. Docker in CI/CD (GitHub Actions)

Automate building and pushing Docker images on every push.

### `.github/workflows/docker.yml`

```yaml
name: Build & Push Docker Image

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout code
      - name: Checkout
        uses: actions/checkout@v4

      # 2. Login to Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 3. Set up Docker Buildx (for multi-platform builds)
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 4. Build and push
      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: |
            yourusername/myapp-backend:latest
            yourusername/myapp-backend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

> Set `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` in your GitHub repo's Settings → Secrets.

---
## 16. Best Practices

### ✅ Dockerfile Best Practices

```dockerfile
# ✅ Use specific versions (not :latest in production)
FROM node:20.11-alpine

# ✅ Use .dockerignore to exclude unnecessary files

# ✅ Order layers from least to most frequently changing
COPY package*.json ./    # Changes rarely
RUN npm install          # Cached if package.json unchanged
COPY . .                 # Changes often

# ✅ Run as non-root user
RUN adduser -S appuser
USER appuser

# ✅ Use COPY over ADD (unless you need tar extraction)
COPY . .

# ✅ Combine RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# ✅ Use multi-stage builds for production
```

### ✅ Docker Compose Best Practices

```yaml
# ✅ Always set restart policy
restart: unless-stopped

# ✅ Use health checks
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 10s
  retries: 3

# ✅ Set resource limits
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M

# ✅ Use depends_on with condition for reliability
depends_on:
  db:
    condition: service_healthy
```

### ✅ Security Best Practices

```bash
# ✅ Never use :latest in production
FROM node:20.11-alpine    # Not: FROM node:latest

# ✅ Scan images for vulnerabilities
docker scout cves myapp:v1

# ✅ Never put secrets in Dockerfile or docker-compose.yml
# Use .env files (excluded from git) or Docker secrets

# ✅ Run as non-root user inside containers

# ✅ Keep base images updated regularly
```

---
## 17. Common Mistakes & How to Fix Them

### ❌ Problem: Container exits immediately

```bash
docker logs mycontainer
# Check what error is shown
```

Usually: wrong CMD, application crash, or missing env variable.

---
### ❌ Problem: Port already in use
```
Error: port is already allocated
```

```bash
# Find what's using the port
lsof -i :3000

# Use a different host port
docker run -p 3001:3000 myapp  # Map to 3001 instead
```
---
### ❌ Problem: Cannot connect to database
```
Error: connect ECONNREFUSED localhost:5432
```

Inside Docker, use the **service name**, not `localhost`:

```javascript
// ❌ Wrong (inside Docker)
DATABASE_URL=postgresql://user:pass@localhost:5432/myapp

// ✅ Correct (inside Docker Compose network)
DATABASE_URL=postgresql://user:pass@db:5432/myapp
```
---
### ❌ Problem: node_modules conflict (bind mount)
```bash
# Symptom: Error: Cannot find module 'express'
# Fix in docker-compose.yml:
volumes:
  - ./backend:/app
  - /app/node_modules   # ← This line saves you!
```
---
### ❌ Problem: Changes not reflected
```bash
# If you changed Dockerfile or package.json:
docker compose up --build    # Force rebuild

# If you just changed source code (with bind mount):
# Changes should reflect automatically (no rebuild needed)
```
---
### ❌ Problem: Image is too large
```bash
# Check image size
docker images myapp
# Fix: Use alpine images + multi-stage builds
FROM node:20-alpine    # Not: FROM node:20
```

---
### ❌ Problem: Database data lost after `docker compose down`
```bash
# ❌ This deletes volumes (your DB data)!
docker compose down -v

# ✅ Use this to keep data
docker compose down

# ✅ Or use named volumes in docker-compose.yml (already shown above)
```
---
## 18. Docker Hub — Push, Pull & Share Your Images

Docker Hub is the **official public registry** for Docker images — think of it like GitHub, but for Docker images. You push your image once, and anyone (or any server) can pull and run it anywhere in the world.
```
Your Machine                    Docker Hub                   Any Server / Teammate
─────────────                   ──────────                   ────────────────────
docker build  →  local image
docker push   ──────────────→   yourusername/myapp:v1   →── docker pull
                                                             docker run
```
---
### Step 1: Create a Docker Hub Account

Go to 👉 https://hub.docker.com and create a free account.

Your username will be part of every image name you publish:

```
yourusername/image-name:tag```

---

### Step 2: Login to Docker Hub from Terminal

```bash
docker login
# Enter your Docker Hub username and password when prompted

# Or login with username inline (prompts for password)
docker login -u yourusername
```

You should see:

```
Login Succeeded
```

> Your credentials are saved in `~/.docker/config.json` — you only need to login once per machine.

---
### Step 3: Tag Your Image Correctly

Docker Hub requires images to be named in the format `username/repository:tag`.

```bash
# Check your existing local images
docker images

# Tag an existing image for Docker Hub
docker tag local-image-name:tag yourusername/repository-name:tag

# Examples:
docker tag my-backend:v1        yourusername/myapp-backend:v1
docker tag my-backend:v1        yourusername/myapp-backend:latest
docker tag my-frontend:v1       yourusername/myapp-frontend:v1
```

> 🔑 **Tip:** Always tag with both a version (`v1`, `v2`) **and** `latest`. `latest` is what gets pulled by default when no tag is specified.

---

### Step 4: Push Image to Docker Hub

```bash
# Push a specific tag
docker push yourusername/myapp-backend:v1

# Push the latest tag
docker push yourusername/myapp-backend:latest

# Push all tags at once
docker push yourusername/myapp-backend --all-tags
```

**Example output:**

```
The push refers to repository [docker.io/yourusername/myapp-backend]
a1b2c3d4e5f6: Pushed
f7g8h9i0j1k2: Pushed
v1: digest: sha256:abc123... size: 1234
```

After pushing, your image is visible at:

```
https://hub.docker.com/r/yourusername/myapp-backend
```

---

### Step 5: Pull the Image (on any machine)

Anyone with Docker installed can now pull your image — no login required for public images.

```bash
# Pull a specific version
docker pull yourusername/myapp-backend:v1

# Pull latest (default)
docker pull yourusername/myapp-backend

# Pull and immediately run
docker run -d -p 3000:3000 yourusername/myapp-backend:v1
```

---

### Step 6: Use the Pulled Image

#### Run directly

```bash
# Basic run
docker run -d \
  -p 3000:3000 \
  --name backend \
  yourusername/myapp-backend:v1

# Run with environment variables
docker run -d \
  -p 3000:3000 \
  --name backend \
  -e DATABASE_URL=postgresql://user:pass@db:5432/myapp \
  -e NODE_ENV=production \
  yourusername/myapp-backend:v1
```

#### Use in docker-compose.yml (no build needed!)

Instead of building from source, teammates or servers can pull your pre-built image:

```yaml
version: '3.9'

services:

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - myapp-network

  backend:
    image: yourusername/myapp-backend:v1    # ← Pull from Docker Hub (no build!)
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      NODE_ENV: production
    depends_on:
      - db
    networks:
      - myapp-network

  frontend:
    image: yourusername/myapp-frontend:v1   # ← Pull from Docker Hub (no build!)
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - myapp-network

volumes:
  postgres-data:

networks:
  myapp-network:
    driver: bridge
```

Now your teammate just needs:

```bash
# No source code, no Node.js, no npm install needed!
docker compose up -d
```

---

### Full Push Workflow (Real Project)

Here is the complete workflow from code change to deployment:

```bash
# 1. Make changes to your code

# 2. Build a new versioned image
docker build -t yourusername/myapp-backend:v2 .

# 3. Also tag as latest
docker tag yourusername/myapp-backend:v2 yourusername/myapp-backend:latest

# 4. Push both tags
docker push yourusername/myapp-backend:v2
docker push yourusername/myapp-backend:latest

# 5. On your server — pull and restart
docker pull yourusername/myapp-backend:latest
docker stop backend && docker rm backend
docker run -d -p 3000:3000 --name backend yourusername/myapp-backend:latest
```

---

### Public vs Private Repositories

|Feature|Free (Public)|Free (Private)|Pro Plan|
|---|---|---|---|
|Public repos|Unlimited|—|Unlimited|
|Private repos|0|1|Unlimited|
|Pull rate limit|100/6hr (anonymous)|200/6hr (logged in)|Unlimited|

```bash
# Create a private repo: go to hub.docker.com → Create Repository → set to Private
# Push works the same way — but pullers must be logged in with access

# Pull from private repo (login first)
docker login
docker pull yourusername/myapp-private:v1
```

---

### Versioning Strategy — Best Practices

```bash
# ✅ Semantic versioning
docker push yourusername/myapp:1.0.0
docker push yourusername/myapp:1.0.1   # patch fix
docker push yourusername/myapp:1.1.0   # new feature
docker push yourusername/myapp:2.0.0   # breaking change

# ✅ Git SHA tagging (great for CI/CD traceability)
docker push yourusername/myapp:a3f5c8b

# ✅ Environment tagging
docker push yourusername/myapp:staging
docker push yourusername/myapp:production

# ✅ Always keep :latest pointing to the stable version
docker tag yourusername/myapp:1.1.0 yourusername/myapp:latest
docker push yourusername/myapp:latest

# ❌ Avoid: only using :latest (no history, no rollback)
```

---

### Rolling Back to a Previous Version

```bash
# Pull a specific old version
docker pull yourusername/myapp-backend:v1

# Run the old version
docker run -d -p 3000:3000 \
  --name backend \
  yourusername/myapp-backend:v1

# In docker-compose.yml — just change the tag and restart
# image: yourusername/myapp-backend:v1   ← change back
docker compose up -d```

---
### Useful Docker Hub Commands

```bash
# Search Docker Hub from terminal
docker search node
docker search postgres

# See image tags (requires curl)
curl -s https://hub.docker.com/v2/repositories/yourusername/myapp-backend/tags \
  | python3 -m json.tool

# Logout
docker logout

# Login to a different registry (e.g., GitHub Container Registry)
docker login ghcr.io -u yourusername
```
---
### Alternative Registries

Docker Hub is not the only option:

|Registry|URL|Best For|
|---|---|---|
|**Docker Hub**|hub.docker.com|Default, public images|
|**GitHub Container Registry**|ghcr.io|Open source projects on GitHub|
|**AWS ECR**|*.amazonaws.com|AWS deployments|
|**Google Artifact Registry**|*.pkg.dev|GCP deployments|
|**Azure Container Registry**|*.azurecr.io|Azure deployments|

```bash
# Example: Push to GitHub Container Registry
docker tag myapp:v1 ghcr.io/yourgithubusername/myapp:v1
docker push ghcr.io/yourgithubusername/myapp:v1
```
---
## 🚀 What's Next?

Now that you know Docker, explore these topics:

|Topic|Description|
|---|---|
|**Docker Swarm**|Simple container orchestration built into Docker|
|**Kubernetes (K8s)**|Production-grade container orchestration at scale|
|**Docker Hub / Registry**|Push and share your images|
|**Watchtower**|Auto-update running containers when new images are pushed|
|**Traefik / Nginx Proxy**|Reverse proxy and SSL termination for Docker|
|**Docker Scout**|Vulnerability scanning for your images|
|**Portainer**|GUI dashboard for managing Docker|

---
## 📋 Quick Reference Card

```
Build image:      docker build -t name:tag .
Run container:    docker run -d -p host:container name:tag
View logs:        docker logs -f container
Shell access:     docker exec -it container sh
Stop container:   docker stop container
Remove container: docker rm container

Compose up:       docker compose up -d
Compose down:     docker compose down
Compose logs:     docker compose logs -f
Compose rebuild:  docker compose up --build

Docker Hub login: docker login
Tag for Hub:      docker tag local-image username/repo:tag
Push to Hub:      docker push username/repo:tag
Pull from Hub:    docker pull username/repo:tag
Run from Hub:     docker run -d -p 3000:3000 username/repo:tag
```

---

_Happy Dockerizing! 🐳_