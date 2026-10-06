# TLS - Transport Layer Security (SSL/HTTPS)

TLS provides encrypted communication over the network. Understanding certificates, expiration, and validation is critical for production SRE work.

## Quick Reference

```bash
# Check certificate info
openssl s_client -connect example.com:443                    # Interactive, see cert
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -text -noout

# Check certificate expiry
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -dates

# Certificate file information
openssl x509 -in /path/to/cert.pem -text -noout
openssl x509 -in /path/to/cert.pem -noout -dates
openssl x509 -in /path/to/cert.pem -noout -subject -issuer

# Check private key
openssl rsa -in /path/to/key.pem -check

# Verify certificate and key match
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
# If hashes match: certificate and key pair correctly

# Create self-signed certificate (testing)
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Create CSR (Certificate Signing Request) for CA
openssl req -new -key key.pem -out request.csr
```

---

## Certificate Components

### X.509 Certificate Structure

```bash
Certificate:
  Data:
    Version: 3 (0x2)
    Serial Number: 0x1234567890abcdef
    Signature Algorithm: sha256WithRSAEncryption
    Issuer: C=US, O=Let's Encrypt, CN=R3
    Validity:
      Not Before: Oct  5 12:00:00 2023 GMT
      Not After : Oct  4 12:00:00 2024 GMT
    Subject: CN=example.com
    Public Key:
      RSA Public-Key: (2048 bit)
    X509v3 extensions:
      Subject Alternative Name:
        DNS:example.com, DNS:*.example.com, DNS:www.example.com
      Key Usage: critical
        Digital Signature, Key Encipherment
```

| Field | Meaning |
|-------|---------|
| **Issuer** | Who signed the certificate (Certificate Authority) |
| **Subject** | Who the certificate is for |
| **Not Before / Not After** | Validity period (cert is invalid outside this range) |
| **Public Key** | Used by clients to encrypt data |
| **Subject Alternative Names (SAN)** | Alternative DNS names covered by cert |
| **Serial Number** | Unique identifier for this certificate |

---

## Understanding Certificate Chains

```
Root CA (self-signed, installed in browser/OS)
  ↓
Intermediate CA (signed by Root)
  ↓
End Entity Certificate (your certificate, signed by Intermediate)
```

**Why the chain?**
- Browsers trust the Root CA (built-in)
- Root CA is offline, secure
- Intermediate CA does actual signing
- If Intermediate compromised, revoke it (Root still valid)

```bash
# See the full chain
openssl s_client -connect example.com:443 -showcerts 2>/dev/null | grep -A5 "-----BEGIN CERTIFICATE-----"

# Output shows 3 certificates:
# 1. End entity (example.com)
# 2. Intermediate CA
# 3. Root CA (may be omitted, browser already has it)
```

---

## Common TLS Errors

### Error 1: Certificate Expired

**Error:** `curl: (60) SSL certificate problem: certificate has expired`

**Check expiry:**
```bash
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -dates
# Output:
# notBefore=Oct 5 12:00:00 2022 GMT
# notAfter=Oct 4 12:00:00 2024 GMT
# (If notAfter is before today = EXPIRED)

# Check days remaining
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -dates | grep notAfter
```

**Root Cause:** Certificate validity period ended.

**Fix:**
```bash
# Get new certificate from CA
# For Let's Encrypt:
certbot renew

# Or manually:
certbot certonly --standalone -d example.com

# Then restart service to load new cert
systemctl restart nginx
```

---

### Error 2: Certificate Doesn't Match Hostname

**Error:** `curl: (60) SSL: certificate subject name does not match target host name`

**Investigation:**
```bash
# Check certificate subject
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -subject
# Output: subject=CN=old-domain.com

# Check Subject Alternative Names
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -text | grep -A1 "Subject Alternative"
# Output: DNS:old-domain.com, DNS:*.old-domain.com
# Notice: example.com NOT in list

# You're connecting to example.com, but cert is for old-domain.com
```

**Root Cause:** Certificate is for different domain (old cert, misconfiguration, CNAME pointing to wrong place).

**Fix:**
```bash
# If it's a CNAME issue:
# Make sure DNS CNAME points to server with correct certificate:
dig example.com CNAME  # May show: example.com -> other-server.com

# If redirecting, server must have certificate for destination domain:
# example.com → other-server.com (must have other-server.com in cert)

# If it's truly wrong cert:
# Upload certificate for correct domain
```

---

### Error 3: Self-Signed Certificate

**Error:** `curl: (60) SSL certificate problem: self signed certificate`

**Investigation:**
```bash
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -issuer -subject
# Output:
# issuer=CN=example.com (self-signed)
# subject=CN=example.com

# If issuer == subject: SELF-SIGNED (not trusted)
```

**Root Cause:** Certificate signed by itself, not by trusted CA. Fine for testing, not for production.

