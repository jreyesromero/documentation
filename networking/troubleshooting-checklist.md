# Network Troubleshooting Checklist

Complete systematic workflow for debugging network connectivity issues. Use this during production incidents or interviews.

## Quick Triage (< 5 minutes)

When something isn't working:

```bash
# 1. CAN YOU REACH IT?
ping api.example.com

# 2. CAN YOU REACH THE PORT?
nc -zv api.example.com 443

# 3. DOES IT RESPOND?
curl -i https://api.example.com

# 4. WHAT'S THE ERROR?
curl -v https://api.example.com 2>&1 | tail -20
```

**Decision:** Which step failed? Jump to that section below.

---

## Failure Type 1: "Name or service not known" (DNS Failure)

**Error:** `curl: (6) Could not resolve host: api.example.com`

### Quick Check
```bash
dig api.example.com +short
# If empty output or "NXDOMAIN": DNS is broken
```

### Full Investigation
```bash
# 1. Is the domain registered?
whois api.example.com | grep -i "domain status"

# 2. Can any nameserver resolve it?
dig api.example.com @8.8.8.8           # Public DNS
dig api.example.com @1.1.1.1           # Cloudflare DNS

# 3. What does your local resolver see?
cat /etc/resolv.conf
dig api.example.com @10.0.0.1          # Your configured resolver

# 4. What's the full resolution chain?
dig api.example.com +trace             # Shows each hop

# 5. Is it cached incorrectly?
sudo systemctl restart systemd-resolved
dig api.example.com +nocache +short    # Bypass cache
```

### Common Causes & Fixes

| Cause | Check | Fix |
|-------|-------|-----|
| Domain doesn't exist | `dig api.example.com` returns NXDOMAIN | Verify spelling, check if domain is registered |
| Wrong nameserver | `cat /etc/resolv.conf` | Point to correct nameserver |
| Nameserver down | `dig @ns1.example.com` fails | Use backup nameserver |
| DNS cache stale | Same result with `+nocache` | Wait for TTL to expire or `systemctl restart systemd-resolved` |
| Internal DNS only | `dig @8.8.8.8` works, local fails | Use correct internal DNS |

### Fix Script
```bash
# If wrong nameserver configured:
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null

# Force DNS refresh:
sudo systemctl restart systemd-resolved
dig api.example.com +short
```

---

## Failure Type 2: "Connection refused" (Service Not Listening)

**Error:** `curl: (7) Failed to connect to api.example.com port 443: Connection refused`

### Quick Check
```bash
# Is anything listening on that port?
ss -tln | grep :443

# If nothing shows: Service not listening
# If shows LISTEN: Service is there, but connection still refused = firewall
```

### Full Investigation
```bash
# 1. Is service running?
systemctl status api-service
ps aux | grep api-service

# 2. What ports is it listening on?
ss -tln | grep api
netstat -tln | grep api

# 3. Is it listening on the right IP?
# 0.0.0.0:443 = listening on all interfaces
# 127.0.0.1:443 = listening on localhost only (won't accept remote)

# 4. Did it just crash?
journalctl -u api-service -n 50
# Look for: "Exited", "Crashed", "segmentation fault"

# 5. Can you reach it locally?
curl http://localhost:8080          # Test locally (port 8080)

# 6. Is firewall blocking?
iptables -L -n | grep 443
firewall-cmd --list-all
```

### Common Causes & Fixes

| Cause | Check | Fix |
|-------|-------|-----|
| Service not running | `ps aux \| grep api` | `systemctl start api-service` |
| Wrong port config | `ss -tln`, check config file | Update config, restart |
| Firewall blocking | `iptables -L`, cloud security groups | `firewall-cmd --add-port=443/tcp --permanent` |
| Port already in use | `lsof -i :443` | Kill other process or use different port |
| Service crashed | `journalctl -u service` | Check logs, fix issue, restart |

### Fix Script
```bash
# Start service
systemctl start api-service

# Verify it's listening
ss -tln | grep :443

# Check logs for errors
journalctl -u api-service -f

# If still not listening, check config
systemctl cat api-service | grep -i "port\|listen"
```

---

## Failure Type 3: "Connection timeout" (Host Unreachable)

**Error:** `curl: (7) Failed to connect to api.example.com port 443: Operation timed out`

### Quick Check
```bash
# Can you reach the host at all?
ping api.example.com

# If timeout or "Destination Host Unreachable": Network problem
```

### Full Investigation
```bash
# 1. Is host reachable?
ping -c 3 api.example.com
# Response times should be < 500ms

# 2. Check routing
ip route | grep api.example.com
traceroute api.example.com            # Shows path

# 3. Check if any hop is dropping packets
mtr -c 10 api.example.com             # Monitor connection

# 4. Check from your IP what's happening
# From within same network:
ping api.example.com

# From outside network (different internet):
# (Compare latency and success rate)

# 5. Check firewall rules
# On your server:
iptables -L -n | grep DROP
firewall-cmd --list-ports

# On cloud (AWS/GCP/Azure):
# Check security groups / network ACLs
```

