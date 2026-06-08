
**Role:** Mid-Level Full-Stack Developer **Project:** Multi-Instance CMS Platform (Next.js) **Estimated Time:** 35–40 hours across 7–10 days **Submission:** GitHub repository link with a README

---
## Overview

You will build and **deploy a production-ready mini CMS platform** to AWS. This is not a local demo — the final submission must be live, accessible via a public URL, and running on real infrastructure.

This task covers the full software delivery lifecycle: database design, API development, frontend, media storage on S3, containerisation with Docker, automated CI/CD via GitHub Actions, and deployment to an EC2 instance.

> We use these exact tools in production. This task mirrors real work you will do on the job.

---

## Time Allocation Guide

|Part|Area|Hours|
|---|---|---|
|1|Database design & schema|3.5 hrs|
|2|Authentication & RBAC|2.5 hrs|
|3|Backend API|6 hrs|
|4|Frontend — admin dashboard|6 hrs|
|5|Media uploads (S3)|2 hrs|
|6|Public-facing frontend|2 hrs|
|7|Docker & containerisation|2.5 hrs|
|8|CI/CD with GitHub Actions|3.5 hrs|
|9|EC2 deployment & production setup|4 hrs|
|10|Testing|3 hrs|
|11|Documentation & README|1.5 hrs|
|**Total**||**~37 hrs**|

---

## Part 1 — Database Design
Design and implement the full database schema using **PostgreSQL** and **Prisma** (or Drizzle).
### Tables Required
---
#### `tenants`

Individual CMS instances — one per client or site.

| Column       | Type           | Constraints                                  |
| ------------ | -------------- | -------------------------------------------- |
| `id`         | `uuid`         | Primary key, default `gen_random_uuid()`     |
| `name`       | `varchar(100)` | Not null                                     |
| `slug`       | `varchar(100)` | Not null, globally unique                    |
| `plan`       | `enum`         | `FREE`, `PRO`, `ENTERPRISE` — default `FREE` |
| `is_active`  | `boolean`      | Default `true`                               |
| `created_at` | `timestamp`    | Default `now()`                              |
|              |                |                                              |

---
#### `users`

Users belong to a tenant and carry a role.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`tenant_id`|`uuid`|FK → `tenants.id`, not null|
|`email`|`varchar(255)`|Not null, globally unique|
|`password_hash`|`text`|Not null|
|`full_name`|`varchar(150)`|Nullable|
|`role`|`enum`|`ADMIN`, `EDITOR`, `VIEWER`|
|`is_active`|`boolean`|Default `true`|
|`last_login_at`|`timestamp`|Nullable|
|`created_at`|`timestamp`|Default `now()`|

---
#### `content_types`
Custom content models defined per tenant.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`tenant_id`|`uuid`|FK → `tenants.id`, not null|
|`name`|`varchar(100)`|Not null|
|`slug`|`varchar(100)`|Not null|
|`description`|`text`|Nullable|
|`is_active`|`boolean`|Default `true`|
|`created_by`|`uuid`|FK → `users.id`|
|`created_at`|`timestamp`|Default `now()`|
|`updated_at`|`timestamp`|Auto-updated|

> **Unique constraint:** `(tenant_id, slug)`

---
#### `fields`

Ordered, typed fields belonging to a content type.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`content_type_id`|`uuid`|FK → `content_types.id`, cascade delete|
|`name`|`varchar(100)`|Not null|
|`slug`|`varchar(100)`|Not null|
|`field_type`|`enum`|`TEXT`, `RICH_TEXT`, `NUMBER`, `BOOLEAN`, `DATE`, `MEDIA`, `RELATION`, `SELECT`|
|`required`|`boolean`|Default `false`|
|`sort_order`|`integer`|Not null|
|`config`|`jsonb`|Optional — e.g. `{ "options": ["red","blue"] }` for SELECT|

> **Unique constraint:** `(content_type_id, slug)`

---
#### `entries`

