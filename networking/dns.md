# DNS - Domain Name System

DNS translates domain names (example.com) into IP addresses. Critical for understanding connectivity issues, caching, and name resolution failures in production.

## Quick Reference

```bash
# Basic queries
dig example.com                    # Full DNS query
dig example.com +short             # IP only
dig example.com A                  # A records (IPv4)
dig example.com AAAA               # AAAA records (IPv6)
dig example.com MX                 # Mail servers
dig example.com NS                 # Name servers
dig example.com CNAME              # Aliases
dig example.com TXT                # Text records

# Advanced queries
dig example.com @8.8.8.8           # Use specific nameserver
dig example.com +trace             # See full resolution chain
dig example.com +nocache           # Bypass local cache
nslookup example.com               # Simpler interface
host example.com                   # Even simpler
```

---

## DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | IPv4 address | example.com → 93.184.216.34 |
| **AAAA** | IPv6 address | example.com → 2606:2800:220:1:248:1893:25c8:1946 |
| **CNAME** | Alias | www.example.com → example.com |
| **MX** | Mail server | Priority + mail server address |
| **NS** | Name server | Which server holds the truth |
| **TXT** | Text data | SPF, DKIM, domain verification |
| **SOA** | Start of Authority | Serial, refresh, retry, expire times |
| **SRV** | Service | Service name + port (Kubernetes uses this) |

---

## Understanding DNS Resolution

### The Full Resolution Flow

```
Your Application
        ↓
    (Step 1) Local /etc/hosts file?
        ↓ NO
    (Step 2) Local resolver cache?
        ↓ NO
    (Step 3) Query Recursive Resolver (8.8.8.8, 1.1.1.1)
        ↓
    (Step 4) Recursive Resolver → Root Nameserver
        "Where is .com?"
        ↓
    (Step 5) Root → TLD Nameserver (.com registry)
        "Where is example.com?"
        ↓
    (Step 6) TLD → Authoritative Nameserver
        "Give me all records for example.com"
        ↓
    (Step 7) Authoritative returns: A, MX, NS records
        ↓
    (Step 8) Recursive Resolver caches + returns to client
        ↓
    Client Application gets: 93.184.216.34
```

### Key Concept: TTL (Time To Live)

```bash
# dig example.com shows:
# example.com.      3599 IN A 93.184.216.34
#          ^^^^^ = TTL in seconds = 59 minutes 59 seconds

# Meaning: Resolver will cache for ~1 hour, then re-query
# Low TTL (300s) = Frequent updates possible, more queries
# High TTL (86400s) = Efficient caching, slower propagation
```

### dig Output Explained

```bash
$ dig example.com

; <<>> DiG 9.10.6 <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 5

;; QUESTION SECTION:
;example.com.            IN  A

;; ANSWER SECTION:
example.com.        3599 IN  A   93.184.216.34

;; AUTHORITY SECTION:
example.com.        172799 IN NS a.iana-servers.net.
example.com.        172799 IN NS b.iana-servers.net.

;; ADDITIONAL SECTION:
a.iana-servers.net. 172799 IN A   199.43.135.53
b.iana-servers.net. 172799 IN A   199.43.135.49
```

**Interpretation:**
- **QUESTION**: "What's the A record for example.com?"
- **ANSWER**: "example.com is 93.184.216.34, good for 3599 seconds"
- **AUTHORITY**: "Ask these nameservers (they know the truth)"
- **ADDITIONAL**: "Here are the IPs of those nameservers"

---

## DNS Caching Layers

```
Browser Cache (1-60 min)
    ↓
OS Resolver Cache (systemd-resolved, dnsmasq)
    ↓
ISP/Cloud Recursive Resolver (8.8.8.8, 1.1.1.1)
    ↓
Authoritative Nameserver (example.com's official server)
```

### Important: Cache Invalidation

```bash
# Flush browser cache
# (varies by browser - Chrome: chrome://net-internals/#dns)

# Flush OS resolver cache (Linux)
sudo systemctl restart systemd-resolved

# macOS
sudo dscacheutil -flushcache

# Verify no cache
dig +nocache example.com
```

