# netstat - Network Statistics

`netstat` displays network connections, routing tables, and network interface statistics. Critical for SRE troubleshooting network issues, port conflicts, and connection states.

## Quick Reference

```bash
netstat -an              # All connections, numeric (no DNS lookups)
netstat -tln             # TCP listening sockets
netstat -tnlp            # TCP listening with process info
netstat -s               # Protocol statistics summary
netstat -r               # Routing table
netstat -i               # Network interface stats
```

---

## Understanding Output

### Connection List (netstat -an)

```
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)    
tcp4       0      0  10.41.56.74.52877     40.79.150.124.443     ESTABLISHED
tcp4       0      0  10.41.56.74.52875     192.178.25.110.443    ESTABLISHED
tcp4       0      0  192.168.1.49.52872    52.123.202.175.443    ESTABLISHED
tcp4       0      0  10.41.56.74.52864     64.233.167.113.443    ESTABLISHED
```

| Column | Meaning |
|--------|---------|
| **Proto** | Protocol (tcp4, tcp6, udp4, etc.) |
| **Recv-Q** | Bytes received but not read by app (should be 0) |
| **Send-Q** | Bytes sent but not acknowledged (should be 0) |
| **Local Address** | IP:port listening/connected locally |
| **Foreign Address** | Remote IP:port |
| **(state)** | Connection state (ESTABLISHED, LISTEN, TIME_WAIT, etc.) |

### Connection States

| State | Meaning | Concern |
|-------|---------|---------|
| **ESTABLISHED** | Active connection | Normal |
| **LISTEN** | Socket waiting for connections | Normal (service listening) |
| **TIME_WAIT** | Waiting after close (2-minute timeout) | Normal, but too many means port churn |
| **CLOSE_WAIT** | Remote closed, local still has files open | App not closing connections properly |
| **SYN_SENT** | Initiating connection | Normal while connecting |
| **SYN_RECV** | Receiving connection request | Normal during handshake |
| **LAST_ACK** | Final acknowledgment | Brief, shouldn't accumulate |
| **FIN_WAIT1/FIN_WAIT2** | Waiting for connection termination | Brief, shouldn't accumulate |

---

## Critical Interview Questions

### 1. Find What's Using a Port

```bash
netstat -tlnp | grep :8080
```

Or for macOS (without -p):
```bash
netstat -tln | grep 8080
lsof -i :8080
```

**Answer format:** "I'd use netstat to check if port 8080 is already in use by another process."

### 2. Check Service Connectivity

```bash
netstat -an | grep 10.0.0.5:3306
```

This shows if app is connected to database at that IP/port.

### 3. Diagnose Connection Leaks

```bash
netstat -an | grep ESTABLISHED | wc -l
```

Run this over time. Increasing count suggests app isn't closing connections.

### 4. Identify Slow Clients

```bash
netstat -an | grep CLOSE_WAIT
```

If many connections in CLOSE_WAIT, remote clients aren't closing properly or app isn't flushing socket buffers.

---

## Common Scenarios

### Connection Refused Error

```bash
# Check if service is listening
netstat -tln | grep :5432

# If nothing shows, service isn't running or listening on wrong port
```

### Port Already in Use

```bash
netstat -an | grep LISTEN | grep :3000

# Shows what's using port 3000
```

### Too Many TIME_WAIT Connections

```bash
netstat -an | grep TIME_WAIT | wc -l

# Many TIME_WAIT means rapid connection churn (might be normal for API servers)
# But if paired with app failures, might need to tune TCP_ABORT_ON_OVERFLOW or decrease TIME_WAIT timeout
```

### Connection Backlog

```bash
netstat -s | grep -i "listen queue"

# Shows if app can't accept connections fast enough
```

---

## Flags Explained

