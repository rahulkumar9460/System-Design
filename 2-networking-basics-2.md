# Networking Basics — Round 4: Public IP vs Private IP

## Public IP Address

A **public IP address** is an IP address that is globally unique and accessible over the internet.  
It allows devices or servers to communicate directly with other systems on the public internet.

Public IPs are assigned by **Internet Service Providers (ISPs)**.

Example:

```
34.102.136.180
```

Servers that host public services usually have public IPs.

Examples:

- Web servers
- Cloud servers
- Public APIs
- Email servers

Any device with a public IP can be reached from anywhere on the internet.

---

# Private IP Address

A **private IP address** is used inside a **local network (LAN)** and cannot be accessed directly from the internet.

Private IPs allow devices inside a network to communicate with each other.

Examples of devices using private IPs:

- Laptops
- Smartphones
- Smart TVs
- Printers
- IoT devices

Example private IP:

```
192.168.1.10
```

Private IPs are not routable on the public internet.

---

# Reserved Private IP Ranges

Certain IP ranges are reserved specifically for private networks.

| Range | Usage |
|------|------|
| 10.0.0.0 – 10.255.255.255 | Large private networks |
| 172.16.0.0 – 172.31.255.255 | Medium private networks |
| 192.168.0.0 – 192.168.255.255 | Home and small networks |

Example home network:

```
Router: 192.168.1.1
Laptop: 192.168.1.10
Phone: 192.168.1.11
TV: 192.168.1.12
```

---

# Why Private IPs Exist

IPv4 addresses are limited.

```
2^32 ≈ 4.3 billion addresses
```

If every device on earth required a public IP, the address space would be exhausted very quickly.

Private IPs allow **millions of devices to share a single public IP address**.

---

# Network Address Translation (NAT)

To allow private devices to access the internet, routers use **Network Address Translation (NAT)**.

NAT translates a device's **private IP address into the router's public IP address** when sending traffic to the internet.

Example:

```
Laptop private IP: 192.168.1.10
Router public IP: 52.91.20.14
```

Outgoing request becomes:

```
52.91.20.14 → google.com
```

The router keeps a **NAT table** to remember which internal device initiated the request.

---

# NAT Translation Example

Private request:

```
192.168.1.10:53001 → google.com:443
```

Router translates to:

```
52.91.20.14:40001 → google.com:443
```

Response arrives:

```
google.com:443 → 52.91.20.14:40001
```

Router checks the NAT table and forwards it to:

```
192.168.1.10:53001
```

---

# Network Example

```
Internet
   │
   │
Public IP
52.91.20.14
   │
 Router
   │
 ┌────────────┬────────────┬────────────┐
192.168.1.10 192.168.1.11 192.168.1.12
Laptop        Phone        TV
```

All devices share **one public IP address**.

---

# Key Interview Takeaways

- Public IP addresses are **globally reachable on the internet**
- Private IP addresses are **used inside local networks**
- Private IPs cannot be routed on the public internet
- Routers use **NAT** to translate private IPs to public IPs
- Many devices can share **one public IP using NAT**

# Networking Basics — Round 5: Packets

## What is a Packet?

A **packet** is the smallest unit of data transmitted across a network.

When large data (like a file, webpage, or video) is sent over the internet, it is **broken into smaller packets** so the network can deliver it efficiently and reliably.

Example:

```
5 MB File
   ↓
Split into thousands of packets
   ↓
Packets travel across the network
   ↓
Receiver reassembles the original data
```

---

# Why Data is Split into Packets

Large data cannot be sent as one single block because of several networking constraints.

## 1. MTU (Maximum Transmission Unit)

Every network link has a **maximum packet size** it can transmit.

Typical Ethernet MTU:

```
1500 bytes
```

If a packet is larger than the MTU, it must be **fragmented into smaller packets**.

---

## 2. Reliable Retransmission

Networks are unreliable.

Packets may be:

- dropped
- corrupted
- delayed

If data were sent as one huge block and it failed, the **entire block would need retransmission**.

With packets:

```
Only the lost packet is retransmitted
```

This improves reliability and efficiency.

---

## 3. Better Network Utilization

Packets can take **different routes through the network**.

This allows:

- load balancing
- congestion handling
- fault tolerance

Routers can dynamically choose the best path for each packet.

---

## 4. Buffer Limitations

Routers and switches have **limited memory buffers**.

Very large packets would:

- increase latency
- consume large memory
- cause packet drops

Smaller packets allow devices to process traffic efficiently.

---

# Packet Structure

Packets consist of **two main parts**:

