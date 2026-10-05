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

