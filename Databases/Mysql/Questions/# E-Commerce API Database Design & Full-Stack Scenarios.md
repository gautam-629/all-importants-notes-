
This document provides a complete database design with **API endpoint mappings**, **request/response examples**, and **real-world full-stack implementation scenarios**.

---
## Table of Contents

1. [Database Schema](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#database-schema)
2. [API Architecture Overview](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#api-architecture-overview)
3. [Authentication & Authorization APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#authentication--authorization-apis)
4. [User Management APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#user-management-apis)
5. [RBAC Management APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#rbac-management-apis)
6. [Category Management APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#category-management-apis)
7. [Product Catalog APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#product-catalog-apis)
8. [Order Management APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#order-management-apis)
9. [Payment Processing APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#payment-processing-apis)
10. [Search & Filter APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#search--filter-apis)
11. [Analytics & Reporting APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#analytics--reporting-apis)
12. [Audit & Activity APIs](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#audit--activity-apis)
13. [Advanced SQL Patterns](https://claude.ai/chat/5e673489-920e-4922-be98-b5796af8303f#advanced-sql-patterns)

---

## Database Schema

### Core Tables with API Considerations

#### users

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  full_name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  phone VARCHAR(20),
  avatar_url TEXT,
  email_verified BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  last_login_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for API performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_active ON users(is_active);
CREATE INDEX idx_users_created_at ON users(created_at DESC);
```

#### roles

```sql
CREATE TABLE roles (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL,
  slug VARCHAR(50) UNIQUE NOT NULL, -- For API queries
  description TEXT,
  is_system BOOLEAN DEFAULT FALSE, -- Prevent deletion of system roles
  created_at TIMESTAMP DEFAULT NOW()
);

-- Pre-populate system roles
INSERT INTO roles (name, slug, is_system) VALUES
  ('Super Admin', 'super-admin', TRUE),
  ('Admin', 'admin', TRUE),
  ('Customer', 'customer', TRUE),
  ('Vendor', 'vendor', TRUE);
```

#### permissions

```sql
CREATE TABLE permissions (
  id SERIAL PRIMARY KEY,
  code VARCHAR(100) UNIQUE NOT NULL, -- e.g., 'products.create'
  resource VARCHAR(50) NOT NULL, -- e.g., 'products'
  action VARCHAR(50) NOT NULL, -- e.g., 'create', 'read', 'update', 'delete'
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_permissions_resource ON permissions(resource);
CREATE INDEX idx_permissions_code ON permissions(code);
```

#### user_roles

```sql
CREATE TABLE user_roles (
  user_id INT REFERENCES users(id) ON DELETE CASCADE,
  role_id INT REFERENCES roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMP DEFAULT NOW(),
  assigned_by INT REFERENCES users(id),
  PRIMARY KEY (user_id, role_id)
);

CREATE INDEX idx_user_roles_user ON user_roles(user_id);
CREATE INDEX idx_user_roles_role ON user_roles(role_id);
```

#### role_permissions

```sql
CREATE TABLE role_permissions (
  role_id INT REFERENCES roles(id) ON DELETE CASCADE,
  permission_id INT REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);
```

#### categories

```sql
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL, -- For SEO-friendly URLs
  parent_id INT REFERENCES categories(id) ON DELETE SET NULL,
  description TEXT,
  image_url TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  display_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_categories_parent ON categories(parent_id);
CREATE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_active ON categories(is_active);
```

#### products

```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  category_id INT REFERENCES categories(id),
  sku VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(150) NOT NULL,
  slug VARCHAR(150) UNIQUE NOT NULL,
  description TEXT,
  price NUMERIC(10,2) NOT NULL,
  compare_at_price NUMERIC(10,2), -- Original price for discounts
  cost_price NUMERIC(10,2), -- For margin calculations
  stock INT DEFAULT 0,
  low_stock_threshold INT DEFAULT 10,
  is_active BOOLEAN DEFAULT TRUE,
  is_featured BOOLEAN DEFAULT FALSE,
  tags TEXT[], -- Array for filtering
  metadata JSONB, -- Flexible attributes
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_active ON products(is_active);
CREATE INDEX idx_products_featured ON products(is_featured);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_tags ON products USING GIN(tags);
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);
```

#### product_images

```sql
CREATE TABLE product_images (
  id SERIAL PRIMARY KEY,
  product_id INT REFERENCES products(id) ON DELETE CASCADE,
  url TEXT NOT NULL,
  alt_text VARCHAR(255),
  display_order INT DEFAULT 0,
  is_primary BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_product_images_product ON product_images(product_id);
```

#### orders

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  order_number VARCHAR(50) UNIQUE NOT NULL, -- Human-readable order ID
  user_id INT REFERENCES users(id),
  status VARCHAR(30) NOT NULL DEFAULT 'pending',
  subtotal NUMERIC(12,2) NOT NULL,
  tax_amount NUMERIC(12,2) DEFAULT 0,
  shipping_amount NUMERIC(12,2) DEFAULT 0,
  discount_amount NUMERIC(12,2) DEFAULT 0,
  total_amount NUMERIC(12,2) NOT NULL,
  shipping_address JSONB,
  billing_address JSONB,
  customer_notes TEXT,
  admin_notes TEXT,
  cancelled_at TIMESTAMP,
  cancelled_reason TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_number ON orders(order_number);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
```

#### order_items

```sql
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INT REFERENCES orders(id) ON DELETE CASCADE,
  product_id INT REFERENCES products(id),
  product_name VARCHAR(150) NOT NULL, -- Snapshot at order time
  product_sku VARCHAR(50),
  quantity INT NOT NULL,
  unit_price NUMERIC(10,2) NOT NULL,
  subtotal NUMERIC(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

#### payments

```sql
CREATE TABLE payments (
  id SERIAL PRIMARY KEY,
  order_id INT REFERENCES orders(id) ON DELETE CASCADE,
  payment_method VARCHAR(50) NOT NULL, -- 'credit_card', 'paypal', etc.
  payment_status VARCHAR(30) NOT NULL DEFAULT 'pending',
  transaction_id VARCHAR(255), -- External payment gateway ID
  paid_amount NUMERIC(12,2),
  currency VARCHAR(3) DEFAULT 'USD',
  gateway_response JSONB, -- Store full response for debugging
  paid_at TIMESTAMP,
  refunded_at TIMESTAMP,
  refund_amount NUMERIC(12,2),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_payments_order ON payments(order_id);
CREATE INDEX idx_payments_status ON payments(payment_status);
CREATE INDEX idx_payments_transaction ON payments(transaction_id);
```

#### audit_logs

```sql
CREATE TABLE audit_logs (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id),
  action VARCHAR(100) NOT NULL, -- 'create', 'update', 'delete'
  resource VARCHAR(100) NOT NULL, -- 'product', 'order', 'user'
  resource_id INT,
  ip_address INET,
  user_agent TEXT,
  changes JSONB, -- Store old/new values
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource, resource_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at DESC);
```

#### reviews

```sql
CREATE TABLE reviews (
  id SERIAL PRIMARY KEY,
  product_id INT REFERENCES products(id) ON DELETE CASCADE,
  user_id INT REFERENCES users(id),
  order_id INT REFERENCES orders(id), -- Verified purchase
  rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),
  title VARCHAR(200),
  comment TEXT,
  is_verified BOOLEAN DEFAULT FALSE,
  is_approved BOOLEAN DEFAULT FALSE,
  helpful_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_reviews_product ON reviews(product_id);
CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_reviews_approved ON reviews(is_approved);
```

---

## API Architecture Overview

### RESTful Endpoint Structure

```
Base URL: https://api.example.com/v1

Authentication:
  POST   /auth/register
  POST   /auth/login
  POST   /auth/logout
  POST   /auth/refresh
  POST   /auth/forgot-password
  POST   /auth/reset-password

Users:
  GET    /users              (Admin only)
  GET    /users/:id
  GET    /users/me
  PATCH  /users/:id
  DELETE /users/:id

Roles & Permissions:
  GET    /roles
  POST   /roles
  GET    /roles/:id
  PATCH  /roles/:id
  DELETE /roles/:id
  GET    /permissions
  POST   /users/:id/roles
  DELETE /users/:id/roles/:roleId

Categories:
  GET    /categories
  GET    /categories/:slug
  POST   /categories
  PATCH  /categories/:id
  DELETE /categories/:id
  GET    /categories/tree

Products:
  GET    /products
  GET    /products/:slug
  POST   /products
  PATCH  /products/:id
  DELETE /products/:id
  GET    /products/:id/reviews

Orders:
  GET    /orders
  GET    /orders/:id
  POST   /orders
  PATCH  /orders/:id
  POST   /orders/:id/cancel

Payments:
  POST   /payments
  GET    /payments/:id
  POST   /payments/:id/refund

Analytics:
  GET    /analytics/sales
  GET    /analytics/products
  GET    /analytics/customers
  GET    /analytics/revenue

Search:
  GET    /search/products
  GET    /search/suggestions
```

---

## Authentication & Authorization APIs

### 1. User Registration

**Endpoint:** `POST /auth/register`

**Request:**

```json
{
  "full_name": "John Doe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "phone": "+1234567890"
}
```

**SQL:**

```sql
-- Insert new user
INSERT INTO users (full_name, email, password_hash, phone)
VALUES ($1, $2, crypt($3, gen_salt('bf')), $4)
RETURNING id, full_name, email, created_at;

-- Assign default 'customer' role
INSERT INTO user_roles (user_id, role_id)
SELECT $1, id FROM roles WHERE slug = 'customer';
```

**Response:**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": 123,
      "full_name": "John Doe",
      "email": "john@example.com",
      "created_at": "2026-02-05T10:30:00Z"
    },
    "access_token": "eyJhbGc...",
    "refresh_token": "eyJhbGc..."
  }
}
```

### 2. Login with Permission Loading

**Endpoint:** `POST /auth/login`

**SQL:**

```sql
-- Authenticate and load user with all permissions
WITH user_auth AS (
  SELECT id, full_name, email, is_active
  FROM users
  WHERE email = $1 
    AND password_hash = crypt($2, password_hash)
    AND is_active = TRUE
),
user_permissions AS (
  SELECT DISTINCT p.code
  FROM user_auth u
  JOIN user_roles ur ON ur.user_id = u.id
  JOIN role_permissions rp ON rp.role_id = ur.role_id
  JOIN permissions p ON p.id = rp.permission_id
)
SELECT 
  u.*,
  array_agg(up.code) as permissions
FROM user_auth u
LEFT JOIN user_permissions up ON TRUE
GROUP BY u.id, u.full_name, u.email, u.is_active;

-- Update last login
UPDATE users SET last_login_at = NOW() WHERE id = $1;
```

**Response:**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": 123,
      "full_name": "John Doe",
      "email": "john@example.com",
      "permissions": [
        "products.read",
        "orders.create",
        "orders.read"
      ]
    },
    "access_token": "eyJhbGc...",
    "refresh_token": "eyJhbGc..."
  }
}
```

### 3. Check API Access Permission

**Middleware SQL:**

```sql
-- Check if user can access specific endpoint
SELECT EXISTS (
  SELECT 1
  FROM users u
  JOIN user_roles ur ON ur.user_id = u.id
  JOIN role_permissions rp ON rp.role_id = ur.role_id
  JOIN permissions p ON p.id = rp.permission_id
  WHERE u.id = $1 
    AND u.is_active = TRUE
    AND p.code = $2 -- e.g., 'products.create'
) as has_permission;
```

---

## User Management APIs

### 4. Get User Profile with Roles

**Endpoint:** `GET /users/me`

**SQL:**

```sql
SELECT 
  u.id,
  u.full_name,
  u.email,
  u.phone,
  u.avatar_url,
  u.email_verified,
  u.created_at,
  json_agg(
    json_build_object(
      'id', r.id,
      'name', r.name,
      'slug', r.slug
    )
  ) as roles
FROM users u
LEFT JOIN user_roles ur ON ur.user_id = u.id
LEFT JOIN roles r ON r.id = ur.role_id
WHERE u.id = $1
GROUP BY u.id;
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 123,
    "full_name": "John Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "email_verified": true,
    "roles": [
      {
        "id": 3,
        "name": "Customer",
        "slug": "customer"
      }
    ],
    "created_at": "2026-01-15T08:00:00Z"
  }
}
```

### 5. List Users with Pagination and Filters

**Endpoint:** `GET /users?page=1&limit=20&role=customer&search=john&active=true`

**SQL:**

```sql
WITH filtered_users AS (
  SELECT DISTINCT u.id
  FROM users u
  LEFT JOIN user_roles ur ON ur.user_id = u.id
  LEFT JOIN roles r ON r.id = ur.role_id
  WHERE 1=1
    AND ($1::VARCHAR IS NULL OR r.slug = $1) -- role filter
    AND ($2::VARCHAR IS NULL OR u.full_name ILIKE '%' || $2 || '%' OR u.email ILIKE '%' || $2 || '%') -- search
    AND ($3::BOOLEAN IS NULL OR u.is_active = $3) -- active filter
)
SELECT 
  u.id,
  u.full_name,
  u.email,
  u.is_active,
  u.created_at,
  COUNT(*) OVER() as total_count,
  json_agg(
    json_build_object(
      'id', r.id,
      'name', r.name
    )
  ) as roles
FROM users u
INNER JOIN filtered_users fu ON fu.id = u.id
LEFT JOIN user_roles ur ON ur.user_id = u.id
LEFT JOIN roles r ON r.id = ur.role_id
GROUP BY u.id
ORDER BY u.created_at DESC
LIMIT $4 OFFSET $5;
```

**Response:**

```json
{
  "success": true,
  "data": {
    "users": [...],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8
    }
  }
}
```

### 6. Update User Profile

**Endpoint:** `PATCH /users/123`

**Request:**

```json
{
  "full_name": "John Updated",
  "phone": "+9876543210"
}
```

**SQL:**

```sql
UPDATE users
SET 
  full_name = COALESCE($2, full_name),
  phone = COALESCE($3, phone),
  updated_at = NOW()
WHERE id = $1
RETURNING id, full_name, email, phone, updated_at;
```

### 7. Soft Delete User

**Endpoint:** `DELETE /users/123`

**SQL:**

```sql
-- Soft delete with audit
UPDATE users
SET is_active = FALSE, updated_at = NOW()
WHERE id = $1
RETURNING id, email;

-- Log the action
INSERT INTO audit_logs (user_id, action, resource, resource_id, ip_address)
VALUES ($2, 'soft_delete', 'user', $1, $3);
```

---

## RBAC Management APIs

### 8. Create Role with Permissions

**Endpoint:** `POST /roles`

**Request:**

```json
{
  "name": "Content Manager",
  "slug": "content-manager",
  "description": "Manages products and categories",
  "permission_ids": [1, 2, 5, 6, 10]
}
```

**SQL:**

```sql
-- Create role
INSERT INTO roles (name, slug, description)
VALUES ($1, $2, $3)
RETURNING id;

-- Assign permissions
INSERT INTO role_permissions (role_id, permission_id)
SELECT $4, unnest($5::INT[]);

-- Return role with permissions
SELECT 
  r.id,
  r.name,
  r.slug,
  r.description,
  json_agg(
    json_build_object(
      'id', p.id,
      'code', p.code,
      'description', p.description
    )
  ) as permissions
FROM roles r
LEFT JOIN role_permissions rp ON rp.role_id = r.id
LEFT JOIN permissions p ON p.id = rp.permission_id
WHERE r.id = $4
GROUP BY r.id;
```

### 9. Assign Roles to User

**Endpoint:** `POST /users/123/roles`

**Request:**

```json
{
  "role_ids": [2, 4]
}
```

**SQL:**

```sql
-- Remove existing roles (if replacing)
DELETE FROM user_roles WHERE user_id = $1;

-- Assign new roles
INSERT INTO user_roles (user_id, role_id, assigned_by)
SELECT $1, unnest($2::INT[]), $3
ON CONFLICT (user_id, role_id) DO NOTHING;

-- Return updated user roles
SELECT 
  u.id,
  u.full_name,
  json_agg(
    json_build_object(
      'id', r.id,
      'name', r.name,
      'slug', r.slug
    )
  ) as roles
FROM users u
JOIN user_roles ur ON ur.user_id = u.id
JOIN roles r ON r.id = ur.role_id
WHERE u.id = $1
GROUP BY u.id;
```

### 10. List Users by Permission

**Endpoint:** `GET /permissions/products.create/users`

**SQL:**

```sql
-- Find all users with specific permission
SELECT DISTINCT
  u.id,
  u.full_name,
  u.email,
  array_agg(DISTINCT r.name) as roles
FROM users u
JOIN user_roles ur ON ur.user_id = u.id
JOIN roles r ON r.id = ur.role_id
JOIN role_permissions rp ON rp.role_id = r.id
JOIN permissions p ON p.id = rp.permission_id
WHERE p.code = $1
  AND u.is_active = TRUE
GROUP BY u.id
ORDER BY u.full_name;
```

---

## Category Management APIs

### 11. Get Category Tree (Nested JSON)

**Endpoint:** `GET /categories/tree`

**SQL (Recursive CTE):**

```sql
WITH RECURSIVE category_tree AS (
  -- Root categories
  SELECT 
    id, name, slug, parent_id, description, image_url,
    0 as level,
    ARRAY[id] as path
  FROM categories
  WHERE parent_id IS NULL AND is_active = TRUE
  
  UNION ALL
  
  -- Child categories
  SELECT 
    c.id, c.name, c.slug, c.parent_id, c.description, c.image_url,
    ct.level + 1,
    ct.path || c.id
  FROM categories c
  JOIN category_tree ct ON c.parent_id = ct.id
  WHERE c.is_active = TRUE
)
SELECT 
  json_agg(
    json_build_object(
      'id', id,
      'name', name,
      'slug', slug,
      'description', description,
      'image_url', image_url,
      'level', level,
      'children', (
        SELECT json_agg(
          json_build_object(
            'id', child.id,
            'name', child.name,
            'slug', child.slug
          )
        )
        FROM category_tree child
        WHERE child.parent_id = category_tree.id
      )
    )
  ) as category_tree
FROM category_tree
WHERE level = 0;
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Electronics",
      "slug": "electronics",
      "children": [
        {
          "id": 10,
          "name": "Laptops",
          "slug": "laptops"
        },
        {
          "id": 11,
          "name": "Phones",
          "slug": "phones"
        }
      ]
    }
  ]
}
```

### 12. Get Category Breadcrumb Path

**Endpoint:** `GET /categories/electronics/laptops/gaming`

**SQL:**

```sql
WITH RECURSIVE breadcrumb AS (
  SELECT id, name, slug, parent_id, 1 as depth
  FROM categories
  WHERE slug = $1
  
  UNION ALL
  
  SELECT c.id, c.name, c.slug, c.parent_id, b.depth + 1
  FROM categories c
  JOIN breadcrumb b ON c.id = b.parent_id
)
SELECT 
  json_agg(
    json_build_object(
      'id', id,
      'name', name,
      'slug', slug
    ) ORDER BY depth DESC
  ) as breadcrumb
FROM breadcrumb;
```

**Response:**

```json
{
  "breadcrumb": [
    {"id": 1, "name": "Electronics", "slug": "electronics"},
    {"id": 10, "name": "Laptops", "slug": "laptops"},
    {"id": 25, "name": "Gaming", "slug": "gaming"}
  ]
}
```

### 13. Create Category with Parent

**Endpoint:** `POST /categories`

**Request:**

```json
{
  "name": "Gaming Laptops",
  "slug": "gaming-laptops",
  "parent_id": 10,
  "description": "High-performance gaming laptops"
}
```

**SQL:**

```sql
INSERT INTO categories (name, slug, parent_id, description)
VALUES ($1, $2, $3, $4)
RETURNING id, name, slug, parent_id, created_at;
```

### 14. Get Categories with Product Count

**Endpoint:** `GET /categories?include_counts=true`

**SQL:**

```sql
SELECT 
  c.id,
  c.name,
  c.slug,
  c.parent_id,
  COUNT(p.id) as product_count,
  COUNT(DISTINCT CASE WHEN p.stock > 0 THEN p.id END) as in_stock_count
FROM categories c
LEFT JOIN products p ON p.category_id = c.id AND p.is_active = TRUE
WHERE c.is_active = TRUE
GROUP BY c.id
ORDER BY c.display_order, c.name;
```

---

## Product Catalog APIs

### 15. List Products with Advanced Filters

**Endpoint:** `GET /products?category=laptops&min_price=500&max_price=2000&in_stock=true&featured=true&tags=gaming,rgb&sort=price_desc&page=1&limit=24`

**SQL:**

```sql
SELECT 
  p.id,
  p.sku,
  p.name,
  p.slug,
  p.description,
  p.price,
  p.compare_at_price,
  p.stock,
  p.is_featured,
  p.tags,
  c.name as category_name,
  c.slug as category_slug,
  (
    SELECT json_agg(
      json_build_object(
        'url', url,
        'alt_text', alt_text
      ) ORDER BY display_order
    )
    FROM product_images
    WHERE product_id = p.id
  ) as images,
  COALESCE(AVG(r.rating), 0) as avg_rating,
  COUNT(r.id) as review_count,
  COUNT(*) OVER() as total_count
FROM products p
LEFT JOIN categories c ON c.id = p.category_id
LEFT JOIN reviews r ON r.product_id = p.id AND r.is_approved = TRUE
WHERE p.is_active = TRUE
  AND ($1::VARCHAR IS NULL OR c.slug = $1) -- category filter
  AND ($2::NUMERIC IS NULL OR p.price >= $2) -- min price
  AND ($3::NUMERIC IS NULL OR p.price <= $3) -- max price
  AND ($4::BOOLEAN IS NULL OR ($4 = TRUE AND p.stock > 0)) -- in stock
  AND ($5::BOOLEAN IS NULL OR p.is_featured = $5) -- featured
  AND ($6::TEXT[] IS NULL OR p.tags && $6) -- tags overlap
  AND ($7::VARCHAR IS NULL OR p.name ILIKE '%' || $7 || '%') -- search
GROUP BY p.id, c.id
ORDER BY 
  CASE WHEN $8 = 'price_asc' THEN p.price END ASC,
  CASE WHEN $8 = 'price_desc' THEN p.price END DESC,
  CASE WHEN $8 = 'newest' THEN p.created_at END DESC,
  CASE WHEN $8 = 'popular' THEN COUNT(r.id) END DESC,
  p.name
LIMIT $9 OFFSET $10;
```

**Response:**

```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": 42,
        "sku": "LAPTOP-001",
        "name": "Gaming Laptop Pro",
        "slug": "gaming-laptop-pro",
        "price": 1499.99,
        "compare_at_price": 1799.99,
        "stock": 15,
        "category_name": "Gaming Laptops",
        "images": [
          {"url": "https://...", "alt_text": "Front view"}
        ],
        "avg_rating": 4.7,
        "review_count": 23
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 24,
      "total": 87,
      "pages": 4
    },
    "filters": {
      "price_range": {"min": 500, "max": 2000},
      "category": "laptops",
      "in_stock": true
    }
  }
}
```

### 16. Get Single Product Details

**Endpoint:** `GET /products/gaming-laptop-pro`

**SQL:**

```sql
SELECT 
  p.id,
  p.sku,
  p.name,
  p.slug,
  p.description,
  p.price,
  p.compare_at_price,
  p.stock,
  p.is_featured,
  p.tags,
  p.metadata,
  json_build_object(
    'id', c.id,
    'name', c.name,
    'slug', c.slug
  ) as category,
  (
    SELECT json_agg(
      json_build_object(
        'id', id,
        'url', url,
        'alt_text', alt_text,
        'is_primary', is_primary
      ) ORDER BY display_order
    )
    FROM product_images
    WHERE product_id = p.id
  ) as images,
  (
    SELECT json_build_object(
      'average', COALESCE(AVG(rating), 0),
      'count', COUNT(*),
      'distribution', json_build_object(
        '5', COUNT(*) FILTER (WHERE rating = 5),
        '4', COUNT(*) FILTER (WHERE rating = 4),
        '3', COUNT(*) FILTER (WHERE rating = 3),
        '2', COUNT(*) FILTER (WHERE rating = 2),
        '1', COUNT(*) FILTER (WHERE rating = 1)
      )
    )
    FROM reviews
    WHERE product_id = p.id AND is_approved = TRUE
  ) as ratings
FROM products p
LEFT JOIN categories c ON c.id = p.category_id
WHERE p.slug = $1 AND p.is_active = TRUE;
```

### 17. Create Product with Images

**Endpoint:** `POST /products`

**Request:**

```json
{
  "category_id": 25,
  "sku": "LAPTOP-042",
  "name": "Ultra Gaming Laptop",
  "slug": "ultra-gaming-laptop",
  "description": "...",
  "price": 1999.99,
  "compare_at_price": 2499.99,
  "stock": 50,
  "tags": ["gaming", "rgb", "rtx"],
  "images": [
    {"url": "https://...", "alt_text": "Front", "is_primary": true},
    {"url": "https://...", "alt_text": "Side"}
  ]
}
```

**SQL (Transaction):**

```sql
BEGIN;

-- Insert product
INSERT INTO products (
  category_id, sku, name, slug, description, 
  price, compare_at_price, stock, tags
)
VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9)
RETURNING id;

-- Insert images
INSERT INTO product_images (product_id, url, alt_text, is_primary, display_order)
SELECT 
  $10, -- product_id from above
  (value->>'url')::TEXT,
  (value->>'alt_text')::TEXT,
  (value->>'is_primary')::BOOLEAN,
  (row_number() OVER ())::INT
FROM json_array_elements($11::JSON);

-- Log action
INSERT INTO audit_logs (user_id, action, resource, resource_id)
VALUES ($12, 'create', 'product', $10);

COMMIT;
```

### 18. Update Product Stock

**Endpoint:** `PATCH /products/42/stock`

**Request:**

```json
{
  "stock": 100,
  "operation": "set" // or "increment" or "decrement"
}
```

**SQL:**

```sql
UPDATE products
SET 
  stock = CASE 
    WHEN $2 = 'set' THEN $3
    WHEN $2 = 'increment' THEN stock + $3
    WHEN $2 = 'decrement' THEN GREATEST(0, stock - $3)
  END,
  updated_at = NOW()
WHERE id = $1
RETURNING id, name, stock;
```

### 19. Get Low Stock Products

**Endpoint:** `GET /products/low-stock`

**SQL:**

```sql
SELECT 
  p.id,
  p.sku,
  p.name,
  p.stock,
  p.low_stock_threshold,
  c.name as category_name,
  COUNT(DISTINCT oi.order_id) as pending_orders
FROM products p
LEFT JOIN categories c ON c.id = p.category_id
LEFT JOIN order_items oi ON oi.product_id = p.id
LEFT JOIN orders o ON o.id = oi.order_id AND o.status IN ('pending', 'processing')
WHERE p.is_active = TRUE
  AND p.stock <= p.low_stock_threshold
GROUP BY p.id, c.id
ORDER BY p.stock ASC;
```

### 20. Get Products Never Ordered

**Endpoint:** `GET /analytics/products/never-ordered`

**SQL:**

```sql
SELECT 
  p.id,
  p.sku,
  p.name,
  p.price,
  p.stock,
  p.created_at,
  c.name as category_name,
  EXTRACT(DAY FROM NOW() - p.created_at) as days_listed
FROM products p
LEFT JOIN categories c ON c.id = p.category_id
LEFT JOIN order_items oi ON oi.product_id = p.id
WHERE p.is_active = TRUE
  AND oi.id IS NULL
ORDER BY p.created_at DESC;
```

---

## Order Management APIs

### 21. Create Order

**Endpoint:** `POST /orders`

**Request:**

```json
{
  "items": [
    {
      "product_id": 42,
      "quantity": 2
    },
    {
      "product_id": 55,
      "quantity": 1
    }
  ],
  "shipping_address": {
    "name": "John Doe",
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zip": "10001",
    "country": "US"
  },
  "billing_address": {...},
  "payment_method": "credit_card"
}
```

**SQL (Complex Transaction):**

```sql
BEGIN;

-- Generate order number
SELECT 'ORD-' || LPAD(NEXTVAL('order_number_seq')::TEXT, 8, '0') INTO order_num;

-- Calculate totals from items
WITH item_totals AS (
  SELECT 
    SUM(p.price * i.quantity) as subtotal
  FROM json_to_recordset($1::JSON) AS i(product_id INT, quantity INT)
  JOIN products p ON p.id = i.product_id
)
SELECT 
  subtotal,
  subtotal * 0.08 as tax,
  15.00 as shipping,
  subtotal * 1.08 + 15.00 as total
INTO order_totals
FROM item_totals;

-- Create order
INSERT INTO orders (
  order_number, user_id, status, subtotal, tax_amount, 
  shipping_amount, total_amount, shipping_address, billing_address
)
VALUES (
  order_num, $2, 'pending', 
  order_totals.subtotal, order_totals.tax, 
  order_totals.shipping, order_totals.total, 
  $3, $4
)
RETURNING id INTO order_id;

-- Insert order items
INSERT INTO order_items (
  order_id, product_id, product_name, product_sku,
  quantity, unit_price, subtotal
)
SELECT 
  order_id,
  p.id,
  p.name,
  p.sku,
  i.quantity,
  p.price,
  i.quantity * p.price
FROM json_to_recordset($1::JSON) AS i(product_id INT, quantity INT)
JOIN products p ON p.id = i.product_id;

-- Reduce product stock
UPDATE products p
SET stock = p.stock - i.quantity
FROM json_to_recordset($1::JSON) AS i(product_id INT, quantity INT)
WHERE p.id = i.product_id;

-- Log action
INSERT INTO audit_logs (user_id, action, resource, resource_id)
VALUES ($2, 'create', 'order', order_id);

COMMIT;

-- Return order
SELECT * FROM orders WHERE id = order_id;
```

### 22. Get Order Details

**Endpoint:** `GET /orders/ORD-00001234`

**SQL:**

```sql
SELECT 
  o.id,
  o.order_number,
  o.status,
  o.subtotal,
  o.tax_amount,
  o.shipping_amount,
  o.total_amount,
  o.shipping_address,
  o.created_at,
  json_build_object(
    'id', u.id,
    'name', u.full_name,
    'email', u.email
  ) as customer,
  json_agg(
    json_build_object(
      'id', oi.id,
      'product_id', oi.product_id,
      'product_name', oi.product_name,
      'sku', oi.product_sku,
      'quantity', oi.quantity,
      'unit_price', oi.unit_price,
      'subtotal', oi.subtotal
    ) ORDER BY oi.id
  ) as items,
  (
    SELECT json_build_object(
      'payment_method', payment_method,
      'status', payment_status,
      'paid_amount', paid_amount,
      'paid_at', paid_at
    )
    FROM payments
    WHERE order_id = o.id
    ORDER BY created_at DESC
    LIMIT 1
  ) as payment
FROM orders o
JOIN users u ON u.id = o.user_id
LEFT JOIN order_items oi ON oi.order_id = o.id
WHERE o.order_number = $1
GROUP BY o.id, u.id;
```

### 23. Get User Order History

**Endpoint:** `GET /orders?page=1&status=completed`

**SQL:**

```sql
SELECT 
  o.id,
  o.order_number,
  o.status,
  o.total_amount,
  o.created_at,
  COUNT(oi.id) as item_count,
  (
    SELECT payment_status 
    FROM payments 
    WHERE order_id = o.id 
    ORDER BY created_at DESC 
    LIMIT 1
  ) as payment_status,
  COUNT(*) OVER() as total_count
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
WHERE o.user_id = $1
  AND ($2::VARCHAR IS NULL OR o.status = $2)
GROUP BY o.id
ORDER BY o.created_at DESC
LIMIT $3 OFFSET $4;
```

### 24. Update Order Status

**Endpoint:** `PATCH /orders/123/status`

**Request:**

```json
{
  "status": "shipped",
  "notes": "Shipped via FedEx, tracking: 123456789"
}
```

**SQL:**

```sql
UPDATE orders
SET 
  status = $2,
  admin_notes = COALESCE($3, admin_notes),
  updated_at = NOW()
WHERE id = $1
RETURNING id, order_number, status, updated_at;

-- Log status change
INSERT INTO audit_logs (user_id, action, resource, resource_id, changes)
VALUES (
  $4,
  'update_status',
  'order',
  $1,
  json_build_object('new_status', $2, 'notes', $3)
);
```

### 25. Cancel Order

**Endpoint:** `POST /orders/123/cancel`

**SQL (Transaction):**

```sql
BEGIN;

-- Update order
UPDATE orders
SET 
  status = 'cancelled',
  cancelled_at = NOW(),
  cancelled_reason = $2
WHERE id = $1
  AND status NOT IN ('shipped', 'delivered', 'cancelled')
RETURNING id;

-- Restore product stock
UPDATE products p
SET stock = p.stock + oi.quantity
FROM order_items oi
WHERE oi.order_id = $1
  AND oi.product_id = p.id;

-- Refund payment if already paid
UPDATE payments
SET payment_status = 'refunded', refunded_at = NOW()
WHERE order_id = $1 AND payment_status = 'completed';

COMMIT;
```

### 26. Get Orders Without Payment

**Endpoint:** `GET /analytics/orders/unpaid`

**SQL:**

```sql
SELECT 
  o.id,
  o.order_number,
  o.total_amount,
  o.created_at,
  u.full_name,
  u.email,
  EXTRACT(HOUR FROM NOW() - o.created_at) as hours_since_order
FROM orders o
JOIN users u ON u.id = o.user_id
LEFT JOIN payments p ON p.order_id = o.id AND p.payment_status = 'completed'
WHERE o.status = 'pending'
  AND p.id IS NULL
  AND o.created_at > NOW() - INTERVAL '7 days'
ORDER BY o.created_at DESC;
```

---

## Payment Processing APIs

### 27. Create Payment

**Endpoint:** `POST /payments`

**Request:**

```json
{
  "order_id": 123,
  "payment_method": "stripe",
  "transaction_id": "ch_3abc123",
  "amount": 1599.99
}
```

**SQL:**

```sql
BEGIN;

-- Create payment record
INSERT INTO payments (
  order_id, payment_method, payment_status,
  transaction_id, paid_amount, paid_at, gateway_response
)
VALUES ($1, $2, 'completed', $3, $4, NOW(), $5)
RETURNING id;

-- Update order status
UPDATE orders
SET status = 'paid', updated_at = NOW()
WHERE id = $1;

COMMIT;
```

### 28. Get Revenue by Payment Method

**Endpoint:** `GET /analytics/revenue/by-payment-method`

**SQL:**

```sql
SELECT 
  payment_method,
  COUNT(*) as transaction_count,
  SUM(paid_amount) as total_revenue,
  AVG(paid_amount) as avg_transaction,
  MIN(paid_at) as first_payment,
  MAX(paid_at) as last_payment
FROM payments
WHERE payment_status = 'completed'
  AND paid_at >= $1 -- start_date
  AND paid_at <= $2 -- end_date
GROUP BY payment_method
ORDER BY total_revenue DESC;
```

---

## Search & Filter APIs

### 29. Global Product Search

**Endpoint:** `GET /search/products?q=gaming laptop rtx&limit=10`

**SQL (Full-text search):**

```sql
SELECT 
  p.id,
  p.name,
  p.slug,
  p.price,
  p.stock,
  c.name as category_name,
  ts_rank(
    to_tsvector('english', p.name || ' ' || COALESCE(p.description, '')),
    plainto_tsquery('english', $1)
  ) as relevance,
  (SELECT url FROM product_images WHERE product_id = p.id AND is_primary = TRUE LIMIT 1) as image
FROM products p
LEFT JOIN categories c ON c.id = p.category_id
WHERE p.is_active = TRUE
  AND (
    to_tsvector('english', p.name || ' ' || COALESCE(p.description, '')) @@ plainto_tsquery('english', $1)
    OR p.tags && string_to_array($1, ' ')
  )
ORDER BY relevance DESC
LIMIT $2;
```

### 30. Search Suggestions (Autocomplete)

**Endpoint:** `GET /search/suggestions?q=gam`

**SQL:**

```sql
-- Product name suggestions
SELECT DISTINCT name, 'product' as type
FROM products
WHERE name ILIKE $1 || '%' AND is_active = TRUE
LIMIT 5

UNION ALL

-- Category suggestions
SELECT DISTINCT name, 'category' as type
FROM categories
WHERE name ILIKE $1 || '%' AND is_active = TRUE
LIMIT 3

ORDER BY type, name
LIMIT 8;
```

---

## Analytics & Reporting APIs

### 31. Sales Dashboard

**Endpoint:** `GET /analytics/dashboard?period=30days`

**SQL:**

```sql
WITH date_range AS (
  SELECT 
    NOW() - INTERVAL '30 days' as start_date,
    NOW() as end_date
),
current_period AS (
  SELECT 
    COUNT(*) as order_count,
    SUM(total_amount) as revenue,
    AVG(total_amount) as avg_order_value,
    COUNT(DISTINCT user_id) as unique_customers
  FROM orders
  WHERE created_at >= (SELECT start_date FROM date_range)
    AND status NOT IN ('cancelled')
),
previous_period AS (
  SELECT 
    COUNT(*) as order_count,
    SUM(total_amount) as revenue,
    AVG(total_amount) as avg_order_value,
    COUNT(DISTINCT user_id) as unique_customers
  FROM orders
  WHERE created_at >= (SELECT start_date FROM date_range) - INTERVAL '30 days'
    AND created_at < (SELECT start_date FROM date_range)
    AND status NOT IN ('cancelled')
)
SELECT 
  json_build_object(
    'revenue', cp.revenue,
    'revenue_change', ROUND((cp.revenue - pp.revenue) / pp.revenue * 100, 2),
    'orders', cp.order_count,
    'orders_change', ROUND((cp.order_count - pp.order_count)::NUMERIC / pp.order_count * 100, 2),
    'avg_order_value', cp.avg_order_value,
    'avg_order_value_change', ROUND((cp.avg_order_value - pp.avg_order_value) / pp.avg_order_value * 100, 2),
    'customers', cp.unique_customers,
    'customers_change', ROUND((cp.unique_customers - pp.unique_customers)::NUMERIC / pp.unique_customers * 100, 2)
  ) as metrics
FROM current_period cp, previous_period pp;
```

**Response:**

```json
{
  "success": true,
  "data": {
    "revenue": 125430.50,
    "revenue_change": 15.3,
    "orders": 342,
    "orders_change": 8.2,
    "avg_order_value": 366.75,
    "avg_order_value_change": 6.5,
    "customers": 287,
    "customers_change": 12.1
  }
}
```

### 32. Top Products by Revenue

**Endpoint:** `GET /analytics/products/top?period=30days&limit=10`

**SQL:**

```sql
SELECT 
  p.id,
  p.name,
  p.sku,
  c.name as category_name,
  COUNT(DISTINCT oi.order_id) as order_count,
  SUM(oi.quantity) as units_sold,
  SUM(oi.subtotal) as revenue,
  ROUND(AVG(oi.unit_price), 2) as avg_selling_price,
  p.cost_price,
  SUM(oi.subtotal) - (SUM(oi.quantity) * p.cost_price) as profit
FROM order_items oi
JOIN products p ON p.id = oi.product_id
LEFT JOIN categories c ON c.id = p.category_id
JOIN orders o ON o.id = oi.order_id
WHERE o.created_at >= NOW() - INTERVAL '30 days'
  AND o.status NOT IN ('cancelled')
GROUP BY p.id, c.id
ORDER BY revenue DESC
LIMIT $1;
```

### 33. Sales by Category with Trends

**Endpoint:** `GET /analytics/categories/revenue`

**SQL:**

```sql
WITH category_sales AS (
  SELECT 
    c.id,
    c.name,
    DATE_TRUNC('month', o.created_at) as month,
    SUM(oi.subtotal) as revenue,
    COUNT(DISTINCT o.id) as orders,
    SUM(oi.quantity) as units
  FROM categories c
  JOIN products p ON p.category_id = c.id
  JOIN order_items oi ON oi.product_id = p.id
  JOIN orders o ON o.id = oi.order_id
  WHERE o.created_at >= NOW() - INTERVAL '12 months'
    AND o.status NOT IN ('cancelled')
  GROUP BY c.id, DATE_TRUNC('month', o.created_at)
)
SELECT 
  id,
  name,
  SUM(revenue) as total_revenue,
  SUM(orders) as total_orders,
  SUM(units) as total_units,
  ROUND(AVG(revenue), 2) as avg_monthly_revenue,
  json_agg(
    json_build_object(
      'month', month,
      'revenue', revenue,
      'orders', orders
    ) ORDER BY month
  ) as monthly_breakdown
FROM category_sales
GROUP BY id, name
ORDER BY total_revenue DESC;
```

### 34. Customer Lifetime Value

**Endpoint:** `GET /analytics/customers/ltv?min_orders=2`

**SQL:**

```sql
SELECT 
  u.id,
  u.full_name,
  u.email,
  COUNT(o.id) as total_orders,
  SUM(o.total_amount) as lifetime_value,
  AVG(o.total_amount) as avg_order_value,
  MIN(o.created_at) as first_order_date,
  MAX(o.created_at) as last_order_date,
  EXTRACT(DAY FROM MAX(o.created_at) - MIN(o.created_at)) as customer_lifespan_days
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.status NOT IN ('cancelled')
GROUP BY u.id
HAVING COUNT(o.id) >= $1
ORDER BY lifetime_value DESC
LIMIT 100;
```

### 35. Product Performance Matrix

**Endpoint:** `GET /analytics/products/matrix`

**SQL:**

```sql
WITH product_stats AS (
  SELECT 
    p.id,
    p.name,
    p.price,
    p.stock,
    COALESCE(SUM(oi.quantity), 0) as total_sold,
    COALESCE(SUM(oi.subtotal), 0) as revenue,
    COALESCE(COUNT(DISTINCT oi.order_id), 0) as order_count,
    COALESCE(AVG(r.rating), 0) as avg_rating,
    COUNT(r.id) as review_count
  FROM products p
  LEFT JOIN order_items oi ON oi.product_id = p.id
  LEFT JOIN orders o ON o.id = oi.order_id AND o.status NOT IN ('cancelled')
  LEFT JOIN reviews r ON r.product_id = p.id AND r.is_approved = TRUE
  WHERE p.is_active = TRUE
  GROUP BY p.id
),
percentiles AS (
  SELECT 
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) as median_revenue,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_sold) as median_sales
  FROM product_stats
)
SELECT 
  ps.*,
  CASE 
    WHEN revenue >= median_revenue AND total_sold >= median_sales THEN 'star'
    WHEN revenue >= median_revenue AND total_sold < median_sales THEN 'premium'
    WHEN revenue < median_revenue AND total_sold >= median_sales THEN 'volume'
    ELSE 'underperformer'
  END as classification
