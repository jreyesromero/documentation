# ps - Process Status

`ps` displays information about active processes. It's essential for troubleshooting, monitoring resource usage, and understanding process relationships.

## Quick Reference

```bash
ps -ef              # Unix syntax: full format
ps aux              # BSD syntax: detailed with CPU/memory
ps -u username      # Show processes for specific user
ps aux | sort -k 3 -rn | head -10    # Top 10 by CPU
ps aux | sort -k 4 -rn | head -10    # Top 10 by memory
```

---

## Syntax: BSD vs Unix

macOS and Linux distributions support two syntax styles:

**Unix-style** (single dash, older):
```bash
ps -ef
```

**BSD-style** (no dash, more common):
```bash
ps aux
```

Both work on most systems. **Key difference:** the output columns vary between them. Choose the one that shows what you need.

---

## Understanding Columns

### ps -ef Output Format

```
UID   PID  PPID   C STIME   TTY           TIME CMD
  0     1     0   0 10:20AM ??         1:41.85 /sbin/launchd
  0   536     1   0 10:21AM ??         1:01.17 /usr/libexec/logd
  0   537     1   0 10:21AM ??         0:00.23 /usr/libexec/smd
```

| Column | Meaning |
|--------|---------|
| **UID** | User ID who owns the process |
| **PID** | Process ID — unique identifier for this process |
| **PPID** | Parent Process ID — the process that started this one |
| **C** | CPU usage percentage (or scheduling class on some systems) |
| **STIME** | Start time of the process |
| **TTY** | Terminal type (? = no controlling terminal, background process) |
| **TIME** | CPU time accumulated by the process |
| **CMD** | Command that started the process |

### ps aux Output Format

```
USER               PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
julianre         18281  17.1  1.2 440898464 469760 s001  S+   12:16PM   0:19.21 claude
root               833  10.6  0.3 435379488 122352   ??  Ss   10:22AM   0:38.34 /usr/libexec/...
_windowserver     1342   8.6  0.6 436536832 236880   ??  Us   10:22AM  17:23.71 /System/Library/...
```

| Column | Meaning |
|--------|---------|
| **USER** | Username of the process owner |
| **PID** | Process ID |
| **%CPU** | CPU usage as a percentage |
| **%MEM** | Physical memory usage as a percentage |
| **VSZ** | Virtual memory size in KB |
| **RSS** | Resident set size (physical memory) in KB |
| **TT** | Controlling terminal (? = background, s = session) |
| **STAT** | Process state (see below) |
| **STARTED** | When the process started |
| **TIME** | CPU time used |
| **COMMAND** | The command that started the process |

---

## Process States (STAT column)

Understanding what each state means helps diagnose issues:

| State | Meaning | Example |
|-------|---------|---------|
| **R** | Running — actively using CPU | Interactive processes |
| **S** | Sleeping — waiting for something | Most background daemons |
| **Ss** | Session leader, sleeping | Parent process of a session |
| **Ss+** | Foreground process, sleeping | Your shell |
| **Z** | Zombie — terminated but not cleaned up | Sign of a parent process issue |
| **T** | Stopped — paused (Ctrl+Z) | Suspended jobs |
| **Us** | Uninterruptible sleep (kernel I/O) | Processes waiting for disk |

The `+` suffix means foreground (controlling terminal), `<` means high priority, `s` means session leader.

---

## Common Use Cases

### 1. Find Resource Hogs

**Top 10 processes by CPU:**
```bash
ps aux | sort -k 3 -rn | head -10
```

```
USER               PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
julianre         18281  17.1  1.2 440898464 469760 s001  S+   12:16PM   0:19.21 claude
root               833  10.6  0.3 435379488 122352   ??  Ss   10:22AM   0:38.34 /usr/libexec/...
_windowserver     1342   8.6  0.6 436536832 236880   ??  Us   10:22AM  17:23.71 /System/Library/...
```

**Top 10 by memory:**
```bash
ps aux | sort -k 4 -rn | head -10
```

```
USER               PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
julianre          2773   0.1  7.1 444097680 2691136   ??  S    10:22AM   4:33.82 /Applications/IntelliJ...
julianre         16592   0.0  5.7 1949467344 2168224   ??  S    11:53AM   1:50.96 /Applications/Google...
julianre          2943   0.0  3.2 1955324992 1220112   ??  S    10:23AM   6:23.04 /Applications/Microsoft...
```

### 2. Find Processes by User

```bash
ps -u username
```

Shows all processes running as a specific user. Useful for debugging user-specific issues or seeing what a service account is doing.

### 3. Find a Specific Process

```bash
ps aux | grep process_name
```

Example:
```bash
ps aux | grep claude
```