Content entries created against a content type.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`tenant_id`|`uuid`|FK → `tenants.id`|
|`content_type_id`|`uuid`|FK → `content_types.id`|
|`slug`|`varchar(200)`|Not null|
|`status`|`enum`|`DRAFT`, `PUBLISHED`, `SCHEDULED`, `ARCHIVED` — default `DRAFT`|
|`published_at`|`timestamp`|Nullable|
|`scheduled_for`|`timestamp`|Nullable|
|`created_by`|`uuid`|FK → `users.id`|
|`updated_by`|`uuid`|FK → `users.id`|
|`created_at`|`timestamp`|Default `now()`|
|`updated_at`|`timestamp`|Auto-updated|

> **Unique constraint:** `(tenant_id, content_type_id, slug)`

---

#### `entry_field_values`

EAV (Entity-Attribute-Value) table for flexible field storage.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`entry_id`|`uuid`|FK → `entries.id`, cascade delete|
|`field_id`|`uuid`|FK → `fields.id`|
|`value_text`|`text`|Nullable|
|`value_number`|`numeric`|Nullable|
|`value_boolean`|`boolean`|Nullable|
|`value_date`|`timestamp`|Nullable|
|`value_json`|`jsonb`|Nullable — for RICH_TEXT, MEDIA, SELECT, RELATION|

> Only one value column should be populated per row, determined by the field's `field_type`.

---

#### `media`

Uploaded files stored in S3, scoped to a tenant.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`tenant_id`|`uuid`|FK → `tenants.id`|
|`filename`|`varchar(255)`|Not null — stored S3 key|
|`original_name`|`varchar(255)`|Not null — original upload name|
|`mime_type`|`varchar(100)`|Not null|
|`size_bytes`|`integer`|Not null|
|`s3_bucket`|`varchar(255)`|Not null|
|`s3_key`|`text`|Not null — full S3 object key|
|`url`|`text`|Public CloudFront or S3 URL|
|`alt_text`|`text`|Nullable|
|`uploaded_by`|`uuid`|FK → `users.id`|
|`created_at`|`timestamp`|Default `now()`|

---

#### `activity_logs`

Audit trail for all significant actions.

|Column|Type|Constraints|
|---|---|---|
|`id`|`uuid`|Primary key|
|`tenant_id`|`uuid`|FK → `tenants.id`|
|`user_id`|`uuid`|FK → `users.id`, nullable (system actions)|
|`action`|`varchar(100)`|e.g. `entry.published`, `media.deleted`, `content_type.created`|
|`resource_type`|`varchar(50)`|e.g. `entry`, `media`, `content_type`|
|`resource_id`|`uuid`|Nullable|
|`metadata`|`jsonb`|Nullable — extra context (old/new status, filename, etc.)|
|`created_at`|`timestamp`|Default `now()`|

---

### Relationship Diagram

```
tenants
  ├── users              (tenant_id)
  ├── content_types      (tenant_id)
  │     └── fields       (content_type_id)
  ├── entries            (tenant_id, content_type_id)
  │     └── entry_field_values  (entry_id, field_id → fields)
  ├── media              (tenant_id)
  └── activity_logs      (tenant_id, user_id)
```

---

### Deliverables

- `schema.prisma` with all tables, enums, relations, and constraints
- All migration files committed (`prisma/migrations/`)
- Seed script `prisma/seed.ts` creating:
    - 2 tenants (`acme-corp`, `globex`)
    - 2 users per tenant (1 ADMIN, 1 EDITOR) with known passwords
    - 2 content types per tenant with 4–5 mixed-type fields each
    - 5+ entries per content type in mixed statuses
    - 3+ media records per tenant (can be placeholder S3 URLs)

---

## Part 2 — Authentication & Role-Based Access Control

### Authentication

- JWT-based auth via **NextAuth.js** (credentials provider) or manual JWT
- Token payload must include: `userId`, `tenantId`, `role`, `exp`
- Refresh token strategy is a bonus
- Passwords stored as **bcrypt** hashes (min cost factor 12)
- Rate-limit login attempts — max 5 failures per IP per 15 minutes (use `rate-limiter-flexible` or equivalent)

### Role Permissions Matrix

|Action|ADMIN|EDITOR|VIEWER|
|---|:-:|:-:|:-:|
|View content types|✅|✅|✅|
|Create / edit / delete content types|✅|❌|❌|
|Create / edit entries|✅|✅|❌|
|Publish / archive entries|✅|✅|❌|
|Delete entries|✅|❌|❌|
|Upload media|✅|✅|❌|
|Delete media|✅|❌|❌|
|View activity logs|✅|❌|❌|
|Invite / deactivate users|✅|❌|❌|