| Flag | Meaning |
|------|---------|
| **-a** | All sockets (including listening) |
| **-n** | Numeric (IP instead of DNS, port number instead of service name) |
| **-t** | TCP only |
| **-u** | UDP only |
| **-l** | Listening sockets only |
| **-p** | Include process ID and name (Linux, requires sudo on some systems) |
| **-s** | Statistics by protocol |
| **-r** | Routing table |
| **-i** | Interface statistics |
| **-c** | Continuous (keep updating) |

---

## Modern Alternative: ss

On newer Linux systems, `ss` replaces `netstat`:

```bash
ss -tln              # TCP listening
ss -an               # All sockets
ss -tan              # TCP, all, numeric
```

`ss` is faster and more informative but may not be available on all systems.

---

## Real-World Scenarios

### API Server Not Responding?

```bash
# Check if service is listening
netstat -tln | grep :8080

# Check existing connections
netstat -an | grep 8080 | head -20

# If too many ESTABLISHED, might be connection pool exhaustion
netstat -an | grep 8080 | grep ESTABLISHED | wc -l
```

### Database Connection Issues

```bash
# Check if app is connecting to database
netstat -an | grep 3306 | grep ESTABLISHED

# Check connection backlog
netstat -s | grep 'socket buffer'
```

### Network Hung Process

```bash
# Find connections in abnormal states
netstat -an | grep FIN_WAIT1
netstat -an | grep CLOSE_WAIT

# If many of these, process might be stuck
```

---

## Performance Tips

- **Use numeric flag (`-n`):** DNS lookups slow down output significantly
- **Grep instead of browsing:** Piping through grep is faster than reading interactive output
- **Use `ss` if available:** Modern, faster, more info

---

## Interview Tip

When asked "How would you diagnose a network issue?" include:

1. `netstat -tln` to verify service is listening
2. `netstat -an` to check connection states
3. `netstat -s` to check for dropped packets/errors
4. Combine with `ps` to identify which process owns the connection

---

## SRE Troubleshooting Scenarios

### Scenario 1: Port Conflict - Service Won't Start

**Problem:** Service fails to start with "Address already in use" error.

**Investigation:**
```bash
# 1. What port is the service trying to use?
PORT=8080

# 2. Check if anything is listening on that port
netstat -tln | grep :$PORT

# 3. Get process info (which PID owns it?)
netstat -tlnp | grep :$PORT

# 4. Identify the owner process
ps aux | grep PID_FROM_ABOVE

# 5. Check what service owns this port
systemctl status service_name  # where service_name owns that PID

# 6. If it's an old service, kill it
kill -9 PID

# 7. Or find and stop the actual service
systemctl stop old_service
systemctl start new_service
```

**Expected Output:**
```bash
$ netstat -tlnp | grep 8080
tcp   0   0 0.0.0.0:8080   0.0.0.0:*   LISTEN   2847/java

# 2847 is the PID using port 8080
```

---

### Scenario 2: Connection Pool Exhaustion - Too Many Connections

**Problem:** "Connection refused" or "Too many open files" errors. Service appears hung.

**Diagnosis:**
```bash
# 1. Count total connections to a service
PORT=3306
netstat -an | grep :$PORT | wc -l

# 2. Break down by state
netstat -an | grep :$PORT | awk '{print $NF}' | sort | uniq -c | sort -rn

# 3. Example output - what does it mean?
#     100 ESTABLISHED   <- 100 active connections
#      50 TIME_WAIT     <- 50 waiting for timeout
#      10 CLOSE_WAIT    <- 10 stuck (not closing properly)

# 4. If many TIME_WAIT, ports are cycling too fast
netstat -an | grep TIME_WAIT | wc -l
# If > 1000, likely connection churn (new conn every second)

# 5. If many CLOSE_WAIT, app is leaking connections
netstat -an | grep CLOSE_WAIT | wc -l
# If > 100, app is not calling close() on connections

# 6. Check connection limits
ulimit -n  # Open file descriptor limit
# Also check: /proc/sys/net/ipv4/tcp_max_syn_backlog
```