### Common Causes & Fixes

| Cause | Check | Fix |
|-------|-------|-----|
| Network down | `ping` times out | Check internet connection, router |
| Firewall blocking | `traceroute` stops, `mtr` shows loss | Add firewall rule, check security groups |
| Host unreachable | `ip route`, no route | Check routing, check DNS |
| Slow network | `ping` response > 1000ms | Contact ISP, check congestion |
| Server down | DNS works, ping fails | Check if host is actually up |

### Fix Script
```bash
# Open firewall port
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload

# Check routing
ip route add default via 192.168.1.1

# Test connectivity
ping api.example.com
nc -zv api.example.com 443
```

---

## Failure Type 4: "SSL Certificate Problem" (HTTPS/TLS Error)

**Error:** `curl: (60) SSL certificate problem`

### Quick Check
```bash
# What's the cert error?
openssl s_client -connect api.example.com:443 < /dev/null 2>/dev/null | \
  openssl x509 -noout -dates

# Shows: notBefore=... notAfter=...
# If current date is after notAfter: EXPIRED
```

### Full Investigation (See tls.md for details)
```bash
# 1. Check certificate
openssl s_client -connect api.example.com:443 < /dev/null 2>/dev/null | \
  openssl x509 -noout -subject -issuer -dates

# 2. Check if expired
openssl s_client -connect api.example.com:443 < /dev/null 2>/dev/null | \
  openssl x509 -noout -dates | grep notAfter

# 3. Check if hostname matches
openssl s_client -connect api.example.com:443 < /dev/null 2>/dev/null | \
  openssl x509 -noout -subject -text | grep -E "CN=|DNS:"

# 4. Check certificate chain (intermediate missing?)
openssl s_client -connect api.example.com:443 -showcerts < /dev/null 2>/dev/null | \
  grep "BEGIN CERTIFICATE" | wc -l
# Should be 2 or 3 (end entity + intermediate + possibly root)
```

### Common Causes & Fixes

| Cause | Check | Fix |
|-------|-------|-----|
| Cert expired | `openssl x509 -dates` | Renew: `certbot renew` |
| Hostname mismatch | `openssl x509 -subject` | Upload correct cert |
| Self-signed | `issuer == subject` | For production, get CA-signed cert |
| Chain incomplete | `showcerts` shows < 2 certs | Use fullchain.pem instead of cert.pem |

### Fix Script
```bash
# Renew Let's Encrypt cert
certbot renew --force-renewal -d api.example.com

# Or get new cert
certbot certonly --standalone -d api.example.com

# Copy to correct location
cp /etc/letsencrypt/live/api.example.com/fullchain.pem /etc/nginx/certs/

# Reload nginx (don't restart)
systemctl reload nginx

# Verify
openssl s_client -connect api.example.com:443 < /dev/null 2>/dev/null | \
  openssl x509 -noout -dates
```

---

## Failure Type 5: "Bad Gateway" / "Service Unavailable" (5xx Errors)

**Error:** `HTTP/1.1 502 Bad Gateway` or `503 Service Unavailable`

### Quick Check
```bash
# 502 = Proxy can't reach backend
# 503 = Service temporarily unavailable

# Check backend service
systemctl status api-service
```

### Full Investigation
```bash
# 1. Is backend service running?
ps aux | grep api-service

# 2. Is it listening on expected port?
ss -tln | grep :8080

# 3. Can you reach it directly?
curl http://localhost:8080/health    # From the host

# 4. What does load balancer see?
# From load balancer server:
curl http://10.0.1.10:8080/health    # Direct backend IP

# 5. Check backend logs
journalctl -u api-service -n 50

# 6. Check load balancer logs
journalctl -u nginx -n 50              # or -u haproxy, etc.
```

### Common Causes & Fixes

| Cause | Check | Fix |
|-------|-------|-----|
| Backend down | `ps aux` | `systemctl start api-service` |
| Backend overloaded | `top`, load average | Scale up, optimize code |
| Backend slow | Response time > 30s | Check database, external deps |
| LB health check failing | LB logs | Fix health check endpoint |
| Network partition | `traceroute` to backend | Check connectivity |

### Fix Script
```bash
# Restart backend service
systemctl restart api-service

# Monitor startup
journalctl -u api-service -f

# Check if it's healthy
curl http://localhost:8080/health

# If still broken, check logs
journalctl -u api-service --since "5 minutes ago" | tail -50
```

---

