# System Troubleshooting Workflow - SRE Interview Guide

**Time to master:** 45 minutes  
**Difficulty:** ⭐⭐⭐⭐ (Hard - but essential)  
**Interview frequency:** CRITICAL (appears in ~95% of SRE interviews)

---

## ⚡ Quick Summary

Troubleshooting is THE core SRE skill. Interviewers want to see:
- **Systematic approach** (not random clicking)
- **Hypothesis-driven** (what's the problem likely to be?)
- **Data-driven** (check logs, metrics, not guesses)
- **Root cause focus** (why, not just what)
- **Communication** (explain what you're checking)

This guide teaches the **SRE troubleshooting framework**.

---

## 🎯 Key Concepts to Know

### 1. **The SRE Troubleshooting Framework**

```
INCIDENT REPORTED
        ↓
    GATHER INFO
    ├─ What's the symptom?
    ├─ When did it start?
    ├─ What changed recently?
    └─ How many users affected?
        ↓
    FORM HYPOTHESIS
    ├─ Most likely cause (80/20 rule)
    ├─ Quick to verify
    └─ Previous similar issues?
        ↓
    TEST HYPOTHESIS
    ├─ Check logs
    ├─ Check metrics
    ├─ Check status
    └─ Narrow down
        ↓
    CONFIRM ROOT CAUSE
    ├─ Found it!
    └─ Verify with multiple data points
        ↓
    IMPLEMENT FIX
    ├─ Quick workaround (immediate)
    └─ Permanent fix (prevent recurrence)
        ↓
    POST-MORTEM
    ├─ Why did this happen?
    ├─ How do we prevent it?
    └─ Update documentation
```

### 2. **Common SRE Scenarios (80/20 Rule)**

These **4 scenarios** cover ~80% of production issues:

| Scenario | Symptom | First Check |
|----------|---------|------------|
| **Service Down** | App not responding | `systemctl status` |
| **High CPU** | System slow, 100% CPU | `top` |
| **High Memory** | OOM errors, swapping | `free -h`, `top` |
| **High I/O/Disk Full** | Disk full, I/O wait high | `df -h`, `df -i` |

### 3. **The Golden Rules**

✅ **Rule 1:** Check **recent changes** first  
(New deployment? Config change? Updated dependency?)

✅ **Rule 2:** Look at **timestamps**  
(Does error time match incident time? Correlation != causation but it helps)

✅ **Rule 3:** Use **multiple data sources**  
(Logs + metrics + status checks, not just one)

✅ **Rule 4:** Think **layers**  
(Application? OS? Network? Database?)

✅ **Rule 5:** **Isolate the problem**  
(Does it reproduce? Always? Only under load?)

---

## ❓ Typical Interview Questions

### **Q1: "Our API service stopped responding 10 minutes ago. Walk me through how you'd debug it."**

**Answer (Systematic approach):**

```bash
# STEP 1: Understand the problem (30 seconds)
# Questions:
# - Is it completely down or slow?
# - All users or some?
# - What changed 10 minutes ago?

# STEP 2: Check service status (1 minute)
systemctl status myapi
# Output: 
# ● myapi.service - Loaded
# Active: failed (Result: exit-code) since 10 min ago

# AHA! Service crashed 10 minutes ago (matches incident time)

# STEP 3: Check logs for why it crashed (2 minutes)
journalctl -u myapi -n 50
# Output: "FATAL: Connection refused to database"

# HYPOTHESIS: Database is down or unreachable

# STEP 4: Test hypothesis (1 minute)
systemctl status postgresql
# Active: running ✓

# Hmm, database is running. Maybe network issue?
psql -h db.example.com -U appuser -d mydb -c "SELECT 1"
# Connection refused

# CONFIRMED: Can't reach database

# STEP 5: Check network (1 minute)
ping db.example.com              # Is host reachable?
netstat -tuln | grep 5432        # Is port 5432 open locally?
ss -tuln | grep 5432             # Alternative to netstat

# Check firewall (might have changed 10 min ago!)
sudo iptables -L -n              # Check rules

# STEP 6: Implement fix (2 minutes)
# Quick fix: Restart database connection pool
systemctl restart myapi

# Check if it works
curl http://localhost:8080/health
# {"status": "ok"} ✓

# STEP 7: Root cause analysis (ongoing)
# Check git log for recent changes
git log --oneline -10 | head -5

# Check if database had issues
journalctl -u postgresql -n 20

# Check metrics for that time period
# (In real job: check Prometheus, Grafana, etc.)
```

**Key points in answer:**
- ✅ Asked clarifying questions first
- ✅ Checked service status (most likely culprit)
- ✅ Found correlation (crash time = incident time)
- ✅ Checked logs (actual error message)
- ✅ Tested hypothesis systematically
- ✅ Implemented quick fix
- ✅ Mentioned post-mortem

---

### **Q2: "System is running slow. CPU is at 80%. How do you find the culprit?"**

**Answer (Isolate the problem):**

```bash
# STEP 1: Confirm high CPU (10 seconds)
top
# Shows: CPU 80%, one process using 75%

# STEP 2: Find the process (10 seconds)
top -b -n 1 | head -20          # One-shot top output
# PID 2847 python myapp.py  75% CPU

# STEP 3: Check what it's doing (30 seconds)
ps aux | grep 2847
ps -o pid,ppid,cmd -p 2847

# Check process state
cat /proc/2847/status | grep State
# State: S (Sleeping) — not actually using CPU now!
# Might be in syscall, not user code

# STEP 4: Drill into process (1 minute)
# Option A: strace (see what it's doing)
sudo strace -p 2847 -c          # Count syscalls

# Option B: perf (detailed performance)
sudo perf top -p 2847            # Real-time function calls

# Option C: Check if it's I/O bound
iotop -p 2847                    # Disk I/O by process

# STEP 5: Check application logs (2 minutes)
tail -f /var/log/myapp.log
# Look for: loop iterations, queries, external calls

# STEP 6: Check context (1 minute)
# Recent changes?
git log --oneline -5

# Restart and monitor
systemctl restart myapp
watch -n 1 'ps aux | grep myapp'

# STEP 7: If it persists
# Option A: Code review recent commits
# Option B: Check for memory leak (growing memory?)
# Option C: Check database queries (slow query log)
```

**Key points:**
- ✅ Used `top` to identify process
- ✅ Drilled down with `ps`, `strace`, `iotop`
- ✅ Checked logs AND code
- ✅ Looked for recent changes
- ✅ Monitored after restart

---

### **Q3: "Service started failing 2 hours ago. Users report 'Connection timeout'. Walk me through it."**

**Answer (Multi-layer debugging):**

```bash
# STEP 1: Timeline (what changed 2 hours ago?)
# Check deployment log
cat /var/log/deployment.log | tail -20
# Output: "2024-10-15 10:30 - Deployed version 2.5.0"
# That's roughly 2 hours ago!

# STEP 2: Check service logs
journalctl -u myservice --since "2 hours ago"
# Output: Multiple "Connection timeout" errors

# STEP 3: Hypothesis: New code has a bug
# But before blaming code, check infrastructure

# STEP 4: Check network (1 minute)
netstat -an | grep ESTABLISHED   # How many connections?
netstat -an | grep TIME_WAIT     # Stuck connections?

# Check if service is listening
netstat -tuln | grep 8080
# tcp LISTEN 0 128 *.8080

# STEP 5: Check resource limits (1 minute)
free -h                          # Memory ok?
df -h                            # Disk ok?
ulimit -a                        # File descriptor limit?

# Common issue: Too many open files (ulimit -n)
lsof | wc -l                     # How many open files?
cat /proc/sys/fs/file-max        # Kernel limit

# STEP 6: Check if database/cache is the issue
# (Might timeout on external service call)
curl -v http://redis:6379        # Timeout?
psql -h db -U user -d db -c "SELECT 1"

# STEP 7: Rollback decision
# If code changed 2 hours ago:
git revert <commit-hash>
systemctl restart myservice

# Monitor
watch -n 1 'systemctl status myservice'

# STEP 8: Debug the new code (after incident)
# Why does 2.5.0 timeout?
# Check: connection pool size, timeout settings, new external call?
```

**Key points:**
- ✅ Correlated incident time with deployment
- ✅ Checked multiple layers (network, resources, database)
- ✅ Made rollback decision quickly
- ✅ Planned root cause analysis for after
- ✅ Used "trust the timestamps" principle

---

### **Q4: "You're on-call, pager goes off with 'Disk 95% full'. What's your systematic approach?"**

**Answer (Follow the framework):**

```bash
# STEP 1: Assess severity (30 seconds)
# Questions:
# - Is service actually down or just warning?
# - Can we still write logs?
# - What's the business impact?

# STEP 2: Quick disk check (30 seconds)
df -h
# /dev/sda1 100G 95G 5G 95%  ← Only 5GB free!

# Check inodes too!
df -i
# Plenty of inodes (not the issue)

# STEP 3: Find the culprit (2 minutes)
du -sh /* | sort -rh
# 50G /var
# 30G /home

du -sh /var/* | sort -rh
# 45G /var/log

du -sh /var/log/* | sort -rh
# 40G /var/log/app.log

# FOUND IT: Old log file

# STEP 4: Immediate action (1 minute)
# Option A: Rotate the log NOW
logrotate -f /etc/logrotate.d/app

# Option B: If rotation fails, manually compress
gzip /var/log/app.log.1 /var/log/app.log.2

# Verify space recovered
df -h
# /dev/sda1 100G 60G 40G 60%  ✓ Breathing room!

# STEP 5: Permanent fix (5 minutes)
# Fix logrotate config
cat > /etc/logrotate.d/app <<EOF
/var/log/app/*.log {
  daily
  rotate 7
  compress
  delaycompress
  missingok
  notifempty
}
EOF

# Test it
logrotate -d /etc/logrotate.d/app

# STEP 6: Monitoring
# Alert when disk reaches 80%, not 95%
# Add alerting rules:
# "Disk usage > 80% for 1 hour → alert"
```

**Key points:**
- ✅ Assessed urgency first
- ✅ Found root cause quickly (du -sh)
- ✅ Implemented quick fix
- ✅ Prevented recurrence
- ✅ Improved monitoring

---

## 🚨 Common Mistakes

❌ **"I'll just restart the service"**  
✅ Restarting hides the problem! Debug first, restart last.

❌ **"The logs don't show anything, so it's not that"**  
✅ Check multiple log sources: journalctl, app logs, syslog

❌ **"The error started 2 hours ago, but I'll check what changed today"**  
✅ Check what changed at EXACTLY that time (git log, deployment log)

❌ **"I'll delete files to free up space"**  
✅ Find out WHY it's full first (is it a leak? wrong config?)

❌ **"It's the database's fault"**  
✅ Verify first! Check connectivity, logs, metrics.

---

## 🛠️ Essential Troubleshooting Commands (Quick Reference)

```bash
# Quick Status Check
systemctl status servicename     # Service state + recent logs
journalctl -u servicename -n 20  # Last 20 logs

# Resource Check
top                              # CPU & memory
free -h                          # Memory summary
df -h                            # Disk space
df -i                            # Inode usage

# Network Check
netstat -tuln                    # Listening ports
ss -tuln                         # Modern alternative
netstat -an | grep ESTABLISHED   # Open connections
curl http://localhost:8080       # Service connectivity

# Process Debugging
ps aux | grep processname         # Find process
lsof -p PID                       # Open files by PID
strace -p PID                     # System calls
iotop -p PID                      # Disk I/O by process

# Log Search
grep ERROR /var/log/app.log       # Find errors
journalctl --since "1 hour ago"   # Time range
tail -f /var/log/app.log          # Follow in real-time

# Disk Debugging
du -sh /*                         # Directory sizes
du -sh /path/* | sort -rh         # Sorted by size
find / -size +1G                  # Large files
```

---

## 🎬 Real-World Multi-Scenario

**Situation:** 3am on-call alert: "API timeout errors spiking"

```bash
# 0:00 - Alert fires
# Message: "API response time > 5 seconds (p99)"

# 0:05 - Start investigation
# STEP 1: Check service status
systemctl status api-server
# Active: running (but unhealthy?)

# STEP 2: Check error logs (last 5 min)
journalctl -u api-server --since "5 minutes ago" | grep -i error
# Output: "Connection pool exhausted"

# HYPOTHESIS: Too many connections, can't handle load

# STEP 3: Check connection metrics
netstat -an | grep ESTABLISHED | wc -l
# 4500 connections (normal is 500-1000)

# STEP 4: Check what's connecting
netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn
# Shows: 2000 connections from 10.0.1.5 (internal service)

# STEP 5: Check that service
ssh 10.0.1.5
systemctl status other-service
# Status: It's hammering the API with requests!

# STEP 6: Check why OTHER service is hammering us
journalctl -u other-service | tail -30
# Output: "Retry loop: 1000 retries/sec"

# AHA! Other service broke and is retry-spamming us

# STEP 7: Quick fix (circuit breaker)
# Temporarily limit connections from that IP
iptables -A INPUT -s 10.0.1.5 -j REJECT
# Now other-service fails fast, stops retry loop

# STEP 8: Monitor our service
watch -n 1 'netstat -an | grep ESTABLISHED | wc -l'
# Connections drop from 4500 → 500 ✓

# STEP 9: Fix the other service (after incident)
# Why was it in retry loop?
ssh 10.0.1.5
git log --oneline -5
# Check what changed
```

**Timeline:**
- 0:00 - Alert
- 0:10 - Root cause found (other service retry loop)
- 0:12 - Quick fix applied (circuit breaker)
- 0:15 - Service recovered
- Later - Post-mortem on why other service failed

---

## 📊 Interview Tips

✅ **Think aloud:** "I suspect it's the database because..."  
✅ **Show methodology:** "I'd check status, then logs, then metrics..."  
✅ **Ask clarifying questions:** "How many users? Started suddenly?"  
✅ **Prioritize:** "Most likely cause first: application or infrastructure?"  
✅ **Explain logic:** "I'm checking X because if Y is true, then Z..."  
✅ **Adapt:** "If that's not it, I'd check..."  

---

## 🔗 Related Files

- [01-process-management-interview.md](./01-process-management-interview.md)
- [02-filesystem-permissions-interview.md](./02-filesystem-permissions-interview.md)
- [04-storage-disk-interview.md](./04-storage-disk-interview.md)
- [05-logs-syslog-interview.md](./05-logs-syslog-interview.md)
- [sre-interview-guide.md](./sre-interview-guide.md)

---

**Next:** Read [07-performance-analysis-interview.md](./07-performance-analysis-interview.md)