### Middleware

- `middleware.ts` protects all `/dashboard/*` routes
- Unauthorised requests redirect to `/login`
- Decoded token attached to request headers for downstream route handlers
- CSRF protection on all state-mutating routes

---

## Part 3 — Backend API Routes

All routes under `/app/api/`. All responses follow this envelope:

**Success:**

```json
{ "data": { ... }, "meta": { "total": 10, "page": 1, "limit": 20 } }
```

**Error:**

```json
{ "error": "Human-readable message", "code": "SLUG_CONFLICT", "statusCode": 409 }
```

All write operations must append a record to `activity_logs`.

---

### Auth

#### `POST /api/auth/login`

- Accepts `{ email, password }`
- Returns signed JWT + user details
- Updates `last_login_at`
- Returns `401` on bad credentials, `429` when rate-limited

---

### Content Types

|Method|Route|Role|Notes|
|---|---|---|---|
|GET|`/api/content-types`|Any|Returns list with `fieldCount`|
|POST|`/api/content-types`|ADMIN|Creates type + fields in one transaction|
|GET|`/api/content-types/[id]`|Any|Returns type with full field list|
|PATCH|`/api/content-types/[id]`|ADMIN|Update name, description, add new fields|
|DELETE|`/api/content-types/[id]`|ADMIN|Cascades to fields and entries|

---

### Entries

|Method|Route|Role|Notes|
|---|---|---|---|
|GET|`/api/content-types/[id]/entries`|Any|Supports `?status=&page=&limit=&search=`|
|POST|`/api/content-types/[id]/entries`|EDITOR+|Validates required fields; default DRAFT|
|GET|`/api/content-types/[id]/entries/[entryId]`|Any|Full field values|
|PATCH|`/api/content-types/[id]/entries/[entryId]`|EDITOR+|Status transitions; sets `published_at` on PUBLISHED|
|DELETE|`/api/content-types/[id]/entries/[entryId]`|ADMIN|Hard delete|

**Status transition rules:**

- `DRAFT` → `PUBLISHED`: set `published_at = now()`
- `DRAFT` → `SCHEDULED`: require `scheduled_for` in body (must be future date)
- `SCHEDULED` → `PUBLISHED`: allowed manually or via cron job
- `PUBLISHED` → `ARCHIVED`: allowed
- `ARCHIVED` → `DRAFT`: allowed (re-open)
- Any other transition: return `422 INVALID_TRANSITION`

---

### Media

|Method|Route|Role|Notes|
|---|---|---|---|
|POST|`/api/media/upload`|EDITOR+|Multipart upload → S3|
|GET|`/api/media`|Any|Paginated list for tenant|
|GET|`/api/media/[id]`|Any|Single media record|
|PATCH|`/api/media/[id]`|EDITOR+|Update `alt_text` only|
|DELETE|`/api/media/[id]`|ADMIN|Delete from S3 + database|

**Upload rules:**

- Accepted MIME types: `image/jpeg`, `image/png`, `image/webp`, `image/gif`, `application/pdf`
- Max file size: **10 MB**
- S3 key pattern: `uploads/{tenantId}/{year}/{month}/{uuid}-{sanitised-filename}`
- Return `400` with clear message for type or size violations

---

### Users

|Method|Route|Role|Notes|
|---|---|---|---|
|GET|`/api/users`|ADMIN|List users in tenant|
|POST|`/api/users/invite`|ADMIN|Create inactive user with temp password|
|PATCH|`/api/users/[id]`|ADMIN|Update role or `is_active`|

---

### Activity Logs

|Method|Route|Role|Notes|
|---|---|---|---|
|GET|`/api/activity-logs`|ADMIN|Paginated; supports `?resource_type=&action=`|

---

### Scheduled Publish Cron

#### `GET /api/cron/publish-scheduled`

- Protected by a secret header: `x-cron-secret: [CRON_SECRET env var]`
- Finds all entries with `status = SCHEDULED` and `scheduled_for <= now()`
- Updates them to `PUBLISHED`, sets `published_at`
- Logs each transition to `activity_logs`
- Intended to be called by a system cron job every minute (set up on EC2)

---

