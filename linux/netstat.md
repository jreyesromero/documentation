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

