# HTTP - HyperText Transfer Protocol

HTTP is the application layer protocol for web communication. Understanding HTTP status codes, methods, and headers is essential for SRE troubleshooting.

## Quick Reference

```bash
# Basic request
curl https://example.com

# See headers
curl -i https://example.com               # Include headers in output
curl -I https://example.com               # Headers only

# Verbose (see request + response)
curl -v https://example.com
curl -v --trace - https://example.com    # Very detailed

# Different methods
curl -X GET https://example.com
curl -X POST -d "data" https://example.com
curl -X PUT -d "data" https://example.com
curl -X DELETE https://example.com

# With headers
curl -H "Authorization: Bearer token123" https://example.com
curl -H "Content-Type: application/json" -d '{"key": "value"}' https://example.com

# Follow redirects
curl -L https://example.com              # Follow 301/302/etc

# Ignore SSL/TLS errors (debugging only, not production)
curl -k https://example.com              # Ignore cert errors

# Set timeout
curl --max-time 5 https://example.com    # Fail after 5 seconds

# Write response to file
curl -o response.html https://example.com
curl -O https://example.com/file.zip     # Keep original filename
```

---

## HTTP Methods

| Method | Purpose | Body? | Idempotent? | Safe? | Cacheable? |
|--------|---------|-------|------------|-------|-----------|
| **GET** | Retrieve resource | No | Yes | Yes | Yes |
| **POST** | Create resource | Yes | No | No | No |
| **PUT** | Replace resource entirely | Yes | Yes | No | No |
| **PATCH** | Partially update resource | Yes | No | No | No |
| **DELETE** | Delete resource | No | Yes | No | No |
| **HEAD** | Like GET, but no body | No | Yes | Yes | Yes |
| **OPTIONS** | What methods allowed? | No | Yes | Yes | No |
| **TRACE** | Loop-back test (diagnostic) | No | Yes | No | No |

**Idempotent:** Running twice = same result (safe to retry)
**Safe:** Doesn't modify state on server

---

## HTTP Status Codes

### 2xx - Success

| Code | Meaning | Details |
|------|---------|---------|
| **200** | OK | Request succeeded, response has body |
| **201** | Created | POST succeeded, new resource created |
| **202** | Accepted | Request accepted but not yet processed (async) |
| **204** | No Content | Success but no body (DELETE often returns this) |

### 3xx - Redirection

| Code | Meaning | Details |
|------|---------|---------|
| **301** | Moved Permanently | Resource moved, update bookmarks (search engines follow) |
| **302** | Found (Temporary Redirect) | Resource moved temporarily, don't update bookmarks |
| **304** | Not Modified | Client cache is current (use cached version) |
| **307/308** | Temporary/Permanent Redirect | Like 302/301 but method preserved |

**See Location header for new URL:**
```bash
curl -i https://example.com 2>&1 | grep -i location
```

### 4xx - Client Error

| Code | Meaning | Root Cause |
|------|---------|-----------|
| **400** | Bad Request | Malformed request (invalid JSON, missing fields) |
| **401** | Unauthorized | Auth required, missing/invalid credentials |
| **403** | Forbidden | Auth OK but no permission (ACL/RBAC) |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | POST to read-only resource, etc. |
| **409** | Conflict | Concurrent update conflict (optimistic locking) |
| **429** | Too Many Requests | Rate limited |

### 5xx - Server Error

| Code | Meaning | Root Cause |
|------|---------|-----------|
| **500** | Internal Server Error | Application error/exception |
| **502** | Bad Gateway | Proxy received invalid response from upstream |
| **503** | Service Unavailable | Server overloaded or under maintenance |
| **504** | Gateway Timeout | Proxy waited too long for upstream |

---

## Understanding a curl Response

```bash
$ curl -v https://api.example.com/users/123

> GET /users/123 HTTP/1.1
> Host: api.example.com
> User-Agent: curl/7.68.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: application/json
< Content-Length: 256
< Server: nginx/1.18.0
< X-Response-Time: 45ms
<
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

| Part | Meaning |
|------|---------|
| `>` | Request (what we sent) |
| `<` | Response (what we got) |
| `HTTP/1.1 200 OK` | Status line (protocol, code, message) |
| Headers | Metadata (content-type, cache, auth, etc.) |
| Body | Actual data (JSON, HTML, binary, etc.) |

---

## Common HTTP Headers

### Request Headers (Client → Server)

```bash
# Authorization
curl -H "Authorization: Bearer token123" https://api.example.com

