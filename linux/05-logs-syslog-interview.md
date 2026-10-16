# Logs & Syslog - SRE Interview Guide

**Time to master:** 60 minutes  
**Difficulty:** ⭐⭐⭐ (Medium)  
**Interview frequency:** Very High (appears in ~85% of SRE interviews)

---

## ⚡ Quick Summary

Logs are your best friend in debugging. SREs spend half their time reading logs. You need to:
- **Know log locations** (/var/log, journalctl)
- **Monitor logs in real-time** (tail -f, journalctl -f)
- **Search for errors** (grep, specific patterns)
- **Understand log rotation** (why old logs are .1, .2, etc.)
- **Manage log disk usage** (logrotate, compression)

---

## 🎯 Key Concepts to Know

### 1. **Log Locations (Syslog Legacy)**

```
/var/log/
├── syslog              # System events (Debian/Ubuntu)
├── messages            # System events (RHEL/CentOS)
├── auth.log            # Authentication attempts
├── secure              # Authentication (RHEL)
├── kern.log            # Kernel messages
├── cron                # Cron job output
├── maillog             # Mail server logs
├── httpd/
│   └── access_log      # Web server requests
│   └── error_log       # Web server errors
├── nginx/
│   └── access.log
│   └── error.log
├── app/
│   └── app.log         # Application logs
└── dmesg               # Kernel ring buffer
```

### 2. **Log Format (Common Pattern)**

```
Oct 15 10:30:45 hostname systemd[1]: Starting nginx...
│     │  │      │       │       │     │
│     │  │      │       │       │     └─ Message
│     │  │      │       │       └───────── Process/PID
│     │  │      │       └───────────────── Program name
│     │  │      └───────────────────────── Hostname
│     │  └───────────────────────────────── Time (default syslog format)
│     └─────────────────────────────────── Month & Day
└─────────────────────────────────────────── Syslog standard header
```

### 3. **Syslog Levels (Low to High Severity)**

| Level | Number | Meaning | Use |
|-------|--------|---------|-----|
| Debug | 7 | Verbose info | Development only |
| Info | 6 | Informational | Normal operation |
| Notice | 5 | Notable but normal | Important events |
| Warning | 4 | Warning | Potential issues |
| Error | 3 | Error | Something failed |
| Crit | 2 | Critical | System unstable |
| Alert | 1 | Alert | Immediate action |
| Emerg | 0 | Emergency | System down |

**SRE focus:** Error, Crit, Alert, Emerg = need investigation

### 4. **Syslog Facilities** (Source of message)

```
auth/authpriv    - Authentication
cron             - Cron jobs
daemon           - Daemons/services
kern             - Kernel
local0-local7    - Local use
mail             - Mail system
news             - News system
syslog           - Syslog itself
user             - User processes
uucp             - UUCP system
```

### 5. **Log Rotation (Why .1, .2, etc.)**

```
/var/log/app.log       # Current, being written to
/var/log/app.log.1     # Yesterday's logs (compressed: .1.gz)
/var/log/app.log.2.gz  # Day before
/var/log/app.log.3.gz  # Older
/var/log/app.log.4.gz  # Even older
# ...then deleted after 7-30 days
```

**Logrotate configuration:**
```
/var/log/app.log {
  daily              # Rotate daily
  rotate 7           # Keep 7 rotations
  compress           # gzip old ones
  delaycompress      # Don't compress today's
  missingok          # Don't error if missing
  notifempty         # Don't rotate if empty
  create 0640 app app # Create new log with these permissions
}
```

### 6. **Systemd Journal (Modern Logging)**