---

## Real-World Scenarios

### Scenario 1: Application Can't Resolve Domain - Debug DNS

**Problem:** Application logs show "unknown host: database.mycompany.com"

**Investigation Steps:**
```bash
# 1. Does DNS resolve from your shell?
dig database.mycompany.com
# If ANSWER section is empty = DNS failure

# 2. Can you reach the nameserver?
dig database.mycompany.com @8.8.8.8  # Try public DNS
# If this works but internal doesn't = internal DNS issue

# 3. Check which nameserver your system uses
cat /etc/resolv.conf
# Shows: nameserver 10.0.0.1 (internal)

# 4. Trace the full resolution
dig database.mycompany.com +trace
# Shows each hop: root → TLD → authoritative

# 5. Check if it's a CNAME chain issue
dig database.mycompany.com +short
# May show multiple CNAME hops

# 6. Check TTL and expiry
dig database.mycompany.com | grep -i database
# High TTL = might be cached somewhere
```

**Common Causes:**
- **NXDOMAIN** (Non-existent domain): Domain doesn't exist
- **SERVFAIL**: Nameserver error, upstream unreachable
- **TIMEOUT**: Nameserver not responding (network issue)
- **Wrong nameserver**: Application using different resolver

**Fix:**
```bash
# If NXDOMAIN, verify spelling and zone setup:
dig database.mycompany.com NS

# If SERVFAIL, check authoritative nameserver:
dig @ns1.mycompany.com database.mycompany.com

# If timeout, check network connectivity to nameserver:
ping 10.0.0.1  # Internal nameserver
```

---

### Scenario 2: DNS Changed But Old Value Still Cached

**Problem:** You updated DNS record (A record IP changed), but requests still go to old IP.

**Investigation:**
```bash
# 1. Check what DNS says NOW
dig example.com +short
# Shows: 93.184.216.35 (new IP)

# 2. Check if your resolver cached the old value
# Clear cache and try again
sudo systemctl restart systemd-resolved
dig example.com +short
# If still old IP = problem persists

# 3. Check authoritative nameserver directly
dig example.com @ns1.example.com +short
# Shows the source of truth

# 4. Check TTL
dig example.com | grep example.com | head -1
# If TTL=0, it expired and should have refreshed

# 5. Check who's caching (browser, app, load balancer)
# May need to restart app/service to flush DNS cache
```

**Root Causes:**
- **Browser cache**: Restart browser or clear cache
- **Application cache**: App may cache DNS internally (needs restart)
- **Load balancer cache**: Check if it caches DNS lookups
- **Connection pooling**: Old connections still open to old IP

**Fix:**
```bash
# Restart application to clear DNS cache
systemctl restart myapp

# Verify new connections use new IP
netstat -antp | grep myapp | head -5
# Should show connections to new IP

# For long-lived apps, may need connection refresh
```

---

### Scenario 3: Intermittent "Name Not Found" Errors

**Problem:** Application intermittently fails to resolve a hostname. Works sometimes, fails sometimes.

**Investigation:**
```bash
# 1. Try multiple times, see if it fails
for i in {1..10}; do dig example.com +short; sleep 1; done

# 2. Check if there are multiple nameservers
cat /etc/resolv.conf
# If multiple nameservers, one might be down

# 3. Check nameserver health
dig @ns1.example.com example.com
dig @ns2.example.com example.com
# One might be failing

# 4. Check TTL expiry pattern
for i in {1..5}; do dig example.com | grep example.com | head -1; sleep 10; done
# TTL counts down (good) or stays same (cached)?

# 5. Check if it's a load balancing issue
# Round-robin DNS returns multiple IPs
dig example.com +short
# May return 5 different IPs

# 6. Check application timeout settings
# App may timeout before DNS responds
strace -e trace=open,read curl example.com 2>&1 | grep -i dns
```

