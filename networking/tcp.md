# TCP - Transmission Control Protocol

TCP is the fundamental transport layer protocol for reliable, ordered delivery. Understanding TCP states and troubleshooting is essential for SRE work.

## Quick Reference

```bash
# View TCP connections
ss -tan                        # All TCP, numeric, no name lookups
ss -tln                        # TCP listening sockets
ss -antp | grep ESTABLISHED    # Active connections
ss -antp | grep TIME_WAIT      # Connections waiting to close

# Older tool (netstat)
netstat -tan
netstat -tln
netstat -antp

# Monitor in real-time
watch -n 1 'ss -tan | head -20'

# Count connections by state
ss -tan | tail -n +2 | awk '{print $NF}' | sort | uniq -c | sort -rn
```

---

## TCP 3-Way Handshake

The process to establish a connection:

```
CLIENT                              SERVER
  |                                   |
  |-------- SYN (seq=x) ------------>| (1)
  |                                   |
  |<------ SYN-ACK (seq=y, ack=x+1) -| (2)
  |                                   |
  |-------- ACK (seq=x+1, ack=y+1) -->| (3)
  |                                   |
  [Connection ESTABLISHED]
```

**Time: ~1-50ms for local network, 100-300ms for internet**

### What Each Step Means

| Step | Name | Flag | Meaning |
|------|------|------|---------|
| 1 | SYN | Client sends | "I want to connect, here's my starting sequence number" |
| 2 | SYN-ACK | Server responds | "OK, I acknowledge you, here's my starting sequence number" |
| 3 | ACK | Client acks back | "I acknowledge you back. Connection is ready." |

### Timeline

```bash
# Establish connection: ~1-50ms

# Server accepts connection: ~1-100μs (microseconds)

# Application processes request: ~10ms - seconds

# Graceful close: 40ms (each side sends FIN, waits for ACK)

# TIME_WAIT: 60 seconds (by default, ~2 × RTT)
```

---

## TCP Connection States

Understanding TCP states is **critical for SRE troubleshooting**:

| State | Direction | Meaning | Concern |
|-------|-----------|---------|---------|
| **LISTEN** | Server | Waiting for connection | Normal, healthy |
| **SYN_SENT** | Client | Sent SYN, waiting for SYN-ACK | Brief (< 1s) |
| **SYN_RECV** | Server | Received SYN, sent SYN-ACK, waiting for ACK | Brief (< 1s) |
| **ESTABLISHED** | Both | Active connection, data flowing | Normal |
| **FIN_WAIT1** | Closer | Sent FIN, waiting for ACK | Brief (< 1s) |
| **FIN_WAIT2** | Closer | Got ACK of our FIN, waiting for their FIN | Brief (< 1s) |
| **CLOSING** | Both | Both sent FIN simultaneously, waiting for ACKs | Very brief (< 100ms) |
| **TIME_WAIT** | Closer | Got their FIN + ACK, waiting before reusing ports | **60 seconds** (configurable) |
| **CLOSE_WAIT** | Receiver | Got FIN from peer, app hasn't called close() yet | **Application bug if > 10s** |
| **LAST_ACK** | Receiver | Sent FIN, waiting for final ACK | Brief (< 1s) |
| **CLOSED** | Both | Fully closed | Not seen in ss/netstat |

### State Transition Diagram (Simplified)

```
LISTEN
  ↓ (client connects)
SYN_RECV → ESTABLISHED → FIN_WAIT1 → FIN_WAIT2 → TIME_WAIT (60s) → CLOSED
                           ↑                                   ↑
                           |                                   |
                      CLOSE_WAIT ← (peer sent FIN)
                           |
                      (app calls close())
                           ↓
                      LAST_ACK
                           ↓
                        CLOSED
```

---

## Troubleshooting Common TCP Issues

### Issue 1: Connection Refused

**Error:** `curl: (7) Failed to connect to example.com port 80: Connection refused`

