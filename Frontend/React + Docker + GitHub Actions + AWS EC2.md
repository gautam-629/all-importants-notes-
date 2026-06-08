
> Complete guide from zero — separate dev/prod environments, Docker Hub sync, EC2 auto-deploy No nginx — uses `vite preview` as the production server
---
## Table of Contents

1. [Project Structure Overview](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#1-project-structure-overview)
2. [Dockerfiles — Dev vs Production](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#2-dockerfiles--dev-vs-production)
3. [Environment Files](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#3-environment-files)
4. [Docker Compose Files](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#4-docker-compose-files)
5. [GitHub Actions — Development Pipeline](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#5-github-actions--development-pipeline)
6. [GitHub Actions — Production Pipeline](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#6-github-actions--production-pipeline)
7. [AWS EC2 Setup (Zero to Ready)](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#7-aws-ec2-setup-zero-to-ready)
8. [GitHub Secrets Setup](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#8-github-secrets-setup)
9. [Full Deploy Flow Diagram](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#9-full-deploy-flow-diagram)
10. [Rollback Strategy](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#10-rollback-strategy)
11. [Troubleshooting](https://claude.ai/chat/f59a391d-cbbf-40e6-89d0-6c2a37667baf#11-troubleshooting)

---
## 1. Project Structure Overview

```
my-frontend/
├── src/
├── public/
├── index.html
├── vite.config.ts
├── tsconfig.json
├── package.json
│
├── docker/
│   ├── Dockerfile.dev          ← Development image (hot reload)
│   └── Dockerfile.prod         ← Production image (build + vite preview)
│
├── .env.development            ← Dev environment variables
├── .env.production             ← Prod environment variables
│
├── docker-compose.dev.yml      ← Run locally in dev mode
├── docker-compose.prod.yml     ← Run locally to test prod mode
│
├── .dockerignore
├── .gitignore
│
└── .github/
    └── workflows/
        ├── dev.yml             ← CI/CD for development branch
        └── prod.yml            ← CI/CD for main/production branch
```

---
## 2. Dockerfiles — Dev vs Production

### `docker/Dockerfile.dev` — Development

```dockerfile
# ── Development Dockerfile ──────────────────────────────────────────
# Hot reload enabled — source is mounted as a volume
# No build step needed; Vite handles everything in memory

FROM node:20-alpine

WORKDIR /app

# Copy package files first (Docker layer caching trick)
# If package.json hasn't changed, Docker reuses the cached npm install layer
COPY package*.json ./

# Install ALL dependencies including devDependencies
RUN npm install

# Copy source code
COPY . .

# Vite dev server runs on port 5173 by default
EXPOSE 5173

# --host 0.0.0.0 makes Vite listen on all interfaces
# Without this, Vite only listens on localhost INSIDE the container
# and you can't reach it from your browser on the host machine
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

---
### `docker/Dockerfile.prod` — Production

```dockerfile
# ── Production Dockerfile ────────────────────────────────────────────
# Stage 1: Install deps, run lint, run build
# Stage 2: Copy the built dist/ and serve it with vite preview
#
# Why two stages?
# Stage 1 needs devDependencies (TypeScript, ESLint, Vite plugins)
# Stage 2 only needs the built files + vite to serve them
# This keeps the final image smaller and cleaner

# ──────────────────────────────────────────────
# Stage 1: Builder
# ──────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files (cached layer if package.json unchanged)
COPY package*.json ./

# npm ci = faster + stricter version of npm install
# Installs exact versions from package-lock.json
# Includes devDependencies needed for build (TypeScript, Vite, ESLint)
RUN npm ci

# Copy all source code into the container
COPY . .

# Run ESLint — if any lint error exists, Docker build STOPS here
# This means a bad push will never make it to Docker Hub or EC2
RUN npm run lint

# Run TypeScript compiler + Vite build
# tsc -b checks types, vite build creates the dist/ folder
RUN npm run build

# ──────────────────────────────────────────────
# Stage 2: Production Server (vite preview)
# ──────────────────────────────────────────────
FROM node:20-alpine AS runner

WORKDIR /app

# Copy only package files
COPY package*.json ./

# Install ALL deps — vite is a devDependency but needed to run vite preview
RUN npm ci

# Copy the built React app from Stage 1
# We only copy dist/ — no source code in the final image
COPY --from=builder /app/dist ./dist

# vite preview serves on port 4173 by default
EXPOSE 4173

# --host 0.0.0.0 = listen on all interfaces (required inside Docker)
# --port 4173    = explicit port (matches EXPOSE above)
CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0", "--port", "4173"]
```

> **Why `vite preview`?** `vite preview` is the built-in way to serve the production `dist/` folder — no extra tools needed. It's simple, requires zero config, and works perfectly for learning and small projects.

---
## 3. Environment Files

### `.env.development`

```bash
# Used by Vite dev server locally and in the dev Docker container
# VITE_ prefix = Vite injects these into the browser bundle at build time
VITE_API_URL=https://typicode.com
VITE_APP_ENV=development
VITE_APP_TITLE=My App (Dev)
```

### `.env.production`

```bash
# Used during npm run build — values are baked into the dist/ files
VITE_API_URL=https://typicode.com
VITE_APP_ENV=production
VITE_APP_TITLE=My App
```
### How to use env variables in your React code

```typescript
// Access anywhere in your React/TypeScript code
const apiUrl = import.meta.env.VITE_API_URL;

// Example fetch call using your API URL
fetch(`${import.meta.env.VITE_API_URL}/posts`)
  .then(res => res.json())
  .then(data => console.log(data));
```

### `.gitignore` — make sure these are included

```bash
# Secrets — never commit these
.env.local
.env.*.local

# Dependencies
node_modules/

# Build output
dist/
build/
```

### `.dockerignore` — speeds up Docker builds

```
node_modules
dist
.git
.gitignore
.env.local
.env.*.local
*.log
README.md
.github
```

---
## 4. Docker Compose Files

### `docker-compose.dev.yml` — Local Development

```yaml
version: '3.9'

services:
  frontend:
    build:
      context: .
      dockerfile: docker/Dockerfile.dev
    container_name: frontend-dev
    ports:
      - "5173:5173"       # host port : container port
    volumes:
      - .:/app            # Bind mount — file changes on host reflect instantly
      - /app/node_modules # Prevent host node_modules from overwriting container's
    env_file:
      - .env.development
    restart: unless-stopped
```

**Run locally in dev mode:**

```bash
docker compose -f docker-compose.dev.yml up --build
# Open: http://localhost:5173
# Edit any file → browser hot-reloads automatically
```

---

### `docker-compose.prod.yml` — Local Production Test

```yaml
version: '3.9'

services:
  frontend:
    image: ${DOCKER_HUB_USERNAME}/my-frontend:latest   # Pulled from Docker Hub
    container_name: frontend-prod
    ports:
      - "4173:4173"       # vite preview default port
    restart: unless-stopped
```

**Test the production image locally before deploying:**

```bash
DOCKER_HUB_USERNAME=yourusername docker compose -f docker-compose.prod.yml up
# Open: http://localhost:4173
```

---

## 5. GitHub Actions — Development Pipeline

### `.github/workflows/dev.yml`

```yaml
name: 🔧 Development CI

# Triggers on any push or pull request to the 'develop' branch
on:
  push:
    branches:
      - develop
  pull_request:
    branches:
      - develop

env:
  IMAGE_NAME: ${{ secrets.DOCKER_HUB_USERNAME }}/my-frontend
  IMAGE_TAG: dev-${{ github.sha }}     # e.g. dev-a3f5c8b (unique per commit)

jobs:
  # ─────────────────────────────────────────────
  # Job 1: Lint + Type Check
  # Fast feedback — runs before anything else
  # ─────────────────────────────────────────────
  lint-and-typecheck:
    name: 🔍 Lint & Type Check
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'              # Cache node_modules between workflow runs

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint           # Fails the job if lint errors found

      - name: Run TypeScript check
        run: npx tsc --noEmit      # Checks types without producing output files

  # ─────────────────────────────────────────────
  # Job 2: Build
  # Only runs if lint + typecheck passed
  # ─────────────────────────────────────────────
  build:
    name: 🏗️ Build
    runs-on: ubuntu-latest
    needs: lint-and-typecheck       # Depends on Job 1 succeeding

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL_DEV }}
          VITE_APP_ENV: development

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist-dev-${{ github.sha }}
          path: dist/
          retention-days: 3         # Keep dev build artifacts for 3 days only

  # ─────────────────────────────────────────────
  # Job 3: Docker Build + Push
  # Pushes a dev-tagged image to Docker Hub
  # Does NOT deploy to EC2 (dev branch only)
  # ─────────────────────────────────────────────
  docker-build-push:
    name: 🐳 Docker Build & Push (Dev)
    runs-on: ubuntu-latest
    needs: build                    # Depends on Job 2 succeeding

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push dev image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: docker/Dockerfile.prod
          push: true
          tags: |
            ${{ env.IMAGE_NAME }}:dev-latest
            ${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          build-args: |
            VITE_API_URL=${{ secrets.VITE_API_URL_DEV }}
            VITE_APP_ENV=development
          cache-from: type=gha      # Use GitHub Actions cache (faster builds)
          cache-to: type=gha,mode=max
```

---

## 6. GitHub Actions — Production Pipeline

### `.github/workflows/prod.yml`

```yaml
name: 🚀 Production CI/CD

# Triggers ONLY on push to 'main' branch
on:
  push:
    branches:
      - main

env:
  IMAGE_NAME: ${{ secrets.DOCKER_HUB_USERNAME }}/my-frontend
  IMAGE_TAG: v${{ github.run_number }}   # e.g. v1, v2, v3 — increments each run

jobs:
  # ─────────────────────────────────────────────
  # Job 1: Lint + Type Check
  # ─────────────────────────────────────────────
  lint-and-typecheck:
    name: 🔍 Lint & Type Check
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run TypeScript check
        run: npx tsc --noEmit

  # ─────────────────────────────────────────────
  # Job 2: Build + Verify
  # ─────────────────────────────────────────────
  build:
    name: 🏗️ Build & Verify
    runs-on: ubuntu-latest
    needs: lint-and-typecheck

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project (production)
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL_PROD }}
          VITE_APP_ENV: production

      - name: Show build output size
        run: |
          echo "── dist/ folder size ──────────────"
          du -sh dist/
          echo "── dist/assets/ contents ──────────"
          ls -lh dist/assets/

      - name: Upload production artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist-prod-${{ github.sha }}
          path: dist/
          retention-days: 30        # Keep prod artifacts for 30 days

  # ─────────────────────────────────────────────
  # Job 3: Docker Build + Push to Docker Hub
  # ─────────────────────────────────────────────
  docker-build-push:
    name: 🐳 Docker Build & Push (Prod)
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push production image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: docker/Dockerfile.prod
          push: true
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
            ${{ env.IMAGE_NAME }}:${{ github.sha }}
          build-args: |
            VITE_API_URL=${{ secrets.VITE_API_URL_PROD }}
            VITE_APP_ENV=production
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─────────────────────────────────────────────
  # Job 4: Deploy to AWS EC2
  # SSHes into your EC2, pulls the new image, restarts the container
  # ─────────────────────────────────────────────
  deploy-to-ec2:
    name: 🖥️ Deploy to AWS EC2
    runs-on: ubuntu-latest
    needs: docker-build-push

    steps:
      - name: Deploy to EC2 via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}       # Your EC2 public IP
          username: ${{ secrets.EC2_USER }}   # ubuntu (for Ubuntu AMI)
          key: ${{ secrets.EC2_SSH_KEY }}     # Contents of your .pem file

          # These commands run directly on your EC2 instance:
          script: |
            echo "──────────────────────────────────────"
            echo "Deploying: ${{ env.IMAGE_TAG }}"
            echo "──────────────────────────────────────"

            # Pull the latest image from Docker Hub
            docker pull ${{ env.IMAGE_NAME }}:latest

            # Stop the old container (|| true prevents error if not running)
            docker stop frontend-prod || true
            docker rm frontend-prod || true

            # Start a new container with the freshly pulled image
            docker run -d \
              --name frontend-prod \
              --restart unless-stopped \
              -p 4173:4173 \
              ${{ env.IMAGE_NAME }}:latest

            # Remove unused old images to free up disk space
            docker image prune -f

            echo "✅ Deploy complete!"
            docker ps --filter name=frontend-prod

  # ─────────────────────────────────────────────
  # Job 5: Health Check
  # Confirms the site is actually up after deploy
  # ─────────────────────────────────────────────
  health-check:
    name: ✅ Health Check
    runs-on: ubuntu-latest
    needs: deploy-to-ec2

    steps:
      - name: Wait for container to start
        run: sleep 15                         # Give the container 15s to boot

      - name: Check HTTP response
        run: |
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${{ secrets.EC2_HOST }}:4173)
          echo "HTTP Status: $STATUS"
          if [ "$STATUS" != "200" ]; then
            echo "❌ Health check failed — got $STATUS instead of 200"
            exit 1
          fi
          echo "✅ Site is live and returning 200"

      - name: Notify on failure
        if: failure()
        run: |
          echo "🚨 DEPLOY FAILED"
          echo "SSH into EC2 and check: docker logs frontend-prod"
```

---

## 7. AWS EC2 Setup (Zero to Ready)

### Step 1: Launch EC2 Instance

1. Go to **AWS Console** → **EC2** → **Launch Instance**
2. Choose **Ubuntu Server 24.04 LTS** (Free tier eligible)
3. Instance type: `t2.micro` (free) or `t3.small` (better performance)
4. Create a new **Key Pair** → download the `.pem` file → **store it safely, you cannot re-download it**
5. Under **Security Group**, add these inbound rules:

|Type|Port|Source|Purpose|
|---|---|---|---|
|SSH|22|0.0.0.0/0|GitHub Actions SSH|
|Custom TCP|4173|0.0.0.0/0|vite preview server|

6. Click **Launch Instance**
7. Note your **Public IPv4 address** — this is your `EC2_HOST` secret

---

### Step 2: Connect to EC2 and Install Docker

```bash
# On your LOCAL machine — set correct permissions on the PEM file
chmod 400 your-key.pem

# SSH into your EC2
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP

# ── You are now INSIDE EC2 ─────────────────────────────

# Update the package list
sudo apt update && sudo apt upgrade -y

# Download and run Docker's official install script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to the docker group (so you don't need sudo every time)
sudo usermod -aG docker ubuntu

# Apply the group change in this session
newgrp docker

# Verify Docker installed correctly
docker --version
docker run hello-world
```

---

### Step 3: Test a Manual Deploy on EC2

Before relying on GitHub Actions, test pulling and running your image manually:

```bash
# Pull your image from Docker Hub (no login needed for public images)
docker pull yourusername/my-frontend:latest

# Run the container
docker run -d \
  --name frontend-prod \
  --restart unless-stopped \
  -p 4173:4173 \
  yourusername/my-frontend:latest

# Confirm it's running
docker ps

# Quick test
curl http://localhost:4173
```

Open in your browser: `http://YOUR_EC2_PUBLIC_IP:4173` — you should see your React app ✅

---

### Step 4: Get Your SSH Key Ready for GitHub Actions

```bash
# On your LOCAL machine — print your PEM key
cat your-key.pem

# Copy the ENTIRE output including the header and footer lines:
# -----BEGIN RSA PRIVATE KEY-----
# MIIEowIBAAKCAQEA...
# -----END RSA PRIVATE KEY-----
```

This full text goes into GitHub Secrets as `EC2_SSH_KEY`.

---

## 8. GitHub Secrets Setup

Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add all 7 of these:

|Secret Name|Example Value|What It Is|
|---|---|---|
|`DOCKER_HUB_USERNAME`|`johndoe`|Your Docker Hub username|
|`DOCKER_HUB_TOKEN`|`dckr_pat_abc123...`|Docker Hub Access Token (not your password)|
|`EC2_HOST`|`18.234.56.78`|Your EC2 public IP address|
|`EC2_USER`|`ubuntu`|SSH username (ubuntu for Ubuntu AMI)|
|`EC2_SSH_KEY`|`-----BEGIN RSA PRIVATE KEY-----...`|Full content of your `.pem` file|
|`VITE_API_URL_DEV`|`https://typicode.com`|API base URL for development builds|
|`VITE_API_URL_PROD`|`https://typicode.com`|API base URL for production builds|

### How to get a Docker Hub Access Token

1. Login to [hub.docker.com](https://hub.docker.com/)
2. Go to **Account Settings** → **Security** → **New Access Token**
3. Name it `github-actions`, set permissions to Read/Write
4. Copy the token — it starts with `dckr_pat_`
5. Paste it as `DOCKER_HUB_TOKEN` in GitHub Secrets

---

## 9. Full Deploy Flow Diagram

```
You push code to GitHub
         │
         ├── push to 'develop' → dev.yml runs
         └── push to 'main'    → prod.yml runs
                  │
                  ▼
         ┌─────────────┐
         │  1. Lint    │  npm run lint
         │   ESLint    │  ← STOPS if lint errors
         └──────┬──────┘
                │ ✅
                ▼
         ┌─────────────┐
         │  2. Type    │  npx tsc --noEmit
         │   Check     │  ← STOPS if TypeScript errors
         └──────┬──────┘
                │ ✅
                ▼
         ┌─────────────┐
         │  3. Build   │  npm run build
         │   Vite      │  → creates dist/ folder
         └──────┬──────┘
                │ ✅
                ▼
         ┌──────────────────────────┐
         │  4. Docker Build & Push  │
         │     Dockerfile.prod      │
         │     lint → build inside  │
         │     → Docker Hub         │
         │     yourusername/app:v3  │
         └──────────┬───────────────┘
                    │ ✅  (prod.yml only from here)
                    ▼
         ┌──────────────────────────┐
         │  5. SSH into EC2         │
         │     docker pull :latest  │
         │     docker stop old      │
         │     docker run new       │
         └──────────┬───────────────┘
                    │ ✅
                    ▼
         ┌──────────────────────────┐
         │  6. Health Check         │
         │     curl EC2_HOST:4173   │
         │     Expect HTTP 200      │
         └──────────────────────────┘
```

---

## 10. Rollback Strategy

If something goes wrong after a deploy, roll back in under a minute:

```bash
# SSH into your EC2
ssh -i your-key.pem ubuntu@YOUR_EC2_IP

# See all available image versions on this machine
docker images yourusername/my-frontend

# Roll back to the previous version
docker stop frontend-prod
docker rm frontend-prod

docker run -d \
  --name frontend-prod \
  --restart unless-stopped \
  -p 4173:4173 \
  yourusername/my-frontend:v2    # ← change to the previous version number

# Confirm it's back up
docker ps
curl http://localhost:4173
```

Or do it via git — this triggers the full pipeline cleanly:

```bash
# Undo the last commit and push to main
git revert HEAD
git push origin main
# prod.yml pipeline runs automatically with the reverted code
```

---

## 11. Troubleshooting

### ❌ Lint fails in GitHub Actions but works locally

```bash
# Make sure you run the exact same command as CI before pushing
npm run lint

# Auto-fix what can be fixed automatically
npx eslint . --ext ts,tsx --fix

# Then commit the fixed files and push again
```

---

### ❌ Docker build fails in GitHub Actions

```bash
# Test the exact same build locally first
docker build -t test-build -f docker/Dockerfile.prod .

# Common cause: .env.production not found
# Fix: make sure .env.production is committed to the repo
# (safe to commit if it only has VITE_ variables — no passwords)
```

---

### ❌ SSH connection refused on EC2

Check these in order:

1. **Port 22 open?** → AWS Console → Security Group → Inbound Rules → Port 22 must allow `0.0.0.0/0`
2. **Correct user?** → Ubuntu AMI = `ubuntu` | Amazon Linux = `ec2-user`
3. **Key format correct?** → The `EC2_SSH_KEY` secret must include the full header/footer lines
4. **No extra spaces?** → When pasting PEM into GitHub Secrets, avoid leading/trailing whitespace

---

### ❌ Site not accessible on EC2 after deploy

```bash
# 1. Check the container is actually running
docker ps

# 2. Check container logs for errors
docker logs frontend-prod

# 3. Test locally on EC2
curl http://localhost:4173

# 4. Check port 4173 is open in EC2 Security Group
#    AWS Console → EC2 → Security Groups → Inbound Rules → add port 4173
```

---

### ❌ API calls fail (CORS errors in browser console)

This is a browser security issue. Fix it in your Vite config for local dev:

```typescript
// vite.config.ts — add a proxy so dev requests don't hit CORS
export default {
  server: {
    proxy: {
      '/api': {
        target: 'https://typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
}

// Then in your React code, use relative paths in dev:
fetch('/api/posts')   // Vite proxies this to https://typicode.com/posts
```

For production (EC2), call the API directly using your env variable:

```typescript
fetch(`${import.meta.env.VITE_API_URL}/posts`)
```

---

### ❌ Old images filling up EC2 disk

```bash
# On EC2 — remove all images not currently used by a running container
docker image prune -a -f

# See current disk usage breakdown
docker system df
```

---

## Quick Start Checklist

```
One-time setup:
  □ Create Docker Hub account + generate an Access Token
  □ Launch EC2 instance (Ubuntu, open ports 22 and 4173)
  □ SSH into EC2 and install Docker
  □ Add all 7 secrets to GitHub repository Settings
  □ Create docker/Dockerfile.dev and docker/Dockerfile.prod
  □ Create docker-compose.dev.yml and docker-compose.prod.yml
  □ Create .github/workflows/dev.yml and prod.yml
  □ Create .env.development and .env.production

Daily workflow:
  □ Develop on 'develop' branch
  □ Push → dev.yml runs: lint → typecheck → build → Docker push (no EC2 deploy)
  □ Merge develop into main
  □ Push → prod.yml runs: lint → typecheck → build → Docker push → EC2 deploy → health check
  □ Visit http://YOUR_EC2_IP:4173 to see your live site
```

---

_Happy shipping! 🚀🐳_