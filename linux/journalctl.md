# journalctl - Systemd Journal

`journalctl` queries the systemd journal (centralized logging system). Shows logs from services, kernel, and system events.

## Quick Reference

```bash
journalctl                              # Show all logs
journalctl -n 50                        # Last 50 lines
journalctl -f                           # Follow (tail -f)
journalctl -u servicename               # Logs for specific service
journalctl -b                           # Since last boot
journalctl --since "1 hour ago"         # Since timestamp
journalctl -p err                       # Error level only
journalctl /usr/bin/systemd             # Logs from specific executable
```

---

## Understanding Output

```
Oct 05 10:20:00 mymachine systemd[1]: Starting nginx.service...
Oct 05 10:20:00 mymachine systemd[1]: Started nginx.service.
Oct 05 10:22:15 mymachine nginx[2848]: [error] 2854#2854: *1 connect() failed
```

| Component | Meaning |
|-----------|---------|
| **Timestamp** | When event occurred |
| **Hostname** | Machine name |
| **Process** | Service/process name and PID |
| **Message** | Log message |

---

## Critical Interview Scenarios

### 1. "Service crashed. Find the error in logs."

```bash
# Step 1: Check recent service logs
journalctl -u myservice -n 50

# Step 2: Last boot only
journalctl -u myservice -b

# Step 3: With timestamps and levels
journalctl -u myservice -o short-iso --all

# Step 4: Error level only
journalctl -u myservice -p err
```

### 2. "System had issues. Check what went wrong."

```bash
# Kernel messages
journalctl -k

# OOM killer events
journalctl | grep -i "killed process"

# Boot sequence
journalctl -b -p err
```

### 3. "Application logging is missing. Debug."

```bash
# Check if app is writing to journal
journalctl -u myapp -n 100

# Check if it even started
journalctl -u myapp --all

# Check for errors during start
journalctl -u myapp -p err
```

---

## Key Options

| Option | Meaning |
|--------|---------|
| **-n N** | Show last N lines |
| **-f** | Follow (continuous) |
| **-u SERVICE** | Specific service |
| **-b** | Since last boot |
| **--since TIME** | Since timestamp |
| **--until TIME** | Until timestamp |
| **-p LEVEL** | Priority level |
| **-k** | Kernel messages only |
| **-o FORMAT** | Output format |
| **--all** | Include no-display lines |
| **--reverse** | Newest first |

---

## Priority Levels

| Level | Meaning |
|-------|---------|
| **emerg** | System emergency |
| **alert** | Urgent action needed |
| **crit** | Critical |
| **err** | Error |
| **warn** | Warning |
| **notice** | Important info |
| **info** | General info |
| **debug** | Debug messages |

---

## Time-Based Filtering

### Relative Times

```bash
# Last hour
journalctl --since "1 hour ago"

# Last 10 minutes
journalctl --since "10 minutes ago"

# Today
journalctl --since today

# Yesterday
journalctl --since yesterday
```

### Absolute Times

```bash
# Since specific date
journalctl --since "2026-10-05"

# Between dates
journalctl --since "2026-10-05" --until "2026-10-06"

# Specific time
journalctl --since "2026-10-05 10:00:00"
```

---

## Real-World Scenarios

### Service keeps restarting

```bash
# Find restart events
journalctl -u myservice | grep -i restart

# See what's causing crash
journalctl -u myservice -p err

# Check if dependency failed
journalctl -u myservice | head -50
```

### OOM (Out of Memory) issues

```bash
# Find OOM killer events
journalctl | grep -i "killed process"

# Or
journalctl -k | grep -i oom

# Show memory pressure
journalctl | grep -i "memory"
```

### Bootup problems

```bash
# Show boot sequence
journalctl -b

# Errors during boot
journalctl -b -p err

# Specific service failure
journalctl -u failed_service -b
```

### Performance degradation

```bash
# Check for disk errors
journalctl -k | grep -i "I/O"

# Check for CPU overload
journalctl | grep -i "throttle"

# Check for network issues
journalctl | grep -i "network"
```

---

## Output Formats

### Short (default)

```
Oct 05 10:20:00 machine systemd[1]: Started service.
```

### Short-iso (ISO 8601 timestamps)