**Workaround (testing only):**
```bash
# Ignore cert validation
curl -k https://example.com  # -k = insecure, don't validate cert

# Or update with real certificate
```

---

### Error 4: Can't Read Private Key

**Error:** `SSL: sslv3 alert handshake failure` or `139904221797184:error:06065064:digital envelope routines:EVP_DecryptFinal_ex:bad decrypt`

**Investigation:**
```bash
# Try to read the key
openssl rsa -in /path/to/key.pem -check
# If error: key is encrypted or corrupted

# If encrypted (passphrase protected):
openssl rsa -in /path/to/key.pem -passin pass:yourpassword -check

# Check if key and cert match
openssl x509 -noout -modulus -in cert.pem | openssl md5 > /tmp/cert.md5
openssl rsa -noout -modulus -in key.pem | openssl md5 > /tmp/key.md5
diff /tmp/cert.md5 /tmp/key.md5
# If different: they don't match
```

**Root Causes:**
- **Key encrypted:** Need passphrase to use
- **Wrong key:** Key doesn't match certificate
- **Corrupted file:** Can't read

**Fix:**
```bash
# If encrypted, remove passphrase (if safe):
openssl rsa -in encrypted_key.pem -out unencrypted_key.pem

# If wrong key, get the right one
# If corrupted, regenerate or recover from backup
```

---

## Real-World Scenarios

### Scenario 1: Certificate Expiring Soon - Production Alert

**Problem:** Monitoring alerts: SSL certificate expires in 7 days.

**Investigation:**
```bash
# Confirm expiry
openssl s_client -connect api.mycompany.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -dates
# Output:
# notBefore=Oct 5 12:00:00 2022 GMT
# notAfter=Oct 5 12:00:00 2024 GMT

# Days until expiry
echo "Days remaining: $((( $(date -d "2024-10-05" +%s) - $(date +%s) ) / 86400))"

# Check which service uses this certificate
# Usually in nginx/apache config:
grep -r "ssl_certificate" /etc/nginx
# Output: /etc/nginx/certs/api.mycompany.com.pem

# Check certificate file location
ls -la /etc/nginx/certs/api.mycompany.com.pem
```

**Root Cause:** Certificate not renewed before expiry.

**Fix (urgent):**
```bash
# Quick path: Renew with Let's Encrypt
sudo certbot renew --force-renewal -d api.mycompany.com

# Or manually get new certificate
sudo certbot certonly --standalone -d api.mycompany.com --preferred-challenges http

# Copy to correct location
sudo cp /etc/letsencrypt/live/api.mycompany.com/fullchain.pem /etc/nginx/certs/
sudo cp /etc/letsencrypt/live/api.mycompany.com/privkey.pem /etc/nginx/certs/

# Test new cert
openssl x509 -in /etc/nginx/certs/fullchain.pem -noout -dates

# Reload web server (don't restart, seamless)
sudo systemctl reload nginx

# Verify from client side
curl https://api.mycompany.com  # Should work without warnings
```

---

### Scenario 2: Wrong Certificate on Server - Clients Can't Connect

**Problem:** Deployed new version of service, clients report certificate errors.

**Investigation:**
```bash
# What cert is server showing?
openssl s_client -connect api.mycompany.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -subject -issuer
# Output:
# subject=CN=old-service.example.com
# issuer=C=US, O=Let's Encrypt, CN=R3

# What cert should it be?
# (Check service documentation or deployment config)
# Expected: subject=CN=api.mycompany.com

# Check server configuration
# For nginx:
sudo grep -A10 "server_name" /etc/nginx/nginx.conf
sudo grep -B5 -A5 "ssl_certificate" /etc/nginx/nginx.conf

# Wrong cert might be in:
ls -la /etc/nginx/certs/
ls -la /etc/ssl/certs/

# Check which cert is actually loaded
openssl s_client -connect api.mycompany.com:443 -showcerts < /dev/null 2>/dev/null | head -20
```

**Root Cause:** Deployment uploaded wrong certificate, or symlink broken.

**Fix:**
```bash
# 1. Get the right certificate
# Copy from cert management system or generate new one
certbot certonly --standalone -d api.mycompany.com

# 2. Update server configuration to point to right cert
sudo nano /etc/nginx/nginx.conf
# Change:
#   ssl_certificate /etc/nginx/certs/api.mycompany.com.pem
#   ssl_certificate_key /etc/nginx/certs/api.mycompany.com.key

# 3. Verify cert and key match
openssl x509 -noout -modulus -in /etc/nginx/certs/api.mycompany.com.pem | openssl md5
openssl rsa -noout -modulus -in /etc/nginx/certs/api.mycompany.com.key | openssl md5
# Hashes should match

# 4. Test configuration
sudo nginx -t
# Output: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

# 5. Reload
sudo systemctl reload nginx

# 6. Verify from client
curl -v https://api.mycompany.com 2>&1 | grep -i "subject\|issuer"
```

