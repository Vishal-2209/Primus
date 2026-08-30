---
created: 2026-08-30
purpose: Computer Networking concepts for technical interview
---
# Networking Cheatsheet

## 1. OSI Model (7 Layers)

| Layer | Name | Data Unit | Protocols/Devices | Your Context |
|-------|------|-----------|-------------------|--------------|
| 7 | Application | Data | HTTP, HTTPS, DNS, SMTP | Django/Flask REST APIs |
| 6 | Presentation | Data | SSL/TLS, JPEG, ASCII | JSON serialization |
| 5 | Session | Data | NetBIOS, RPC | JWT session management |
| 4 | Transport | Segments | TCP, UDP | WebSocket vs HTTP |
| 3 | Network | Packets | IP, ICMP, Router | Load balancer routing |
| 2 | Data Link | Frames | Ethernet, Switch | LAN communication |
| 1 | Physical | Bits | Cables, Hub | Physical infrastructure |

### Mnemonic: **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

---

## 2. TCP vs UDP

| Aspect | TCP | UDP |
|--------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery, retransmission | Best effort, no guarantee |
| Order | Ordered delivery | No ordering guarantee |
| Speed | Slower (overhead) | Faster (minimal overhead) |
| Use Case | HTTP, email, file transfer | DNS, video streaming, gaming |

### TCP 3-Way Handshake
```
Client -> Server: SYN
Server -> Client: SYN-ACK
Client -> Server: ACK
Connection established
```

---

## 3. HTTP/HTTPS

### HTTP Methods
| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Update/replace | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

### Status Codes
| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful request |
| 201 | Created | Resource created (POST) |
| 204 | No Content | Successful DELETE |
| 301 | Moved Permanently | URL changed |
| 304 | Not Modified | Cache still valid |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Not authorized |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unhandled exception |

### HTTPS = HTTP + TLS
- TLS handshake encrypts communication
- Default port: 443 (vs HTTP 80)

---

## 4. REST API Design

### Principles
1. **Stateless**: Each request contains all info needed
2. **Uniform Interface**: Consistent resource naming
3. **Client-Server**: Separation of concerns
4. **Cacheable**: Responses can be cached

### URL Design
```
GET    /api/v1/cases          -> List all cases
GET    /api/v1/cases/123      -> Get case 123
POST   /api/v1/cases          -> Create new case
PUT    /api/v1/cases/123      -> Update case 123
DELETE /api/v1/cases/123      -> Delete case 123
```

---

## 5. DNS

### Resolution Flow
```
Browser cache -> OS cache -> Router -> ISP DNS -> Root -> TLD -> Authoritative
```

### Record Types
| Type | Purpose | Example |
|------|---------|---------|
| A | Domain -> IPv4 | example.com -> 93.184.216.34 |
| CNAME | Alias to another domain | www.example.com -> example.com |
| MX | Mail server | example.com -> mail.example.com |
| TXT | Text records (verification) | Domain verification |

---

## 6. WebSocket vs HTTP

| Aspect | HTTP | WebSocket |
|--------|------|-----------|
| Connection | Request-response only | Persistent, bidirectional |
| Server Push | Polling or SSE | Native push |
| Use Case | CRUD APIs | Real-time chat, live updates |

---

## 7. Load Balancing

### Algorithms
| Algorithm | Description | Best For |
|-----------|-------------|----------|
| Round Robin | Distribute sequentially | Equal capacity servers |
| Least Connections | Send to fewest active conns | Varying request durations |
| IP Hash | Same IP hits same server | Session persistence |

### L4 vs L7 Load Balancing
- **L4 (Transport)**: Routes based on IP + port. Faster.
- **L7 (Application)**: Routes based on HTTP headers, URL. More intelligent.

---

## 8. CORS (Cross-Origin Resource Sharing)

### Same-Origin Policy
Browser blocks requests from one origin to another unless server allows it.

### CORS Headers
```
Access-Control-Allow-Origin: https://va7.dev
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

---

## 9. Common Interview Questions

**Q: What happens when you type google.com in browser?**
A:
1. DNS resolution (domain -> IP)
2. TCP handshake with server
3. TLS handshake (if HTTPS)
4. HTTP GET request
5. Server processes request, returns HTML
6. Browser parses HTML, loads CSS/JS/images
7. Browser renders page

**Q: TCP vs UDP?**
A: TCP: reliable, ordered, connection-oriented. UDP: fast, connectionless, no guarantee. Use TCP for web/email, UDP for video/gaming/DNS.

**Q: What is CORS?**
A: Browser security mechanism restricting cross-origin HTTP requests. Server must explicitly allow other origins via Access-Control-Allow-Origin header.

**Q: REST vs GraphQL?**
A: REST: fixed endpoints, multiple routes. GraphQL: single endpoint, client specifies exact data needed. REST is simpler; GraphQL reduces over-fetching.

---

> **Key for Vishal**: Your LawPrix serves REST APIs through DRP. Nexus proxies Google Drive API. PGPulse uses real-time updates. Connect networking to what you've built.