## Part 4 — Frontend: Admin Dashboard

Build with **Next.js App Router**, **React**, **TypeScript**, and **Tailwind CSS** or **shadcn/ui**.

### Pages

|Route|Description|
|---|---|
|`/login`|Email + password login, redirect on success|
|`/dashboard`|Summary cards + recent entries + recent activity feed|
|`/dashboard/content-types`|Table list; create button (ADMIN only)|
|`/dashboard/content-types/new`|Create form with dynamic field builder|
|`/dashboard/content-types/[id]`|Detail view — fields list + edit option|
|`/dashboard/content-types/[id]/entries`|Filterable entries table|
|`/dashboard/content-types/[id]/entries/new`|Dynamic entry creation form|
|`/dashboard/content-types/[id]/entries/[entryId]/edit`|Dynamic entry edit form|
|`/dashboard/media`|Media library grid with upload|
|`/dashboard/users`|User list; invite and deactivate (ADMIN only)|
|`/dashboard/activity-logs`|Paginated activity log (ADMIN only)|

### UI Requirements

- Fully responsive — desktop sidebar collapses to bottom nav on mobile
- Loading skeletons during data fetches (not just spinners)
- Empty states with helpful copy for all lists
- Toast notifications for all success and error actions
- Confirmation dialogs for all destructive actions (delete, deactivate)
- Role-aware rendering — hide or disable actions the user is not permitted to perform
- Status badges using distinct colours: DRAFT (gray), PUBLISHED (green), SCHEDULED (amber), ARCHIVED (red)

### Dynamic Entry Form

The entry form must render fields dynamically based on the content type definition:

|Field type|UI element|
|---|---|
|`TEXT`|`<input type="text">`|
|`RICH_TEXT`|Textarea or a basic rich text editor|
|`NUMBER`|`<input type="number">` with min/max from `config`|
|`BOOLEAN`|Toggle switch|
|`DATE`|`<input type="datetime-local">`|
|`SELECT`|Dropdown using options from field `config.options`|
|`MEDIA`|Button that opens media library modal; shows preview thumbnail|
|`RELATION`|Searchable dropdown of entries from the related content type|

---

## Part 5 — Media Uploads to AWS S3

### Configuration

All S3 settings must come from environment variables:

```env
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
S3_BUCKET_NAME=your-cms-media-bucket
S3_PUBLIC_URL_PREFIX=https://your-bucket.s3.amazonaws.com
```

### S3 Bucket Setup (document in README)

- Bucket must have **public read** on the `uploads/` prefix (or use CloudFront)
- Enable **versioning** on the bucket
- Configure **CORS** to allow uploads from your EC2 domain
- Set a **lifecycle rule** to move objects older than 180 days to S3 Glacier Instant Retrieval

### Upload Flow

1. Client sends file to `POST /api/media/upload`
2. API validates MIME type and file size
3. API streams file directly to S3 using `@aws-sdk/client-s3` (`PutObjectCommand`)
4. API writes a record to the `media` table with the S3 key and public URL
5. Return the created media record to the client

### Delete Flow

1. `DELETE /api/media/[id]` is called
2. API calls `DeleteObjectCommand` on S3
3. API deletes the database record
4. Log action to `activity_logs`

---

## Part 6 — Public-Facing Frontend

Server-rendered pages for published content, accessible without authentication.

### Routes

#### `GET /sites/[tenantSlug]`

- Lists all published entries grouped by content type
- Shows entry name (from a `title` or `name` field if present, else slug), content type, and published date
- Returns `404` if tenant does not exist or `is_active = false`

#### `GET /sites/[tenantSlug]/[contentTypeSlug]/[entrySlug]`

- Renders a single published entry with all field values
- Field rendering by type: text as paragraph, number as formatted value, boolean as Yes/No, date as formatted string, media as `<img>` with alt text, rich text as sanitised HTML
- Returns `404` if entry not found, not `PUBLISHED`, or tenant mismatch
- Include basic Open Graph meta tags (`og:title`, `og:description`)

### Requirements

- Use Next.js **Server Components** — no client-side fetching
- No authentication required
- Sanitise rich text output before rendering (use `DOMPurify` or `sanitize-html`)
- Basic, readable layout — not heavily styled, but not unstyled either

---

