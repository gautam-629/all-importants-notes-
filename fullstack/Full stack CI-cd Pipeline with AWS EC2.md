
### Node.js + React + PostgreSQL + Docker + GitHub Actions + AWS EC2

---

## 📋 Task Overview

Build and deploy a **full-stack Task Manager application** with a complete CI/CD pipeline. Every time you push code to GitHub, it automatically builds Docker images, pushes them to Docker Hub, and deploys to your AWS EC2 server — with zero manual steps.

**What you will build:**

- A working full-stack app (React frontend + Node.js API + PostgreSQL)
- Dockerized with Docker Compose
- Automated CI/CD with GitHub Actions
- Live on a real AWS EC2 server accessible from the internet

---

## 🏗️ Architecture

```
Developer Machine
      │
      │  git push
      ▼
┌─────────────────┐
│    GitHub Repo  │
│                 │
│  GitHub Actions │  ← Triggered on push to main
│  CI/CD Pipeline │
└────────┬────────┘
         │
    ┌────┴─────┐
    │          │
    ▼          ▼
┌────────┐  ┌──────────────────────────┐
│ Tests  │  │  Docker Build & Push     │
│ Lint   │  │  → Docker Hub            │
└────────┘  │    yourusername/api:sha  │
            │    yourusername/web:sha  │
            └────────────┬─────────────┘
                         │  SSH Deploy
                         ▼
              ┌─────────────────────┐
              │    AWS EC2 Instance │
              │  Ubuntu 22.04       │
              │                     │
              │  ┌───────────────┐  │
              │  │  Docker       │  │
              │  │  Compose      │  │
              │  │               │  │
              │  │ [React:80]    │  │
              │  │ [Node:3000]   │  │
              │  │ [Postgres]    │  │
              │  └───────────────┘  │
              └─────────────────────┘
                         │
                  http://your-ec2-ip
```

---

## 📁 Project Structure

```
taskmanager/
├── backend/
│   ├── src/
│   │   ├── index.js
│   │   ├── routes/
│   │   │   └── tasks.js
│   │   └── db/
│   │       └── pool.js
│   ├── package.json
│   ├── Dockerfile
│   └── .dockerignore
│
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   └── TaskList.jsx
│   │   └── main.jsx
│   ├── package.json
│   ├── Dockerfile
│   ├── nginx.conf
│   └── .dockerignore
│
├── .github/
│   └── workflows/
│       └── deploy.yml          ← CI/CD pipeline
│
├── docker-compose.yml           ← Development
├── docker-compose.prod.yml      ← Production (uses Hub images)
├── .env.example
└── README.md
```

---

## 📝 Task Breakdown

---

### ✅ TASK 1 — Build the Backend API

**Goal:** Create a Node.js/Express REST API for task management.

#### 1.1 Initialize the project

```bash
mkdir taskmanager && cd taskmanager
mkdir backend && cd backend
npm init -y
npm install express pg cors dotenv
npm install --save-dev nodemon
```

#### 1.2 `backend/src/db/pool.js` — Database connection

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

// Create table if it doesn't exist
pool.query(`
  CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    completed BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW()
  )
`).catch(console.error);

module.exports = pool;
```

#### 1.3 `backend/src/routes/tasks.js` — Task routes

```javascript
const express = require('express');
const router = express.Router();
const pool = require('../db/pool');

