# WebSocket & RPC

---

# 1. WebSocket Overview

**WebSocket** is a protocol that enables **full-duplex, bidirectional communication** between client and server over a single TCP connection.

Key points:

- Starts as an **HTTP/HTTPS request** with `Upgrade: websocket` header
- Connection **switches from HTTP to WebSocket** after handshake
- Server can **push data to clients** without client requests
- Low latency and persistent connection make it ideal for **real-time applications**

**Real-world examples:** Chat applications, live dashboards, multiplayer games, video/voice calls

---

# 2. WebSocket Handshake

**Step 1: HTTP Upgrade Request**

Client sends:

```
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: <key>
```

**Step 2: Server Accepts Upgrade**

Server responds:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: <key>
```

**Step 3: Persistent Connection Established**

- Connection switches from HTTP → WebSocket
- Both client and server can now **send messages anytime**

---

# ASCII Diagram — WebSocket Handshake & Communication

```
Client                     Server
------                     ------
HTTP GET /chat HTTP/1.1 -------------------->
Headers: Upgrade: websocket

                             HTTP 101 Switching Protocols <----------------
                             Connection upgraded to WebSocket

Persistent TCP connection established
Client <---> Server
Bidirectional messages flow freely
```

---

# 3. Advantages of WebSocket over Polling

1. **Reduces CPU & network overhead** – no repeated HTTP requests  
2. **Persistent connection** – single TCP connection reused  
3. **Server push support** – real-time updates without client polling  
4. **Lower latency** – messages sent immediately  
5. **Efficient resource usage** – less network traffic  

**ASCII Diagram — Polling vs WebSocket**

**Polling:**

```
Client                      Server
------                      ------
GET /updates ---------------->
                             Response: No new data
Wait 5 sec
GET /updates ---------------->
                             Response: New data
```

**WebSocket:**

```
Client                     Server
------                     ------
TCP + WebSocket handshake ---------------->
                            ---------------- 101 Switching Protocols

Persistent connection established
Server --> Client: New message
Server --> Client: Another message
```

---

# 4. WebSocket Use Cases

- **Chat applications** – instant messaging  
- **Video/voice calls** – low latency streaming  
- **Online multiplayer games** – real-time game state updates  
- **Live dashboards** – stock tickers, IoT updates  

**ASCII Diagram — Real-Time Chat Flow**

```
Client                     Server
------                     ------
WebSocket connection -------------------->
                             101 Switching Protocols

Chat message "Hi"  -------------------->
                             Delivered to Server
                             Broadcast to other clients
Other client receives "Hi"  <----------------
```

---

# 5. RPC Overview

**RPC (Remote Procedure Call)** allows a program to **execute functions on a remote server as if they were local functions**.

Key points:

- Client and server agree on a **shared contract** (proto files, IDL)
- Data is often **binary-encoded** (Protobuf, Thrift)
- Reduces **serialization overhead**, bandwidth, and latency
- Used for **microservice-to-microservice communication**

**Real-world example:** Communication between backend microservices

---

# 6. RPC vs HTTP API

| Feature | HTTP API | RPC |
|---------|---------|-----|
| Communication | Request-response (often textual, JSON/XML) | Function call semantics |
| Data format | Text (JSON/XML) | Binary (Protobuf/Thrift) |
| Latency | Higher (more overhead) | Lower (compact, binary) |
| Contract | Loose | Strongly typed, shared contract |
| Ease of use | Manual parsing | Call looks like local function |

---

# 7. RPC Call Flow

**Client calls remote function:**

```
Client                     Server
------                     ------
getUser(1) ---------------->
                             Server executes getUser(1)
                             Result: {name: "Rahul", age: 27}
<---------------------------- Response
Client receives return value
```

**Key Points:**

- Call appears **local to the client**  
- Under the hood, request is **serialized**, sent over network, deserialized by server  
- Supports **synchronous or asynchronous calls**  

---

# 8. RPC Advantages

1. **Reduced serialization/deserialization overhead** – shared proto/IDL contract  
2. **Low latency** – binary encoding, fewer round-trips  
3. **Reduced bandwidth** – smaller message sizes than JSON over HTTP  
4. **Strong typing & contract enforcement** – prevents request/response errors  
5. **Simplified client code** – call looks like a local function  

**ASCII Diagram — RPC vs HTTP API**

**HTTP API:**

```
Client                     Server
------                     ------
GET /getUser?id=1 -------->
                             Server processes request
                             Returns full JSON:
                             { "name": "Rahul", "age": 27 }
<---------------------------- Response
```

**RPC:**

```
Client                     Server
------                     ------
getUser(1) ---------------->
                             Server executes getUser(1)
                             Returns binary encoded response
<---------------------------- Response
Client receives return value as native object
```

---

# ✅ Summary — WebSocket & RPC

| Feature | WebSocket | RPC |
|---------|-----------|-----|
| Communication | Bidirectional, full-duplex | Remote function call |
| Connection | Persistent TCP | TCP/HTTP/HTTP2 (gRPC) |
| Latency | Low | Low (binary + contract) |
| Server Push | Yes | Only via request-response |
| Protocol Upgrade | HTTP → WebSocket | Usually binary or HTTP2 (gRPC) |
| Use Cases | Chat apps, live feeds, multiplayer games | Microservice communication, gRPC APIs |

---

