# Networking Tools - curl, nc, tcpdump, traceroute

Essential command-line tools for network troubleshooting and SRE diagnostics.

## curl - HTTP/HTTPS Client

```bash
# Basic request
curl https://example.com

# See headers
curl -i https://example.com               # Include headers
curl -I https://example.com               # Headers only
curl -v https://example.com               # Verbose (full request/response)

# Detailed timing breakdown
curl -w "Total time: %{time_total}s\n" https://example.com
curl -w "DNS: %{time_namelookup}s, Connect: %{time_connect}s, TLS: %{time_appconnect}s, Total: %{time_total}s\n" https://example.com

# Different methods
curl -X POST -d "data" https://example.com
curl -X PUT -H "Content-Type: application/json" -d '{"key":"value"}' https://example.com
curl -X DELETE https://example.com

# Headers
curl -H "Authorization: Bearer token123" https://example.com
curl -H "User-Agent: MyApp/1.0" https://example.com

# Cookies
curl -b "session=abc123" https://example.com
curl -c /tmp/cookies.txt https://example.com     # Save cookies
curl -b /tmp/cookies.txt https://example.com     # Load cookies

# Redirect handling
curl -L https://example.com                      # Follow redirects
curl -L -i https://example.com                   # Follow + show headers

# SSL/TLS
curl -k https://example.com                      # Ignore cert errors (testing only)
curl --cacert /path/to/ca.pem https://example.com  # Custom CA

# Timeout
curl --max-time 5 https://example.com            # Fail after 5 seconds
curl --connect-timeout 2 https://example.com     # Connection timeout

# Output
curl -o filename.html https://example.com        # Save as filename.html
curl -O https://example.com/file.zip             # Keep original name

# Authentication
curl -u username:password https://example.com    # Basic auth
curl -H "Authorization: Bearer token" https://example.com  # Bearer token
```

### curl for Health Checks

```bash
# Simple health check (return HTTP code)
curl -s -o /dev/null -w "%{http_code}\n" https://api.example.com/health
# Returns just the status code: 200, 500, etc.

# With timeout
curl -s --max-time 5 -o /dev/null -w "%{http_code}\n" https://api.example.com/health

# Loop health checks (watch for failures)
while true; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health)
  echo "$(date '+%H:%M:%S') Status: $STATUS"
  sleep 5
done
```

---

## nc (netcat) - Network Swiss Army Knife

```bash
# Test if port is open
nc -zv example.com 443                           # Check if port open (-z = scan, -v = verbose)

# With timeout
nc -zv -w 5 example.com 443                      # 5 second timeout

# Port scanning (find open ports)
nc -zv example.com 80 443 8080 8443

# Listen for connections (server)
nc -l -p 9000                                    # Listen on port 9000

# Connect to server
nc example.com 9000                              # Connect

# Send data
echo "hello" | nc example.com 9000

# Transfer file
nc -l -p 9000 < file.txt                         # Server: send file
nc example.com 9000 > received_file.txt          # Client: receive file

# Test raw TCP connection (like telnet)
nc -v example.com 443
# Then type HTTP request manually or Ctrl+C to exit

# Get banner from service
nc -v example.com 22                             # SSH banner
nc -v example.com 80                             # HTTP banner
```

### nc for Troubleshooting

```bash
# Verify connectivity to specific port
if nc -zv -w 2 database.internal 5432; then
  echo "Database is reachable"
else
  echo "Database is NOT reachable"
fi

# Measure latency (simple)
# nc isn't perfect for timing, but can estimate:
time nc -zv example.com 443

# Check if port is listening locally
netstat -tln | grep :8080  # Better approach than nc for local check
```

---

## tcpdump - Packet Capture

```bash
# Capture packets on any interface
tcpdump -i any                                   # See all traffic

# Specific interface
tcpdump -i eth0                                  # Only eth0

# Specific port
tcpdump -i any port 443                          # Only port 443
tcpdump -i any port 80 or port 443               # Multiple ports

# Specific host
tcpdump -i any host example.com                  # To/from example.com
tcpdump -i any src 10.0.0.1                      # Only from this IP
tcpdump -i any dst 10.0.0.1                      # Only to this IP

# Specific protocol
tcpdump -i any tcp                               # Only TCP
tcpdump -i any udp                               # Only UDP

# Verbose output
tcpdump -i any -v port 443                       # Show more details
tcpdump -i any -vv port 443                      # Even more details

# Combine filters
tcpdump -i any tcp and port 443 and host example.com

# Show packet payload (first 100 bytes)
tcpdump -i any -A port 443 | head -50

# Save to file
tcpdump -i any -w /tmp/capture.pcap port 443
tcpdump -r /tmp/capture.pcap                     # Read file

# Count packets
tcpdump -i any -c 100 port 443                   # Capture 100 packets and stop

# Don't resolve hostnames (faster)
tcpdump -i any -n port 443

# Show just SYN packets (connection attempts)
tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0'
```

### tcpdump for Troubleshooting

```bash
# Count DNS queries
tcpdump -i any -nn 'udp port 53' | wc -l

# See DNS requests
tcpdump -i any -nn 'udp port 53' -A | grep -i "example"

# See TCP connection attempts
tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0' | head -20

# Monitor specific connection
tcpdump -i any -nn 'host 10.0.0.5 and port 3306'

# See all HTTP requests (raw)
tcpdump -i any -A 'tcp port 80' | grep -i "GET\|POST"

# See TLS handshake
tcpdump -i any -nn 'tcp port 443' | grep -i "Client Hello\|Server Hello"
```