# Content type
curl -H "Content-Type: application/json" https://api.example.com

# Accept (what format do you want back?)
curl -H "Accept: application/json" https://api.example.com

# User-Agent (who are you?)
curl -H "User-Agent: MyApp/1.0" https://api.example.com

# Cookie (session state)
curl -H "Cookie: session_id=abc123" https://api.example.com
```

### Response Headers (Server → Client)

```bash
# Content-Type (format of body)
Content-Type: application/json; charset=utf-8

# Location (where to find resource)
Location: https://example.com/new-location
# Usually with 301/302 redirects

# Set-Cookie (store this session)
Set-Cookie: session_id=abc123; Path=/; Max-Age=3600

# Cache-Control (how long to cache)
Cache-Control: max-age=3600  # Cache for 1 hour

# Server (what software is running)
Server: nginx/1.18.0

# X-Response-Time (diagnostic header)
X-Response-Time: 45ms
```

---

## Real-World Scenarios

### Scenario 1: API Returning 500 Error

**Problem:** `curl https://api.mycompany.com/endpoint` returns `500 Internal Server Error`

**Investigation:**
```bash
# 1. Verify the error
curl -i https://api.mycompany.com/endpoint
# Shows: HTTP/1.1 500 Internal Server Error

# 2. Check response body for error message
curl -s https://api.mycompany.com/endpoint | jq .
# May show: {"error": "database connection failed"}

# 3. Check if all endpoints are broken or just one
curl -i https://api.mycompany.com/health
curl -i https://api.mycompany.com/other

# 4. Check server logs
journalctl -u api-service -n 50
# Look for: stack trace, database error, out of memory

# 5. Check if it's intermittent
for i in {1..10}; do curl -w "Status: %{http_code}\n" -s https://api.mycompany.com/endpoint; done
# If alternating 200 and 500: race condition or memory issue
```

**Root Causes:**
- **Database down:** Can't connect to database
- **Out of memory:** Application crashed with OOMKilled
- **Crashed service:** Application threw unhandled exception
- **External API down:** Service depends on something offline
- **Code deploy:** Bad deployment introduced bug

**Fix:**
```bash
# Check service status
systemctl status api-service
ps aux | grep api-service

# Check database connectivity
# (depends on your database)
psql -h database.internal -d myapp -c "SELECT 1;"

# Check logs for specific error
journalctl -u api-service --since "1 minute ago"

# Restart service
systemctl restart api-service

# Monitor for recovery
journalctl -u api-service -f
```

---

### Scenario 2: 502 Bad Gateway - Proxy Issue

**Problem:** `curl https://api.example.com` returns `502 Bad Gateway`

**Meaning:** There's a proxy/load balancer in front, and it can't reach the backend service.

**Investigation:**
```bash
# 1. Confirm the 502
curl -i https://api.example.com
# Shows: HTTP/1.1 502 Bad Gateway

# 2. Check if backend service is running
# SSH to backend server
systemctl status backend-api
ps aux | grep backend

# 3. Check if load balancer can reach backend
# From load balancer:
curl http://10.0.1.5:8080/health  # Direct backend IP
# If this works but 502 still happens: LB config issue

# 4. Check load balancer logs
journalctl -u nginx -n 50  # or -u haproxy
# Look for: "no available backend", "connection refused", timeout

# 5. Check backend service logs
journalctl -u backend-api -n 50

# 6. Check if backend is listening
ss -tln | grep :8080
```

**Root Causes:**
- **Backend service down:** Crashed or not started
- **Backend overloaded:** All connections exhausted
- **Health check failing:** LB marked backend as down
- **Network issue:** LB can't reach backend (firewall, network partition)
- **Timeout:** Backend slow, LB gave up

**Fix:**
```bash
# Restart backend service
systemctl restart backend-api

# If it keeps crashing, check logs
journalctl -u backend-api --since "10 minutes ago" | tail -50

# Check if it's a startup issue
systemctl start backend-api
systemctl status backend-api

# Monitor logs during restart
journalctl -u backend-api -f
```

