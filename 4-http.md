# Networking Topic 3 — HTTP / HTTPS (Comprehensive)
---

# 1. What is HTTP?

**HTTP (Hypertext Transfer Protocol)** is an **application layer protocol** used for communication between clients (browsers) and servers.

Key points:

- Defines how requests and responses are formatted and transmitted
- Stateless: each request is independent
- Used to fetch resources like web pages, images, scripts, and APIs
- Security (authentication/encryption) is added via **HTTPS**  

**Example:** Browsing a website, calling a REST API

**ASCII Diagram — Request/Response Flow:**

```
Client (Browser)                     Server
-----------------                    -----------------
GET /index.html -------------------->
                                   Processes request
                                   Returns response
<-------------------- 200 OK + HTML
Renders page
```

---

# 2. HTTP Methods

| Method | Purpose | Idempotent? |
|--------|---------|-------------|
| GET | Retrieve a resource | Yes |
| POST | Create a new resource | No |
| PUT | Replace a resource entirely (or create if not exists) | Yes |
| PATCH | Partially update a resource | No |
| DELETE | Remove a resource | Yes |

**ASCII Diagram — CRUD Example:**

```
Client                         Server
GET /users -------------------->
                              Returns list of users

POST /users ------------------->
   Body: {name: "Rahul"}       Creates new user

PUT /users/1 ------------------>
   Body: {name: "Rahul Kumar"} Replaces user #1

PATCH /users/1 ---------------->
   Body: {name: "Rahul K"}     Updates only name field

DELETE /users/1 ---------------> 
                              Deletes user #1
```

---

# 3. Statelessness

- HTTP is **stateless**: server does not retain information about previous requests
- Each request is independent
- Makes scaling easier but requires mechanisms like cookies or tokens for maintaining user sessions

**ASCII Diagram — Stateless Request Flow:**

```
Client                     Server
-----------------          -----------------
GET /profile ---------------->
                             Returns profile data
GET /dashboard ---------------> 
                             Returns dashboard data

Note: Server treats each request independently
```

---

# 4. HTTP Headers

- Carry **metadata** about requests or responses
- Examples: Authorization, Content-Type, Content-Length, Cache-Control
- Help the client and server handle data properly (authentication, caching, content negotiation, etc.)

**Common HTTP Status Codes:**

| Code | Meaning |
|------|---------|
| 200 | OK – Request succeeded |
| 202 | Accepted – Request accepted but not yet processed |
| 301 / 302 | Redirect |
| 304 | Not Modified – cached resource still valid |
| 404 | Not Found |
| 500 | Internal Server Error |

**ASCII Diagram — Request & Response with Headers:**

```
Client                        Server
------                        ------
GET /profile HTTP/1.1 -------->
Headers: 
  Authorization: Bearer xyz
  Accept: application/json
  Content-Type: application/json

                              Processes request
                              Response:
                              Status: 200 OK
                              Headers:
                                Content-Type: application/json
                                Content-Length: 256
                              Body: { "name": "Rahul" }
<---------------------------- Response
```

---

# 5. HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Security | None | Encryption, Authentication, Integrity |
| Port | 80 | 443 |
| Transport | TCP | TCP + TLS |
| Use | Public websites (no sensitive data) | Secure websites, online banking, APIs |

**HTTPS Handshake (after TCP connection):**

```
Client                         Server
---------                      ---------
TCP 3-Way Handshake:
SYN ------------------------>
          <----------------- SYN+ACK
ACK ------------------------>

TLS Handshake:
Client Hello ---------------->
          <----------------- Server Hello + Certificate
Client Key Exchange --------->
Finished -------------------->
          <----------------- Finished
Secure Connection Established
```

---

# 6. HTTP Versions

| Version | Key Features |
|---------|-------------|
| HTTP/1.1 | Mostly sequential requests, persistent connections (keep-alive) |
| HTTP/2 | Multiplexing multiple requests/responses over one connection, header compression (HPACK) |
| HTTP/3 | Uses QUIC over UDP, faster handshake, no TCP head-of-line blocking |

**ASCII Diagram — Request Multiplexing:**

```
HTTP/1.1 (TCP):
GET /a -------------------->
          <----------------- Response A
GET /b -------------------->
          <----------------- Response B

HTTP/2 (TCP Multiplexing):
GET /a -------------------->
GET /b -------------------->
          <----------------- Response A
          <----------------- Response B

HTTP/3 (QUIC over UDP):
GET /a -------------------->
GET /b -------------------->
          <----------------- Response A
          <----------------- Response B
```

---

# Key Takeaways

- HTTP is **stateless, request-response protocol** at the application layer  
- Methods (GET, POST, PUT, PATCH, DELETE) define **CRUD operations**  
- Headers provide **metadata and instructions**  
- Status codes indicate **request outcomes**  
- HTTPS secures HTTP traffic using **TLS encryption**  
- HTTP/2 and HTTP/3 improve **performance and multiplexing**  

---