## Part 7 — Docker & Containerisation

### `Dockerfile` (multi-stage build)

```dockerfile
# Stage 1 — deps
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2 — builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

# Stage 3 — runner
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["npm", "start"]
```

### `docker-compose.yml` (for local development)

```yaml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: cms
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: cmsdb
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Requirements

- `.dockerignore` must exclude: `.git`, `node_modules`, `.env`, `.next` (non-standalone)
- Image must build successfully with `docker build -t cms-app .`
- App must start successfully with `docker compose up`
- Keep final image under **300 MB** — use alpine base and multi-stage build

---

## Part 8 — CI/CD with GitHub Actions

Create the following workflows under `.github/workflows/`.

---

### `ci.yml` — runs on every push and pull request to `main`

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: cms
          POSTGRES_PASSWORD: secret
          POSTGRES_DB: cmsdb_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run Prisma migrations (test DB)
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://cms:secret@localhost:5432/cmsdb_test

      - name: Run tests
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://cms:secret@localhost:5432/cmsdb_test
          JWT_SECRET: test-secret
          NODE_ENV: test

      - name: Type check
        run: npx tsc --noEmit

      - name: Lint
        run: npm run lint
```

---

### `deploy.yml` — runs only on push to `main` after CI passes

```yaml
name: Deploy to EC2

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: []

    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t cms-app:${{ github.sha }} .

      - name: Log in to Docker Hub (or ECR)
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Push image
        run: |
          docker tag cms-app:${{ github.sha }} ${{ secrets.DOCKER_USERNAME }}/cms-app:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/cms-app:latest

      - name: SSH into EC2 and deploy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/cms
            docker compose pull
            docker compose up -d --remove-orphans
            docker image prune -f
            docker exec cms-app npx prisma migrate deploy
```

### Required GitHub Secrets

Document all secrets needed in your README:

|Secret|Description|
|---|---|
|`EC2_HOST`|Public IP or domain of your EC2 instance|
|`EC2_SSH_KEY`|Private SSH key (PEM format) for EC2 access|
|`DOCKER_USERNAME`|Docker Hub username|
|`DOCKER_PASSWORD`|Docker Hub password or access token|
|`DATABASE_URL`|Production Postgres connection string|
|`JWT_SECRET`|Random 64-character string for signing JWTs|
|`S3_BUCKET_NAME`|Production S3 bucket name|
|`AWS_ACCESS_KEY_ID`|AWS IAM user key|
|`AWS_SECRET_ACCESS_KEY`|AWS IAM user secret|
|`CRON_SECRET`|Secret header value for the cron endpoint|
|`NEXTAUTH_SECRET`|Required by NextAuth.js|
|`NEXTAUTH_URL`|Full public URL of the deployed app|

---

## Part 9 — EC2 Deployment & Production Setup

### EC2 Instance Setup

**Recommended instance:** `t3.small` (2 vCPU, 2 GB RAM) — free tier eligible alternatives: `t2.micro` or `t3.micro`

**Operating system:** Ubuntu 22.04 LTS

**Security group inbound rules:**

|Port|Protocol|Source|Purpose|
|---|---|---|---|
|22|TCP|Your IP only|SSH access|
|80|TCP|0.0.0.0/0|HTTP (redirects to HTTPS)|
|443|TCP|0.0.0.0/0|HTTPS|
|5432|TCP|EC2 security group only|PostgreSQL (internal only)|

**Do NOT open port 3000 to the public.** Traffic must pass through Nginx.

---

### Software to Install on EC2

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu

# Install Docker Compose plugin
sudo apt install -y docker-compose-plugin

# Install Nginx
sudo apt install -y nginx

# Install Certbot (SSL)
sudo apt install -y certbot python3-certbot-nginx

