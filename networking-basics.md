## How a Web Request Works (From Browser to Server)
## What happens when you type https://google.com in browser and press enter

1. **Browser checks DNS cache**  
   The browser first checks its local DNS cache to see if the IP address for the domain is already stored.

2. **DNS Query to Resolver**  
   If the cache misses, the request is sent to a **recursive DNS resolver** (usually provided by the ISP).

3. **DNS Resolution Process**
   - Resolver queries the **Root DNS server**
   - Root server points to the **TLD server** (e.g., `.com`)
   - TLD server points to the **Authoritative DNS server**
   - Authoritative server returns the **IP address** of the domain

4. **Browser Receives Server IP**  
   The browser now knows the IP address of the destination server.

5. **ARP Lookup**  
   The computer checks the **ARP cache** to find the **MAC address of the default gateway** (router).  
   If not found, an **ARP request** is broadcast to obtain it.

6. **TCP Connection Establishment (3-Way Handshake)**
   - Client → **SYN**
   - Server → **SYN-ACK**
   - Client → **ACK**

7. **TLS Handshake (If HTTPS)**
   If the site uses HTTPS, a **TLS handshake** occurs to establish encryption and exchange session keys.

8. **HTTP Request Sent**  
   The browser sends the HTTP request (e.g., `GET /index.html`).

9. **Segmentation**
   The request data is broken into **TCP segments**.

10. **Packet Encapsulation**
    Each TCP segment is encapsulated inside an **IP packet**.

11. **Packet Transmission**
    Packets travel across multiple **routers and networks on the internet** until they reach the destination server.

12. **Server Processing**
    The server receives the request, processes it (possibly querying databases or services), and prepares a response.

13. **Response Transmission**
    The response is also broken into **TCP segments and IP packets** and sent back to the client.

14. **TCP Reliability**
    TCP ensures:
    - **Ordered delivery**
    - **Retransmission of lost packets**
    - **Flow control**
    - **Congestion control**

15. **Browser Reassembly and Rendering**
    The browser reassembles the packets, reconstructs the HTTP response, parses the HTML/CSS/JS, and **renders the web page**.


# Networking Basics — IP Address

## What is an IP Address?

An **IP (Internet Protocol) address** is a unique identifier assigned to a device or network interface on a network. It allows devices to **locate and communicate with each other across networks**, including the internet.

When data is sent across the internet, it is broken into **packets**, and each packet contains:

- **Source IP address** (sender)
- **Destination IP address** (receiver)

Routers use these IP addresses to **forward packets to the correct destination**.

Examples of devices that have IP addresses:

- Servers
- Laptops
- Smartphones
- Routers
- IoT devices

---

# IPv4 (Internet Protocol Version 4)

## Structure

IPv4 addresses are **32-bit numbers** divided into **4 sections (octets)**.

Example:

```
10.2.1.12
```

Each section ranges from **0–255**.

Binary representation:

```
00001010.00000010.00000001.00001100
```

Each block = **8 bits**

```
8 bits × 4 = 32 bits
```

---

## Total Number of IPv4 Addresses

Since IPv4 is **32 bits**:

```
2^32 = 4,294,967,296
```

Approximately:

```
≈ 4.3 billion addresses
```

However, not all are usable because some ranges are **reserved**.

Examples of reserved ranges:

| Range | Purpose |
|------|--------|
| 127.0.0.0/8 | Loopback |
| 10.0.0.0/8 | Private networks |
| 192.168.0.0/16 | Private networks |
| 172.16.0.0–172.31.255.255 | Private networks |

---

# Problem with IPv4

The internet grew rapidly:

- Billions of users
- Smartphones
- IoT devices
- Cloud servers

The **4.3 billion address limit became insufficient**, leading to **IPv4 exhaustion**.

Temporary solutions included:

- **NAT (Network Address Translation)**
- **CIDR (Classless Inter-Domain Routing)**

But a long-term solution was required.

---

# IPv6 (Internet Protocol Version 6)

IPv6 was introduced to solve the **address exhaustion problem**.

## Structure

IPv6 addresses are **128-bit numbers** written in hexadecimal format.

Example:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

It consists of **8 groups of 16 bits**.

```
16 bits × 8 = 128 bits
```

---

## Total Number of IPv6 Addresses

```
2^128 ≈ 340 undecillion
```

Exact value:

```
340,282,366,920,938,463,463,374,607,431,768,211,456
```

This number is so large that **every device on earth can have billions of IP addresses**.

---

# Why IPv6 Was Introduced

IPv6 solves several problems:

1. **Massive address space**
2. **Better routing efficiency**
3. **Simpler packet headers**
4. **Improved auto-configuration**
5. **Better support for modern internet infrastructure**

---

# Key Differences

| Feature | IPv4 | IPv6 |
|------|------|------|
| Address size | 32 bits | 128 bits |
| Format | Decimal | Hexadecimal |
| Example | 192.168.1.1 | 2001:db8::1 |
| Total addresses | ~4.3 billion | ~340 undecillion |
| Introduced | 1981 | 1998 |

---

# Important Interview Insight

Most devices today **do not have a public IPv4 address directly**.

Instead, many devices share **one public IP** using **NAT (Network Address Translation)**.

Example:

```
Router Public IP: 52.10.20.30

Devices behind router:
192.168.1.2 (Laptop)
192.168.1.3 (Phone)
192.168.1.4 (TV)
```

The router translates **private IPs → public IP** when sending traffic to the internet.

---

# Key Takeaways (Interview Cheat Sheet)

- IP address uniquely identifies a **device or network interface**
- IPv4 uses **32 bits (~4.3B addresses)**
- IPv6 uses **128 bits (~340 undecillion addresses)**
- IPv6 was introduced because **IPv4 addresses were exhausted**
- Most devices use **private IP + NAT** instead of direct public IP