// GET all tasks
router.get('/', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT * FROM tasks ORDER BY created_at DESC'
    );
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// POST create task
router.post('/', async (req, res) => {
  const { title } = req.body;
  if (!title) return res.status(400).json({ error: 'Title is required' });
  try {
    const result = await pool.query(
      'INSERT INTO tasks (title) VALUES ($1) RETURNING *',
      [title]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// PATCH toggle complete
router.patch('/:id/toggle', async (req, res) => {
  try {
    const result = await pool.query(
      'UPDATE tasks SET completed = NOT completed WHERE id = $1 RETURNING *',
      [req.params.id]
    );
    if (result.rows.length === 0)
      return res.status(404).json({ error: 'Task not found' });
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// DELETE task
router.delete('/:id', async (req, res) => {
  try {
    await pool.query('DELETE FROM tasks WHERE id = $1', [req.params.id]);
    res.json({ message: 'Task deleted' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

module.exports = router;
```

#### 1.4 `backend/src/index.js` — Main server

```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const taskRoutes = require('./routes/tasks');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

app.use('/api/tasks', taskRoutes);

app.listen(PORT, () => {
  console.log(`🚀 API running on port ${PORT}`);
});
```

#### 1.5 `backend/package.json` scripts

```json
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js"
  }
}
```

#### 1.6 `backend/Dockerfile`

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

EXPOSE 3000

CMD ["node", "src/index.js"]
```

#### 1.7 `backend/.dockerignore`

```
node_modules
.env
*.log
```

---

### ✅ TASK 2 — Build the React Frontend

**Goal:** Create a React app that consumes the API.

#### 2.1 Initialize React project

```bash
cd ..
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install axios
```

#### 2.2 `frontend/src/App.jsx`

```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

const API = import.meta.env.VITE_API_URL || 'http://localhost:3000';

function App() {
  const [tasks, setTasks] = useState([]);
  const [title, setTitle] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchTasks();
  }, []);

  const fetchTasks = async () => {
    const res = await axios.get(`${API}/api/tasks`);
    setTasks(res.data);
    setLoading(false);
  };

  const addTask = async (e) => {
    e.preventDefault();
    if (!title.trim()) return;
    const res = await axios.post(`${API}/api/tasks`, { title });
    setTasks([res.data, ...tasks]);
    setTitle('');
  };

  const toggleTask = async (id) => {
    const res = await axios.patch(`${API}/api/tasks/${id}/toggle`);
    setTasks(tasks.map(t => t.id === id ? res.data : t));
  };

  const deleteTask = async (id) => {
    await axios.delete(`${API}/api/tasks/${id}`);
    setTasks(tasks.filter(t => t.id !== id));
  };

  return (
    <div style={{ maxWidth: 600, margin: '40px auto', padding: '0 20px' }}>
      <h1>📝 Task Manager</h1>

      <form onSubmit={addTask} style={{ display: 'flex', gap: 8, marginBottom: 24 }}>
        <input
          value={title}
          onChange={e => setTitle(e.target.value)}
          placeholder="Add a new task..."
          style={{ flex: 1, padding: '8px 12px', fontSize: 16 }}
        />
        <button type="submit" style={{ padding: '8px 16px' }}>Add</button>
      </form>

      {loading ? (
        <p>Loading...</p>
      ) : tasks.length === 0 ? (
        <p>No tasks yet. Add one above!</p>
      ) : (
        <ul style={{ listStyle: 'none', padding: 0 }}>
          {tasks.map(task => (
            <li
              key={task.id}
              style={{
                display: 'flex',
                alignItems: 'center',
                gap: 12,
                padding: '12px 0',
                borderBottom: '1px solid #eee',
              }}
            >
              <input
                type="checkbox"
                checked={task.completed}
                onChange={() => toggleTask(task.id)}
              />
              <span style={{
                flex: 1,
                textDecoration: task.completed ? 'line-through' : 'none',
                color: task.completed ? '#999' : '#000',
              }}>
                {task.title}
              </span>
              <button onClick={() => deleteTask(task.id)}>🗑</button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default App;
```

#### 2.3 `frontend/nginx.conf` — Nginx config for production

```nginx
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;   # SPA routing support
    }

    location /api/ {
        proxy_pass http://backend:3000/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        proxy_pass http://backend:3000/health;
    }
}
```

> This nginx config does two things: serves your React app AND proxies `/api/` requests to the backend container. This way your frontend and backend are both accessible on port 80.

#### 2.4 `frontend/Dockerfile` — Production multi-stage

```dockerfile
# Stage 1: Build React app
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve with nginx
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### 2.5 `frontend/.dockerignore`

```
node_modules
.env
dist
```

---

### ✅ TASK 3 — Docker Compose Setup

#### 3.1 `docker-compose.yml` — Development (with hot reload)

```yaml
version: '3.9'

services:

  db:
    image: postgres:16-alpine
    container_name: taskmanager-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER:-admin}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
      POSTGRES_DB: ${DB_NAME:-taskmanager}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - taskmanager-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-admin}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
    container_name: taskmanager-backend
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://${DB_USER:-admin}:${DB_PASSWORD:-secret}@db:5432/${DB_NAME:-taskmanager}
      PORT: 3000
      NODE_ENV: development
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      db:
        condition: service_healthy
    networks:
      - taskmanager-net

  frontend:
    build:
      context: ./frontend
    container_name: taskmanager-frontend
    restart: unless-stopped
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    networks:
      - taskmanager-net

volumes:
  postgres-data:

networks:
  taskmanager-net:
    driver: bridge
```

#### 3.2 `docker-compose.prod.yml` — Production (uses Docker Hub images)

```yaml
version: '3.9'

services:

  db:
    image: postgres:16-alpine
    container_name: taskmanager-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - taskmanager-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: ${DOCKERHUB_USERNAME}/taskmanager-backend:${IMAGE_TAG:-latest}
    container_name: taskmanager-backend
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      PORT: 3000
      NODE_ENV: production
    depends_on:
      db:
        condition: service_healthy
    networks:
      - taskmanager-net

  frontend:
    image: ${DOCKERHUB_USERNAME}/taskmanager-frontend:${IMAGE_TAG:-latest}
    container_name: taskmanager-frontend
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - taskmanager-net

volumes:
  postgres-data:

networks:
  taskmanager-net:
    driver: bridge
```

#### 3.3 `.env.example`

```bash
# Copy this to .env and fill in real values
DB_USER=admin
DB_PASSWORD=changeme
DB_NAME=taskmanager
DOCKERHUB_USERNAME=yourdockerhubusername
IMAGE_TAG=latest
```

> ⚠️ Add `.env` to your `.gitignore`! Never commit real secrets to GitHub.

---

### ✅ TASK 4 — GitHub Actions CI/CD Pipeline

**Goal:** On every push to `main`, automatically build images, push to Docker Hub, and deploy to EC2.

#### 4.1 Create `.github/workflows/deploy.yml`

```yaml
name: CI/CD — Build, Push & Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: docker.io
  BACKEND_IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/taskmanager-backend
  FRONTEND_IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/taskmanager-frontend

jobs:

  # ─── JOB 1: Test ──────────────────────────────────────────────────────────
  test:
    name: Run Tests
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Install backend dependencies
        working-directory: backend
        run: npm ci

      - name: Run backend tests
        working-directory: backend
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          NODE_ENV: test
        run: npm test --if-present

  # ─── JOB 2: Build & Push ──────────────────────────────────────────────────
  build-and-push:
    name: Build & Push Docker Images
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    outputs:
      image-tag: ${{ steps.meta.outputs.version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Generate image metadata (tag = git short SHA)
        id: meta
        run: echo "version=${GITHUB_SHA::8}" >> $GITHUB_OUTPUT

      - name: Build & Push Backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: |
            ${{ env.BACKEND_IMAGE }}:${{ steps.meta.outputs.version }}
            ${{ env.BACKEND_IMAGE }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Build & Push Frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: |
            ${{ env.FRONTEND_IMAGE }}:${{ steps.meta.outputs.version }}
            ${{ env.FRONTEND_IMAGE }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─── JOB 3: Deploy to EC2 ─────────────────────────────────────────────────
  deploy:
    name: Deploy to AWS EC2
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via SSH to EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            set -e

            echo "📦 Navigating to app directory..."
            mkdir -p ~/taskmanager
            cd ~/taskmanager

            echo "📄 Writing docker-compose.prod.yml..."
            cat > docker-compose.prod.yml << 'COMPOSE_EOF'
            ${{ needs.build-and-push.result == 'success' && '' }}
            COMPOSE_EOF

            # Copy compose file from repo via scp alternative
            echo "🔑 Logging in to Docker Hub..."
            echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

            echo "⬇️  Pulling latest images..."
            IMAGE_TAG=${{ needs.build-and-push.outputs.image-tag }} \
            DOCKERHUB_USERNAME=${{ secrets.DOCKERHUB_USERNAME }} \
            docker compose -f docker-compose.prod.yml pull

            echo "🚀 Starting services..."
            IMAGE_TAG=${{ needs.build-and-push.outputs.image-tag }} \
            DOCKERHUB_USERNAME=${{ secrets.DOCKERHUB_USERNAME }} \
            DB_USER=${{ secrets.DB_USER }} \
            DB_PASSWORD=${{ secrets.DB_PASSWORD }} \
            DB_NAME=${{ secrets.DB_NAME }} \
            docker compose -f docker-compose.prod.yml up -d --remove-orphans

            echo "🧹 Cleaning up old images..."
            docker image prune -f

            echo "✅ Deployment complete!"
            docker compose -f docker-compose.prod.yml ps
```

> **Better deploy approach** — copy the compose file via SCP before SSH:

Replace the Deploy step with this cleaner version:

```yaml
      - name: Copy compose file to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          source: docker-compose.prod.yml
          target: ~/taskmanager/

      - name: SSH and deploy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            set -e
            cd ~/taskmanager

            echo "${{ secrets.DOCKERHUB_TOKEN }}" | \
              docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

            export IMAGE_TAG=${{ needs.build-and-push.outputs.image-tag }}
            export DOCKERHUB_USERNAME=${{ secrets.DOCKERHUB_USERNAME }}
            export DB_USER=${{ secrets.DB_USER }}
            export DB_PASSWORD=${{ secrets.DB_PASSWORD }}
            export DB_NAME=${{ secrets.DB_NAME }}

            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d --remove-orphans
            docker image prune -f

            echo "✅ Done! Running containers:"
            docker compose -f docker-compose.prod.yml ps
```

---

### ✅ TASK 5 — AWS EC2 Setup

**Goal:** Launch and configure an EC2 instance to host the application.

#### 5.1 Launch EC2 Instance

1. Go to **AWS Console** → EC2 → **Launch Instance**
2. Configure:

|Setting|Value|
|---|---|
|Name|`taskmanager-server`|
|AMI|Ubuntu Server 22.04 LTS|
|Instance type|`t2.micro` (free tier) or `t3.small`|
|Key pair|Create new → download `.pem` file → **save safely**|
|Storage|20 GB gp3|

3. **Security Group** — allow these inbound rules:

|Type|Protocol|Port|Source|
|---|---|---|---|
|SSH|TCP|22|Your IP (or 0.0.0.0/0 for dev)|
|HTTP|TCP|80|0.0.0.0/0|
|Custom TCP|TCP|3000|0.0.0.0/0|

4. Launch the instance. Note your **Public IPv4 address**.

---

#### 5.2 Connect to EC2 and Install Docker

```bash
# Connect from your machine
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Once inside the EC2 terminal:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sh

# Add ubuntu user to docker group (no sudo needed)
sudo usermod -aG docker ubuntu

# Apply group change (or re-login)
newgrp docker

# Install Docker Compose plugin
sudo apt install docker-compose-plugin -y

# Verify
docker --version
docker compose version
```

---

#### 5.3 Configure GitHub Secrets

Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add all of these:

|Secret Name|Value|Where to get it|
|---|---|---|
|`DOCKERHUB_USERNAME`|Your Docker Hub username|hub.docker.com|
|`DOCKERHUB_TOKEN`|Docker Hub access token|hub.docker.com → Account Settings → Security → Access Tokens|
|`EC2_HOST`|EC2 public IP address|AWS Console → EC2 → Instances|
|`EC2_SSH_KEY`|Contents of your `.pem` file|`cat your-key.pem` → copy everything|
|`DB_USER`|`admin`|Your choice|
|`DB_PASSWORD`|`yourStrongPassword123`|Your choice (keep it strong!)|
|`DB_NAME`|`taskmanager`|Your choice|

> 🔑 **Getting your Docker Hub token:**  
> hub.docker.com → click your profile → Account Settings → Security → New Access Token → give it "Read, Write, Delete" access → copy the token immediately (shown only once)

> 🔑 **Getting your SSH key for GitHub:**
> 
> ```bash
> cat your-key.pem   # Copy the ENTIRE output including -----BEGIN----- and -----END-----
> ```

---

### ✅ TASK 6 — Test the Full Pipeline

#### 6.1 Push code and watch the magic

```bash
# Initialize git and push to GitHub
git init
git add .
git commit -m "feat: initial full-stack app with Docker"
git remote add origin https://github.com/yourusername/taskmanager.git
git push -u origin main
```

#### 6.2 Watch the pipeline run

1. Go to your GitHub repo → **Actions** tab
2. You'll see the workflow running with 3 jobs: `Test → Build & Push → Deploy`
3. Click into each job to see real-time logs

#### 6.3 Verify on EC2

```bash
# SSH into EC2
ssh -i your-key.pem ubuntu@YOUR_EC2_IP

# Check running containers
docker ps

# Check logs
docker logs taskmanager-backend
docker logs taskmanager-frontend

# Check compose status
cd ~/taskmanager
docker compose -f docker-compose.prod.yml ps
```

#### 6.4 Open in browser

```
http://YOUR_EC2_PUBLIC_IP        ← React frontend (port 80)
http://YOUR_EC2_PUBLIC_IP/api/tasks  ← API via nginx proxy
http://YOUR_EC2_PUBLIC_IP:3000/health  ← Backend health check
```

---

### ✅ TASK 7 — Make a Change and Re-Deploy

This is the real test of your CI/CD pipeline.

```bash
# Make a small change
# e.g., change the h1 in App.jsx to "📝 My Task Manager v2"
# Then push:

git add .
git commit -m "feat: update app title to v2"
git push
```

Watch GitHub Actions → automatically runs tests → builds new images with new SHA tag → pushes to Docker Hub → SSHes into EC2 → pulls new images → restarts containers.

**Your site updates automatically. Zero manual steps. ✅**

---

## 🔐 Security Checklist

Before considering this production-ready:

- [ ] Never commit `.env` to git (it's in `.gitignore`)
- [ ] Use strong, unique passwords for `DB_PASSWORD`
- [ ] EC2 Security Group: restrict SSH (port 22) to your IP only, not `0.0.0.0/0`
- [ ] Set up a domain name + SSL (HTTPS) with Let's Encrypt / Certbot
- [ ] Enable EC2 automatic security updates: `sudo apt install unattended-upgrades -y`
- [ ] Use AWS IAM roles instead of root account
- [ ] Rotate Docker Hub access tokens periodically
- [ ] Never store secrets in `docker-compose.yml` — always use environment variables

---

## 🐛 Troubleshooting

### Pipeline fails at "Test" job

```
# Check: does your backend have a test script?
# In backend/package.json, either add tests or this won't fail the pipeline:
"scripts": {
  "test": "echo 'No tests yet' && exit 0"
}
```

### Pipeline fails at "Deploy" — SSH connection refused

```bash
# Check EC2 Security Group allows port 22 from GitHub Actions IPs
# Or allow from 0.0.0.0/0 temporarily for debugging

# Verify your EC2_SSH_KEY secret includes the full pem content:
# -----BEGIN RSA PRIVATE KEY-----
# ...all lines...
# -----END RSA PRIVATE KEY-----
```

### Containers not starting on EC2

```bash
# SSH into EC2 and check logs
docker logs taskmanager-backend
docker logs taskmanager-frontend

# Common cause: DB not ready yet — the healthcheck handles this
# but you can also check:
docker logs taskmanager-db
```

### Frontend can't reach API

```bash
# Check nginx.conf proxy_pass points to correct service name
# Check both containers are on same docker network
docker network inspect taskmanager_taskmanager-net
```

### Old containers still running after deploy

```bash
# SSH into EC2
cd ~/taskmanager
docker compose -f docker-compose.prod.yml down
docker compose -f docker-compose.prod.yml up -d
```

---

## 📊 Pipeline Summary

```
git push origin main
        │
        ▼
┌───────────────────┐
│   JOB 1: Test     │  ~1 min
│   npm test        │
│   (with real PG)  │
└────────┬──────────┘
         │ pass
         ▼
┌───────────────────┐
│ JOB 2: Build&Push │  ~3-5 min
│ docker build      │
│ docker push       │  → Docker Hub: yourusername/taskmanager-backend:a1b2c3d4
│ (backend+frontend)│  → Docker Hub: yourusername/taskmanager-frontend:a1b2c3d4
└────────┬──────────┘
         │ success
         ▼
┌───────────────────┐
│ JOB 3: Deploy     │  ~1-2 min
│ scp compose file  │
│ ssh into EC2      │
│ docker pull       │
│ docker compose up │
└───────────────────┘
         │
         ▼
  🌐 Live at http://your-ec2-ip
  Total time: ~5-8 minutes from push to live
```

---

## 🎯 Bonus Challenges

Once you complete the main task, try these:

|Challenge|Description|
|---|---|
|**Add HTTPS**|Install Certbot + Let's Encrypt SSL on EC2 with a real domain|
|**Add a staging environment**|Deploy to a separate EC2 on push to `develop` branch|
|**Database migrations**|Add `db-migrate` or `knex` migrations that run on deploy|
|**Health check endpoint**|Add a `/health` check to the GitHub Actions deploy step to verify deployment succeeded|
|**Slack/Discord notifications**|Post to a channel when deployment succeeds or fails|
|**Rollback command**|Create a GitHub Actions workflow you can trigger manually to roll back to a previous image tag|
|**AWS ECR**|Replace Docker Hub with Amazon Elastic Container Registry|

---

## ✅ Completion Checklist

- [ ] Backend API working locally (`/health`, `/api/tasks` CRUD)
- [ ] Frontend React app connected to API
- [ ] Both Dockerized and running with `docker compose up`
- [ ] Images pushed to Docker Hub manually (before automating)
- [ ] EC2 instance launched and Docker installed
- [ ] All GitHub Secrets configured
- [ ] GitHub Actions workflow file created
- [ ] First push triggers the full pipeline successfully
- [ ] App accessible at `http://YOUR_EC2_IP`
- [ ] Made a code change, pushed, and verified auto-deploy worked
- [ ] Verified data persists in PostgreSQL after container restart

---

_You now have a production-grade CI/CD pipeline. Every code change ships automatically. 🚀_