```
2026-10-05T10:20:00.123456+00:00 machine systemd[1]: Started service.
```

### Verbose

```bash
journalctl -o verbose
```

Shows all metadata including priority, unit, etc.

### JSON

```bash
journalctl -o json

# More readable
journalctl -o json-pretty
```

---

## Exporting and Analysis

### Save to file

```bash
journalctl > journal_backup.txt

# Or specific service
journalctl -u myservice > myservice.log
```

### Combine with grep

```bash
# Find specific errors
journalctl | grep "specific error message"

# Count errors
journalctl -p err | wc -l

# Real-time filtering
journalctl -f | grep ERROR
```

### Analyze patterns

```bash
# Count errors by hour
journalctl -p err | grep "Oct 05 1[0-5]" | wc -l

# Find most common error
journalctl -p err | awk -F': ' '{print $NF}' | sort | uniq -c | sort -rn | head
```

---

## Storage and Cleanup

### Check journal size

```bash
# Get journal disk usage
journalctl --disk-usage

# Vacuum to size
journalctl --vacuum-size=100M

# Vacuum by time
journalctl --vacuum-time=30d
```

### Configure retention (edit /etc/systemd/journald.conf)

```
SystemMaxUse=100G      # Max disk usage
SystemMaxFileSize=100M # Max file size
MaxRetentionSec=30day  # Keep 30 days
```

---

## Interview Tips

1. **Know `-u SERVICE`:** Most common use for troubleshooting
2. **Know `-f` for following:** Like `tail -f` but for systemd
3. **Know `-b` for boot:** Critical for debugging boot issues
4. **Know priority levels:** `-p err` to find errors
5. **Know time filters:** `--since "1 hour ago"` for recent events
6. **Know to combine with grep:** `journalctl -u service | grep ERROR`

---

## Comparison: journalctl vs traditional logs

| Aspect | journalctl | /var/log files |
|--------|-----------|-----------------|
| **Storage** | Binary journal | Text files |
| **Parsing** | Structured | Line-by-line |
| **Filtering** | Rich options | grep/awk |
| **Centralized** | Yes | Scattered |
| **Performance** | Fast queries | Slower |
| **Retention** | Configurable | logrotate |

---

## SRE Troubleshooting Scenarios

### Scenario 1: Production Alert - Service Down - Find Root Cause in 5 Minutes

**Problem:** Monitoring shows service offline 3 minutes ago. Need immediate root cause.

**Fast Investigation:**
```bash
# 1. IMMEDIATE: Check service status
systemctl status myservice

# 2. IMMEDIATE: Get last 30 lines of logs
journalctl -u myservice -n 30

# 3. Check errors only
journalctl -u myservice -p err -n 20

# 4. Check since service was last known working (5 mins ago)
journalctl -u myservice --since "5 minutes ago"

# 5. Look for the exact failure
journalctl -u myservice --since "5 minutes ago" | grep -iE "error|fail|killed|core"
```

**What You're Looking For:**
- **Exit code 1 or 127** = application error (check logs)
- **Exit code 137** = killed (likely OOMKilled, check memory)
- **"Connect to X failed"** = dependency unavailable
- **"Port 8080: Address already in use"** = port conflict
- **"Permission denied"** = file/directory permission issue

**Expected Timeline:**
- Line 1: "Starting myservice.service..."
- Line 2: "myservice[PID]: Application started" (or error)
- Last line: "Stopped myservice.service" (if crashed)

**Recovery:** Use the error message to fix and restart:
```bash
systemctl restart myservice
# Monitor restart
journalctl -u myservice -f
```

---

### Scenario 2: Incident: Service Crashes Every Hour at :15

**Problem:** Service has a pattern: crashes at consistent time. Need to diagnose.

**Analysis Pattern:**
```bash
# 1. Find crash times
journalctl -u myservice | grep "Stopped\|crashed\|failed" | tail -20

# 2. Check what else happens at those times
journalctl --since "2 hours ago" --until now | grep -E "cron|backup|rotate|vacuum"

# 3. Check if another service is stopping this one
journalctl --since "2 hours ago" | grep myservice

# 4. Look for resource spikes at crash times
journalctl -k --since "2 hours ago" | grep -i "memory\|oom"

# 5. Get full context around crash
journalctl -u myservice -b | head -100  # From boot onwards
```

