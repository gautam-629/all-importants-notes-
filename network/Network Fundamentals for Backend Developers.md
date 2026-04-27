
---
## Table of Contents

1. [What is a Network?](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#1-what-is-a-network)
2. [The OSI Model](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#2-the-osi-model)
3. [TCP/IP — The Internet's Foundation](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#3-tcpip--the-internets-foundation)
4. [IP Addresses & Subnetting](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#4-ip-addresses--subnetting)
5. [DNS — Domain Name System](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#5-dns--domain-name-system)
6. [HTTP & HTTPS](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#6-http--https)
7. [TCP vs UDP](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#7-tcp-vs-udp)
8. [Ports & Sockets](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#8-ports--sockets)
9. [REST APIs over HTTP](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#9-rest-apis-over-http)
10. [WebSockets — Real-Time Communication](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#10-websockets--real-time-communication)
11. [Load Balancers & Reverse Proxies](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#11-load-balancers--reverse-proxies)
12. [CDN — Content Delivery Network](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#12-cdn--content-delivery-network)
13. [Firewalls, NAT & Security Basics](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#13-firewalls-nat--security-basics)
14. [TLS/SSL — How HTTPS Works](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#14-tlsssl--how-https-works)
15. [Common Backend Networking Patterns](https://claude.ai/chat/c66c23d0-f511-4aff-a32f-62eaeedc9be8#15-common-backend-networking-patterns)

---

## 1. What is a Network?

A **network** is two or more computers connected together to share data.

```
Your Laptop  ──────────────────  Google Server
(Client)          Internet          (Server)
192.168.1.5                      142.250.190.46
```

### Key Terms

|Term|Meaning|
|---|---|
|**Client**|The machine that requests data (browser, mobile app)|
|**Server**|The machine that responds with data (your backend)|
|**Protocol**|A set of rules both sides agree on (HTTP, TCP, FTP)|
|**Packet**|A small chunk of data sent over the network|
|**Bandwidth**|Max data transfer rate (e.g., 100 Mbps)|
|**Latency**|Time for a packet to travel from A to B (ms)|

### Real Example
When you type `https://github.com` in your browser:
1. Your browser (client) sends a request
2. It travels across the internet as packets
3. GitHub's server receives it and sends back HTML
4. Your browser renders the page
---
## 2. The OSI Model
The **OSI (Open Systems Interconnection)** model is a 7-layer framework that describes how data moves through a network. Think of it as a factory assembly line — each layer does one job.

```
┌─────────────────────────────────────────┐
│  7. Application   │ HTTP, FTP, SMTP, DNS │  ← Your code lives here
├─────────────────────────────────────────┤
│  6. Presentation  │ TLS/SSL, JPEG, JSON  │  ← Encoding & encryption
├─────────────────────────────────────────┤
│  5. Session       │ Sessions, Auth tokens│  ← Managing connections
├─────────────────────────────────────────┤
│  4. Transport     │ TCP, UDP             │  ← Reliable delivery
├─────────────────────────────────────────┤
│  3. Network       │ IP, ICMP, Routers    │  ← Routing packets
├─────────────────────────────────────────┤
│  2. Data Link     │ Ethernet, MAC, WiFi  │  ← Local delivery
├─────────────────────────────────────────┤
│  1. Physical      │ Cables, Radio waves  │  ← Actual wire/signal
└─────────────────────────────────────────┘
```

### What Backend Developers Care About Most

|Layer|Why You Care|
|---|---|
|**Layer 7 (Application)**|HTTP methods, headers, status codes, REST|
|**Layer 4 (Transport)**|TCP vs UDP, ports, connection pooling|
|**Layer 3 (Network)**|IP addresses, routing, VPCs on AWS/GCP|

### Real Example — Sending a POST request

```
You write:  POST /users HTTP/1.1   ← Layer 7 (Application)
            Body: {"name": "Ram"}
                        ↓
Encrypted:  TLS wraps your data    ← Layer 6 (Presentation)
                        ↓
TCP splits it into segments        ← Layer 4 (Transport)
Port 443 assigned
                        ↓
IP adds source/dest address        ← Layer 3 (Network)
                        ↓
WiFi sends via radio waves         ← Layer 1 (Physical)
```

---

## 3. TCP/IP — The Internet's Foundation

**TCP/IP** is the actual protocol suite that powers the internet. It's a simplified 4-layer version of OSI.

```
┌──────────────────────────────────┐
│  Application Layer   (HTTP, DNS) │
├──────────────────────────────────┤
│  Transport Layer     (TCP, UDP)  │
├──────────────────────────────────┤
│  Internet Layer      (IP)        │
├──────────────────────────────────┤
│  Network Access      (Ethernet)  │
└──────────────────────────────────┘
```
### The TCP 3-Way Handshake
Before any HTTP data is sent, TCP establishes a connection:
```
Client                          Server
  │                               │
  │──── SYN ─────────────────────▶│   "Hey, can we talk?"
  │                               │
  │◀─── SYN-ACK ──────────────────│   "Yes! Ready!"
  │                               │
  │──── ACK ─────────────────────▶│   "Great, let's go!"
  │                               │
  │═══ Data Transfer Begins ══════│
```

### Real Example — cURL with verbose TCP info

```bash
curl -v https://api.example.com/users

# Output shows:
# * Trying 93.184.216.34:443...     ← TCP connect attempt
# * Connected to api.example.com    ← 3-way handshake done
# * SSL handshake complete          ← TLS layer
# > GET /users HTTP/2               ← HTTP request sent
# < HTTP/2 200                      ← Response received
```

---

## 4. IP Addresses & Subnetting

An **IP address** is like a home address for a computer on a network.

### IPv4 vs IPv6

```
IPv4:   192.168.1.100        (32-bit, ~4 billion addresses)
IPv6:   2001:0db8::1         (128-bit, virtually unlimited)
```
- Each device (phone, laptop, router) needs a **unique IP address**
- IPv4 can only create about **4.3 billion unique “numbers”**
- But the world has **more than 4.3 billion devices**, so we ran out
### How devices “decide” which IP to use
Devices don’t randomly pick addresses. Usually:
- A **router (DHCP server)** automatically assigns IPs
- Example:
    - Your phone connects to Wi-Fi
    - Router says: “You are `192.168.1.100`”
### Public vs Private IPs

```
Private (internal only):          Public (internet-facing):
  10.0.0.0    – 10.255.255.255     Your actual internet IP
  172.16.0.0  – 172.31.255.255     e.g., 203.45.112.78
  192.168.0.0 – 192.168.255.255
```
### 🏠 Private IP (inside your home/network)
- Used **only inside your local network (Wi-Fi, office, etc.)**
- Example: `192.168.1.100`
- Assigned by your **router**
- **Not visible on the internet**
👉 Analogy:  
Your **room number inside a hotel** (Room 101, 102…)  
People outside the hotel **can’t see or use i**
### 🌍 Public IP (internet-facing)
- Used to identify your network **on the internet**
- Example: `203.45.112.78`
- Assigned by your **ISP (internet provider)**
- **Visible to websites and servers**
👉 Analogy:  
The **hotel’s street address** (e.g., “123 Main Street”)  
This is what the outside world sees

> Your home WiFi gives devices `192.168.x.x` — these never reach the internet directly. NAT(Network Address Translation) converts them (covered in Section 13).
### 🔄 How NAT connects them
NAT = **Network Address Translation**
Your router acts like a middleman:
1. Your phone (private IP `192.168.1.100`) sends a request (e.g., open Google)
2. Router **replaces** that private IP with your **public IP**
3. Google replies to the public IP
4. Router sends the response back to your phone
### Subnetting (CIDR Notation)

```
192.168.1.0/24  means:
  Network:  192.168.1.xxx   (the /24 means first 24 bits are fixed)
  Hosts:    192.168.1.1 → 192.168.1.254  (254 usable IPs)
  
10.0.0.0/16 means:
  Hosts:    10.0.0.1 → 10.0.255.254  (65,534 usable IPs)
```

### Real Example — AWS VPC Subnets

```
VPC: 10.0.0.0/16  (your entire cloud network)
  ├── Public Subnet:  10.0.1.0/24   (web servers, load balancers)
  ├── Private Subnet: 10.0.2.0/24   (app servers, no public access)
  └── DB Subnet:      10.0.3.0/24   (databases, most restricted)
```

### Check Your IP (Terminal)

```bash
# Your local IP
ip addr show          # Linux
ipconfig              # Windows

# Your public IP
curl ifconfig.me

# Trace the route to a server
traceroute google.com
```

---

## 5. DNS — Domain Name System

DNS is the internet's **phone book**. It converts human-readable names into IP addresses.

```
You type:    github.com
DNS returns: 140.82.121.4
Browser connects to: 140.82.121.4:443
```

### DNS Resolution Flow

```
Browser                Resolver          Root NS       .com NS      github NS
   │                      │                 │              │             │
   │── "github.com?" ────▶│                 │              │             │
   │                      │── "github.com?"▶│              │             │
   │                      │◀── "ask .com" ──│              │             │
   │                      │─────────── "github.com?" ────▶│             │
   │                      │◀────────── "ask github's NS" ──│             │
   │                      │──────────────────── "github.com?" ─────────▶│
   │                      │◀──────────────────── "140.82.121.4" ─────────│
   │◀──── 140.82.121.4 ───│
```

### DNS Record Types

|Record|Purpose|Example|
|---|---|---|
|**A**|Domain → IPv4|`api.example.com → 93.184.216.34`|
|**AAAA**|Domain → IPv6|`api.example.com → 2606:2800::1`|
|**CNAME**|Domain → Domain alias|`www.example.com → example.com`|
|**MX**|Mail server|`example.com → mail.example.com`|
|**TXT**|Arbitrary text (used for verification)|SPF, DKIM records|
|**NS**|Nameserver for domain|`example.com → ns1.cloudflare.com`|

### Real Example — Lookup DNS Records

```bash
# Lookup A record
nslookup github.com
dig github.com A

# Lookup MX record (mail servers)
dig gmail.com MX

# Trace full DNS resolution
dig +trace github.com

# Output:
# github.com.    60  IN  A  140.82.121.4
```

### DNS Caching & TTL

```
TTL (Time To Live) = how long DNS result is cached

dig github.com A | grep TTL
# github.com. 57 IN A 140.82.121.4
#             ^^
#         57 seconds left in cache
```

> **Backend Tip:** When you deploy to a new server and change DNS, set a low TTL (60s) before the change, then increase it after (3600s).

---

## 6. HTTP & HTTPS

**HTTP (HyperText Transfer Protocol)** is the language clients and servers use to communicate.

### HTTP Request Structure

```
POST /api/users HTTP/1.1                ← Method + Path + Version
Host: api.example.com                   ←┐
Content-Type: application/json          │  Headers
Authorization: Bearer eyJhb...          │
Content-Length: 27                      ←┘
                                        ← Empty line (separates headers/body)
{"name": "Ram", "age": 25}             ← Body (optional)
```

### HTTP Response Structure

```
HTTP/1.1 201 Created                    ← Version + Status Code + Message
Content-Type: application/json         ←┐
X-Request-Id: abc-123                  │  Response Headers
Date: Mon, 22 Apr 2026 10:00:00 GMT   ←┘
                                        ← Empty line
{"id": 42, "name": "Ram", "age": 25}   ← Response Body
```

### HTTP Methods

|Method|Purpose|Has Body?|Idempotent?|
|---|---|---|---|
|`GET`|Retrieve data|No|✅ Yes|
|`POST`|Create resource|Yes|❌ No|
|`PUT`|Replace resource|Yes|✅ Yes|
|`PATCH`|Partially update|Yes|❌ No|
|`DELETE`|Remove resource|No|✅ Yes|
|`HEAD`|GET without body|No|✅ Yes|
|`OPTIONS`|Check CORS/methods|No|✅ Yes|

### HTTP Status Codes

```
1xx → Informational
  100 Continue

2xx → Success
  200 OK
  201 Created
  204 No Content

3xx → Redirect
  301 Moved Permanently
  302 Found (temporary redirect)
  304 Not Modified (use cached version)

4xx → Client Error
  400 Bad Request        (malformed JSON, missing fields)
  401 Unauthorized       (no/bad auth token)
  403 Forbidden          (authenticated but no permission)
  404 Not Found
  409 Conflict           (duplicate email, etc.)
  422 Unprocessable      (validation failed)
  429 Too Many Requests  (rate limited)

5xx → Server Error
  500 Internal Server Error
  502 Bad Gateway        (upstream server failed)
  503 Service Unavailable (overloaded or down)
  504 Gateway Timeout    (upstream took too long)
```

### HTTP Versions

```
HTTP/1.1  → One request per connection (keep-alive helps)
HTTP/2    → Multiplexed (many requests over one TCP connection, faster)
HTTP/3    → Uses QUIC (UDP-based, even faster, especially on mobile)
```

```bash
# Check which HTTP version a server uses
curl -I --http2 https://github.com
# HTTP/2 200
```

### Headers You'll Use Every Day

```http
# Authentication
Authorization: Bearer <jwt_token>
Authorization: Basic dXNlcjpwYXNz   (base64 of user:pass)

# Content negotiation
Content-Type: application/json
Accept: application/json

# CORS
Access-Control-Allow-Origin: https://yourfrontend.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE

# Caching
Cache-Control: no-cache
ETag: "33a64df5"
If-None-Match: "33a64df5"

# Rate limiting (custom, common convention)
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1714000000
```

---

## 7. TCP vs UDP

|Feature|TCP|UDP|
|---|---|---|
|Connection|Requires handshake|No handshake|
|Delivery|Guaranteed, ordered|Not guaranteed|
|Speed|Slower|Faster|
|Use case|HTTP, databases, files|Video streaming, DNS, gaming|
|Error checking|Yes, retransmits lost packets|No retransmit|

### Real Example

```
TCP — Downloading a file:
  Client: "Give me bytes 1-1000"
  Server: sends bytes 1-1000
  (packet 500-600 gets lost)
  Client: "I didn't get 500-600, resend!"
  Server: resends 500-600
  ✅ File is 100% complete

UDP — Live video stream:
  Server: sends frames continuously
  (a few frames get lost)
  Client: just skips them
  ✅ Video keeps playing (minor glitch)
  ← Waiting for retransmit would cause worse lag
```

### When You'll Use UDP as a Backend Dev

```
- WebRTC (video calls, voice, peer-to-peer)
- DNS queries (fast lookup, small data)
- Game servers (real-time position updates)
- HTTP/3 (built on QUIC, which uses UDP)
- Logging pipelines (can afford some loss)
```

---

## 8. Ports & Sockets

A **port** is like a door on a server. One IP address can run many services on different ports.
### Well-Known Ports

|Port|Protocol|Service|
|---|---|---|
|22|TCP|SSH|
|25|TCP|SMTP (email)|
|53|UDP/TCP|DNS|
|80|TCP|HTTP|
|443|TCP|HTTPS|
|3306|TCP|MySQL|
|5432|TCP|PostgreSQL|
|6379|TCP|Redis|
|27017|TCP|MongoDB|
|8080|TCP|HTTP (alternative / dev)|

### Socket = IP + Port

```
A socket uniquely identifies a connection:

Client socket:  192.168.1.5:52341   (random ephemeral port)
Server socket:  93.184.216.34:443

Connection = (client IP, client port, server IP, server port)
```

### Real Example — Check What's Running on Your Ports

```bash
# See all listening ports
ss -tlnp          # Linux (modern)
netstat -tlnp     # Linux (older)

# Output:
# State   Recv-Q  Send-Q  Local Address:Port
# LISTEN  0       128     0.0.0.0:3000     ← your Node.js app
# LISTEN  0       128     0.0.0.0:5432     ← PostgreSQL
# LISTEN  0       128     0.0.0.0:6379     ← Redis

# Test if a port is open on a remote host
nc -zv api.example.com 443
telnet api.example.com 80
```

### Binding to 0.0.0.0 vs 127.0.0.1

```
127.0.0.1  = Only accept connections from THIS machine (loopback)
0.0.0.0    = Accept connections from ANY network interface

# In Node.js:
app.listen(3000, '127.0.0.1')  // only local access
app.listen(3000, '0.0.0.0')    // accessible from network
app.listen(3000)               // defaults to 0.0.0.0
```

---

## 9. REST APIs over HTTP

**REST (Representational State Transfer)** is a design style for APIs using HTTP.

### Core Principles

```
1. Stateless       — Server doesn't remember client between requests
2. Resource-based  — URLs represent nouns, not verbs
3. HTTP methods    — Use GET/POST/PUT/DELETE for actions
4. Uniform interface — Consistent URL patterns
```

### URL Design

```
❌ Bad (verb-based):
  GET  /getUsers
  POST /createUser
  POST /deleteUser?id=5

✅ Good (resource-based):
  GET    /users          → list all users
  POST   /users          → create a user
  GET    /users/42       → get user #42
  PUT    /users/42       → replace user #42
  PATCH  /users/42       → partial update user #42
  DELETE /users/42       → delete user #42

  GET    /users/42/posts → posts belonging to user #42
  POST   /users/42/posts → create post for user #42
```

### Real Example — Full API Interaction

```bash
# Create a user
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGci..." \
  -d '{"name": "Sita", "email": "sita@example.com"}'

# Response: 201 Created
# {"id": 7, "name": "Sita", "email": "sita@example.com", "createdAt": "2026-04-22T10:00:00Z"}

# Get the user
curl https://api.example.com/users/7 \
  -H "Authorization: Bearer eyJhbGci..."

# Response: 200 OK
# {"id": 7, "name": "Sita", ...}

# Update email only (PATCH)
curl -X PATCH https://api.example.com/users/7 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGci..." \
  -d '{"email": "newemail@example.com"}'

# Delete
curl -X DELETE https://api.example.com/users/7 \
  -H "Authorization: Bearer eyJhbGci..."
# Response: 204 No Content
```

### Query Parameters

```
Filtering:   GET /users?role=admin&active=true
Pagination:  GET /users?page=2&limit=20
Sorting:     GET /users?sort=createdAt&order=desc
Search:      GET /users?q=ram
Fields:      GET /users?fields=id,name,email
```

### CORS — Cross-Origin Resource Sharing

```
Problem:
  Frontend: https://myapp.com          (Origin A)
  Backend:  https://api.myapp.com      (Origin B = different subdomain!)
  
  Browser BLOCKS the request by default for security.

Solution (backend must add headers):
  Access-Control-Allow-Origin: https://myapp.com
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  Access-Control-Allow-Headers: Content-Type, Authorization

Preflight (OPTIONS request):
  Browser first sends OPTIONS to check if CORS is allowed,
  then sends the real request.
```

```javascript
// Express.js CORS setup
const cors = require('cors');

app.use(cors({
  origin: 'https://myapp.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## 10. WebSockets — Real-Time Communication

**WebSockets** provide a persistent, full-duplex connection. Unlike HTTP (request → response), both sides can send messages anytime.

### HTTP vs WebSocket

```
HTTP (request-response):
  Client ──── GET /messages ────▶ Server
  Client ◀─── 200 [{...}] ─────── Server
  (must keep polling for new messages)

WebSocket (persistent):
  Client ══════ ws://chat.com ═══════ Server
         ──── "Hello!" ───────────▶
         ◀─── "Hi there!" ──────────
         ──── "How are you?" ──────▶
         ◀─── "Great!" ─────────────
  (connection stays open, real-time!)
```

### WebSocket Handshake

```http
# Client upgrades the HTTP connection:
GET /chat HTTP/1.1
Host: chat.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZQ==
Sec-WebSocket-Version: 13

# Server responds:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### Real Example — Node.js WebSocket Server

```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws, req) => {
  console.log(`Client connected from ${req.socket.remoteAddress}`);

  ws.on('message', (data) => {
    const message = JSON.parse(data);
    console.log('Received:', message);

    // Broadcast to all connected clients
    wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({
          user: message.user,
          text: message.text,
          time: new Date().toISOString()
        }));
      }
    });
  });

  ws.on('close', () => console.log('Client disconnected'));
});
```

### When to Use WebSockets vs HTTP

|Use WebSocket|Use HTTP|
|---|---|
|Live chat|CRUD operations|
|Real-time dashboards|File uploads|
|Multiplayer games|Authentication|
|Live sports scores|Fetching static data|
|Collaborative editing|Webhooks|

---

## 11. Load Balancers & Reverse Proxies

### Reverse Proxy

A **reverse proxy** sits in front of your backend and forwards requests.

```
Client ──────▶ Reverse Proxy (Nginx) ──────▶ Backend App (Node.js :3000)
```

**Benefits:**

- SSL termination (handles HTTPS so app doesn't have to)
- Caching static files
- Rate limiting
- Hiding internal IPs
- Routing (path-based, host-based)

### Load Balancer

Distributes traffic across multiple backend instances:

```
                         ┌──▶ Server 1 (10.0.0.1:3000)
Client ──▶ Load Balancer ├──▶ Server 2 (10.0.0.2:3000)
                         └──▶ Server 3 (10.0.0.3:3000)
```

### Load Balancing Algorithms

```
Round Robin:    1→2→3→1→2→3   (equal distribution)
Least Connections: send to server with fewest active connections
IP Hash:        same client always goes to same server (sticky sessions)
Weighted:       Server1 gets 50%, Server2 gets 30%, Server3 gets 20%
```

### Real Example — Nginx Config

```nginx
# Reverse proxy + load balancer
upstream backend_servers {
    least_conn;                       # algorithm
    server 10.0.0.1:3000 weight=3;
    server 10.0.0.2:3000 weight=2;
    server 10.0.0.3:3000 weight=1;
}

server {
    listen 80;
    server_name api.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate     /etc/ssl/certs/example.crt;
    ssl_certificate_key /etc/ssl/private/example.key;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static/ {
        root /var/www;               # serve static files directly
        expires 7d;
    }
}
```

---

## 12. CDN — Content Delivery Network

A CDN stores copies of your content on servers around the world (edge nodes). Users get content from the closest server.

```
Without CDN:
  User in Kathmandu ──────────── 200ms ──────────── Server in US

With CDN:
  User in Kathmandu ── 10ms ── CDN Edge (India/Singapore)
                                    (cached copy of content)
```

### What CDNs Cache

```
✅ Good for CDN:
  - Images, videos, fonts
  - JS/CSS bundles
  - API responses (with proper Cache-Control headers)
  - HTML pages (for static sites)

❌ Not for CDN (dynamic per-user):
  - User dashboard data
  - Shopping cart
  - Auth endpoints
```

### Cache-Control Headers

```http
# Cache for 1 day on CDN and browser
Cache-Control: public, max-age=86400

# Cache on CDN only (not browser), 1 hour
Cache-Control: public, s-maxage=3600, max-age=0

# Never cache (authenticated, private data)
Cache-Control: private, no-store

# Revalidate each time
Cache-Control: no-cache
ETag: "abc123"
```

### Real Example — CDN Cache Invalidation

```bash
# Cloudflare: purge a specific file
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/ZONE_ID/purge_cache" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"files": ["https://example.com/logo.png"]}'
```

---

## 13. Firewalls, NAT & Security Basics

### Firewall

A firewall filters network traffic based on rules.

```
Internet ──▶ Firewall ──▶ Your Server

Firewall rules:
  ALLOW port 443 (HTTPS) from anywhere
  ALLOW port 22  (SSH)   from 203.45.x.x only (your IP)
  DENY  port 5432 (PostgreSQL) from anywhere
  DENY  all other inbound traffic
```

### NAT (Network Address Translation)

Allows many private IP devices to share one public IP:

```
Home Network:
  Laptop    192.168.1.2  ─┐
  Phone     192.168.1.3  ─┼──▶ Router ──▶ Internet (203.45.12.78)
  Smart TV  192.168.1.4  ─┘   (NAT)

Outgoing packet:
  Source:  192.168.1.2:52341 → translated to → 203.45.12.78:52341
  
Return packet:
  Dest: 203.45.12.78:52341 → translated back → 192.168.1.2:52341
```

### Security Groups (AWS example)

```
Inbound rules for Web Server:
  Port 80    (HTTP)   from 0.0.0.0/0  (public)
  Port 443   (HTTPS)  from 0.0.0.0/0  (public)
  Port 22    (SSH)    from 10.0.0.0/8 (internal only)

Inbound rules for DB Server:
  Port 5432  (PostgreSQL)  from Web Server Security Group only
  Port 22    (SSH)         from Bastion Host only
  # No public access at all!
```

### Real Example — Linux iptables / ufw

```bash
# UFW (simple firewall for Ubuntu)

# Allow SSH
ufw allow 22/tcp

# Allow HTTP and HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Allow PostgreSQL from internal network only
ufw allow from 10.0.0.0/24 to any port 5432

# Enable firewall
ufw enable

# Check rules
ufw status verbose
```

---

## 14. TLS/SSL — How HTTPS Works

**TLS (Transport Layer Security)** encrypts data between client and server. HTTPS = HTTP + TLS.

### The TLS Handshake (Simplified)

```
Client                                  Server
  │                                       │
  │──── ClientHello ─────────────────────▶│  "I support TLS 1.3, AES-256"
  │                                       │
  │◀─── ServerHello + Certificate ────────│  "Here's my SSL cert"
  │                                       │
  │  (Client verifies cert with CA)       │
  │                                       │
  │──── Key Exchange ────────────────────▶│  Establish shared secret
  │                                       │
  │◀────────── Encrypted Data ────────────│
  │─────────── Encrypted Data ───────────▶│
```

### SSL Certificate Chain

```
Root CA (DigiCert, Let's Encrypt, etc.)
  └── Intermediate CA
        └── Your Domain Certificate (api.example.com)

Browser trusts Root CA → trusts Intermediate → trusts your cert ✅
```

### Real Example — Get Free SSL with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate (auto-configures Nginx)
sudo certbot --nginx -d api.example.com -d www.api.example.com

# Auto-renew (Let's Encrypt certs expire in 90 days)
sudo certbot renew --dry-run

# Check certificate details
openssl s_client -connect api.example.com:443 < /dev/null \
  | openssl x509 -noout -dates -subject
```

### Check a Site's TLS

```bash
# See certificate info
curl -vI https://github.com 2>&1 | grep -E "SSL|TLS|issuer|expire"

# Test TLS version support
nmap --script ssl-enum-ciphers -p 443 github.com
```

---
## 15. Common Backend Networking Patterns
### Connection Pooling
Opening a new DB connection for every request is slow. Pool reuses connections:

```javascript
// PostgreSQL with pg pool (Node.js)
const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  user: 'admin',
  password: 'secret',
  max: 20,                // max 20 connections in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Reuses connections from pool
const result = await pool.query('SELECT * FROM users WHERE id = $1', [42]);
```

### Rate Limiting

Protect your API from abuse:

```javascript
// Express rate limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // max 100 requests per window
  standardHeaders: true,      // adds X-RateLimit-* headers
  message: { error: 'Too many requests, please try again later.' }
});

app.use('/api/', limiter);
```

### Health Check Endpoint

Used by load balancers and container orchestrators:

```javascript
app.get('/health', async (req, res) => {
  try {
    await pool.query('SELECT 1');     // Check DB
    await redis.ping();               // Check Redis
    res.json({
      status: 'healthy',
      db: 'connected',
      redis: 'connected',
      uptime: process.uptime()
    });
  } catch (err) {
    res.status(503).json({ status: 'unhealthy', error: err.message });
  }
});
```

### Retry with Exponential Backoff

Network calls fail sometimes. Retry intelligently:

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return await response.json();
    } catch (err) {
      if (attempt === maxRetries) throw err;
      
      const delay = Math.pow(2, attempt) * 100;  // 200ms, 400ms, 800ms
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Circuit Breaker Pattern

Stop hammering a failing service:

```
CLOSED → (normal operation, requests go through)
   ↓ (too many failures)
OPEN   → (requests fail immediately, don't hit service)
   ↓ (after timeout, try again)
HALF-OPEN → (allow one request through as a test)
   ↓ success      ↓ failure
CLOSED          OPEN
```

### Webhook Pattern

Server-to-server callbacks for async events:

```
Your Backend                        Stripe
     │                                │
     │ ─── POST /charge ─────────────▶│
     │ ◀─── 202 Accepted ─────────────│  (async, processing)
     │                                │
     │ (minutes later...)             │
     │                                │
     │ ◀─── POST /webhook/stripe ─────│  Stripe calls YOU back
     │      {"event":"charge.succeeded",│
     │       "amount": 5000}          │
     │ ─── 200 OK ────────────────────▶│
```

```javascript
// Verify webhook signature (Stripe example)
app.post('/webhook/stripe', express.raw({type: 'application/json'}), (req, res) => {
  const sig = req.headers['stripe-signature'];
  let event;
  
  try {
    event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET);
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }
  
  if (event.type === 'payment_intent.succeeded') {
    const paymentIntent = event.data.object;
    // fulfill the order
  }
  
  res.json({ received: true });
});
```

---

## 🗺️ Quick Reference — Commands Cheat Sheet

```bash
# ── DNS ──────────────────────────────────────────────────────
nslookup github.com              # basic DNS lookup
dig github.com A                 # DNS A record
dig github.com MX                # DNS mail records
dig +trace github.com            # full DNS resolution trace

# ── Connectivity ─────────────────────────────────────────────
ping google.com                  # test reachability
traceroute google.com            # show hops to destination
curl -I https://example.com      # fetch HTTP headers only
curl -v https://example.com      # verbose (shows TLS, headers)

# ── Ports & Sockets ──────────────────────────────────────────
ss -tlnp                         # show listening ports (Linux)
netstat -an | grep LISTEN        # older alternative
nc -zv example.com 443           # test if port is open
lsof -i :3000                    # what's using port 3000

# ── HTTP Testing ─────────────────────────────────────────────
curl -X POST https://api.ex.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}'

# ── TLS / SSL ────────────────────────────────────────────────
openssl s_client -connect example.com:443
curl -vI https://example.com 2>&1 | grep SSL

# ── Network Info ─────────────────────────────────────────────
ip addr show                     # your IP addresses
ip route show                    # routing table
cat /etc/resolv.conf             # your DNS servers
```

---

## 🎯 Summary — What Every Backend Dev Must Know

|Concept|Why You Need It|
|---|---|
|**HTTP methods & status codes**|Building and consuming REST APIs|
|**TCP/IP & ports**|Configuring servers, debugging connections|
|**DNS**|Deployments, custom domains, debugging|
|**TLS/HTTPS**|Every production app must use it|
|**Load balancers**|Scaling your app horizontally|
|**CORS**|Any frontend will hit your API|
|**Firewalls/Security Groups**|Deploying to cloud (AWS, GCP)|
|**WebSockets**|Real-time features (chat, notifications)|
|**CDN & caching headers**|Performance optimization|
|**Connection pooling**|Database performance|

---

_Happy building! 🚀 — Network knowledge makes you a 10x better backend developer._