**What Each State Means:**
- **ESTABLISHED** = active connection (counts toward limit)
- **TIME_WAIT** = waiting to timeout (eventually closes, temporary)
- **CLOSE_WAIT** = remote closed, local hasn't closed = **APP BUG**
- **SYN_RECV** = handshake in progress = normal during load

**Fix:**
```bash
# If CLOSE_WAIT is high (app not closing):
# Restart the application and check logs for connection handling bug

# If TIME_WAIT is high:
# Check /proc/sys/net/ipv4/tcp_tw_reuse or tcp_tw_recycle
# Or add delays between connections in code

# If hitting ulimit:
# Increase: ulimit -n 65536
# Or add to service file: LimitNOFILE=65536
```

---

### Scenario 3: Hung Connection - FIN_WAIT or CLOSE_WAIT Accumulation

**Problem:** Connections not closing properly. Services stop responding.

**Investigation:**
```bash
# 1. Find all hung states
netstat -an | grep -E "FIN_WAIT|CLOSE_WAIT|TIME_WAIT"

# 2. Count each state
netstat -an | awk '{print $NF}' | sort | uniq -c | sort -rn | head -10

# 3. Get the actual connections in bad state
netstat -an | grep CLOSE_WAIT | head -10

# 4. Find which process owns them
# Example output: 10.0.0.1:52345 - 10.0.0.2:3306 CLOSE_WAIT
# This means local process on :52345 talking to :3306

# 5. Find PID using source port
netstat -tlnp | grep :52345

# 6. Check that process
ps aux | grep PID

# 7. For Linux, use lsof for finer detail
lsof -i | grep CLOSE_WAIT
```

**Problem Analysis:**
- **FIN_WAIT1/FIN_WAIT2 accumulating** = remote not acknowledging close
- **CLOSE_WAIT accumulating** = local app not closing connection
- **TIME_WAIT > 1000** = system running out of ports (need SO_REUSEADDR)

**Solution:**
```bash
# For app bug (CLOSE_WAIT):
systemctl restart app_service

# For network issue (FIN_WAIT):
# Check if firewall is dropping packets
tcpdump -i any port 3306 -c 10

# For SO_REUSEADDR issue:
# Add to service or systemd config:
sysctl -w net.ipv4.tcp_tw_reuse=1
```

---

### Scenario 4: Database Connection Refused - Debug Connectivity

**Problem:** Application can't connect to database. Need to verify connectivity.

**Troubleshooting:**
```bash
# 1. Is database listening?
DB_HOST=192.168.1.10
DB_PORT=5432

netstat -tlnp | grep :$DB_PORT

# 2. Can you reach the database host?
curl -v telnet://$DB_HOST:$DB_PORT

# Or
nc -zv $DB_HOST $DB_PORT

# 3. Check established connections from app host to DB
netstat -an | grep $DB_HOST | grep $DB_PORT

# 4. Check connection states
netstat -an | grep $DB_HOST | awk '{print $NF}' | sort | uniq -c

# 5. Check for connection timeouts
netstat -s | grep -i timeout

# 6. Check routing to DB
traceroute $DB_HOST

# 7. Check firewall
# On database host
netstat -tlnp | grep $DB_PORT  # Is it listening?

# 8. Check network interface stats
netstat -i
# Look for errors/dropped on relevant interface
```

**Expected Healthy State:**
```bash
$ netstat -an | grep 192.168.1.10 | grep 5432
tcp   0   0 10.0.0.5:52345   192.168.1.10:5432   ESTABLISHED
```

**Possible Issues:**
- No ESTABLISHED connections = connection refused (firewall or DB down)
- Many SYN_SENT = firewall dropping packets or slow network
- Connection hangs = DB slow or overloaded

---

### Scenario 5: Listening Port Verification - Is Service Really Listening?

**Problem:** Service reports "running" but not responding. Verify it's actually listening.