**Investigation Output Example:**
```
Oct 05 10:15:03 mymachine systemd[1]: Starting myservice.service...
Oct 05 10:15:05 mymachine myservice[2847]: Application error: connection timeout
Oct 05 10:15:05 mymachine systemd[1]: myservice.service: Main process exited
Oct 05 10:15:05 mymachine systemd[1]: Stopped myservice.service
Oct 05 10:15:10 mymachine systemd[1]: Starting myservice.service...
```

**Root Causes to Check:**
- Cron job: `journalctl -u cron | grep -B5 -A5 ":15"`
- Database backup: check postgres/mysql logs
- Metrics scrape: check if monitoring system is scraping the port
- Log rotation: check `/etc/logrotate.d/`

**Fix:**
Reschedule conflicting job to different time or resource-limit it.

---

### Scenario 3: Memory Leak Suspected - Need Evidence from Logs

**Problem:** Memory monitoring shows process growing. Before we restart, verify it's a real leak from logs.

**Log Analysis:**
```bash
# 1. Check if app logs memory allocations
journalctl -u myservice | grep -i "memory\|alloc\|freed" | tail -20

# 2. Check for resource warnings
journalctl -u myservice | grep -i "memory pressure\|swap\|limit"

# 3. Check system memory pressure
journalctl -k | grep -i "memory\|swap\|oom" | tail -10

# 4. Look for app-specific memory error messages
journalctl -u myservice | grep -iE "out of memory|oom|malloc failed"

# 5. Timeline: when did memory start growing?
journalctl -u myservice --since "6 hours ago" | wc -l
journalctl -u myservice --since "3 hours ago" | wc -l
journalctl -u myservice --since "1 hour ago" | wc -l
# If last one is much larger = recent memory growth
```

**What to Look For:**
```
Oct 05 10:00:00 mymachine myservice: Memory usage: 500MB
Oct 05 11:00:00 mymachine myservice: Memory usage: 1000MB
Oct 05 12:00:00 mymachine myservice: Memory usage: 1500MB
Oct 05 12:30:00 mymachine kernel: myservice invoked oom-killer
```

**Evidence of Leak:**
- Memory grows during idle periods (no user load)
- No corresponding "freed" messages
- Continues growing until OOMKilled

**Action:** Schedule restart during maintenance window, then monitor post-restart.

---

### Scenario 4: Startup Sequence Failed - Dig Through Boot Logs

**Problem:** Server rebooted unexpectedly. Multiple services failed. Need to understand sequence of failure.

**Boot Failure Diagnostics:**
```bash
# 1. Get clean boot sequence
journalctl -b | head -100  # First boot items

# 2. Find what failed during boot
journalctl -b -p err

# 3. Timeline of critical failures
journalctl -b | grep -i "failed\|dependency\|timeout" | head -20

# 4. Check if init/systemd itself had issues
journalctl -b /usr/lib/systemd/systemd-* | grep -i "error\|fail"

# 5. Check filesystem/mount issues (often first failure)
journalctl -b | grep -i "mount\|filesystem\|fsck"

# 6. Check network bringup
journalctl -b | grep -i "network\|dhcp\|ip"

# 7. Look for deadlocks or circular dependencies
journalctl -b | grep -i "dependency\|requires\|after"
```

**Typical Boot Failure Sequence:**
```
1. Filesystem errors → mount fails → service start fails
2. Network unavailable → network-dependent service fails → cascade
3. Dependency missing → service won't start until dependency ready
4. Timeout on startup → service marked as failed
```

**Recovery Strategy:**
```bash
# Check systemd default target
systemctl get-default

# Check which services failed
systemctl --failed

# Get status of each failed service
systemctl status service_name

# Check dependencies
systemctl list-dependencies service_name

# Manual start to get error
systemctl start service_name
journalctl -u service_name -n 20
```

---

### Scenario 5: Application Not Logging - Investigate Why

**Problem:** Application is supposedly running but no logs in journalctl.

