
---
# 📚 Table of Contents — Database Notes

---
## Foundations

- [1. What is a Database? 🗄️)
- [2. What is a DBMS? ⚙️](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#2-what-is-a-dbms-)
## Database Types

- [3. Types of Databases 🧠](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#3-types-of-databases-)
    - [3.1 Relational Database (SQL)](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#1-relational-database-sql)
    - [3.2 NoSQL Databases](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#2-nosql-databases)
        - [A. Document Database](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#a-document-database)
        - [B. Key-Value Database](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#b-key-value-database)
        - [C. Wide Column Database](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#c-wide-column-database)
        - [D. Graph Database](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#d-graph-database)

## Deployment

- [4. Database Deployment Types 🚀](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#4-database-deployment-types-)
    - [4.1 Cloud Database ☁️](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#cloud-database-%EF%B8%8F)
- [6. How to Run Database on Cloud ☁️](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#6-how-to-run-database-on-cloud-%EF%B8%8F)
    - [6.1 Managed Database](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#managed-database)
    - [6.2 Self Hosted Database (VM)](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#self-hosted-database-vm)
- [7. If Database Not Available on Cloud ❗](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#7-if-database-not-available-on-cloud-)
    - [7.1 Option 1: Self Host on VM](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#option-1-self-host-on-vm)
    - [7.2 Option 2: Docker Deployment](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#option-2-docker-deployment)
    - [7.3 Option 3: Kubernetes Deployment](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#option-3-kubernetes-deployment)
    - [7.4 Option 4: Hybrid Architecture](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#option-4-hybrid-architecture)

## AWS & Cloud Services

- [8. Database Services](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#8-database-services)
- [9. AWS Database Services ☁️🔥](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#9-aws-database-services-%EF%B8%8F)
    - [9.1 AWS RDS](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#1-aws-rds)
    - [9.2 AWS DynamoDB](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#2-aws-dynamodb)
    - [9.3 AWS ElastiCache](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#3-aws-elasticache)
    - [9.4 Amazon MemoryDB](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#4-amazon-memorydb)
    - [9.5 Amazon DocumentDB](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#5-amazon-documentdb)

---

> 💡 **Tip:** In most Markdown viewers (VS Code, GitHub, Obsidian), clicking a link above will jump directly to that section.
# 1. What is a Database? 🗄️

A **database** is an organized collection of data stored electronically so it can be easily accessed, managed, and updated.
# 2. What is DBMS (Database Management System)? ⚙️

A **DBMS** is software used to **create, manage, and interact** with databases.
### Examples
- MySQL
- PostgreSQL
- MongoDB
- Redis
- SQLite
### DBMS Provides
- Data storage
- Querying (SQL / NoSQL)
- Indexing
- Transactions
- Security
- Backup & Restore
- Replication
---
# 3. Types of Databases 🧠
## 1. Relational Database (SQL)

Data stored in **tables (rows & columns)**
### Examples
- MySQL
- PostgreSQL
- SQL Server
- Oracle
### Example

```
Users
Orders
Payments
```

Use When:
- Structured data
- Relationships required
- Transactions needed
---
## 2. NoSQL Databases
Flexible schema and high scalability.
### A. Document Database
Stores JSON-like data
Example:

```
{
  name: "Binod",
  skills: ["React", "Node"]
}
```

Examples:
- MongoDB
- DocumentDB
---
### B. Key-Value Database
Fastest database type
Example:
```
user:1 → {name:"binod"}
```
Examples:
- Redis
- DynamoDB
---
### C. Wide Column Database
Examples:
- Cassandra
- HBase
Used for:
- Big data
- Analytics
---
### D. Graph Database
Used for relationships
Examples:
- Neo4j
- Amazon Neptune
---
# 4. Database Deployment Types 🚀

---
## Cloud Database ☁️

Architecture:

```
App → Internet → Cloud Database
```

Example:

```
AWS RDS
Mongo Atlas
Firebase
```
### Pros
- Auto scaling
- Backup
- Monitoring
- High availability
### Cons
- Cost
- Network dependency
---
# 6. How to Run Database on Cloud ☁️

## Managed Database

Example:

```
AWS RDS
Mongo Atlas
Firebase
```

Connection:

```
postgres://user:pass@cloud-url:5432/db
```

---
## Self Hosted Database (VM)

```
EC2 instance
Install postgres
Expose port
```

You manage:
- Backup
- Scaling
- Security
---
# 7. If Database Not Available on Cloud ❗
Sometimes database not supported in cloud.
Solutions:
## Option 1: Self Host on VM

```
1.AWS EC2 (Unpredictable traffic, high scalability, complex infrastructure (big data, IoT), and large teams.)
2.VPS(DigitalOcean,hetzner,linode)(Simple hosting, predictable workloads, development/testing, and cost-conscious projects.)
```
Install DB manually.
---
## Option 2: Docker Deployment

```
Docker container
Deploy anywhere
```
---
## Option 3: Kubernetes Deployment

```
Deploy DB as pod
Persistent volume
```

---
## Option 4: Hybrid Architecture

```
Cloud App → Local DB → Sync
```

---
# 8. Database Services
Database services are **managed database providers**
Examples:
- AWS
- Azure
- GCP
- Mongo Atlas
- Supabase
- PlanetScale
They provide:
- Managed DB
- Auto backup
- Scaling
- Monitoring
- Security
---
# 9. AWS Database Services ☁️🔥
## 1. AWS RDS

Relational Database Service
Supports:
- MySQL
- PostgreSQL
- MariaDB
- Oracle
- SQL Server
Use:
- Production apps
- Transactions
---
## 2. AWS DynamoDB

NoSQL Key-Value Database
Features:
- Serverless
- Auto scale
- Fast
Use:
- High traffic apps
- Sessions
- Realtime apps
---
## 3. AWS ElastiCache
In-memory cache database
Supports:
- Redis
- Memcached
Use:
- Cache
- Sessions
- Leaderboards
Architecture:

```
App → Redis → Database
```

---
## 4. Amazon MemoryDB
Redis compatible primary database
Difference:
```
ElastiCache = cache
MemoryDB = primary DB
```
Use:
- Ultra low latency apps
- Realtime apps
---
## 5. Amazon DocumentDB
MongoDB compatible document database
Stores:
```
JSON documents
```
Use:
- Flexible schema
- JSON data
---
# 10. AWS Database Comparison Table

|Service|Type|Use Case|
|---|---|---|
|RDS|SQL|Normal apps|
|DynamoDB|NoSQL|Massive scale|
|ElastiCache|Cache|Speed|
|MemoryDB|In-memory DB|Ultra fast|
|DocumentDB|Document|JSON data|

---
# 11. Real World Architecture Example 🏗️

```
Frontend
   ↓
Backend API
   ↓
ElastiCache (Redis)
   ↓
RDS Database
   ↓
Read Replica
```
---
# 12. When to Use What 🎯
Use RDS when:
- Structured data
- Transactions
Use DynamoDB when:
- Massive scale
- Serverless

Use ElastiCache when:
- Caching
- Sessions
Use MemoryDB when:
- Ultra fast DB
- Low latency

Use DocumentDB when:
- JSON data
- Flexible schema
---