**Diagnosis:**
```bash
# Step 1: Is the service listening?
ss -tln | grep :80

# Expected: Shows LISTEN on 0.0.0.0:80 or 127.0.0.1:80
# If nothing: Service is not listening

# Step 2: Check if service is running
systemctl status nginx

# Step 3: Check if it's a firewall issue
# Try localhost first
curl localhost:80

# If localhost works but remote doesn't:
# Firewall is blocking inbound

# Step 4: Check which IP the service is bound to
ss -tln | grep nginx
# 0.0.0.0:80 = listening on all interfaces
# 127.0.0.1:80 = listening on loopback only (localhost)
```

**Root Causes:**
- Service not running: `systemctl start service`
- Listening on wrong IP: `ss -tln | grep :PORT`
- Firewall blocking: `iptables -L` or check security groups
- Service crashed: `journalctl -u service -n 50`
- Port already in use: `lsof -i :PORT`

---

### Issue 2: Connection Timeout

**Error:** `curl: (7) Failed to connect to example.com port 443: Operation timed out`

**Diagnosis:**
```bash
# Step 1: Can you reach the host at all?
ping example.com
# If timeout: Host unreachable

# Step 2: Can you reach the specific port?
# Try with timeout
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/example.com/443'
# If timeout: Port not responding

# Step 3: Check routing to the host
traceroute example.com
# Shows each hop; where does it stop?

# Step 4: Check firewall rules
# If on cloud: Check security group inbound rules
# If on-premise: Check iptables or firewall

# Step 5: Check if service is listening
ss -tln | grep :443

# Step 6: Check if service is hung
ps aux | grep service
# Is it running? What's its state?
```

**Root Causes:**
- **Network unreachable:** No route to host, firewall blocking
- **Service not listening:** Service crashed or not started
- **Service hung:** Process exists but not accepting connections
- **Slow network:** TCP timeout (usually 30-120 seconds)

**Fixes:**
```bash
# Restart service
systemctl restart nginx

# Check and open firewall port
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload

# Verify reachability
curl -v --max-time 5 https://example.com
```

---

### Issue 3: Too Many TIME_WAIT Connections

**Problem:** `ss -tan | grep TIME_WAIT | wc -l` returns 10,000+

**Investigation:**
```bash
# Count by state
ss -tan | tail -n +2 | awk '{print $NF}' | sort | uniq -c | sort -rn

# Example output:
#   15000 TIME_WAIT
#    2000 ESTABLISHED
#     500 LISTEN

# What's creating so many connections?
# Find the local port range being exhausted
ss -tan | grep TIME_WAIT | head -10

# Each connection cycles through TIME_WAIT for 60 seconds
# If you're creating 250 new connections per second:
# 250 connections/sec × 60 sec = 15,000 TIME_WAIT connections

# Check new connection rate
# Capture for 10 seconds
tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0' -c 1000 2>/dev/null | tail -1
# High count = creating connections rapidly
```

**Root Causes:**
- **Creating connections rapidly:** Load testing, connection per request
- **Not reusing connections:** Each request makes new connection (HTTP 1.0 style)
- **Low TIME_WAIT timeout:** Should be 60s, but sometimes lower
- **Load balancer cycling connections:** Each client gets different backend

**Impact:**
- System runs out of available ports (max ~65,535)
- New connections fail with "cannot assign requested address"
- Eventually: `curl: (7) Failed to connect`

**Fixes:**
```bash
# Option 1: Enable SO_REUSEADDR (allow reusing TIME_WAIT ports)
sysctl -w net.ipv4.tcp_tw_reuse=1

# Option 2: Shorten TIME_WAIT (not recommended, can cause issues)
sysctl -w net.ipv4.tcp_fin_timeout=30

# Option 3: Connection pooling in your application
# Instead of new connection per request, reuse
# (Recommended for production)

# Option 4: Increase available port range
sysctl -w net.ipv4.ip_local_port_range="1024 65535"

# Verify
ss -tan | grep TIME_WAIT | wc -l  # Should decrease over time
```

---