```
+------------------+
| Header           |
+------------------+
| Payload (Data)   |
+------------------+
```

## Header

The header contains metadata used for routing and delivery.

Common fields include:

- Source IP address
- Destination IP address
- Protocol type
- Packet length
- Fragmentation information
- TTL (Time To Live)

## Payload

The payload contains the **actual application data** being transmitted.

Example:

```
HTTP request
file data
video stream
```

---

# Packet Encapsulation

Data moves through multiple networking layers.

Each layer adds its own header.

```
Application Data
       ↓
+-----------------------+
| TCP Header            |
| Ports, Sequence No    |
+-----------------------+
| IP Header             |
| Source/Destination IP |
+-----------------------+
| Ethernet Header       |
| MAC Addresses         |
+-----------------------+
| Payload Data          |
+-----------------------+
```

---

# Packet Ordering

Packets may **arrive out of order** because:

- they take different routes
- some packets experience congestion
- routers process packets independently

Reliable protocols like TCP use **sequence numbers** to reorder packets correctly.

Example:

```
Sent order:     1 2 3 4 5
Received order: 1 3 2 5 4
```

Receiver rearranges them back into the correct order.

---

# Packet Loss

Packets can be lost due to:

- network congestion
- router buffer overflow
- hardware failures
- transmission errors

Protocols like TCP detect missing packets and **request retransmission**.

---

# Real Internet Example

Sending a webpage:

```
Browser → request webpage
Server → splits response into packets
Packets travel through routers
Packets reach browser
Browser reassembles packets
Page renders
```

---

# Key Interview Takeaways

- A packet is the smallest unit of network transmission
- Large data is split into packets for efficiency and reliability
- Each packet contains a header and payload
- Packets may arrive out of order
- Reliable protocols reorder packets and retransmit lost ones

# Networking Basics — Round 6: Router vs Switch

## Overview

Routers and switches are fundamental networking devices used to connect systems and enable communication across networks.

The key difference:

- **Switch** connects devices within a **local network (LAN)**
- **Router** connects **different networks together**

---

# Network Switch

A switch is a device used to connect multiple devices inside a **Local Area Network (LAN)**.

Examples of devices connected to a switch:

- laptops
- servers
- printers
- IoT devices
- desktop computers

Example LAN:

```
Laptop
Phone
Printer
Server
   │
   │
 Switch
```

The switch forwards network frames to the correct device inside the LAN.

---

# How a Switch Works

A switch uses **MAC addresses** to identify devices.

Each device has a unique **MAC address** assigned to its network interface card.

Example MAC address:

```
00:1A:2B:3C:4D:5E
```

The switch maintains a **MAC address table**.

Example:

```
MAC Address        Port
AA:11:BB:22        Port 1
CC:33:DD:44        Port 2
EE:55:FF:66        Port 3
```

If a packet is destined for:

```
CC:33:DD:44
```

The switch forwards the frame only to **Port 2**.

---

# Switch OSI Layer

Switch operates at:

```
OSI Layer 2 — Data Link Layer
```

Address used:

```
MAC Address
```

---

# Router

A router connects **multiple networks together**.

Examples:

- home network to the internet
- office network to cloud infrastructure
- data center networks

Example:

```
Home LAN → Router → Internet
```

---

# How a Router Works

Routers forward packets using **IP addresses**.

Routers maintain a **routing table** to determine where packets should go next.

Example routing table:

```
Destination Network     Next Hop
10.0.0.0/24             Router A
192.168.1.0/24          Router B
0.0.0.0                 Internet Gateway
```

When a packet arrives, the router checks the **destination IP address** and forwards the packet to the correct next network.

---

# Router OSI Layer

Router operates at:

```
OSI Layer 3 — Network Layer
```

Address used:

```
IP Address
```

---

# Switch vs Router Comparison

| Feature | Switch | Router |
|------|------|------|
| Primary Purpose | Connect devices in LAN | Connect different networks |
| OSI Layer | Layer 2 | Layer 3 |
| Address Used | MAC Address | IP Address |
| Typical Use | Inside LAN | Network-to-network communication |

---

# Example Network Architecture

```
Laptop ─┐
Phone  ─┼── Switch ── Router ── Internet
Printer ─┘
```

Flow explanation:

1. Devices communicate locally through the **switch**
2. When communication needs to go outside the LAN, traffic goes to the **router**
3. The router forwards packets to the **internet**

---

# Key Interview Takeaways

- A switch connects devices inside a LAN
- Switch uses MAC addresses
- Switch operates at OSI Layer 2
- A router connects different networks
- Router uses IP addresses
- Router operates at OSI Layer 3