# Install PostgreSQL (or run in Docker — your choice)
sudo apt install -y postgresql postgresql-contrib
```

---

### Nginx Configuration

Create `/etc/nginx/sites-available/cms` and symlink to `sites-enabled`:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 15M;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

### SSL Certificate

```bash
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
sudo certbot renew --dry-run  # verify auto-renewal works
```

---

### Environment File on EC2

Create `/home/ubuntu/cms/.env` on the server with all production values. **Never commit `.env` to the repository.**

```env
NODE_ENV=production
DATABASE_URL=postgresql://cms_user:strongpassword@localhost:5432/cmsdb
JWT_SECRET=your-64-char-random-string
NEXTAUTH_SECRET=your-nextauth-secret
NEXTAUTH_URL=https://your-domain.com
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
S3_BUCKET_NAME=your-cms-media-bucket
S3_PUBLIC_URL_PREFIX=https://your-bucket.s3.ap-south-1.amazonaws.com
CRON_SECRET=your-cron-secret
```

---

### Cron Job for Scheduled Publishing

Add to EC2 crontab (`crontab -e`):

```cron
* * * * * curl -s -H "x-cron-secret: your-cron-secret" https://your-domain.com/api/cron/publish-scheduled >> /var/log/cms-cron.log 2>&1
```

---

### PostgreSQL Production Setup

```sql
-- Run as postgres superuser
CREATE USER cms_user WITH PASSWORD 'strongpassword';
CREATE DATABASE cmsdb OWNER cms_user;
GRANT ALL PRIVILEGES ON DATABASE cmsdb TO cms_user;
```

Configure `/etc/postgresql/14/main/postgresql.conf`:

```
max_connections = 100
shared_buffers = 256MB
effective_cache_size = 768MB
log_min_duration_statement = 500   # log slow queries over 500ms
```

---

### Deployment Directory Structure on EC2

```
/home/ubuntu/cms/
  ├── docker-compose.yml   (production compose — pulls from Docker Hub)
  └── .env                 (production environment variables)