---

### Scenario 3: Mixed Content Warning - HTTP + HTTPS

**Problem:** Clients report "Mixed Content" warning or "not fully secure" indicator.

**Investigation:**
```bash
# Check HTML source for mixed content
curl -s https://example.com | grep -i "http://" | head -10
# May show: <img src="http://example.com/image.png">
# (http instead of https)

# Check all resource types
curl -s https://example.com | grep -oP 'href=|src=' | head -20

# Use browser developer tools
# Chrome DevTools → Console → look for mixed content warnings
# Or: curl -v with grep
```

**Root Cause:** HTML page loaded over HTTPS, but resources loaded over HTTP.

**Fix:**
```bash
# Update HTML/CSS to use HTTPS for all resources
sed -i 's|http://|https://|g' /var/www/html/index.html

# Or use protocol-relative URLs
# Change: <img src="http://example.com/img.png">
# To: <img src="//example.com/img.png">
# Browser will use same protocol as page

# For dynamic content, fix in backend
# If using templates, ensure all links use https://
```

---

## SRE Troubleshooting Scenarios

### Scenario 1: Certificate Chain Incomplete - Browsers Complain, curl Doesn't

**Problem:** Firefox/Safari show cert error, but `curl -k` works and `openssl s_client` succeeds.

**Cause:** Server is missing intermediate certificate in chain.

**Investigation:**
```bash
# Check what browser receives
openssl s_client -connect example.com:443 -showcerts < /dev/null 2>/dev/null
# Might show only 2 certs (missing intermediate)

# Some clients (curl) trust cached intermediates
# But browsers don't have them cached

# Check server configuration
# For nginx:
cat /etc/nginx/nginx.conf | grep -A5 ssl_certificate

# May show:
# ssl_certificate /path/to/cert.pem;
# But cert.pem doesn't include intermediate

# Should be:
# ssl_certificate /path/to/fullchain.pem;
# (which includes: cert + intermediate)
```

**Fix:**
```bash
# 1. Get the full chain
# Let's Encrypt provides:
# - cert.pem = end entity only
# - fullchain.pem = end entity + intermediate

# 2. Use fullchain instead of cert
sudo nano /etc/nginx/nginx.conf
# Change:
#   ssl_certificate /etc/letsencrypt/live/example.com/cert.pem;
# To:
#   ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;

# 3. Reload
sudo nginx -t && sudo systemctl reload nginx

# 4. Verify chain
openssl s_client -connect example.com:443 -showcerts < /dev/null 2>/dev/null | grep "BEGIN CERTIFICATE" | wc -l
# Should show 3 certificates
```

---

### Scenario 2: Interview Question - "Certificate Shows as Expired But Should Be Valid"

**Investigation Steps (explain in interview):**

```bash
# "I'd check several things:

# 1. Check cert expiry
openssl x509 -in /path/to/cert.pem -noout -dates
# notBefore=... notAfter=...

# 2. Check system time (might be wrong)
date
# If system time is before 'notBefore', cert appears invalid
# If system time is after 'notAfter', cert is actually expired

# 3. Check timezone
timedatectl status  # Linux
ntpstat             # Is NTP synced?

# 4. For live cert on server
openssl s_client -connect example.com:443 < /dev/null 2>/dev/null | openssl x509 -noout -dates

# 5. Check if there's a newer cert already installed
ls -la /etc/ssl/certs/ | grep example
# Might have multiple versions

# 6. Check certificate serial number (shouldn't change)
openssl x509 -in cert.pem -noout -serial

# Action: If system time wrong, fix it. If truly expired, renew.
"
```

---

## Interview Tips

1. **Know certificate structure:** Subject, Issuer, Validity dates, SAN
2. **Know certificate chain:** End entity → Intermediate → Root
3. **Know `openssl` commands:** `s_client`, `x509`, check dates and subject
4. **Know common errors:** Expired, hostname mismatch, self-signed, wrong key
5. **Know troubleshooting flow:** Check expiry → Check subject → Check chain → Check cert/key match
6. **Know certificate renewal:** Let's Encrypt `certbot`, manual CA renewal
7. **Know how to verify:** `curl` with detailed output, `openssl` commands, browser DevTools

---

## Common Pitfalls

1. **Not checking full certificate chain:** Missing intermediate causes browser errors
2. **Forgetting to reload service:** Certificate changes don't apply until reload
3. **Using cert.pem instead of fullchain.pem:** Works for some clients, not others
4. **Not checking system time:** Wrong system clock makes valid cert appear invalid
5. **Confusing certificate and key:** They're different files with different purposes
6. **Not renewing before expiry:** Wait until last minute → service goes down
7. **Self-signed in production:** Should only use for testing, not production