**Verification:**
```bash
# 1. Service status says it's running
systemctl status myservice
# Shows: Active (running)

# 2. Check if it's actually listening
PORT=$(systemctl cat myservice | grep -oP 'port[=:\s]+\K\d+' | head -1)
# Or get from ps:
ps aux | grep myservice

# 3. Look for port in netstat
netstat -tlnp | grep myservice

# 4. Check specific port
netstat -tln | grep :$PORT

# 5. If not in netstat, maybe it's a Unix socket
netstat -ln | grep -i sock
# Or
ls -la /var/run/myservice.sock

# 6. If not listening at all - that's the problem
# Service is running but not listening = application initialization error
# Check logs:
journalctl -u myservice -n 50
```

**Expected Healthy:**
```bash
$ netstat -tlnp | grep myservice
tcp   0   0 0.0.0.0:8080   0.0.0.0:*   LISTEN   2847/myservice
```

**If Service Not in netstat Output:**
- Service started but failed to bind = configuration error
- Service crashed after starting = check logs immediately
- Service blocked by firewall = unlikely, but check `/proc/net/nf_conntrack`

---

### Scenario 6: Network Saturation - Check Traffic and Errors

**Problem:** Network appears slow. Check if links are saturated or dropping packets.

**Analysis:**
```bash
# 1. Check interface statistics
netstat -i

# Expected output format:
# Name    Mtu Network       RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR Flg
# eth0   1500 10.0.0.0/24  1.2M    0      0      0    950K    0      0      0     BMRU
# lo    65536 127.0.0.1    2.5M    0      0      0    2.5M    0      0      0     LRU

# 2. Look for errors/dropped
netstat -i | grep -v "^Iface\|^lo" | awk '{if ($4 > 0 || $5 > 0 || $7 > 0) print}'

# 3. Check connections per second
# Get snapshot 1
netstat -s > /tmp/net1.txt
sleep 60
netstat -s > /tmp/net2.txt

# Compare
diff /tmp/net1.txt /tmp/net2.txt | grep -i established

# 4. Look for protocol errors
netstat -s | grep -i "error\|drop\|timeout\|reset\|overflow"

# 5. Check TCP retransmissions
netstat -s | grep -i "retran"

# 6. Check if running out of connections
netstat -s | grep -i "socket"
```

**What to Look For:**
- **RX-ERR, TX-ERR** = physical layer problems
- **RX-DRP, TX-DRP** = dropped packets (buffer full or errors)
- **High TCP retransmit rate** = packet loss or congestion
- **High SYN_RECV** = under SYN flood attack or overwhelmed

**If Errors Detected:**
```bash
# Check MTU size (sometimes mismatch causes drops)
netstat -i | awk '{print $1, $2}'

# Check if it's a hardware issue
ethtool -S eth0  # Detailed interface stats (if available)

# Check Linux socket buffer sizes
sysctl net.core.rmem_max
sysctl net.core.wmem_max
# If connections are dropping, increase these
```

---

### Scenario 7: Interview Question - "How Do You Debug 'Connection Refused'?"

**Answer (comprehensive):**

```bash
# Step 1: Verify the service is running
systemctl status myservice
ps aux | grep myservice

# Step 2: Verify the port is listening
netstat -tlnp | grep myservice

# Step 3: If not listening, check logs
journalctl -u myservice -n 50

# Step 4: Check firewall if service is on different host
# From client:
nc -zv remote_host port

# Step 5: If service listening locally, test locally first
curl localhost:port

# Step 6: If remote, check routing
traceroute remote_host

# Step 7: Check if it's actually the right port/service
systemctl cat myservice | grep -i port
netstat -tlnp | head -20
```

**Interview Tips:**
- "Start with systemctl status to confirm running"
- "Then netstat to confirm listening"
- "Then logs to find the actual error"
- "Use nc or telnet to isolate network from application"
- "Always verify on the actual service host first"

