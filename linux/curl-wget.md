# curl & wget - HTTP Testing

`curl` makes HTTP requests from the command line, useful for testing APIs, checking connectivity, and debugging HTTP issues. `wget` is similar but focuses on downloading files.

## Quick Reference

```bash
# curl - HTTP requests
curl https://api.example.com                      # GET request
curl -X POST https://api.example.com              # POST request
curl -H "Authorization: Bearer TOKEN" URL         # Add header
curl -d '{"key": "value"}' -H "Content-Type: application/json" URL  # POST with JSON
curl -I https://example.com                       # HEAD request (headers only)
curl -v https://example.com                       # Verbose (debug)
curl -o filename https://example.com              # Save to file

# wget - download files
wget https://example.com/file.tar.gz              # Download file
wget -r https://example.com                       # Recursive download
wget --limit-rate=100K https://example.com        # Limit bandwidth
```

---

## curl - Making HTTP Requests

### Basic Syntax

```bash
curl [options] URL
```

### Common Options

| Option | Meaning |
|--------|---------|
| **-X METHOD** | HTTP method (GET, POST, PUT, DELETE, PATCH) |
| **-H "Header"** | Add header |
| **-d "data"** | POST data |
| **-I** | Headers only (HEAD request) |
| **-v** | Verbose (show request/response headers) |
| **-i** | Include response headers in output |
| **-L** | Follow redirects |
| **-u user:pass** | Basic authentication |
| **-w "%{http_code}"** | Show HTTP status code |
| **-o filename** | Save response to file |

---

## Critical Interview Scenarios

### 1. "Check if API endpoint is responding"

```bash
curl -v https://api.example.com/health

# Or just get status code
curl -w "%{http_code}\n" -o /dev/null -s https://api.example.com/health
```

Output interpretation:
- **200** — OK
- **301/302** — Redirect
- **401** — Unauthorized
- **403** — Forbidden
- **404** — Not Found
- **500** — Server Error
- **503** — Service Unavailable

### 2. "Test API with authentication"

```bash
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/endpoint

# With API key
curl -H "X-API-Key: $API_KEY" https://api.example.com/endpoint
```

### 3. "POST JSON data to API"

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"username": "test", "password": "pass"}' \
  https://api.example.com/login
```

### 4. "Check response headers without body"

```bash
curl -I https://example.com

# Or with verbose for more detail
curl -v -I https://example.com
```

### 5. "Debug SSL/TLS issues"

```bash
curl -v https://example.com

# Ignore certificate warnings (NOT for production!)
curl -k https://example.com

# Show certificate details
curl --cacert /path/to/cert.pem https://example.com
```

### 6. "Test with custom headers"

```bash
curl -H "X-Custom-Header: value" \
     -H "Authorization: Bearer token" \
     https://api.example.com
```

---

## Troubleshooting Common Issues

### Connection Refused

```bash
curl -v https://localhost:8080

# Check if service is listening
netstat -tln | grep 8080
```

### Timeout

```bash
# Increase timeout
curl --max-time 30 https://slow-api.example.com

# Connect timeout only
curl --connect-timeout 10 https://api.example.com
```

### SSL Certificate Error

```bash
# Trust self-signed certs (test only!)
curl -k https://self-signed.example.com

# Or use specific certificate
curl --cacert /path/to/ca.pem https://example.com
```

### Redirect Not Followed

```bash
# Follow redirects
curl -L https://example.com
```

---

## Advanced Scenarios

### Monitor API response time

```bash
curl -w "Time: %{time_total}s\n" -o /dev/null -s https://api.example.com

# More detailed timing
curl -w "\nConnect time: %{time_connect}s\nTotal time: %{time_total}s\n" \
  -o /dev/null -s https://api.example.com
```

### Load test endpoint (repeatedly)

```bash
for i in {1..100}; do
  curl -w "%{http_code}\n" -o /dev/null -s https://api.example.com
done | sort | uniq -c
```

### Upload file

```bash
curl -F "file=@/path/to/file.txt" https://upload.example.com
```

### Using environment variables (secure passwords)

```bash
curl -u $USERNAME:$PASSWORD https://api.example.com
```

### Pretty print JSON response

```bash
curl -s https://api.example.com | jq
```

---

## wget - Downloading Files

### Common Usage

```bash
wget https://example.com/file.tar.gz

# Save with different name
wget https://example.com/file.tar.gz -O myfile.tar.gz

# Continue partial download
wget -c https://example.com/largefile.iso

# Limit bandwidth (100K per second)
wget --limit-rate=100K https://example.com/file.iso
```

---

## Real-World Examples

### Test database connectivity via HTTP proxy

```bash
curl -v http://proxy.internal:8080 \
  -H "Host: database.internal:5432"
```

### Check if microservice is healthy

```bash
for service in api auth db cache; do
  echo -n "$service: "
  curl -s -o /dev/null -w "%{http_code}\n" \
    http://$service.internal:8080/health
done
```

### Get JSON data and parse it

```bash
curl -s https://api.example.com/users | jq '.users[] | .name'
```

### Monitor API status continuously

```bash
watch 'curl -s -w "%{http_code}\n" -o /dev/null https://api.example.com/health'
```

---

## Interview Tips

1. **Know basic curl syntax:** `curl -X METHOD -H "Header" -d "data" URL`
2. **Know HTTP status codes:** 2xx (success), 3xx (redirect), 4xx (client error), 5xx (server error)
3. **Use `-v` for verbose debugging** — shows request/response headers
4. **Know `-w "%{http_code}"` for status codes** — useful for checking endpoints
5. **Know basic authentication:** `-u user:pass`
6. **Know how to POST JSON:** `-H "Content-Type: application/json" -d '...'`

---

## curl vs wget

| Aspect | curl | wget |
|--------|------|------|
| **Purpose** | HTTP requests & testing | File downloading |
| **Auth** | Built-in (headers, basic) | Limited |
| **Output** | stdout by default | File |
| **Use case** | API testing, SRE debugging | Downloads, mirrors |
| **Speed** | Lightweight | Can be heavier |

