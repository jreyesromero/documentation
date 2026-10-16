# Networking Interview Guide - SRE Focus (1 Hour Prep)

> **Goal:** Master the core networking concepts & troubleshooting skills for a 1-hour SRE screening interview.

**Time allocation:** 50 min studying + 10 min review before interview.

---

## 📌 Interview Structure (What to Expect)

Most SRE networking interviews follow this pattern:
1. **10 min** - Warm-up: "Walk me through a `curl` request"
2. **30 min** - Troubleshooting scenarios + protocol concepts
3. **15 min** - Practical problem-solving (live or scenario-based)
4. **5 min** - Questions for interviewer

---

## 🎯 Typical SRE Networking Interview Questions

### **1. DNS & NAME RESOLUTION** (10 min - ~2 questions)

**Q1: "Domain is resolving to the wrong IP. How do you debug?"**
- Use `dig domain.com +trace` to see full resolution chain
- Check which nameserver your system uses: `cat /etc/resolv.conf`
- Test against authoritative nameserver: `dig @ns1.example.com domain.com`
- Check TTL and cache: `dig +nocache domain.com`
- Look for CNAME chains (adds latency)
- **Red flag:** Assuming cache is the problem without testing with `+nocache`

**Q2: "Application intermittently fails with 'cannot resolve host'. Why?"**
- One of multiple nameservers is down: `dig @ns1.com`, `dig @ns2.com`
- Network timeout to nameserver (slow DNS)
- Application's DNS timeout too short
- Round-robin DNS returning some dead IPs
- **Answer structure:** "I'd test from multiple nameservers, check network latency to DNS, then review app timeout settings"

---

### **2. TCP & CONNECTION STATES** (10 min - ~2 questions)

**Q1: "Users see 'Connection refused'. What's your debugging flow?"**
- Check if service is listening: `ss -tln | grep :PORT`
- Check if service is running: `systemctl status service`
- Check if it's a firewall issue: try `localhost` first
- Check if port is already in use: `lsof -i :PORT`
- **The order matters:** Listen → Reach → Firewall → Logs

**Q2: "We have 50,000 TIME_WAIT connections. Why is this a problem?"**
- Each port can only be used once during TIME_WAIT (60 second grace period)
- Only ~65,535 total ports available
- Quickly runs out of available ports for new connections
- **Fix:** Enable `tcp_tw_reuse` or implement connection pooling
- **Why it happens:** Creating new connections instead of reusing

**Q3: "CLOSE_WAIT connections growing. What does it mean?"**
- Application hasn't called `close()` on a socket
- Remote side closed connection first
- This is **always an application bug**, not system config
- **Answer:** "This is an application memory leak. Need to fix the code."

---

### **3. HTTP & STATUS CODES** (10 min - ~2 questions)

**Q1: "Server returns 502 'Bad Gateway'. How do you investigate?"**
- 502 = Frontend got bad response from backend (load balancer problem usually)
- Check if backend service is up: `curl http://backend:8080`
- Check if backend is listening: `ss -tln | grep :8080`
- Check backend logs: `journalctl -u backend_service`
- Check network connectivity: `ping backend_host`, `traceroute backend_host`
- Check load balancer configuration and health check settings

**Q2: "Requests randomly return 500 but service says it's running."**
- Service is crashing on certain requests
- Memory leak causing OOM kills
- Resource exhaustion (file descriptors, connections)
- **Approach:** Watch logs while making requests, look for crash patterns
- `journalctl -u service -f &` in background while testing

---

### **4. TLS & CERTIFICATES** (8 min - ~2 questions)

**Q1: "Certificate shows expired but cert file says it's valid. What gives?"**
- Browser cache showing old cert
- Client has old cert cached
- Multiple certificates on same server (wrong one being served)
- System time is wrong: `date` (NTP sync issue)
- **Check:** `openssl s_client -connect domain:443` to see what's actually being served

**Q2: "HTTPS connection fails with 'certificate name mismatch'."**
- Hostname doesn't match certificate's CN or SAN
- You're accessing `https://10.0.0.5` but cert is for `api.example.com`
- Certificate was auto-generated with wrong name
- **Fix:** Either fix DNS/hosts entry OR regenerate cert with correct name