Modern systems use **journalctl** (systemd's logging):

```bash
# journalctl stores in /var/log/journal/ or /run/log/journal/
# Binary format (more efficient, searchable)
# Indexes timestamps, process names, severities
```

---

## ❓ Typical Interview Questions

### **Q1: "A service crashed. Walk me through how you'd debug it using logs."**

**Answer:**
```bash
# Step 1: Check service status
systemctl status myapp
# Shows last few log lines, state, pid

# Step 2: View logs
journalctl -u myapp -n 50       # Last 50 lines for myapp
# OR
tail -f /var/log/app/error.log  # Follow new logs

# Step 3: Look for ERROR or CRITICAL
journalctl -u myapp | grep -i error | tail -20

# Step 4: Check timestamps
# Does crash happen at specific time? Compare with system logs:
journalctl --since "10:00" --until "10:05"

# Step 5: Cross-reference with system logs
journalctl PRIORITY=err         # All errors system-wide

# Step 6: Check what happened before crash
journalctl -u myapp -n 200 | tail -100  # More context

# Step 7: Check system resources at crash time
# Was it out of memory? Check dmesg:
dmesg | tail -20
# Look for "Out of memory: Kill process"

# Step 8: Reproduce (if possible)
/opt/myapp/bin/start            # Try starting again
tail -f /var/log/app/error.log  # Watch logs during startup
```

**What to look for:**
- Exception stack traces (Python, Java)
- "Connection refused" (service not running)
- "Permission denied" (ownership/permissions issue)
- "Out of memory" (memory leak or insufficient RAM)
- "No space left on device" (disk full)
- Port already in use (another service on same port)

---

### **Q2: "Logs are growing too fast and filling up disk. What do you do?"**

**Answer:**
```bash
# Step 1: Identify the culprit
du -sh /var/log/*
# Shows which log file is largest

# Step 2: Check log size
ls -lh /var/log/app/app.log
# -rw-r--r-- 1 app app 50G Oct 15 10:30 /var/log/app/app.log

# Step 3: Check what's being logged
tail -f /var/log/app/app.log    # See what's in there
# Maybe too verbose (DEBUG level)

# Step 4: Immediate fix - rotate the log
logrotate -f /etc/logrotate.d/app  # Force rotation

# Step 5: Compress old logs
gzip /var/log/app/app.log.1

# Step 6: Long-term fix - adjust verbosity
# Edit /opt/app/config.yaml or /etc/app/app.conf
# Change: LOG_LEVEL=DEBUG → LOG_LEVEL=INFO

# Restart application
systemctl restart myapp

# Step 7: Monitor
df -h                           # Verify space recovered
du -sh /var/log/app/

# Step 8: Set up proper logrotate
cat > /etc/logrotate.d/app <<EOF
/var/log/app/*.log {
  daily
  rotate 7
  compress
  delaycompress
  missingok
  notifempty
  copytruncate              # For apps that don't handle SIGHUP
}
EOF
```

**Prevention:**
- Set log level to INFO (not DEBUG) in production
- Use log aggregation (ELK, Splunk, Datadog) instead of local files
- Implement proper logrotate rules
- Monitor disk usage regularly

---

### **Q3: "How would you monitor a log file for specific errors in real-time?"**

**Answer:**
```bash
# Simple: Follow and grep
tail -f /var/log/app.log | grep ERROR

# Better: Follow with context
tail -f /var/log/app.log | grep -C 3 ERROR  # 3 lines before/after

# With timestamp and highlighting
tail -f /var/log/app.log | grep --color=auto ERROR

# Systemd journal equivalent
journalctl -u myapp -f         # Follow new entries
journalctl -u myapp -f PRIORITY=err  # Only errors

# Multiple patterns
tail -f /var/log/app.log | grep -E "ERROR|CRITICAL|EXCEPTION"

# Exclude noise
tail -f /var/log/app.log | grep ERROR | grep -v "DEBUG"

# Real-world: Monitor for alerts
tail -f /var/log/app.log | while read line; do
  if echo "$line" | grep -q "CRITICAL"; then
    echo "ALERT!" | mail -s "App critical error" admin@site.com
  fi
done
```

**Better solutions:**
- Use `journalctl -u myapp -f` (systemd is standard now)
- Use log aggregation + alerting (Splunk, ELK)
- Use monitoring tools (Prometheus, Grafana, Datadog)

---

### **Q4: "What's the difference between syslog and journalctl? When would you use each?"**

**Answer:**

| Aspect | Syslog (Legacy) | Journalctl (Systemd) |
|--------|----------|-----------|
| Storage | Text files (/var/log/) | Binary (/var/log/journal/) |
| Accessible | `cat`, `grep`, `tail` | `journalctl` only |
| Searchable | Text search only | Indexed (faster) |
| Persistence | Only if configured | Survives reboot (by default) |
| Format | Human-readable | Structured (JSON) |
| Where from | /etc/rsyslog.conf | /etc/systemd/journald.conf |

**Syslog use cases:**
```bash
# Old systems (pre-systemd)
# Remote logging (syslog protocol to central server)
# Simple grepping: grep "ERROR" /var/log/syslog
```

**Journalctl use cases:**
```bash
# Modern Linux (systemd)
# Service-specific logs: journalctl -u nginx
# Boot logs: journalctl -b           # Current boot
# Previous boot: journalctl -b -1
# Time-based: journalctl --since "2 hours ago"
# Structured search: journalctl UNIT=nginx SERVICE_RESULT=timeout
```

**Both coexist:**
```bash
# Services log to journalctl
journalctl -u myapp

# rsyslog reads from journalctl and writes to /var/log/syslog
# So grep /var/log/syslog also works
```

---

## 🚨 Common Mistakes

❌ **"I'll just increase log verbosity to find the issue"**  
✅ That fills disk fast! Look for specific patterns, use grep

❌ **"Logs are in /var/log, I need to find all of them"**  
✅ Some apps log to stdout (docker), others to files, others to journalctl

❌ **"The log says 'error' so the service must be broken"**  
✅ Not all errors are fatal. Context matters. Check service status.

❌ **"I'll delete old logs to free space"**  
✅ Use logrotate! Manual deletion leaves running processes with orphaned FDs

❌ **"Grep for ERROR is enough"**  
✅ Some errors are in CRITICAL or WARNING level. Search multiple levels

---

## 🛠️ Essential Commands with Examples

### **View Logs**

```bash
# View current logs
cat /var/log/syslog              # Entire file (big!)
less /var/log/syslog             # Page through (better)
head -50 /var/log/syslog         # First 50 lines
tail -50 /var/log/syslog         # Last 50 lines

# Follow in real-time
tail -f /var/log/syslog          # Append mode (ctrl+C to stop)
tail -F /var/log/syslog          # Better (handles log rotation)
```

### **Search Logs**

```bash
# Search for patterns
grep "ERROR" /var/log/syslog
grep -i "error" /var/log/syslog  # Case-insensitive

# Count occurrences
grep -c "ERROR" /var/log/syslog

# Show context
grep -B 5 "ERROR" /var/log/syslog  # 5 lines before
grep -A 5 "ERROR" /var/log/syslog  # 5 lines after
grep -C 3 "ERROR" /var/log/syslog  # 3 lines both sides

# Multiple patterns
grep -E "ERROR|CRITICAL|FATAL" /var/log/syslog

# Exclude patterns
grep "ERROR" /var/log/syslog | grep -v "DEBUG"

# Time range (approximate)
tail -50000 /var/log/syslog | grep "ERROR"  # Last many lines
```

### **Journalctl (Systemd)**

```bash
# Basic viewing
journalctl                       # All logs (newest last)
journalctl -n 50                 # Last 50 lines
journalctl -f                    # Follow (like tail -f)

# Service-specific
journalctl -u nginx              # Only nginx
journalctl -u nginx -n 100       # Last 100 nginx lines
journalctl -u nginx -f           # Follow nginx logs

# Time-based
journalctl --since today         # Since midnight
journalctl --since "1 hour ago"
journalctl --since "2024-10-15" --until "2024-10-16"

# By boot
journalctl -b                    # Current boot
journalctl -b -1                 # Previous boot
journalctl --list-boots          # Available boots

# Priority/severity
journalctl -p err                # Only errors and critical
journalctl -p warning            # Warnings and above

# Output formats
journalctl -o json               # JSON (for parsing)
journalctl -o json-pretty        # Pretty JSON
journalctl -o short              # Concise format
journalctl -o verbose            # All fields
```

### **Log Rotation**

```bash
# Force immediate rotation
logrotate -f /etc/logrotate.d/app

# Test logrotate config (dry run)
logrotate -d /etc/logrotate.d/app  # -d = debug, doesn't actually rotate

# Manual rotation (if logrotate disabled)
mv /var/log/app.log /var/log/app.log.1
gzip /var/log/app.log.1
# App must reopen log or SIGHUP
systemctl reload app             # Reopen logs
```

### **Disk Usage**

```bash
# Find largest log files
du -sh /var/log/* | sort -h | tail -10

# Monitor in real-time
watch -n 1 'du -sh /var/log'

# Find logs older than X days
find /var/log -mtime +30 -type f  # Modified >30 days ago
find /var/log -mtime +30 -delete  # Delete them (careful!)
```

---

## 🎬 Real-World Scenario

**Situation:** Service keeps crashing, need to find why.

```bash
# 1. Check status
systemctl status myapp
# Shows failed, last exit code 1

# 2. View logs
journalctl -u myapp -n 30
# Shows: "Connection refused" at 10:30:45

# 3. Check what it connects to
grep -i "connection" /opt/myapp/config.yml
# Connects to port 5432 (PostgreSQL)

# 4. Check if PostgreSQL is running
systemctl status postgresql
# It's running ✓

# 5. Check logs at that exact time
journalctl --since "10:30:40" --until "10:30:50"

# 6. Check if PostgreSQL crashed
journalctl -u postgresql --since "10:30:40" --until "10:30:50"
# Shows: "FATAL: out of memory"

# 7. Fix: Increase memory or clear cache
systemctl restart postgresql
journalctl -u myapp -f
# Logs show starting up...
# Should be healthy now

# 8. Permanent fix: Set memory limit or enable swap
# If frequent: add more RAM or optimize queries
```

---

## 📊 Interview Tips

✅ **Show diagnostic approach:** "First I check status, then follow logs..."  
✅ **Use right tools:** "I'd use journalctl for service logs, tail -f for following..."  
✅ **Understand context:** "That error might be harmless, let me check the full output..."  
✅ **Think about prevention:** "Set up logrotate so disk doesn't fill up..."  

---

## 🔗 Related Files

- [journalctl.md](./journalctl.md) - Detailed journalctl reference
- [tail-grep.md](./tail-grep.md) - tail and grep commands
- [sre-interview-guide.md](./sre-interview-guide.md) - Master interview guide

---

**Congratulations!** You've completed all 5 core interview guides. Now review and practice scenarios.

**Next steps:**
1. Review [sre-interview-guide.md](./sre-interview-guide.md) daily for 1 week
2. Practice each command on a test system
3. Do mock interviews with these scenarios
4. Good luck! 🎯