**Note:** Use `grep -v grep` to exclude the grep process itself:
```bash
ps aux | grep claude | grep -v grep
```

### 4. See Parent-Child Relationships

```bash
ps -ef | head -20
```

Look at PID and PPID columns to understand which process spawned which. The PPID tells you who started the process.

Example from output:
```
UID   PID  PPID   C STIME   TTY           TIME CMD
  0     1     0   0 10:20AM ??         1:41.85 /sbin/launchd        <- PID 1 is init
  0   536     1   0 10:21AM ??         1:01.17 /usr/libexec/logd    <- Started by PID 1
```

### 5. Find Zombie Processes

```bash
ps aux | grep Z
```

Zombies have state `Z` and indicate a parent process isn't properly reaping child processes.

### 6. Monitor a Specific Process Over Time

```bash
while true; do clear; ps -p PID -o pid,vsz,rss,comm; sleep 2; done
```

Monitors PID continuously (Ctrl+C to stop).

---

## Advanced Filtering

**Exclude root processes:**
```bash
ps aux | awk '$1 != "root"'
```

**Show only processes using more than 10% memory:**
```bash
ps aux | awk '$4 > 10'
```

**Find all processes in a process group:**
```bash
ps -g groupid
```

---

## Real-World Scenarios

### Investigating a Slow System

1. Check top CPU consumers:
   ```bash
   ps aux | sort -k 3 -rn | head -5
   ```

2. Check top memory consumers:
   ```bash
   ps aux | sort -k 4 -rn | head -5
   ```

3. Look for zombie processes:
   ```bash
   ps aux | grep Z
   ```

4. Check if legitimate services are running:
   ```bash
   ps -ef | grep service_name
   ```

### Finding Process Family Trees

To understand if a process is spawned by another:
```bash
ps -ef | grep parent_process
```

Then look at PPID values to trace the chain.

### Monitoring Application Startup

```bash
ps aux | grep application_name
```

Check the STARTED column to see when it began. Compare VSZ and RSS to see memory growth.

---

## Tips & Tricks

- **Redirect to file for comparison:**
  ```bash
  ps aux > snapshot1.txt
  # (do something)
  ps aux > snapshot2.txt
  diff snapshot1.txt snapshot2.txt
  ```

- **Real-time alternative:** If `ps` output is too static, use `top`, `htop`, or `watch`:
  ```bash
  watch 'ps aux | sort -k 3 -rn | head -10'
  ```

- **Know your TTY:** 
  - `??` = background (daemon)
  - `s001`, `s002` = foreground sessions
  - This helps identify which processes you're responsible for

- **VSZ vs RSS:**
  - **VSZ** = virtual memory (includes swapped, memory-mapped, etc.)
  - **RSS** = actual physical RAM currently in use
  - RSS is usually the better indicator of real memory consumption

---

## Common Pitfalls

1. **Forgetting root processes:** `ps aux` shows everything including root processes. Some system services won't show under your user.

2. **Confusing CPU% with actual usage:** On multi-core systems, %CPU can exceed 100% if a process uses multiple cores. This is normal.

3. **Not using `grep -v grep`:** When searching, your grep command itself appears in results. Filter it out.

4. **Memory appears to exceed 100%:** With modern memory management, this is possible due to memory sharing, swapping, etc.

---

## SRE Troubleshooting Scenarios

### Scenario 1: Service Memory Leak Detected in Production

**Problem:** Your monitoring alerts show a service memory growing unbounded. It was at 500MB this morning, now at 2GB. You need to investigate before it crashes.

**Investigation Steps:**
```bash
# 1. Find the process
ps aux | grep service_name | grep -v grep

# 2. Get initial snapshot (note the RSS value)
ps aux | grep service_name

# 3. Check memory growth over time
while true; do
  echo "$(date) - $(ps aux | grep service_name | grep -v grep | awk '{print $6}')"
  sleep 60
done

# 4. If confirmed leak, check child processes
ps -ef | grep service_name

# 5. Get detailed process info including file descriptors
ps aux | grep service_name | grep -v grep | awk '{print "PID:", $2, "Memory:", $6, "Time:", $10}'
```

**What to Look For:**
- RSS (column 6 in `ps aux`) continuously increasing = likely memory leak
- High %MEM and process age (TIME column) correlate = sustained high memory
- Multiple child processes = check if one specific child is leaking
- VSZ >> RSS = potential memory fragmentation

**Next Steps:** Use `top -pid PID` for real-time monitoring, then check application logs with `journalctl`.

---

### Scenario 2: Orphaned/Zombie Processes Accumulating

**Problem:** Monitoring shows process count growing, but no new services started. You suspect zombie processes.