---

### **5. TROUBLESHOOTING APPROACH** (10 min - ~1 question)

**Q1: "Walk me through: `curl https://api.example.com` fails with timeout."**

**Your systematic answer:**
```
1. "First, I check DNS:
   dig api.example.com
   If fails → DNS issue
   If works → DNS is fine"

2. "Then I check if host is reachable:
   ping <IP from dig>
   If fails → Network/routing issue"

3. "Then I check the specific port:
   nc -zv api.example.com 443
   timeout 5 bash -c 'cat < /dev/null > /dev/tcp/api.example.com/443'
   If fails → Firewall or service not listening"

4. "Check if service is listening:
   ss -tln | grep :443
   If not there → Service crashed/not started"

5. "Check service logs:
   journalctl -u api-service -n 50
   Look for: crashes, errors, resource issues"

6. "If intermittent:
   Watch both service and attempts:
   watch -n 1 'ss -tln | grep :443' &
   curl https://api.example.com"
```

**This shows:**
- ✅ Layered debugging (DNS → Network → Service)
- ✅ Systematic elimination
- ✅ Use of tools (dig, ping, nc, ss, curl, journalctl)
- ✅ Understanding of the problem
- ✅ Communication of thought process

---

## 🚨 Red Flags (What NOT to Say)

❌ "I'll just restart the service"  
✅ "Let me check logs first to understand why"

❌ "DNS must be broken"  
✅ "I'll verify DNS with dig, then check network, then the service"

❌ "The certificate is expired"  
✅ "Let me check what cert is actually being served with openssl s_client"

❌ "Connection refused means firewall"  
✅ "First I check if the service is listening, then consider firewall"

❌ "TIME_WAIT is bad and needs to be fixed"  
✅ "TIME_WAIT is normal, but if there are too many, it's a sign of too many new connections"

---

## 📚 Quick Reference: Most Important Commands

```bash
# DNS
dig domain.com                  # Full query
dig domain.com +trace           # Full resolution chain
dig @nameserver domain.com      # Test specific nameserver
dig domain.com +nocache         # Bypass cache

# TCP Connection Debugging
ss -tln | grep :PORT           # Is service listening?
ss -tan | grep ESTABLISHED     # Active connections
ss -tan | grep TIME_WAIT       # Waiting to close
lsof -i :PORT                  # What's using the port?
ss -tan | awk '{print $NF}' | sort | uniq -c  # Count by state

# HTTP Testing
curl -v https://domain.com     # Verbose (see headers)
curl -w "%{http_code}\n" url   # Just the status code
curl --trace-time url          # Timing breakdown

# TLS/Certificate
openssl s_client -connect domain:443  # See actual certificate
openssl x509 -in cert.pem -text       # Certificate details
openssl x509 -in cert.pem -noout -dates  # Expiration

# Service Debugging
systemctl status service       # Is it running?
journalctl -u service -n 50   # Recent logs
journalctl -u service -f      # Follow logs

# Network
ping host                      # Host reachable?
traceroute host                # Path to host
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/host/port'  # Port open?
```

---

## 🎓 Study Plan (1 Week, Networking Focus)

| Day | Topic | Files to Read | Time |
|-----|-------|---------------|------|
| 1 | TCP/IP Fundamentals | `tcp-ip-fundamentals-interview.md` | 30 min |
| 1 | DNS Fundamentals | `dns.md` (Quick Ref + Scenarios) | 45 min |
| 2 | TCP Concepts & Troubleshooting | `tcp.md` (States + Issues) | 60 min |
| 3 | HTTP & Status Codes | `http.md` (Methods + Codes + Scenarios) | 45 min |
| 4 | TLS & Certificates | `tls.md` (Certs + Validation) | 45 min |
| 5 | Tools & Debugging | `curl-tools.md` + practice with real commands | 60 min |
| 6 | End-to-End Troubleshooting | `troubleshooting-checklist.md` + mock scenarios | 60 min |
| 7 | Review + Mock Interview | All files, focus on weak areas | 90 min |

---

## 🔗 File Map (Quick Navigation)