FROM product_stats ps, percentiles
ORDER BY revenue DESC;
```

---

## Audit & Activity APIs

### 36. Activity Log with Filters

**Endpoint:** `GET /audit/logs?user_id=123&resource=product&action=update&from=2026-01-01&to=2026-02-01&page=1`

**SQL:**

```sql
SELECT 
  al.id,
  al.action,
  al.resource,
  al.resource_id,
  al.ip_address,
  al.changes,
  al.created_at,
  json_build_object(
    'id', u.id,
    'name', u.full_name,
    'email', u.email
  ) as user,
  COUNT(*) OVER() as total_count
FROM audit_logs al
LEFT JOIN users u ON u.id = al.user_id
WHERE 1=1
  AND ($1::INT IS NULL OR al.user_id = $1)
  AND ($2::VARCHAR IS NULL OR al.resource = $2)
  AND ($3::VARCHAR IS NULL OR al.action = $3)
  AND ($4::TIMESTAMP IS NULL OR al.created_at >= $4)
  AND ($5::TIMESTAMP IS NULL OR al.created_at <= $5)
ORDER BY al.created_at DESC
LIMIT $6 OFFSET $7;
```

### 37. User Activity Summary

**Endpoint:** `GET /analytics/users/123/activity`

**SQL:**

```sql
SELECT 
  json_build_object(
    'total_actions', COUNT(*),
    'actions_by_type', (
      SELECT json_object_agg(action, count)
      FROM (
        SELECT action, COUNT(*) as count
        FROM audit_logs
        WHERE user_id = $1
        GROUP BY action
      ) t
    ),
    'resources_modified', (
      SELECT json_object_agg(resource, count)
      FROM (
        SELECT resource, COUNT(*) as count
        FROM audit_logs
        WHERE user_id = $1
        GROUP BY resource
      ) t
    ),
    'first_activity', MIN(created_at),
    'last_activity', MAX(created_at)
  ) as activity_summary