### Issue 4: CLOSE_WAIT Connections Accumulating

**Problem:** `ss -tan | grep CLOSE_WAIT | wc -l` returns 500+ and growing

**Investigation:**
```bash
# Confirm the state
ss -tan | grep CLOSE_WAIT | wc -l

# See which process owns them
lsof -i | grep CLOSE_WAIT | head -10

# Get the PID
PROCESS_PID=$(lsof -i | grep CLOSE_WAIT | head -1 | awk '{print $2}')

# Check what process it is
ps aux | grep $PROCESS_PID

# Check how many connections it has
lsof -p $PROCESS_PID | wc -l
```

**Root Cause: APPLICATION BUG**

CLOSE_WAIT means:
- Remote peer closed the connection (sent FIN)
- Local application hasn't called `close()` yet
- Connection stays open indefinitely

```bash
# Example: Bad application code
for request in many_requests:
    socket = open_socket(server, port)
    socket.send(request)
    # BUG: Never calls socket.close() or close() returns early
    # Socket stays in CLOSE_WAIT forever
```

**Verification:**
```bash
# If CLOSE_WAIT is growing over time, it's definitely a leak:
for i in {1..10}; do
    ss -tan | grep CLOSE_WAIT | wc -l
    sleep 10
done
# Count growing = application bug
```

**Fix:**
```bash
# This is an APPLICATION BUG, not a system config issue

# Workaround (temporary): Restart the service
systemctl restart buggy_app

# Permanent fix: Fix the application
# Ensure all code paths call socket.close()
# Use try/finally to guarantee cleanup
# Use context managers (with statements in Python)
```

---

### Issue 5: Connection Reset

**Error:** `Connection reset by peer` or `curl: (56) Recv error: Connection reset by peer`

**Investigation:**
```bash
# Step 1: See the state when reset happens
ss -tan | grep example.com

# Connection will show RST flag or disappear

# Step 2: Check server-side logs for the reset
journalctl -u nginx -n 50
# Look for: "Connection reset", "Broken pipe", errors

# Step 3: Check if it's a timeout
# Connection stays open for a while then reset

# Step 4: Check if specific requests reset
curl -v https://example.com 2>&1 | grep -i reset

# Step 5: Check if it's a load balancer
# Load balancers sometimes reset connections after timeout
# Check load balancer timeout settings
```

**Root Causes:**
- **Server crashed:** Process died while connection open
- **Idle timeout:** Server closes idle connections
- **Memory pressure:** Server ran out of memory, killed connection
- **Firewall resets:** Some firewalls reset instead of dropping
- **Load balancer:** Closed connection per timeout or max connections

**Fixes:**
```bash
# Check server health
systemctl status nginx
ps aux | grep nginx

# Check if it's timing out
# Enable TCP keepalive
curl --keepalive-time 60 https://example.com

# For application:
socket.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)

# Check firewall timeouts
# AWS: ALB connection timeout (default 60s)
# HAProxy: server timeout default
```

---

## SRE Troubleshooting Scenarios

### Scenario 1: Port Exhaustion - Can't Create New Connections

**Problem:** Application getting "cannot assign requested address" errors.

**Investigation:**
```bash
# Step 1: Check total number of sockets
ss -tan | tail -n +2 | wc -l
# May show 60,000+ sockets

# Step 2: Break down by state
ss -tan | tail -n +2 | awk '{print $NF}' | sort | uniq -c | sort -rn

# Example:
#   50000 TIME_WAIT
#    5000 ESTABLISHED
#    2000 LISTEN

# Step 3: Calculate how long until these clear
# If 50k TIME_WAIT connections, and each is 60s:
# At 60 second timeout, approximately:
# New connections created = Total / 60
# If creating 1000/sec, will cycle through in 50 seconds

# Step 4: Check available ports
# Usually 65535 - 1024 = 64,511 usable ports
# But many reserved, realistically ~30,000

# Step 5: Monitor port usage trend
for i in {1..10}; do
    echo "$(date): $(ss -tan | wc -l) sockets"
    sleep 5
done
```