---

### Scenario 3: 429 Too Many Requests - Rate Limited

**Problem:** `curl https://api.mycompany.com/endpoint` returns `429 Too Many Requests`

**Investigation:**
```bash
# 1. Check response headers for rate limit info
curl -i https://api.mycompany.com/endpoint | head -20
# Look for: X-RateLimit-*, Retry-After

# Example:
# X-RateLimit-Limit: 100
# X-RateLimit-Remaining: 0
# X-RateLimit-Reset: 1634567890

# 2. Check when limit resets
# If Retry-After: 60
# Wait 60 seconds and retry

# 3. Check if you're making too many requests
# Count requests in last minute:
journalctl -u api-service --since "1 minute ago" | grep "GET /endpoint" | wc -l

# 4. Check if it's from a specific client (IP)
# If you're load testing:
for i in {1..100}; do curl -w "%{http_code}\n" -s https://api.mycompany.com/endpoint; done
# You'll eventually hit 429

# 5. Check rate limiter configuration
# May be in nginx, load balancer, or application
```

**Root Causes:**
- **Too many requests:** Legitimate high load or load testing
- **API credentials exhausted:** Service account has rate limit
- **DDoS:** Malicious request flood
- **Retries:** Application retrying failed requests in loop

**Fix:**
```bash
# If legitimate high load:
# - Implement request batching
# - Add jitter to retries
# - Cache responses
# - Upgrade to higher tier (if paid API)

# If DDoS:
# - Enable rate limiting at CDN/proxy level
# - Block IPs with excessive requests
# - WAF rules

# Wait for limit reset
Retry-After=$(curl -s -i https://api.mycompany.com | grep -i retry-after | awk '{print $2}')
sleep $Retry-After
curl https://api.mycompany.com
```

---

### Scenario 4: 401 Unauthorized - Auth Not Working

**Problem:** `curl https://api.mycompany.com/private` returns `401 Unauthorized`

**Investigation:**
```bash
# 1. Verify you're sending auth
curl -v https://api.mycompany.com/private 2>&1 | grep -i "authorization\|auth"
# If no Authorization header: You're not sending credentials

# 2. Send auth header
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.mycompany.com/private
# Check response

# 3. Verify token is valid
# Check token endpoint:
curl -X POST https://api.mycompany.com/token \
  -H "Content-Type: application/json" \
  -d '{"user": "me", "password": "secret"}'
# Get new token if expired

# 4. Check token expiry
# Tokens often have expiration:
curl -H "Authorization: Bearer token123" https://api.mycompany.com/profile
# If: 401 Unauthorized + "token expired"
# Need to refresh

# 5. Check server logs for auth failures
journalctl -u api-service | grep -i "auth\|unauthorized\|invalid"
```

**Root Causes:**
- **Missing auth header:** Not sending credentials
- **Invalid token:** Wrong token or malformed
- **Expired token:** Need to refresh/re-authenticate
- **Invalid credentials:** Wrong username/password
- **Insufficient permissions:** Auth OK but no access (403 instead)

**Fix:**
```bash
# Get valid token
TOKEN=$(curl -s -X POST https://api.mycompany.com/token \
  -H "Content-Type: application/json" \
  -d '{"user": "me", "password": "secret"}' | jq -r '.token')

# Use token in requests
curl -H "Authorization: Bearer $TOKEN" https://api.mycompany.com/private

# Or if using API key:
curl -H "X-API-Key: your-api-key" https://api.mycompany.com/private
```

---

## SRE Troubleshooting Scenarios

### Scenario 1: Slow API Response - Timing Investigation

**Problem:** API is responding slowly (5+ seconds). Users complaining.

