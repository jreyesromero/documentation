# dig & nslookup - DNS Queries

`dig` and `nslookup` query DNS servers to resolve domain names and troubleshoot DNS issues. Critical for diagnosing connectivity and routing problems.

## Quick Reference

```bash
# dig - modern DNS querying
dig example.com                       # Query A record
dig example.com +short                # Short output
dig example.com MX                    # Query MX records
dig example.com NS                    # Query nameservers
dig @8.8.8.8 example.com              # Query specific DNS server
dig +trace example.com                # Show full DNS resolution path

# nslookup - traditional DNS queries
nslookup example.com                  # Query A record
nslookup example.com 8.8.8.8          # Query specific server
nslookup -type=MX example.com         # Query MX records
```

---

## dig - Domain Information Groper

### Output Format

```
; <<>> DiG 9.10.6 <<>> google.com +short
; (1 server found)
;; global options: +cmd
216.58.205.46
```

With more detail:
```
; <<>> DiG 9.10.6 <<>> google.com
;; Query time: 50 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Mon Oct 05 10:37:32 UTC 2026
;; MSG SIZE  rcvd: 55
```

### Common Queries

```bash
# A record (IPv4)
dig example.com A

# AAAA record (IPv6)
dig example.com AAAA

# MX record (mail servers)
dig example.com MX

# NS record (nameservers)
dig example.com NS

# CNAME record (aliases)
dig example.com CNAME

# TXT record (text, SPF, DKIM)
dig example.com TXT

# All records
dig example.com ANY
```

### Flags

| Flag | Meaning |
|------|---------|
| **+short** | Minimal output (just answers) |
| **@server** | Query specific nameserver |
| **+trace** | Show full resolution chain |
| **+nocmd** | Suppress command line echoing |
| **+noall +answer** | Show only answers |
| **+noall +authority** | Show only authority section |

---

## nslookup - Name Server Lookup

### Basic Syntax

```bash
nslookup [domain] [server]
```

### Common Record Types

```bash
nslookup example.com                 # Default: A record
nslookup -type=MX example.com        # MX records
nslookup -type=NS example.com        # Nameservers
nslookup -type=CNAME example.com     # Aliases
```

---

## Critical Interview Scenarios

### 1. "Domain isn't resolving. Debug the DNS issue."

```bash
# Step 1: Check if domain resolves at all
dig example.com +short

# Step 2: Check with public DNS
dig @8.8.8.8 example.com

# Step 3: Trace the resolution path
dig +trace example.com

# Step 4: Check the authoritative nameserver
dig @ns1.example.com example.com
```

### 2. "Service discovery isn't working. Check DNS."

```bash
# In Kubernetes/service mesh context
dig +short service.namespace.svc.cluster.local

# In traditional DNS
dig @dns.server.com service.internal
```

### 3. "Mail isn't being routed. Check MX records."

```bash
dig example.com MX

# Output example:
# example.com.  3600  IN  MX  10 mail.example.com.
# example.com.  3600  IN  MX  20 mail2.example.com.
```

### 4. "Check if DNS is causing connectivity issues"

```bash
# Compare different DNS servers
dig @8.8.8.8 example.com +short
dig @1.1.1.1 example.com +short

# If different results, DNS issue
```

### 5. "Verify SPF/DKIM records"

```bash
# SPF record
dig example.com TXT | grep "v=spf1"

# DKIM record
dig selector._domainkey.example.com TXT
```

---

## Practical Troubleshooting

### DNS Resolution Not Working

```bash
# Test with local resolver
nslookup example.com

# Test with specific DNS server
nslookup example.com 8.8.8.8

# If one works and other doesn't, DNS configuration issue
```

### Slow DNS Queries

```bash
# Check query time
dig example.com | grep "Query time"

# Try different DNS server
dig @8.8.8.8 example.com | grep "Query time"

# If local DNS is slow, network or resolver issue
```

### Wrong IP for Domain

```bash
# Check A record
dig example.com A

# Check if it matches expected IP
# If not, DNS cache or record not updated
```

### Trace DNS Resolution Chain

```bash
# See full path from root nameserver down
dig +trace example.com

# Output shows:
# . -> .com nameserver -> example.com nameserver -> result
```

---

## Real-World Examples

### Find email servers for a domain

```bash
dig example.com MX

# Result: Mail should route to these IPs
```

### Test DNS failover

```bash
# Query primary DNS
dig @ns1.example.com example.com

# Query secondary DNS
dig @ns2.example.com example.com

# Results should match
```

### Check DNS propagation

```bash
# Check multiple nameservers
for ns in 8.8.8.8 1.1.1.1 208.67.222.222; do
  echo "Checking $ns:"
  dig @$ns example.com +short
done
```

### Verify CNAME vs A records

```bash
# Check if it's a CNAME
dig www.example.com

# Check target
dig example.com A
```

---

## Interview Tips

1. **Know the difference:** `dig` is modern and powerful, `nslookup` is simpler and more universal
2. **Know common record types:** A (IPv4), AAAA (IPv6), MX (mail), NS (nameserver), CNAME (alias)
3. **Always use `+short` for cleaner output** in interviews
4. **Know how to query specific servers:** `dig @server.com domain.com`
5. **Know how to trace resolution:** `dig +trace` shows the full path

---

## Comparison: dig vs nslookup

| Aspect | dig | nslookup |
|--------|-----|----------|
| **Output** | Detailed, verbose | Simple, legacy |
| **Query types** | All standard types | Basic types |
| **Performance** | Fast | Slower |
| **Modern systems** | Preferred | Legacy |
| **Trace** | `+trace` flag | Limited |
| **Use case** | SRE debugging, detailed queries | Simple lookups |

---

## Common Record Types Reference

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | 216.58.205.46 |
| **AAAA** | IPv6 address | 2607:f8b0:4004:809::200e |
| **CNAME** | Alias to another domain | www -> example.com |
| **MX** | Mail server | mail.example.com (priority 10) |
| **NS** | Nameserver | ns1.example.com |
| **TXT** | Text records (SPF, DKIM, etc.) | v=spf1 include:... |
| **SOA** | Start of Authority | Serial, refresh times |
| **PTR** | Reverse DNS | IP to domain mapping |