---

## traceroute - Trace Network Path

```bash
# Basic traceroute
traceroute example.com

# Specify maximum hops
traceroute -m 20 example.com                     # Max 20 hops (default 30)

# UDP instead of ICMP (some networks block ICMP)
traceroute -U example.com

# TCP (more likely to pass firewalls)
traceroute -T example.com

# Show IP addresses only (no DNS resolution)
traceroute -n example.com

# Fast mode (less wait time)
traceroute -w 1 example.com                      # 1 second timeout per hop

# Combine options
traceroute -T -n -m 20 example.com
```

### traceroute Output Explained

```bash
$ traceroute example.com

traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1    10.0.0.1 (10.0.0.1)  0.123 ms  0.145 ms  0.134 ms
 2    192.168.1.1 (192.168.1.1)  1.234 ms  1.456 ms  1.345 ms
 3    40.79.150.1 (40.79.150.1)  12.345 ms  12.567 ms  12.456 ms
 4    64.233.167.1 (64.233.167.1)  23.456 ms  23.678 ms  23.567 ms
 5    93.184.216.34 (93.184.216.34)  45.678 ms  45.890 ms  45.789 ms
```

| Column | Meaning |
|--------|---------|
| **Hop #** | Route number (1 = first router, etc.) |
| **IP** | IP address of router |
| **Times** | Round-trip time (RTT) in milliseconds (3 attempts) |
| `*` | No response (router doesn't respond to traceroute) |

**Interpretation:**
- Each line shows latency to that router
- Timing should increase as you get further away
- Large jump = slow hop or distant network
- `*` responses = router not responding (usually OK, just not helpful)

---

## Real-World Troubleshooting Workflows

### Workflow 1: API Endpoint Not Responding

**Quick diagnosis (under 2 minutes):**

```bash
# 1. Can you reach the host?
ping api.example.com
# Or without DNS:
ping 93.184.216.34

# 2. Can you reach the specific port?
nc -zv api.example.com 443

# 3. Does HTTP work?
curl -i https://api.example.com/health

# 4. Is it slow? Get timing breakdown
curl -w "DNS: %{time_namelookup}s, Connect: %{time_connect}s, TLS: %{time_appconnect}s, Wait: %{time_starttransfer}s, Total: %{time_total}s\n" https://api.example.com/health

# 5. What's the actual error?
curl -v https://api.example.com/health 2>&1 | tail -30
```

**Decision tree:**
- If ping fails → Network issue, use `traceroute`
- If nc fails → Port not listening or firewall blocking
- If curl shows 5xx → Server issue, check server logs
- If curl slow → Network latency or server processing slow

---

### Workflow 2: Port Exhaustion - Can't Create New Connections

**Diagnosis:**

```bash
# 1. How many connections exist?
netstat -tan | tail -n +2 | wc -l

# 2. Break down by state
netstat -tan | tail -n +2 | awk '{print $NF}' | sort | uniq -c | sort -rn

# 3. Monitor new connections being created
# In one terminal:
watch -n 1 'netstat -tan | wc -l'

# In another, create connections:
for i in {1..100}; do
  nc -zv example.com 443 &
done

# 4. See which states are accumulating
netstat -tan | tail -n +2 | awk '{print $NF}' | sort | uniq -c | sort -rn
```

---

### Workflow 3: Intermittent DNS Failures

**Diagnosis:**

```bash
# 1. Check DNS resolution repeatedly
for i in {1..20}; do
  dig example.com +short
  sleep 1
done
# Look for pattern of failures

# 2. Monitor DNS queries with tcpdump
tcpdump -i any -nn 'udp port 53' -A | grep example

# 3. Check resolver health
curl -w "DNS: %{time_namelookup}s\n" https://example.com
# If DNS time > 1000ms, resolver is slow

# 4. Check nameserver response
dig example.com @8.8.8.8              # Try different resolver
dig example.com @1.1.1.1
```

---

## SRE Troubleshooting Checklist

```bash
# When something is broken, run this systematic check:

# 1. LAYER 1: Can you reach the host?
ping example.com                        # Measures latency, tests network

# 2. LAYER 2: Can you reach the port?
nc -zv example.com 443                  # Tests TCP connectivity

# 3. LAYER 3: Is service responding?
curl -I https://example.com             # Get headers, see status code

# 4. LAYER 4: What's the response?
curl -v https://example.com             # See full request/response

# 5. LAYER 5: How long does it take?
curl -w "Total: %{time_total}s\n" https://example.com

# 6. LAYER 6: What's the path?
traceroute example.com                  # See network path

# Each layer narrows down the problem
```

---

## Interview Tips

1. **curl is your primary tool:** Understand verbose mode, headers, timing
2. **nc is your connectivity tester:** Quick check if port is open
3. **tcpdump is your deep dive:** When you need to see actual packets
4. **traceroute shows path:** Where latency is hidden
5. **Layered approach:** Start broad (ping), narrow down (curl)
6. **Timing breakdowns matter:** DNS/Connect/TLS/Wait times tell the story

---

## Common Pitfalls

1. **Not using `-v` or `-i` with curl:** Can't see what's happening
2. **Running tcpdump without filters:** Too much data, easy to miss the issue
3. **Assuming firewall without testing:** Test locally first with `nc`
4. **Forgetting DNS resolution:** Slow DNS looks like slow server
5. **Not checking full response with curl:** Just seeing status isn't enough
