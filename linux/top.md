# top - Table of Processes

`top` displays a dynamic, real-time view of running processes and system statistics. It's the go-to command for monitoring overall system health.

## Quick Reference

```bash
# macOS syntax
top                    # Start interactive mode
top -l 1              # Show once and exit
top -pid PID            # Monitor specific process
top -o %CPU           # Sort by CPU
top -o %MEM           # Sort by memory

# Linux syntax
top -b -n 1           # Batch mode, one iteration
top -pid PID -b -n 1    # Monitor specific process (non-interactive)
top -u username       # Show processes for user
top -o %CPU -b -n 1   # Show sorted by CPU (batch mode)
```

---

## Understanding the Output

### Header Information

```
Processes: 863 total, 2 running, 861 sleeping, 4335 threads 
Load Avg: 1.35, 1.75, 2.06 
CPU usage: 3.93% user, 10.49% sys, 85.56% idle 
MemRegions: 473937 total, 11G resident, 892M private, 4733M shared.
PhysMem: 33G used (2473M wired, 1551M compressor), 2034M unused.
```

| Metric | Meaning |
|--------|---------|
| **Processes** | Total processes, running count, sleeping count, threads |
| **Load Avg** | Average system load over 1, 5, 15 minutes (rule: load should be < # of CPU cores) |
| **CPU usage** | User %, system %, idle % |
| **PhysMem** | Physical memory used, wired, available |

### Process List Columns

```
PID    COMMAND          %CPU TIME     #TH    MEM   
20455  head             0.0  00:00.00 1      896K  
20454  top              0.0  00:00.35 1      4864K 
20451  zsh              0.0  00:00.00 1      1712K 
```

| Column | Meaning |
|--------|---------|
| **PID** | Process ID |
| **COMMAND** | Process name |
| **%CPU** | CPU usage percentage |
| **TIME** | CPU time accumulated |
| **#TH** | Number of threads |
| **MEM** | Memory used |
| **STATE** | running, sleeping, zombie, stopped |

---

## Real-World Scenarios

### 1. Identify Resource Hogs

In interactive mode, press `shift+p` (sort by CPU) or `shift+m` (sort by memory) to see what's consuming resources.

```bash
top
# Then type: shift+p (CPU) or shift+m (memory)
```

### 2. Monitor Specific Process

```bash
top -pid 12345
```

Useful for watching a single application's resource consumption over time.

### 3. Check System Load

```bash
top -l 1 | head -3
```

Look at "Load Avg" — if it's consistently > number of CPU cores, the system is overloaded.

### 4. Find Memory Leaks

Watch a long-running process:
```bash
top -pid PID -l 100 -s 1
```

If memory keeps growing, it's likely a leak. Note the resident memory (RSS in ps output) over time.

### 5. Understand CPU Spikes

- **High user %** — Applications consuming CPU
- **High sys %** — Kernel overhead (I/O, context switching, etc.)
- **Low idle %** — System is busy

If you see system % spike suddenly, check `iostat` or `vmstat` for disk/memory issues.

---

## Common Interview Questions

**Q: How do you identify if a process is the cause of high CPU?**
A: Run `top`, sort by %CPU, and watch for processes consistently using >50% CPU.

**Q: What does "Load Avg: 2.5" mean on a 4-core system?**
A: System is under-utilized. Load should be watched against core count. If 2.5 on 4-core, it's healthy. If 2.5 on 2-core, it's overloaded.

**Q: How would you monitor if a service is leaking memory?**
A: Run `top -pid PID` and watch the MEM column over 10+ minutes. If it keeps increasing, there's a leak.

---

## Tips & Tricks

- **Sort by different metrics (interactive mode):**
  - `shift+p` — CPU
  - `shift+m` — Memory
  - `shift+t` — Time
  - `shift+n` — PID (newest first)

- **Filter in interactive mode:**
  - `o` — Open filter dialog
  - Example: filter by user with `user=username`

- **Batch mode for scripting (Linux only):**
  ```bash
  top -b -n 1 | tail -n +8
  ```
  
  Note: macOS doesn't support batch mode. Use `top -l 1` instead.

- **Compare with `ps aux`:**
  - `ps aux` — Static snapshot
  - `top` — Real-time updates (better for monitoring)

---

## Comparison with Alternatives

| Tool | Real-Time | Sorted | Interactive | Use Case |
|------|-----------|--------|-------------|----------|
| **top** | Yes | Yes | Yes | General monitoring |
| **ps aux** | No | No | No | Quick snapshot |
| **htop** | Yes | Yes | Yes | Enhanced top (if available) |
| **Activity Monitor** (macOS) | Yes | Yes | Yes | GUI alternative |

---

## SRE Troubleshooting Scenarios

### Scenario 1: System Load Spike at 3 AM

**Problem:** Monitoring alert: Load average jumped from 1.5 to 8.2 on a 4-core system. Applications timing out.

**Investigation Steps:**
```bash
# 1. Immediate check - what's load right now?
top -l 1 | head -5

# 2. Check if it's sustained or spiking
top -l 5 -s 2  # 5 iterations, 2 second intervals

# 3. Identify top CPU consumers
top -b -n 1 -o %CPU | head -20

# 4. Check if it's one process or many
top -b -n 1 | awk 'NR>12 {print $1, $11}' | sort -k2 -rn | head -10

# 5. Look at system CPU split
top -l 1 | grep "CPU usage"
```

**Analysis:**
- Load > CPU cores = system oversubscribed
- High "sys %" = kernel overhead, I/O wait (disk thrashing)
- High "user %" = applications consuming CPU
- Check which specific process is the culprit

**If high `sys %`:** Check disk I/O with `iostat` or `vmstat`
**If high `user %`:** Restart the offending application or check for runaway job

---

### Scenario 2: Memory Creeping Up - Is It Normal?

**Problem:** `PhysMem` in top shows 28GB used on a 32GB system. Need to know if this is a leak or just caching.

**Investigation Steps:**
```bash
# 1. Check memory breakdown
top -l 1 | grep PhysMem

# 2. Important: Wired memory vs Compressor vs Unused
# On macOS: wired = locked in RAM, compressor = compressed, unused = free
# On Linux: look for "used" and "available"

# 3. Watch a specific process over time
top -pid 12345 -l 100 -s 5  # 100 samples, 5 sec intervals

# 4. Check if memory usage is stable or growing
for i in {1..10}; do top -l 1 | grep PhysMem; sleep 10; done
```

**What This Means:**
- **Wired memory** increasing = real leak (locked, can't free)
- **Compressor** growing = normal (macOS managing pressure)
- **Used increasing, Available decreasing** = real memory pressure
- **Unused = 0** = system under memory pressure

**Action:**
- If wired/real memory growing: find and restart the leaking process
- If just compressor growing: system is okay, managing cache

---

### Scenario 3: Database Server Slow - Why?

**Problem:** Queries running 10x slower than usual. Database team asking what changed on the host.

**Investigation Steps:**
```bash
# 1. Check overall system health
top -l 1 | head -8

# 2. Check if database process is using CPU
top -b -n 1 -o %CPU | grep -i postgres  # or mysql, mongodb, etc

# 3. Check memory usage of database
top -b -n 1 -o %MEM | grep -i postgres

# 4. Monitor database process in real-time
top -pid DB_PID -l 50 -s 1  # Watch for 50 seconds

# 5. Check if other processes are starving the DB
top -b -n 1 | head -20
```

**Analysis:**
- DB using low CPU but query slow = I/O bound (check disk with iostat)
- DB using high CPU but still slow = check if it's context switching (load > cores)
- Other process using high CPU = resource contention
- DB using lots of memory = possibly doing heavy table scan (check queries)

**Next Steps:** 
- If I/O bound: check `iostat -x` for disk utilization
- If CPU bound: check if it's lock contention in application
- If memory: check for swap usage with `vmstat`

---

### Scenario 4: Container Crashing - Memory or CPU?

**Problem:** Docker/Kubernetes reports container OOMKilled. Was it CPU throttling or actual OOM?

**Investigation Steps:**
```bash
# Before it dies, if you catch it:
# 1. Check container process memory
top -b -n 1 -o %MEM | head -15

# 2. Check if it's hitting memory limit
top -pid PID -l 100 -s 1  # Watch memory trend

# 3. Check if it's hitting CPU limits
top -pid PID -l 50 | grep CPU

# After container crashes, in logs:
# Check journalctl or Docker logs for OOMKill signal
journalctl | grep -i oom
docker logs container_name | grep -i oom
```

**What Happened:**
- **OOMKilled** = hit memory limit, kernel terminated
- **CPU throttling** = hit CPU limit but still running (slowed down)
- **Restart loop** = repeatedly hitting limits

**Fix:**
- OOM: increase container memory limit or optimize app
- CPU throttling: increase CPU limit or optimize hot path

---

### Scenario 5: Load High But CPU Low - What's Happening?

**Problem:** Load average 6.0 on 4-core, but top shows CPU mostly idle. System feels slow but doesn't add up.

**Investigation Steps:**
```bash
# 1. Check CPU usage carefully
top -l 1 | grep "CPU usage"

# 2. Check process states (key insight)
top -l 1 | head -3

# 3. Look for processes in uninterruptible sleep (D state)
top -b -n 1 | grep -E '\bD\b'

# 4. Check disk I/O - this is the smoking gun
# (need iostat or vmstat, not top, but check for clues)
top -l 1 | grep -i disk

# 5. Check if waiting on network I/O
lsof -i
```

**What's Really Happening:**
- **High load, low CPU, idle** = I/O wait (processes blocked on disk/network)
- Look for processes in state `D` (uninterruptible sleep) = they're blocked
- CPU showing low because processes aren't running, they're waiting

**The Real Issue:** Disk I/O is the bottleneck
- Check `iostat -x` for disk utilization
- Check network with `netstat` or `ss`
- Likely: heavy disk reads/writes or network timeout waiting

---

### Scenario 6: Runaway Process Takes System Down

**Problem:** A process memory grows from 1GB to 8GB in minutes, system becomes unresponsive.

**Detection Steps (before it fully dies):**
```bash
# 1. Quick assessment of top hogs
top -b -n 1 -o %MEM | head -10

# 2. Identify the culprit PID
CULPRIT_PID=$(top -b -n 1 -o %MEM | sed -n '8p' | awk '{print $1}')

# 3. Get its process info
ps aux | grep $CULPRIT_PID | grep -v grep

# 4. Kill it before it crashes everything
kill -TERM $CULPRIT_PID  # Graceful
kill -9 $CULPRIT_PID      # Forced
```

**After The Fact:**
```bash
# 1. Check if process is still running
top -l 1 | grep process_name

# 2. Get historical memory usage (if logs available)
journalctl -u service_name | grep -i memory

# 3. Restart the service
systemctl restart service_name

# 4. Set memory limits to prevent recurrence
# Edit service file to add MemoryLimit
systemctl edit service_name
```

**Prevention:**
Add to service file:
```
MemoryLimit=2G
MemoryMax=3G
```

---

### Scenario 7: Interview Question: "How to Detect Memory Leak?"

**Answer (practical):**
```bash
# Method 1: Watch RSS grow over time while app is idle
while true; do
  top -b -n 1 | grep -i app_name | awk '{print strftime("%H:%M:%S"), $6, "MB"}'
  sleep 60
done

# If RSS (column 6) continuously grows while app is idle, it's a leak

# Method 2: Batch monitoring for 1 hour
for i in {1..60}; do
  top -b -n 1 | grep -i app_name | awk '{print $6}'
  sleep 60
done | awk '{print "Sample", NR, "RSS:", $1}' | tail -5

# If last sample >> first sample with no activity, it's a leak
```

**Interview Tip:** Explain your thinking:
- "I'd capture baseline memory"
- "Let it run idle for time (no user traffic)"
- "If memory keeps growing = leak"
- "If stable = normal caching"