FROM audit_logs
WHERE user_id = $1;
```

---

## Advanced SQL Patterns

### 38. Window Functions: Rank Products by Category

**Use Case:** Display "Top 3 in Category" badge

**SQL:**

```sql
WITH ranked_products AS (
  SELECT 
    p.*,
    c.name as category_name,
    COALESCE(SUM(oi.quantity), 0) as total_sold,
    ROW_NUMBER() OVER (PARTITION BY p.category_id ORDER BY COALESCE(SUM(oi.quantity), 0) DESC) as rank_in_category
  FROM products p
  LEFT JOIN categories c ON c.id = p.category_id
  LEFT JOIN order_items oi ON oi.product_id = p.id
  WHERE p.is_active = TRUE
  GROUP BY p.id, c.name
)
SELECT *
FROM ranked_products
WHERE rank_in_category <= 3
ORDER BY category_name, rank_in_category;
```

### 39. Running Totals: Cumulative Revenue

**SQL:**

```sql
SELECT 
  DATE(created_at) as order_date,
  COUNT(*) as daily_orders,
  SUM(total_amount) as daily_revenue,
  SUM(SUM(total_amount)) OVER (ORDER BY DATE(created_at)) as cumulative_revenue
FROM orders
WHERE status NOT IN ('cancelled')
  AND created_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY order_date;