```

---

## Part 10 — Testing

Use **Vitest** or **Jest** with **React Testing Library**.

### Required Tests (minimum 15 passing)

#### API — integration tests

|#|Test|
|---|---|
|1|`POST /api/auth/login` — success returns JWT|
|2|`POST /api/auth/login` — wrong password returns 401|
|3|`POST /api/content-types` — ADMIN creates type + fields in one transaction|
|4|`POST /api/content-types` — duplicate slug returns 409|
|5|`POST /api/content-types` — no token returns 401|
|6|`POST /api/content-types` — EDITOR role returns 403|
|7|`DELETE /api/content-types/[id]` — ADMIN deletes successfully|
|8|`DELETE /api/content-types/[id]` — cross-tenant returns 404|
|9|`PATCH /api/.../entries/[id]` — PUBLISHED sets `published_at`|
|10|`PATCH /api/.../entries/[id]` — SCHEDULED without `scheduled_for` returns 422|
|11|`POST /api/media/upload` — rejects file over 10 MB|
|12|`POST /api/media/upload` — rejects invalid MIME type|
|13|`DELETE /api/media/[id]` — deletes from S3 and database (mock S3)|
|14|`GET /api/cron/publish-scheduled` — wrong secret returns 401|
|15|`GET /api/cron/publish-scheduled` — publishes due scheduled entries|

#### Frontend — component tests

|#|Test|
|---|---|
|16|Content types list renders all items from mocked API|
|17|Delete button hidden for EDITOR and VIEWER roles|
|18|Entry form shows `scheduled_for` picker only when SCHEDULED selected|
|19|Entry form shows validation error for missing required field on submit|
|20|Media upload rejects non-image file before hitting the API|

### Coverage Target

Aim for at least **70% line coverage** on API route handlers. Run:

```bash
npm test -- --coverage
```

---

## Part 11 — Documentation Requirements

Your `README.md` must include:

### Setup (local)

- Prerequisites: Node 20+, PostgreSQL 16+, Docker, AWS account
- Step-by-step: clone → install → env setup → DB migrate → seed → run
- Seeded login credentials (email + password for each test user)

### Setup (production)

- EC2 instance type and AMI used
- Step-by-step server provisioning guide
- How to configure GitHub Secrets for the CI/CD pipeline
- How to run the first deployment manually

### Architecture decisions

- Why EAV for entry field values — trade-offs acknowledged
- How tenant isolation is enforced at API level
- How RBAC middleware is structured
- S3 key naming convention and why

### What you would improve

- Honest list of shortcuts taken due to time
- What you would add next (e.g. CDN, Redis cache, queue for scheduled jobs)

### AI assistance disclosure

- Which parts were AI-assisted
- How you reviewed or adapted the output

---

## Submission Requirements Checklist

- [ ] GitHub repository (public or invite `@reviewer-handle`)
- [ ] App live at a public HTTPS URL on EC2
- [ ] `schema.prisma` with all 8 tables, enums, relations, and constraints
- [ ] All migration files committed
- [ ] Seed script fully populates the database
- [ ] All API endpoints implemented, protected, and tenant-scoped
- [ ] All 11 frontend pages functional and role-aware
- [ ] Media upload working end-to-end with real S3
- [ ] Public-facing frontend pages working
- [ ] Docker multi-stage build working
- [ ] `docker-compose.yml` for local development working
- [ ] `ci.yml` — CI pipeline runs and passes on every push
- [ ] `deploy.yml` — auto-deploys to EC2 on merge to `main`
- [ ] Nginx configured as reverse proxy with HTTPS
- [ ] Cron job running on EC2 for scheduled publish
- [ ] Minimum 15 passing tests with 70%+ coverage on API handlers
- [ ] `README.md` covering all sections above

---

## Evaluation Criteria

|Area|Weight|What We Look For|
|---|---|---|
|**Database design**|High|Correct relations, constraints, EAV usage, tenant isolation|
|**API quality**|High|Auth, RBAC, tenant scoping, validation, error handling, transactions|
|**CI/CD pipeline**|High|Both workflows work; secrets handled correctly; deploy is automated|
|**EC2 & production setup**|High|App is live; Nginx + HTTPS configured; no open ports except 80/443/22|
|**S3 integration**|Medium|Real uploads to S3; correct key structure; delete cleans up S3|
|**Code quality**|Medium|Clean TypeScript; logical structure; no business logic in route handlers|
|**Frontend**|Medium|Functional, responsive, role-aware, dynamic form rendering|
|**Testing**|Medium|Meaningful tests; not just coverage padding|
|**Documentation**|Medium|Clear README; honest trade-off analysis; reproducible setup|
|**Editorial workflow**|Medium|Status transitions correct; `published_at` / `scheduled_for` handled|

---

## Tech Stack Reference

|Layer|Required|
|---|---|
|Framework|Next.js 14+ (App Router)|
|Language|TypeScript (strict mode)|
|Database|PostgreSQL 16|
|ORM|Prisma or Drizzle|
|Auth|NextAuth.js or manual JWT + bcrypt|
|Styling|Tailwind CSS or shadcn/ui|
|Testing|Vitest or Jest + React Testing Library|
|File storage|AWS S3 (`@aws-sdk/client-s3`)|
|Containerisation|Docker (multi-stage) + Docker Compose|
|CI/CD|GitHub Actions|
|Web server|Nginx (reverse proxy + SSL termination)|
|SSL|Let's Encrypt via Certbot|
|Hosting|AWS EC2 (Ubuntu 22.04)|
|Cron|System crontab on EC2|

---

## Bonus Challenges (optional)

Attempting one well is better than rushing all of them.

|Bonus|Description|
|---|---|
|**CloudFront CDN**|Put CloudFront in front of S3 for media delivery|
|**Redis cache**|Cache published entry responses with a 60-second TTL using Upstash or self-hosted Redis|
|**SES email**|Send a welcome email when a new user is invited using AWS SES|
|**Terraform**|Provision the EC2 instance, S3 bucket, and security group using Terraform|
|**Staging environment**|Add a `staging` branch that deploys to a separate EC2 instance|
|**GraphQL**|Expose published entries via `/api/graphql`|

---

## Important Notes

- **Security:** Never commit `.env`, AWS keys, SSH keys, or secrets to the repository under any circumstances.
- **Cost:** Use `t3.micro` or `t2.micro` to stay within AWS free tier. Shut down the instance if you finish before the deadline.
- **AI tools:** You may use Cursor, Copilot, Claude, or ChatGPT. Disclose usage in your README. We are evaluating your judgement in reviewing and adapting AI output, not your ability to avoid it.
- **Prioritise working over complete:** A deployed app with 8 of 11 pages beats a local app with all 11. Deployment is a core requirement.
- **Ask questions:** If anything is unclear, ask. Clear communication is part of what we are evaluating.

---

_Shortlisted candidates will be invited for a 60–90 minute technical interview to walk through the submission, discuss architectural decisions, and pair on a small live extension._