**Foundational (Start here if new to networking):**
- [TCP/IP Fundamentals](./tcp-ip-fundamentals-interview.md) - Layers, MAC vs IP, encapsulation, ports

**By Problem Type:**
- **"Can't reach service"** → Start with [troubleshooting-checklist.md](./troubleshooting-checklist.md)
- **"DNS issue"** → [dns.md](./dns.md) - Scenarios 1-4
- **"Connection refused"** → [tcp.md](./tcp.md) - Issue 1
- **"Connection timeout"** → [tcp.md](./tcp.md) - Issue 2
- **"500 error"** → [http.md](./http.md) - Scenario 1
- **"Certificate expired"** → [tls.md](./tls.md) - Scenario 1
- **"How do I test?"** → [curl-tools.md](./curl-tools.md)

**By Topic:**
- [TCP/IP Fundamentals](./tcp-ip-fundamentals-interview.md) - Layers, addressing, encapsulation
- [DNS](./dns.md) - Domain resolution, TTL, caching, nameservers
- [TCP](./tcp.md) - Connection states, handshake, port issues
- [HTTP](./http.md) - Methods, status codes, redirects, caching
- [TLS](./tls.md) - Certificates, validation, common errors
- [Curl & Tools](./curl-tools.md) - curl, nc, tcpdump, traceroute
- [Troubleshooting Checklist](./troubleshooting-checklist.md) - Systematic approach

---

## 📊 What Interviewers Are Looking For

✅ **Systematic thinking:** Start broad, narrow down methodically  
✅ **Layer awareness:** Know DNS → TCP → HTTP → TLS → Application  
✅ **Tool knowledge:** Can use curl, dig, ss, nc, ping effectively  
✅ **Problem-solving:** Not just knowing commands, but why and when to use them  
✅ **Communication:** Explain your reasoning aloud  
✅ **Root cause focus:** "Why" not just "how to fix"  

---

## 🎬 Mock Interview Scenario

**Setup:** Your manager says "API is down, users can't reach it. 10 minute fix window."

**Your answer (speak aloud):**
1. "First, I need to understand the error. Are users getting connection refused or timeout?"
2. "Let me test from my machine: `curl https://api.example.com -v`"
3. "DNS resolves, so it's not that. Let me check if the service is running on the backend..."
4. "I'm checking if it's listening: `ss -tln | grep :8080`"
5. "Service is listening, so it's not crashed. Let me check its logs..."
6. "Logs show memory errors. Looks like a memory leak. I'm restarting the service..."
7. "Testing again: `curl https://api.example.com` - it works!"
8. "But this is a temporary fix. We need to investigate why memory is leaking. Let me add it to the post-mortem."

**Why this works:**
- Shows structured thinking
- Makes intermediate tests visible
- Communicates each step
- Distinguishes quick fix from root cause
- Follows up with long-term plan

---

## 💡 Interview Tips

✅ **Ask clarifying questions first:** "What error do users see?"  
✅ **State your assumption:** "I'm assuming it's a DNS issue, let me verify..."  
✅ **Test hypotheses:** Don't guess, run commands  
✅ **Eliminate systematically:** DNS → Network → Service → Application  
✅ **Explain your reasoning:** "I'm checking this because..."  
✅ **Prioritize:** Quick fix first (get users working), then root cause  
✅ **Show humility:** "I'm not familiar with that error, but I'd check the logs..."  

---

## 🔗 Related Files

**Interview Preparation:**
- [sre-interview-guide.md](../linux/sre-interview-guide.md) - Linux preparation (complementary)
- [Linux Troubleshooting Guide](../linux/06-system-troubleshooting-workflow.md) - Systematic approach

**Networking References:**
- [dns.md](./dns.md) - DNS deep dive
- [tcp.md](./tcp.md) - TCP/IP concepts
- [http.md](./http.md) - HTTP protocol
- [tls.md](./tls.md) - HTTPS/certificates
- [curl-tools.md](./curl-tools.md) - Debugging tools
- [troubleshooting-checklist.md](./troubleshooting-checklist.md) - Step-by-step guide

---

**Last updated:** 2026-10-16  
**Confidence level:** High (networking content is comprehensive and interview-tested)

**Next:** Practice with the troubleshooting checklist scenarios, then do mock interviews.