```

### 40. Pivot Table: Sales by Day of Week

**SQL:**

```sql
SELECT 
  EXTRACT(HOUR FROM created_at) as hour,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 0) as sunday,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 1) as monday,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 2) as tuesday,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 3) as wednesday,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 4) as thursday,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 5) as friday,
  COUNT(*) FILTER (WHERE EXTRACT(DOW FROM created_at) = 6) as saturday
FROM orders
WHERE created_at >= NOW() - INTERVAL '90 days'
GROUP BY EXTRACT(HOUR FROM created_at)
ORDER BY hour;
```

### 41. JSON Aggregation: Nested Order Export

**SQL:**

```sql
SELECT json_agg(
  json_build_object(
    'order_number', o.order_number,
    'customer', json_build_object(
      'name', u.full_name,
      'email', u.email
    ),
    'items', (
      SELECT json_agg(
        json_build_object(
          'product', oi.product_name,
          'quantity', oi.quantity,
          'price', oi.unit_price
        )
      )
      FROM order_items oi
      WHERE oi.order_id = o.id
    ),
    'total', o.total_amount
  )
) as orders_export
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.created_at >= $1 AND o.created_at <= $2;
```

### 42. Inventory Forecasting

**SQL:**

```sql
WITH daily_sales AS (
  SELECT 
    p.id as product_id,
    DATE(o.created_at) as sale_date,
    SUM(oi.quantity) as units_sold
  FROM products p
  LEFT JOIN order_items oi ON oi.product_id = p.id
  LEFT JOIN orders o ON o.id = oi.order_id AND o.status NOT IN ('cancelled')
  WHERE o.created_at >= NOW() - INTERVAL '30 days'
  GROUP BY p.id, DATE(o.created_at)
),
avg_daily_sales AS (
  SELECT 
    product_id,
    AVG(units_sold) as avg_daily_units
  FROM daily_sales
  GROUP BY product_id
)
SELECT 
  p.id,
  p.name,
  p.stock,
  ROUND(ads.avg_daily_units, 2) as avg_daily_sales,
  CASE 
    WHEN ads.avg_daily_units > 0 
    THEN ROUND(p.stock / ads.avg_daily_units, 1)
    ELSE NULL
  END as days_of_stock_remaining,
  CASE 
    WHEN p.stock / NULLIF(ads.avg_daily_units, 0) < 7 THEN 'urgent'
    WHEN p.stock / NULLIF(ads.avg_daily_units, 0) < 14 THEN 'soon'
    ELSE 'ok'
  END as reorder_priority