**Common Causes:**
- **Nameserver down**: One of N nameservers is failing
- **Network timeout**: DNS query takes too long
- **Round-robin**: Multiple IPs, one host is down
- **Low TTL + high load**: Cache misses cause delays

**Fix:**
```bash
# Increase application timeout for DNS
# Check app config for DNS timeout setting

# For nameserver redundancy:
# Verify at least 2 nameservers are healthy
dig NS example.com

# If one is down, alert infrastructure team
```

---

### Scenario 4: Internal DNS Works From Some Servers, Not Others

**Problem:** App on Server A can resolve internal.corp, but App on Server B cannot.

**Investigation:**
```bash
# On Server A (working):
cat /etc/resolv.conf
# nameserver 10.0.0.1

# On Server B (broken):
cat /etc/resolv.conf
# nameserver 8.8.8.8 (different resolver!)

# Test each nameserver
dig internal.corp @10.0.0.1    # Should work
dig internal.corp @8.8.8.8     # Will fail (external DNS doesn't know internal domains)

# Check network connectivity from Server B to internal DNS
ping 10.0.0.1  # May be blocked by firewall

# Check DHCP/system config
ip route  # Routing table
systemctl status systemd-resolved
```

**Root Causes:**
- **Different DNS server**: Server B using external instead of internal
- **Network isolation**: Server B can't reach internal DNS (firewall)
- **DHCP misconfiguration**: Server B got wrong nameserver from DHCP

**Fix:**
```bash
# Manually set internal nameserver on Server B
sudo nano /etc/resolv.conf
# Add: nameserver 10.0.0.1

# Or fix DHCP to assign correct nameserver
# (Check DHCP server configuration)

# Verify
dig internal.corp
```

---

### Scenario 5: DNS Propagation Issue - Old Registrar DNS Still Used

**Problem:** Changed DNS nameservers at registrar, but old nameserver still getting queries.

**Investigation:**
```bash
# 1. What does the registrar think the nameservers are?
whois example.com | grep -i nameserver
# Shows nameservers registered at ICANN

# 2. Query the ROOT nameserver to see what it knows
dig example.com @a.root-servers.net +short

# 3. Query the old nameserver
dig example.com @old-ns.example.com

# 4. Query the new nameserver
dig example.com @new-ns.example.com

# 5. Check propagation globally
# Use online tools: https://dnschecker.org
# Shows which global resolvers have updated
```

**Root Cause:** Nameserver change takes 24-48 hours to propagate globally.

**Fix:**
```bash
# Wait for propagation OR
# Lower TTL before the change so old values expire faster

# In the meantime, update local /etc/hosts as workaround:
echo "93.184.216.34 example.com" >> /etc/hosts

# Or point app to specific nameserver:
dig @new-ns.example.com example.com
```

---

### Scenario 6: DNS Spoofing / DNSSEC Validation

**Problem:** Security concern: Is DNS response trustworthy?

**Check DNSSEC:**
```bash
# Check if domain has DNSSEC enabled
dig example.com +dnssec

# Look for "ad" flag (authenticated data)
# If present = DNSSEC validated response

# Verify DNSKEY records
dig example.com DNSKEY

# Check for tampering risk
dig example.com +cd +dnssec
# If result changes with +cd, DNSSEC prevented spoofing
```

---

## SRE Troubleshooting Scenarios

### Scenario 1: Production Outage - Service Unreachable "Host Not Found"

**Problem:** Sudden outage. Users report "could not resolve api.mycompany.com". Service restarted but same error.

**Emergency Investigation (5 minutes):**
```bash
# 1. Is DNS working at all?
dig api.mycompany.com
# If ANSWER is empty = DNS failure

# 2. Try from different network
# If on corporate network, try 8.8.8.8
dig api.mycompany.com @8.8.8.8

# 3. Check if it's cached incorrectly
dig api.mycompany.com +nocache @8.8.8.8

# 4. Check nameserver status
dig NS api.mycompany.com

# 5. Query authoritative directly
dig api.mycompany.com @ns1.mycompany.com

# 6. Check if domain expired (SOA record)
dig api.mycompany.com SOA
```