**Investigation Steps:**
```bash
# 1. Find all zombies
ps aux | grep Z

# 2. Count them
ps aux | grep ' Z ' | wc -l

# 3. Identify their parents (PPID column)
ps aux | grep ' Z ' | awk '{print "Zombie PID:", $2, "Parent PID:", $3}'

# 4. Find the parent process
ps aux | grep PPID_from_above

# 5. Check if parent is misbehaving
ps -ef | grep PARENT_PID
```

**What This Means:**
- Zombies = process exited but parent didn't call `wait()`
- Parent PID still running = parent has a bug (not reaping children)
- If parent is PID 1 (init) = child's parent died, orphaned

**Fix:** Restart the parent process or the entire service:
```bash
systemctl restart service_name
```

---

### Scenario 3: Sudden CPU Spike - Find the Culprit

**Problem:** CPU jumps from 5% to 80% unexpectedly. Users reporting slowness.

**Investigation Steps:**
```bash
# 1. Find top CPU consumers
ps aux | sort -k 3 -rn | head -5

# 2. Check if it's a legitimate service
ps aux | grep -E "nginx|postgres|app_name"

# 3. Check if multiple processes are offending
ps aux | awk '$3 > 20 {print $0}'  # Show all processes using >20% CPU

# 4. Check process creation time
ps aux | grep culprit_process | grep -v grep | awk '{print "Started:", $9, "Time Used:", $10}'

# 5. Check if it spawned children
ps -ef | grep culprit_pid
```

**What to Look For:**
- One process with >50% CPU = that's your issue
- Multiple processes with high CPU = possible DDoS or batch job
- STIME (start time) very recent = new process causing issue
- TIME very high = process has been running long accumulating CPU

**Quick Fix:** Check logs, then restart if needed:
```bash
systemctl restart service_name
```

---

### Scenario 4: Process Vanished - Was It Running?

**Problem:** Investigating an incident. Need to prove if a service was running at a specific time.

**Investigation Steps:**
```bash
# Current snapshot
ps aux | grep service_name

# Check if it's in process list at all
ps -ef | grep -i service | grep -v grep

# If not running, check logs for when it crashed
journalctl -u service_name --since "2 hours ago" | grep -E "EXIT|FAIL|ERROR"

# Check restart history
systemctl status service_name | grep -i active

# Get detailed historical view
journalctl -u service_name -n 100 | head -20
```

**What This Tells You:**
- Process not in `ps` output = either crashed or never started
- Check Exit status in logs to understand why it stopped
- Use timestamps to build incident timeline

---

### Scenario 5: Resource Limits - Process Hitting Ceiling

**Problem:** A process keeps failing with "too many open files" or similar errors.

**Investigation Steps:**
```bash
# 1. Find the process
ps aux | grep service_name | grep -v grep

# 2. Check resource usage
ps aux | grep service_name | awk '{print "VSZ:", $5, "RSS:", $6, "Threads:", $11}'

# 3. Check actual limits (need lsof or /proc)
# On Linux:
cat /proc/PID/limits

# 4. Check if hitting memory ceiling
ps aux | awk '$4 > 80 {print "High memory:", $0}'

# 5. Check parent process for inherited limits
ps -ef | grep PID
```

**What to Look For:**
- %MEM approaching 95%+ = hitting memory ceiling
- VSZ much larger than RSS = memory available but committed
- Check systemd service file for MemoryLimit settings

**Solution:** Either increase limits or optimize the application.

---

### Scenario 6: Application Won't Die - Stuck Process

**Problem:** `systemctl stop service` hangs. Process seems stuck.

**Investigation Steps:**
```bash
# 1. Check its state
ps aux | grep stuck_process

# 2. Look for state (column 8)
ps aux | grep stuck_process | awk '{print "State:", $8}'

# 3. If state is 'D' (uninterruptible sleep), it's waiting for I/O
#    If 'T' (stopped), someone paused it
#    If 'S' (sleeping), it should respond

# 4. Check what it's doing
# On Linux, check if blocked on I/O:
cat /proc/PID/status | grep State

# 5. Check if it has network connections holding it
lsof -i -p PID

# 6. Force kill if necessary
kill -9 PID
```

**What Each State Means:**
- **S** = sleeping (responsive, can be stopped)
- **D** = uninterruptible sleep (waiting for disk I/O, cannot be killed normally)
- **T** = stopped (paused, someone ran kill -STOP)
- **Z** = zombie (dead but parent hasn't reaped)

**Fix:**
- If `D` state: wait for I/O or restart system
- If `T` state: `kill -CONT PID`
- If stuck: `kill -9 PID` then `systemctl start service`