**Investigation:**
```bash
# 1. Measure response time
curl -w "@format.txt" https://api.mycompany.com/endpoint

# Create format.txt:
# time_namelookup: %{time_namelookup}
# time_connect: %{time_connect}
# time_appconnect: %{time_appconnect}
# time_pretransfer: %{time_pretransfer}
# time_redirect: %{time_redirect}
# time_starttransfer: %{time_starttransfer}
# time_total: %{time_total}

# Output shows:
# time_namelookup: 0.002      # DNS lookup
# time_connect: 0.050         # TCP handshake
# time_appconnect: 0.150      # TLS handshake
# time_pretransfer: 0.152
# time_starttransfer: 2.500   # SERVER RESPONSE (slow!)
# time_total: 2.510

# 2. Identify bottleneck:
# starttransfer > appconnect = Server processing slow

# 3. Check if it's consistent
for i in {1..5}; do
  curl -w "Time: %{time_total}s\n" -s -o /dev/null https://api.mycompany.com/endpoint
  sleep 1
done

# 4. Check if it's that endpoint or all endpoints
curl -w "Time: %{time_total}s\n" -s -o /dev/null https://api.mycompany.com/health
curl -w "Time: %{time_total}s\n" -s -o /dev/null https://api.mycompany.com/users

# 5. Check server load
ssh api-server
top  # or: uptime, load average

# 6. Check application logs
journalctl -u api-service --since "1 minute ago" | grep -i slow
```

**Root Causes:**
- **Server overloaded:** Too many requests
- **Slow database:** Query taking too long
- **External API call:** Service calls slow external API
- **Large response:** Slow data serialization
- **Memory issues:** Swapping, cache misses

**Fix:**
```bash
# Short-term: Scale up (add more servers)
# Long-term: Optimize code/query
#   - Add caching
#   - Optimize database queries
#   - Reduce response payload
```

---

### Scenario 2: Interview Question - "Walk Me Through `curl https://example.com`"

**Your detailed explanation (what interviewers want to hear):**

```
"There are several layers involved:

1. DNS RESOLUTION (0-100ms)
   - curl needs to resolve example.com to an IP
   - Queries recursive resolver (8.8.8.8, 1.1.1.1, or internal)
   - If cached, immediate; if not, goes to authoritative
   - Result: example.com → 93.184.216.34

2. TCP HANDSHAKE (10-100ms typical)
   - curl sends SYN to 93.184.216.34:443
   - Server responds with SYN-ACK
   - curl sends final ACK
   - TCP connection established

3. TLS HANDSHAKE (50-200ms typical)
   - curl: 'Client Hello' with supported ciphers
   - Server: 'Server Hello' + certificate
   - curl: Verifies certificate against CA
   - Both: Key exchange (Diffie-Hellman or similar)
   - Result: Encrypted tunnel established

4. HTTP REQUEST (1-100ms)
   - curl sends: GET / HTTP/1.1 headers
   - Server receives and processes request
   - Server sends: HTTP/1.1 200 OK status + headers + body

5. RESPONSE (depends on size)
   - curl receives response headers
   - curl receives response body
   - curl displays in terminal (or saves to file)

POTENTIAL FAILURES at each stage:
- DNS timeout → can't resolve
- TCP timeout → host unreachable or firewall
- DNS returns NXDOMAIN → domain doesn't exist
- TLS cert invalid → certificate error
- Connection reset → server crashed
- 500 error → application error
- 429 rate limit → too many requests
"
```

---

## Interview Tips

1. **Know the main status codes:** 200, 301, 400, 401, 403, 404, 500, 502, 503
2. **Know the difference:** 401 (auth) vs 403 (permission)
3. **Know curl basics:** `-v` for verbose, `-H` for headers, `-X` for method
4. **Understand HTTP methods:** GET (safe), POST (creates), PUT (replaces), PATCH (updates)
5. **Know idempotency:** GET, PUT, DELETE are idempotent (safe to retry)
6. **Know common patterns:** 301/302 redirect, 304 not modified, 429 rate limit
7. **Understand layering:** HTTP runs on TLS/TCP/DNS

---

## Common Pitfalls

1. **Not checking response headers:** Status code alone doesn't explain everything
2. **Ignoring timeouts:** Slow responses often indicate deeper issues
3. **Following redirects blindly:** Sometimes you want to see the 301, not the final page
4. **Using wrong HTTP method:** POST instead of GET for fetch operations
5. **Not validating SSL certificates:** `-k` in curl ignores cert errors (debugging only)
6. **Confusing 401 and 403:** 401 = no auth, 403 = no permission
7. **Assuming 5xx is always server bug:** Could be downstream service down
