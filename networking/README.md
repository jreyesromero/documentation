# Network Troubleshooting & Connectivity Guide

Welcome to the Networking reference guide for SRE, DevOps, and Platform Engineers. This section covers essential networking concepts, protocols, and troubleshooting workflows for production environments.

## Table of Contents

### Core Protocols & Fundamentals

- [DNS](./dns.md) - Domain Name System
  - DNS resolution flow, record types, TTL, caching
  - 6 production troubleshooting scenarios
  - dig, nslookup commands with examples

- [TCP](./tcp.md) - Transmission Control Protocol  
  - TCP 3-way handshake and connection states
  - Understanding TIME_WAIT, CLOSE_WAIT, and other states
  - 5 critical troubleshooting scenarios
  - Connection refused, timeout, port exhaustion issues

- [HTTP](./http.md) - HyperText Transfer Protocol
  - HTTP methods (GET, POST, PUT, PATCH, DELETE)
  - Status codes (2xx, 3xx, 4xx, 5xx) with detailed explanations
  - Headers, caching, redirects
  - 4 production scenarios (500 errors, 502 gateway, rate limiting, auth)

- [TLS](./tls.md) - Transport Layer Security (HTTPS/SSL)
  - X.509 certificates and certificate chains
  - Certificate validation and expiration
  - openssl commands for certificate inspection
  - Real-world scenarios: expired certs, hostname mismatches, chain issues

### Tools & Commands

- [curl & Network Tools](./curl-tools.md) - Essential Utilities
  - curl: HTTP testing, verbose mode, timing breakdowns, authentication
  - nc (netcat): Port testing, connectivity checks
  - tcpdump: Packet capture, filtering, protocol inspection
  - traceroute: Network path visualization

### Troubleshooting Workflows

- [Complete Troubleshooting Checklist](./troubleshooting-checklist.md) - End-to-End Guide
  - Quick triage (< 5 minutes)
  - 6 failure type investigations:
    * DNS failures ("name not found")
    * Service not listening ("connection refused")  
    * Network unreachable ("timeout")
    * HTTPS/TLS errors (certificate problems)
    * Backend failures (5xx errors)
    * Unexpected closes ("connection reset")
  - Interview scenario walkthrough
  - Printable production checklist

---

## Quick Reference by Problem Type

### "Can't reach the service"
1. **DNS not resolving?** → See [DNS](./dns.md) - Scenarios 1-2
2. **Service not responding?** → See [TCP](./tcp.md) - Issues 1-2
3. **Host unreachable?** → See [Troubleshooting Checklist](./troubleshooting-checklist.md) - Failure Type 3
4. **Port already in use?** → See [TCP](./tcp.md) - Issue 3

### "Connection issues"
- Connection refused → [TCP](./tcp.md) - Issue 1
- Connection timeout → [TCP](./tcp.md) - Issue 2
- Connection reset → [TCP](./tcp.md) - Issue 5
- Too many TIME_WAIT → [TCP](./tcp.md) - Scenario 1

### "HTTPS/Certificate problems"
- Certificate expired → [TLS](./tls.md) - Scenario 1
- Hostname mismatch → [TLS](./tls.md) - Scenario 2
- Certificate chain incomplete → [TLS](./tls.md) - Scenario 3
- Can't read private key → [TLS](./tls.md) - Error 4

### "API/HTTP errors"
- 500 Internal Server Error → [HTTP](./http.md) - Scenario 1
- 502 Bad Gateway → [HTTP](./http.md) - Scenario 2
- 429 Too Many Requests → [HTTP](./http.md) - Scenario 3
- 401 Unauthorized → [HTTP](./http.md) - Scenario 4

### "Debugging tools"
- Test connectivity → [curl & Network Tools](./curl-tools.md) - nc section
- Analyze HTTP response → [curl & Network Tools](./curl-tools.md) - curl section
- Capture packets → [curl & Network Tools](./curl-tools.md) - tcpdump section
- Check network path → [curl & Network Tools](./curl-tools.md) - traceroute section

---

## Interview Focus

**Most Important (master these first):**
1. DNS resolution flow (dig, dig +trace, NXDOMAIN)
2. TCP 3-way handshake and states (LISTEN, ESTABLISHED, TIME_WAIT, CLOSE_WAIT)
3. HTTP methods and status codes (GET/POST, 200/404/500)
4. curl: verbose mode, timing breakdowns (-w flags)
5. TLS certificate validation (openssl s_client)

**Common Interview Questions:**
- "Walk me through `curl https://example.com`" → See [Troubleshooting Checklist](./troubleshooting-checklist.md) - Scenario Type
- "DNS changed but old value cached" → See [DNS](./dns.md) - Scenario 2
- "What's the difference between TCP states?" → See [TCP](./tcp.md) - Connection States
- "How do you debug a 502 error?" → See [HTTP](./http.md) - Scenario 2
- "Certificate is expired, how to fix?" → See [TLS](./tls.md) - Scenario 1

---

## Coverage Summary

| Topic | Concepts | Commands | Scenarios | Coverage |
|-------|----------|----------|-----------|----------|
| DNS | 8 concepts | 5+ | 6 | ✅ Complete |
| TCP | 8 states | 2+ | 5 | ✅ Complete |
| HTTP | Methods/codes | curl | 4 | ✅ Complete |
| TLS | Certs/chains | openssl | 3 | ✅ Complete |
| Tools | 4 tools | Full ref | Workflows | ✅ Complete |
| **Total** | **28+ concepts** | **15+ commands** | **20+ scenarios** | **Comprehensive** |

---

## How to Use This Guide

1. **For Quick Lookup:** Use "Quick Reference by Problem Type" above to jump to the right section
2. **For Learning:** Read through each protocol file in order (DNS → TCP → HTTP → TLS)
3. **For Production Troubleshooting:** Use [Troubleshooting Checklist](./troubleshooting-checklist.md) and follow the systematic flow
4. **For Interview Prep:** Master the "Interview Focus" list above, then practice walkthroughs

---

**Note:** Each file includes detailed command syntax, practical examples with real output, troubleshooting scenarios with fixes, common pitfalls, and interview tips.

**Key Principle:** Network issues are diagnosed by testing layer-by-layer: DNS → TCP → HTTP/TLS → Application. Start broad, narrow down systematically.