**Root Cause:**
Creating connections faster than they close.

**Fix:**
```bash
# Immediate: Enable SO_REUSEADDR
sysctl -w net.ipv4.tcp_tw_reuse=1

# Reduce TIME_WAIT
sysctl -w net.ipv4.tcp_fin_timeout=30

# Increase port range
sysctl -w net.ipv4.ip_local_port_range="1024 65535"

# Long-term: Connection pooling in application
# Don't create new connection per request
```

---

### Scenario 2: Intermittent "Connection refused" But Service Running

**Problem:** `curl example.com` works sometimes, fails sometimes. Service shows as running.

**Investigation:**
```bash
# Step 1: Service is actually running
systemctl status nginx
ps aux | grep nginx

# Step 2: But what port is it listening on?
ss -tln | grep nginx

# Expected: Should show LISTEN
# If not there: Service crashed/died

# Step 3: Service crashed and auto-restarted
journalctl -u nginx --since "5 minutes ago" | grep -iE "restart|fail|error"

# Step 4: Check if it's crashing on certain requests
# Monitor and test simultaneously
while true; do
    curl example.com -w "%{http_code}\n" -o /dev/null
    sleep 1
done
# In another terminal:
watch -n 1 'ss -tln | grep nginx'

# If service disappears after a request, that's the pattern
```

**Root Causes:**
- **Service crashes on certain requests:** Check logs
- **Auto-restart failing:** Startup error
- **Memory limit hit:** OOMKilled
- **Resource exhaustion:** File descriptor limit

**Fix:**
```bash
# Check why it crashes
journalctl -u nginx -n 100

# Increase limits if needed
systemctl edit nginx
# Add: LimitNOFILE=65536

systemctl restart nginx
```

---

### Scenario 3: Interview Question - "How Do You Debug TCP Connection Issues?"

**Your answer (structured):**

```bash
# "I'd start by checking three things:

# 1. Is the SERVICE listening on the port?
ss -tln | grep :8080

# 2. Can you REACH the host?
ping example.com

# 3. Can you REACH the specific PORT?
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/example.com/8080'
# Or: curl -v https://example.com:8080

# If service is listening but still times out:
# It's a network issue (firewall, routing)

# If service is NOT listening but should be:
# Check why it's not running:
systemctl status service
journalctl -u service -n 50

# If service was listening but now isn't:
# Check if it crashed:
ps aux | grep service
# If gone: check logs for crash

# For intermittent issues:
# Watch both the service and the connection attempts
watch -n 1 'ss -tan | grep :8080'
# While in another terminal:
for i in {1..20}; do curl example.com; sleep 1; done

# This shows if service is disappearing or restarting
"
```

---

## Interview Tips

1. **Know the 3-way handshake:** SYN → SYN-ACK → ACK (3 packets)
2. **Know the states:** LISTEN, SYN_SENT, ESTABLISHED, FIN_WAIT, TIME_WAIT, CLOSE_WAIT
3. **Know TIME_WAIT:** 60 seconds, when closer waits before reusing port
4. **Know CLOSE_WAIT:** Application bug, not calling close()
5. **Know common errors:** Connection refused (not listening), Timeout (unreachable), Reset (crashed)
6. **ss vs netstat:** `ss` is newer, faster, preferred
7. **Debugging approach:** Listen → Reach → Port → Firewall → Logs

---

## Common Pitfalls

1. **Assuming firewall blocking without checking:** Test locally first with `localhost`
2. **Not checking if service is actually listening:** `ss -tln` before debugging further
3. **Confusing CLOSE_WAIT with normal states:** It's a bug, not temporary
4. **Ignoring TIME_WAIT explosion:** Can cause "port exhaustion"
5. **Not checking logs:** Service crash info is in `journalctl`
6. **Forgetting the full path:** Service runs, but on different IP/port than expected
7. **Network vs Application confusion:** Use layered approach (DNS → ping → port → service)
