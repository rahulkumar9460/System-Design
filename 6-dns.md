# DNS (Domain Name System)

This document contains a **complete guide to DNS**, including:

- DNS fundamentals
- DNS resolution flow
- Root, TLD, and Authoritative servers
- Recursive vs Iterative queries
- DNS caching
- DNS record types
- DNS load balancing
- GeoDNS routing
- Detailed end-to-end DNS resolution flow

These concepts are commonly asked in **system design interviews**.

---

# 1. What is DNS?

DNS (Domain Name System) translates **human-readable domain names** into **IP addresses**.

Example:

```
google.com → 142.250.183.206
```

Why DNS exists:

- Humans remember **domain names**
- Computers communicate using **IP addresses**

DNS acts like the **phonebook of the internet**.

---

# 2. DNS Hierarchy

DNS is structured as a **hierarchical distributed system**.

Hierarchy:

```
Root Servers
     |
     v
TLD Servers (.com, .org, .net)
     |
     v
Authoritative DNS Servers
     |
     v
IP Address
```

---

# 3. DNS Resolution Overview

When a user types:

```
www.google.com
```

The browser must find the IP address.

Steps:

1. Browser checks **browser cache**
2. OS checks **OS DNS cache**
3. Query sent to **recursive DNS resolver**
4. Resolver queries **Root DNS server**
5. Root returns **list of TLD servers**
6. Resolver selects one TLD server
7. Resolver queries TLD server
8. TLD returns **list of authoritative servers**
9. Resolver selects one authoritative server
10. Resolver queries authoritative server
11. Authoritative server returns **IP address**
12. Resolver returns IP to browser
13. Browser connects to server using **TCP/TLS**

---

# 4. Complete DNS Resolution Flow

Example:

User enters:

```
www.google.com
```

Detailed resolution process:

```
User Browser
     |
     | Check browser DNS cache
     v
Operating System DNS Cache
     |
     | If cache miss
     v
Recursive DNS Resolver (ISP / Public DNS)
     |
     | Query Root DNS Server
     v
Root Server
     |
     | Returns list of .com TLD servers
     v
Resolver chooses one TLD server
     |
     v
TLD Server (.com)
     |
     | Returns authoritative DNS servers for google.com
     v
Resolver chooses one authoritative server
     |
     v
Authoritative DNS Server
     |
     | Returns IP address
     v
DNS Resolver
     |
     v
Browser receives IP
```

Example result:

```
www.google.com → 142.250.183.206
```

---

# 5. Root DNS Servers

Root servers are the **top level of DNS hierarchy**.

There are **13 logical root servers** worldwide.

Example root servers:

```
a.root-servers.net
b.root-servers.net
c.root-servers.net
...
```

These are operated by organizations like:

- ICANN
- Verisign
- NASA

Each root server is replicated globally using **Anycast**, so there are **hundreds of physical servers**.

---

# 6. Root Server Response

When the resolver asks:

```
Resolver → Root : Where is .com ?
```

The root server does **not return one TLD server**.

Instead it returns **a list of TLD servers**.

Example response:

```
.com TLD servers

a.gtld-servers.net
b.gtld-servers.net
c.gtld-servers.net
d.gtld-servers.net
```

Resolver then **chooses one** server to query.

Selection strategies:

- Random selection
- Round robin
- Lowest latency
- Cached preference

---

# ASCII Diagram

```
Resolver → Root Server
            |
            | returns list
            v

      a.gtld-servers.net
      b.gtld-servers.net
      c.gtld-servers.net
```

Resolver picks one:

```
Resolver → b.gtld-servers.net
```

---

# 7. TLD Servers

TLD stands for **Top Level Domain**.

Examples:

```
.com
.org
.net
.edu
```

TLD servers know **which authoritative server manages a domain**.

Example query:

```
Resolver → TLD : Where is google.com ?
```

Response:

```
ns1.google.com
ns2.google.com
ns3.google.com
ns4.google.com
```

These are **authoritative DNS servers**.

---

# 8. Authoritative DNS Servers

Authoritative servers contain the **actual DNS records**.

Example query:

```
Resolver → ns1.google.com
```

Response:

```
google.com → 142.250.183.206
```

This is the **final IP address**.

---

# 9. Recursive vs Iterative Queries

## Recursive Query

Client asks resolver to get the **final answer**.

Example:

```
Client → Resolver → Root → TLD → Authoritative → Resolver → Client
```

The resolver performs the entire lookup.

---

## Iterative Query

DNS servers respond with **referrals** instead of final answers.

Example:

```
Resolver → Root
Root → "Ask TLD server"

Resolver → TLD
TLD → "Ask authoritative server"

Resolver → Authoritative
Authoritative → Returns IP
```

---

# Real Internet Behavior

Actual flow:

```
Client → Resolver (recursive query)

Resolver → Root/TLD/Auth (iterative queries)
```

---

# 10. DNS Caching

DNS lookups introduce **latency and server load**, so results are cached.

Caching happens at:

```
Browser Cache
OS Cache
Recursive Resolver Cache
```

Each record has a **TTL (Time To Live)**.

Example:

```
google.com → 142.250.183.206
TTL = 300 seconds
```

Meaning it can be cached for **5 minutes**.

---

# ASCII Diagram — DNS Caching

```
User enters domain
      |
      v
Browser Cache
      |
      v
OS DNS Cache
      |
      v
Resolver Cache
      |
      v
Root → TLD → Authoritative
```

---

# 11. DNS Record Types

## A Record

Maps **domain → IPv4**

```
example.com → 93.184.216.34
```

---

## AAAA Record

Maps **domain → IPv6**

```
example.com → 2606:2800:220:1:248:1893:25c8:1946
```

---

## CNAME Record

Domain alias.

```
www.example.com → example.com
```

Resolution flow:

```
www.example.com
      ↓
example.com
      ↓
IP Address
```

---

## MX Record

Mail server mapping.

```
gmail.com → mail server
```

---

## TXT Record

Metadata.

Used for:

- domain verification
- SPF
- DKIM
- security validation

---

# DNS Record Summary

| Record | Purpose |
|------|------|
| A | Domain → IPv4 |
| AAAA | Domain → IPv6 |
| CNAME | Domain alias |
| MX | Mail routing |
| TXT | Metadata |

---

# 12. DNS Load Balancing

DNS can distribute traffic by returning **multiple IP addresses**.

Example:

```
example.com → 10.1.1.1
example.com → 10.1.1.2
example.com → 10.1.1.3
```

Different users may receive different IPs.

---

# ASCII Diagram

```
User 1 → DNS → 10.1.1.1 → Server 1
User 2 → DNS → 10.1.1.2 → Server 2
User 3 → DNS → 10.1.1.3 → Server 3
```

---

# 13. GeoDNS (Location Based Routing)

Advanced DNS providers route users based on **geographic location**.

Example:

```
User in India → Mumbai server
User in EU → Frankfurt server
User in US → Virginia server
```

---

# ASCII Diagram

```
           Global Users
                |
                v
           Authoritative DNS
            /      |      \
           v       v       v
        India     EU      USA
       Server    Server   Server
```

---

# 14. Key Takeaways

- DNS translates **domain names to IP addresses**
- DNS hierarchy: **Root → TLD → Authoritative**
- Root and TLD servers return **multiple servers**
- Resolver selects one server to query
- DNS uses **recursive and iterative queries**
- **Caching + TTL** improves performance
- DNS supports **load balancing and geo routing**

---