**Investigation:**
```bash
# 1. Confirm app is running
systemctl status myapp
ps aux | grep myapp

# 2. Check if journalctl even shows systemd messages for it
journalctl -u myapp -n 100 | head -20

# 3. Check if it's configured to log to journal
systemctl cat myapp | grep -i "standard\|output\|journal"

# 4. Check if there's a custom log file
systemctl cat myapp | grep -E "ExecStart|WorkingDirectory"
# If it specifies a log file, check that file instead:
tail -f /var/log/myapp.log

# 5. Check if there's stderr redirection
systemctl cat myapp | grep -i "redirect\|error"

# 6. Check if container/cgroup is hiding logs
journalctl -u myapp --all --reverse | head -20
```

**Common Causes:**
- App configured to log to file, not journal
- App logs to `/dev/null` or `/dev/stderr`
- Logs rotated and deleted
- Journal storage is too small, oldest logs purged

**Fix:**
```bash
# Option 1: Configure app to use journal (best)
systemctl edit myapp
# Add: StandardOutput=journal
# Add: StandardError=journal

# Option 2: Follow the actual log file
tail -f /var/log/myapp.log

# Option 3: Check app configuration for log path
cat /etc/myapp/config.conf | grep -i log
```

---

### Scenario 6: Network Service Down - Debug Networking Issue

**Problem:** Web service running but requests fail. Check logs for network clues.

**Network Debugging:**
```bash
# 1. Check if service is even listening
journalctl -u myapp -n 50 | grep -i "listen\|bind\|port"

# 2. Check for port already in use error
journalctl -u myapp | grep -i "address in use"

# 3. Check for DNS resolution errors
journalctl -u myapp | grep -i "dns\|resolve\|getaddrinfo"

# 4. Check for connection timeouts
journalctl -u myapp | grep -i "timeout\|timed out\|connect"

# 5. Check for TLS/SSL errors
journalctl -u myapp | grep -i "certificate\|ssl\|tls\|handshake"

# 6. Check network interface logs
journalctl -k | grep -i "network\|eth\|wlan\|lo\|bridge"

# 7. Check systemd-resolved for DNS issues
journalctl -u systemd-resolved | grep -i "fail\|error"
```

**Expected vs Actual:**
```
EXPECTED:
Oct 05 10:00:00 server myapp: Server listening on 0.0.0.0:8080

ACTUAL FAILURE EXAMPLES:
Oct 05 10:00:00 server myapp: Address already in use
Oct 05 10:00:00 server myapp: Failed to resolve hostname
Oct 05 10:00:00 server myapp: Connection refused
```

**Fix:**
```bash
# If port conflict
lsof -i :8080
kill -9 conflicting_pid

# If DNS issue
systemctl restart systemd-resolved

# If firewall blocking
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

---

### Scenario 7: Performance Degradation - Check Logs for Clues

**Problem:** System was fast 2 hours ago, now slow. Need to identify what changed from logs.

**Log-Based Performance Analysis:**
```bash
# 1. Find resource allocation changes
journalctl --since "2 hours ago" | grep -i "memory\|cpu\|throttle\|limit"

# 2. Find service restarts (might indicate cascading failures)
journalctl --since "2 hours ago" | grep -i "started\|stopped\|restarted"

# 3. Find disk/IO errors
journalctl --since "2 hours ago" | grep -i "i/o\|disk\|async"

# 4. Find network errors
journalctl --since "2 hours ago" | grep -i "network\|connection"

# 5. Find OOM pressure
journalctl --since "2 hours ago" | grep -i "oom\|memory.*pressure"

# 6. Estimate when performance degraded
journalctl --since "2 hours ago" | wc -l  # Total events
journalctl --since "1.5 hours ago" | wc -l  # Recent events
# If recent event count is high, degradation started recently
```

**Timeline Analysis:**
```bash
# Find the exact moment of degradation
journalctl -o short-iso --since "2026-10-05 10:00" | 
  awk -F'T' '{print $2}' | cut -d: -f1-2 | sort | uniq -c | sort -rn
# Spike in event count = degradation started

# Then check what changed at that moment
journalctl --since "2026-10-05 10:15" --until "2026-10-05 10:25"
```

**Common Causes Found in Logs:**
- OOMKiller events → memory pressure
- Disk errors → I/O bottleneck
- Service restart cascades → dependency failure
- Network timeout messages → connectivity degradation