## Failure Type 6: "Connection Reset by Peer" (Unexpected Close)

**Error:** `curl: (56) Recv error: Connection reset by peer`

### Quick Check
```bash
# Is the server running?
systemctl status api-service

# Intermittent or consistent?
for i in {1..10}; do curl -s https://api.example.com/endpoint && echo "OK" || echo "FAIL"; done
```

### Full Investigation
```bash
# 1. Check service stability
# Monitor and test simultaneously:
watch -n 1 'systemctl status api-service'

# In another terminal:
while true; do
  curl -s https://api.example.com/endpoint && echo "Success" || echo "Fail"
  sleep 1
done

# 2. Check if service crashes on certain requests
# Add detail to requests to see pattern:
for i in {1..20}; do
  curl -v https://api.example.com/endpoint 2>&1 | grep -i "reset\|close"
  sleep 2
done

# 3. Check service logs for crashes
journalctl -u api-service | grep -i "segfault\|crashed\|exception"

# 4. Check memory/resources
ps aux | grep api-service | head -1
# Check MEM% column

# 5. Check if firewall is resetting
# Unlikely, but possible:
tcpdump -i any 'host api.example.com' | grep RST
```

### Common Causes & Fixes

| Cause | Check | Fix |
|-------|-------|-----|
| Service crashes | `journalctl` after reset | Fix crash, update code |
| Resource exhausted | `ps aux` %MEM high | Increase limits, optimize |
| Timeout on idle | Idle > threshold | Keep-alive, shorter timeout |
| Server overloaded | Load average, response time | Scale up, optimize |
| Firewall active reset | `tcpdump` | Allow connections, check rules |

### Fix Script
```bash
# Check what's crashing it
journalctl -u api-service --since "10 minutes ago" | head -100

# Increase resource limits
systemctl edit api-service
# Add: MemoryMax=2G

# Restart
systemctl restart api-service

# Monitor
journalctl -u api-service -f
```

---

## Interview Scenario: "Complete Diagnosis"

**Problem Given:** "Users report the API is down. What do you do?"

**Your response (2-3 minutes of talking):**

```
"I'd approach this systematically, starting with the broadest checks:

Step 1 - LOCAL VERIFICATION (Does problem exist?)
- From my terminal: curl -v https://api.example.com/health
- If this works: Problem is user-side, geographic, or intermittent

Step 2 - CONNECTIVITY (Can I reach the server?)
- ping api.example.com          # Network reachability
- nc -zv api.example.com 443    # Port connectivity

Step 3 - DNS (Is name resolving?)
- dig api.example.com +short    # Check DNS
- If fails: DNS issue (see DNS troubleshooting)

Step 4 - SERVICE (Is it running?)
- SSH to api server
- ps aux | grep api-service     # Is it running?
- ss -tln | grep :8080          # Is it listening?
- journalctl -u api-service -n 50  # Did it crash?

Step 5 - RESPONSE (What's the error?)
- If service running: curl -v http://localhost:8080
- Check logs for recent errors

Step 6 - SCALE CHECK (Is it overloaded?)
- top (CPU/memory?)
- netstat -an | grep ESTABLISHED | wc -l (connection count?)

Step 7 - DEPENDENCIES (Did a dependency break?)
- Check if database is up
- Check if cache layer is up
- Check external API dependencies

Action:
- If service crashed: restart and monitor
- If service slow: check database/external deps
- If overloaded: scale or optimize
- If resource limit: increase and restart
"
```

---

## Complete Checklist (Print & Keep Handy)

```
[ ] 1. Verify issue exists locally (curl -v)
[ ] 2. Test DNS (dig)
[ ] 3. Test connectivity (ping)
[ ] 4. Test port (nc -zv)
[ ] 5. Check service status (systemctl)
[ ] 6. Check service listening (ss -tln)
[ ] 7. Check logs (journalctl)
[ ] 8. Check resources (top, free, df)
[ ] 9. Check connections (netstat -antp)
[ ] 10. Check firewall (iptables, firewall-cmd)
[ ] 11. Check routing (ip route, traceroute)
[ ] 12. Check certificates (openssl s_client)
[ ] 13. Monitor while reproducing (watch, tail -f)
[ ] 14. Document findings
[ ] 15. Implement fix
[ ] 16. Verify resolution
[ ] 17. Monitor for stability
```

---

## Key Principles

1. **Start broad, narrow down:** Network → Service → Logs
2. **Layer by layer:** DNS → TCP → HTTP → Application
3. **Isolate the problem:** Test locally first, then remote
4. **Distinguish client vs server:** Is it your problem or theirs?
5. **Check logs first:** Usually shows exact error
6. **Verify before fixing:** Confirm root cause
7. **Monitor after fixing:** Ensure it stays fixed