FROM products p
LEFT JOIN avg_daily_sales ads ON ads.product_id = p.id
WHERE p.is_active = TRUE
ORDER BY days_of_stock_remaining NULLS LAST;
```

### 43. Cohort Analysis: Customer Retention

**SQL:**

```sql
WITH first_orders AS (
  SELECT 
    user_id,
    DATE_TRUNC('month', MIN(created_at)) as cohort_month
  FROM orders
  WHERE status NOT IN ('cancelled')
  GROUP BY user_id
),
order_months AS (
  SELECT 
    o.user_id,
    fo.cohort_month,
    DATE_TRUNC('month', o.created_at) as order_month,
    EXTRACT(MONTH FROM AGE(DATE_TRUNC('month', o.created_at), fo.cohort_month)) as months_since_first
  FROM orders o
  JOIN first_orders fo ON fo.user_id = o.user_id
  WHERE o.status NOT IN ('cancelled')
)
SELECT 
  cohort_month,
  COUNT(DISTINCT user_id) FILTER (WHERE months_since_first = 0) as month_0,
  COUNT(DISTINCT user_id) FILTER (WHERE months_since_first = 1) as month_1,
  COUNT(DISTINCT user_id) FILTER (WHERE months_since_first = 2) as month_2,
  COUNT(DISTINCT user_id) FILTER (WHERE months_since_first = 3) as month_3,
  ROUND(
    COUNT(DISTINCT user_id) FILTER (WHERE months_since_first = 1)::NUMERIC / 
    NULLIF(COUNT(DISTINCT user_id) FILTER (WHERE months_since_first = 0), 0) * 100, 
    2
  ) as retention_month_1