**Triage Path:**
- **dig shows IP** → DNS is fine, problem is elsewhere (connectivity, service down)
- **dig returns NXDOMAIN** → Domain missing or deleted
- **dig timeout** → Nameserver unreachable (network issue, firewall blocking)
- **dig SERVFAIL** → Nameserver itself has problem

**Actions:**
```bash
# If NXDOMAIN:
# Check if domain was accidentally deleted
# Roll back or re-add A record immediately

# If SERVFAIL:
# Escalate to infrastructure team (nameserver down)
# Failover to backup nameserver if available

# Temporary workaround:
echo "10.0.0.5 api.mycompany.com" >> /etc/hosts
systemctl restart myapp  # Force app to re-resolve
```

---

### Scenario 2: Slow DNS Causing Timeouts

**Problem:** Intermittent 10+ second hangs, then requests work. Application timeouts.

**Investigation:**
```bash
# 1. Measure DNS query time
time dig example.com

# Expected: < 100ms
# If > 500ms: nameserver is slow or unreachable

# 2. Check which nameserver is slow
time dig example.com @8.8.8.8
time dig example.com @1.1.1.1
time dig example.com @10.0.0.1  (internal)

# 3. Check network latency to nameserver
ping -c 3 10.0.0.1
# High latency = network problem

# 4. Check if nameserver is overloaded
# Count DNS queries from this host
tcpdump -i any dst port 53 | wc -l
# High count = DNS storms (something querying constantly)

# 5. Check application timeout for DNS
# Add timeout to curl/app config
curl --max-time 2 https://example.com
```

**Root Causes:**
- **Slow nameserver**: Infrastructure issue
- **High latency to nameserver**: Network/firewall issue
- **DNS queries timing out**: Nameserver overloaded

**Fix:**
```bash
# Use faster public DNS
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf

# Or cache DNS locally
# Install dnsmasq for local caching DNS resolver

# Increase application DNS timeout
export CURL_TIMEOUT=5
```

---

### Scenario 3: Interview Question - "Application Can't Connect to database.internal"

**Your debugging flow (show in interview):**

```bash
# "I'd start by verifying DNS resolution:"
dig database.internal +short

# "If that returns an IP, DNS is fine. If not:"
dig database.internal +trace
# Shows where the failure is in the chain

# "Next, I'd check if it's a network issue:"
nc -zv database.internal 5432  # Can we reach the IP on the port?

# "If DNS works but nc fails, it's a network/firewall issue, not DNS:"
ping <IP_from_dig>

# "I'd also check which nameserver my app is using:"
cat /etc/resolv.conf

# "And check if caching might be stale:"
sudo systemctl restart systemd-resolved
dig database.internal +short

# "Finally, check if there's a CNAME chain:"
dig database.internal CNAME
# Each CNAME adds latency
```

---

## Interview Tips

1. **Know the layers:** Application → Resolver → Root → TLD → Authoritative
2. **Know DNS commands:** `dig`, `nslookup`, `host`, understand flags like `+trace`, `+short`
3. **Know record types:** A, AAAA, CNAME, MX, NS, TXT, SOA
4. **Know TTL implications:** Low TTL = more queries, High TTL = slower propagation
5. **Know caching layers:** Browser, OS, resolver, authoritative
6. **Common errors:** NXDOMAIN, SERVFAIL, TIMEOUT = different root causes
7. **Debugging approach:** Start broad (can you resolve?) then narrow down (where does it fail?)

---

## Common Pitfalls

1. **Forgetting to check nameserver specifically:** `dig example.com @ns1.example.com`
2. **Not understanding CNAME chains:** Each CNAME adds a lookup
3. **Assuming DNS is cached:** Always test with `+nocache` if caching suspected
4. **Not checking TTL before DNS changes:** Change TTL first, then wait, then change IP
5. **Using `nslookup` for production troubleshooting:** `dig` is more powerful and scriptable
6. **Not verifying /etc/resolv.conf:** Different servers may have different nameservers
