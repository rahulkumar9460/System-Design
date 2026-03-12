# TCP and UDP
---

# 1. TCP Overview

**TCP (Transmission Control Protocol)** is a transport layer protocol that runs on top of IP and provides **reliable, ordered communication** between applications.

Key responsibilities:

- Reliable data delivery
- Ordered packets
- Retransmission of lost packets
- Flow control (prevents receiver overload)
- Congestion control (prevents network overload)

**Protocols that use TCP:**

- HTTP / HTTPS
- FTP
- SMTP
- Telnet

---

# 2. UDP Overview

**UDP (User Datagram Protocol)** is a transport layer protocol designed for **fast, connectionless communication**.

Characteristics:

- Connectionless (no handshake)
- Minimal overhead
- Low latency
- No guarantee of delivery or ordering

**Applications that use UDP:**

- Video calls (Zoom, Teams)
- Online gaming (Fortnite, PUBG)
- Live streaming
- DNS queries

---

# 3. TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed | Not guaranteed |
| Ordering | Yes | No |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Speed | Slower | Faster |
| Use Cases | Web, Files, Email | Gaming, Streaming, Calls |

---

# 4. TCP Three-Way Handshake

Before data transfer, TCP establishes a connection using a **three-way handshake**.

**Steps:**

```
Client → SYN --------------------> Server
Client waits

Server → SYN + ACK <------------- Client
Server waits

Client → ACK --------------------> Server
Connection Established
```

**Purpose:**

1. Verify both client and server are alive
2. Synchronize sequence numbers
3. Ensure reliable communication

---

# 5. TCP ACK (Acknowledgment)

**ACK** confirms that a receiver successfully received data.

**Example:**

```
Sender → Packet (Seq=100)
Receiver → ACK=101
```

This means: "I received bytes up to 100. Please send from 101 next."

**If ACK is not received:**

- TCP retransmits the packet
- Ensures reliability over an unreliable IP network

**ASCII Diagram:**

```
Sender                     Receiver
Packet 1  -------------------->
Packet 2  -------------------->
           <-------------------- ACK=3
Packet 3  -------------------->
           <-------------------- ACK=4
```

---

# 6. TCP Flow Control

Flow control prevents the **sender from overwhelming the receiver**.

**Mechanism: Receive Window (rwnd)**

- Receiver advertises available buffer space
- Sender sends **up to the window size**

**Example:**

```
Receiver buffer size = 10 KB
Already used = 6 KB
Available = 4 KB
```

Receiver sends:

```
ACK + Window Size = 4 KB
```

**ASCII Diagram:**

```
Sender                        Receiver
Packet 1 -------------------->
Packet 2 -------------------->
Packet 3 -------------------->
             Buffer filling
ACK + Window Size = 2 packets <-------
Sender slows down
```

---

# 7. TCP Congestion Control

Congestion control prevents the **network from being overloaded**.

**How it happens:**

- Too many packets in network → router buffer overflow → packet loss
- TCP detects congestion via:
  - Packet loss / timeout
  - Duplicate ACKs

**How TCP reacts:**

- Maintains **congestion window (cwnd)**
- Uses algorithms:
  - Slow Start
  - Congestion Avoidance
  - Fast Retransmit
  - Fast Recovery

**ASCII Diagram (simplified):**

```
Sender Rate ↑
        │
        │    Packet Loss
        │      ↓
        │     /\
        │    /  \  reduce rate
        │   /    \
        │  /
        └────────────── Time
```

- Sender gradually increases sending rate when network is free
- Reduces quickly on congestion detection

---

# 8. When to Use TCP

Use TCP when **reliability, ordered delivery, and data integrity** are critical.

**Examples:**

1. Web browsing (HTTP/HTTPS)
2. File transfer (FTP)
3. Email (SMTP)
4. Banking / transactions

Reason: **Missing or unordered data is unacceptable**.

---

# 9. When to Use UDP

Use UDP when **low latency is more important than reliability**.

**Examples:**

1. Video calls (Zoom, Teams)
2. Online gaming (Fortnite, PUBG)
3. Live streaming
4. DNS queries

Reason: **Acceptable to lose packets to maintain speed and reduce delay**.

---

# 10. Summary ASCII Diagram — TCP vs UDP Flow

**TCP Connection Flow:**

```
Client                     Server
SYN ------------------------>
          <------------------ SYN+ACK
ACK ------------------------>
Data1 -----------------------> 
          <------------------ ACK
Data2 -----------------------> 
          <------------------ ACK
```

**UDP Connectionless Flow:**

```
Client                     Server
Data1 -----------------------> 
Data2 -----------------------> 
Data3 -----------------------> 
No ACK required
```

---

# Key Takeaways

- TCP ensures **reliable, ordered, and congestion-aware communication**
- UDP provides **fast, connectionless, low-latency communication**
- TCP handshake establishes a connection; ACKs ensure delivery
- Flow control protects the receiver; congestion control protects the network
- Use TCP for **web, files, email**; use UDP for **real-time, latency-sensitive apps**

---