FROM order_months
GROUP BY cohort_month
ORDER BY cohort_month DESC;
```

---

## Best Practices Summary

### API Design Principles

1. **Use Consistent Response Format**

```json
{
  "success": true/false,
  "data": {...},
  "error": {...},
  "meta": {
    "pagination": {...},
    "timestamp": "..."
  }
}
```

2. **Implement Proper Pagination**

- Always include total count
- Support cursor-based pagination for large datasets
- Return next/previous page indicators

3. **Filter & Sort Parameters**

- Use query parameters for filters
- Support multiple sort fields
- Implement sensible defaults

4. **Security Best Practices**

- Never return password hashes
- Validate permissions in middleware
- Use parameterized queries (prevent SQL injection)
- Log all sensitive operations

5. **Performance Optimization**

- Create indexes on foreign keys
- Use `EXPLAIN ANALYZE` for slow queries
- Implement Redis caching for frequently accessed data
- Use database connection pooling

6. **Error Handling**

- Return meaningful error messages
- Use appropriate HTTP status codes
- Log errors with context
- Never expose internal errors to users

---

## Complete API Implementation Checklist

- [ ] Authentication & JWT middleware
- [ ] Permission-based authorization
- [ ] Request validation (JSON schema)
- [ ] Rate limiting
- [ ] API versioning
- [ ] CORS configuration
- [ ] Logging & monitoring
- [ ] Database migrations
- [ ] API documentation (OpenAPI/Swagger)
- [ ] Unit & integration tests
- [ ] Caching layer
- [ ] Error tracking (Sentry)
- [ ] Performance monitoring (New Relic/DataDog)

---

**This document provides production-ready SQL queries for building a complete e-commerce REST API. Each query is optimized for performance and includes proper indexing, filtering, and pagination support.**