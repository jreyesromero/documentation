# Performance Analysis & Tuning - SRE Interview Guide

**Time to master:** 60 minutes  
**Difficulty:** ⭐⭐⭐⭐ (Hard)  
**Interview frequency:** Very High (appears in ~75% of SRE interviews)

---

## ⚡ Quick Summary

Performance issues are the #1 production problem. SREs need to:
- **Identify bottlenecks** (is it CPU, memory, disk I/O, or network?)
- **Measure correctly** (not all "100% CPU" is the same)
- **Understand queuing** (why is system slow with low CPU?)
- **Know the tools** (top, iostat, vmstat, netstat, perf)
- **Think systematically** (layers: application → OS → hardware)

---

## 🎯 Key Concepts to Know

### 1. **The Four System Bottlenecks**

```
                    SLOW APPLICATION
                           ↓
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
      CPU          MEMORY / I/O           NETWORK
    (compute)      (data access)          (latency)
    
    - High CPU    - High memory use      - High latency
    - Low CPU     - Disk I/O wait       - Packet loss
      (idle?)     - Swap usage          - Connection timeouts
```

**Key insight:** Just because CPU is 100% doesn't mean it's the bottleneck!

### 2. **Load Average vs CPU Usage (Common Confusion)**

```bash
uptime
# 10:30:45 up 5 days, 3:15, 4 users, load average: 8.5, 6.2, 4.1
#                                      ↑1min  ↑5min  ↑15min

# This system has 4 CPU cores
# Load 8.5 = 2.1x oversubscribed (8.5 / 4 = 2.125)
# Meaning: 4 processes running + 4.5 waiting for CPU

# NOT the same as "CPU usage is 212%"!
# Load = queue depth
# CPU % = utilization of available CPU
```

**Critical understanding:**
- Load 1.0 on 4-core = 25% CPU (1 process using 1 core)
- Load 4.0 on 4-core = 100% CPU (all cores busy, queue=0)
- Load 8.0 on 4-core = 100% CPU (queue of 4 waiting)

### 3. **Wait States (I/O vs CPU)**

```bash
# From /proc/stat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  1   1024  2048   512  4096    0    0    50   100  1000  500 30 10 50 10  0
 │  │                                                          │  │  │  │
 │  └─ Processes in I/O wait (b = blocked on I/O)             │  │  │  └─ wa = I/O wait %
 └─ Runnable processes (r)                                     │  │  └─ id = idle %
                                                               │  └─ sy = system time %
                                                               └─ us = user time %

Key insight: If wa% is high (>20%), bottleneck is DISK/NETWORK, not CPU!
```

### 4. **The Problem Layers**

```
        User Process (your app)
                ↓
        Kernel (syscalls, scheduling)
                ↓
        Hardware (CPU, disk, network)

Symptoms can look the same but causes are different:
- "CPU 100%" might be:
  ✓ Actually CPU-bound work (need optimization)
  ✗ Process waiting on I/O (disk/network slow)
  ✗ Runqueue too long (not enough CPU cores)
  
- "System slow" might be:
  ✓ Memory: swap, OOM killer
  ✓ Disk: high I/O wait, slow storage
  ✓ Network: high latency, packet loss
  ✓ CPU: actual compute-bound work
```

### 5. **Context Switches (Hidden Killer)**

```bash
# From vmstat: cs = context switches per second
vmstat 1 3
       CS column: if >10000/sec, too many context switches!
       
Why this matters:
- Context switch = CPU saves state, loads new process (100ns each)
- Too many switches = time spent switching, not working
- Cause: Too many threads, lock contention, oversubscription

Example:
- 10,000 switches/sec × 100ns = 1% CPU just switching!
- But if you have 100 threads on 4 cores, expect thousands
```

---

## ❓ Typical Interview Questions

### **Q1: "System is slow but CPU is only 20%. What could be wrong?"**

**Answer (I/O-bound problem):**

```bash
# Step 1: Check all resources
top                              # What does top show?
free -h                          # Memory usage
df -h                            # Disk space
uptime                           # Load average

# Common findings:
# CPU: 20% ✓ Low
# Memory: 90% used, swapping heavily ✗ Problem!
# Disk: 95% full, 90% I/O wait ✗ Problem!

# Step 2: If swap is high, that's your culprit
free -h
#                 total        used        free
# Mem:            16Gi        14Gi        2Gi
# Swap:           8Gi         6Gi         2Gi ✗ Heavy swapping!

# Why: Physical memory is full, using slow disk swap
# Every memory access = disk I/O (vs RAM)
# That's why system is slow, not CPU

# Step 3: Check what's consuming memory
ps aux --sort=-%mem | head -10
# Shows: Python process using 10GB (memory leak?)

# Step 4: Check I/O metrics
iostat -x 1 3                    # Disk I/O stats

# Look for: %util (utilization), await (latency)
# If %util = 100% and await > 10ms, disk is bottleneck

# Step 5: Fix
# Option A: Kill memory hog
kill -9 <PID>
# System should be responsive again

# Option B: If it's normal, add RAM or optimize code

# Option C: If disk I/O is high
# Check what's reading/writing:
iotop                           # Top processes by disk I/O
```

**Key insight:** Low CPU + slow system = almost always memory or I/O

---

### **Q2: "CPU is at 100% but system responds slowly to new requests. Why?"**

**Answer (Runqueue depth vs utilization):**

```bash
# Step 1: Check CPU metrics
top
# CPU: 100% (looks bad)
# Load: 16 (for 4-core system = 4x oversubscribed!)

# Step 2: This is a clue
# 100% CPU utilization = cores are busy
# But load 16 = 12 processes waiting in queue!

# Step 3: Verify with vmstat
vmstat 1 3
# r  b   ...  r = runnable processes = 12
#             b = blocked on I/O = 0

# This means: 12 processes waiting to run
# When new request comes in, it joins the queue

# Step 4: Why is load so high?
# Check what's running:
ps aux | sort -k3 -rn | head -5
# Shows: 8 Python processes, each 80% CPU

# Step 5: Options
# A) Reduce number of processes (configuration)
# B) Optimize code to use less CPU
# C) Add more CPU cores (scale up)
# D) Scale horizontally (more servers)

# Step 6: Quick workaround
# Lower priority of CPU-intensive processes
ps aux | grep intensive_process | awk '{print $2}' | xargs -I {} renice +10 -p {}

# This doesn't speed up single requests, but reduces starvation
```

**Key learning:** 100% CPU doesn't mean system is bad. Context matters.

---

### **Q3: "Disk I/O is 95%. How do you find what's causing it?"**

**Answer (Identify disk hog):**

```bash
# Step 1: Confirm high disk I/O
iostat -x 1 3
# Look for: %util = 95%, await = 50ms (high latency!)

# Step 2: Identify slow disk
# Multiple disks?
lsblk
# Shows: /dev/sda (busy), /dev/sdb (quiet)

# Step 3: Find the process
iotop -o                         # -o = only processes with I/O
# Shows: process 1234 (python) doing 100MB/s reads

# Step 4: Check what it's doing
ps aux | grep 1234
# python /opt/app/batch_job.py

# Step 5: Check logs
tail -f /var/log/app/batch.log
# Shows: "Processing 10GB file, scanning all data"

# Step 6: Determine if it's normal or bug
# Questions:
# - Is this scheduled (batch job at 3am)?
# - Is this expected (large data processing)?
# - Did it start recently (new code)?

# Step 7: Optimize or reschedule
# Option A: Optimize I/O (add index, cache, etc.)
# Option B: Add disk (SSD faster than HDD)
# Option C: Reschedule batch job to off-peak
# Option D: Parallelize (multiple disks)
```

**Common causes of high disk I/O:**
- Swapping (memory pressure)
- Batch jobs (nightly backups, reports)
- Database intensive queries
- Log flushing
- Filesystem checks

---

### **Q4: "p99 latency is spiking every 5 minutes. What's happening?"**

**Answer (Periodic behavior pattern):**

```bash
# Step 1: Correlate with time
# Every 5 minutes = likely periodic task
# Check: cron? GC? Cache cleanup? Monitoring job?

# Step 2: Check system metrics at spike time
# Collect baseline (now vs during spike)
# Run this during spike:
vmstat 1 3
iostat -x 1 3
netstat -an | wc -l

# Compare to baseline values

# Step 3: Check cron jobs
crontab -l
# Look for: tasks running every 5 minutes

ls /etc/cron.d/
# System cron jobs

# Step 4: Check if it's GC (Java/Python)
# For Java processes:
ps aux | grep java | grep -i "verbose:gc"
tail -f /path/to/gc.log

# Look for: pause times aligning with latency spikes

# Step 5: Check logs at spike time
# Grab a spike time: 10:00:30
journalctl --since "10:00:25" --until "10:00:35"
tail -30 /var/log/app.log | grep -B5 -A5 "10:00:3"

# Look for: "Starting GC", "Flushing cache", "Syncing logs"

# Step 6: Identify the cause
# Common culprits:
# - Garbage collection (GC pause, every 5 min = heap setup)
# - Periodic stats collection (every 5 min = normal)
# - Cache eviction (LRU cleanup every N items)
# - Log rotation/flush
# - External API calls (timeout recovery?)

# Step 7: Fix
# If GC: increase heap size or tune GC
# If batch: make async/parallelized
# If monitoring: run async, don't block main thread
```

**Pattern recognition tip:**
> Periodic symptoms = periodic cause (cron, GC, timers, polling)

---

## 🚨 Common Mistakes

❌ **"CPU is 100%, so we need to optimize code"**  
✅ Check I/O wait! Might be disk, not code.

❌ **"Load 8 is bad, we need to scale"**  
✅ Load 8 on 16-core system is fine (50% utilized). Check CPU %.

❌ **"Swap is high, so memory is leaking"**  
✅ Swap is normal! High swap + high memory = problem.

❌ **"iostat shows 100% utilization, disk is broken"**  
✅ 100% utilization is ok if await time is low (<5ms, SSD).

❌ **"I'll just kill the process to speed up system"**  
✅ That doesn't fix the root cause. What was the process doing?

---

## 🛠️ Essential Performance Debugging Commands

### **System Overview**

```bash
# One-liner performance snapshot
echo "=== LOAD ===" && uptime && echo "=== CPU ===" && top -bn1 | head -3 && echo "=== MEM ===" && free -h && echo "=== DISK ===" && df -h / && echo "=== I/O ===" && iostat -x 1 2
```

### **CPU Analysis**

```bash
# Top CPU consumers
top -bn1 -o %CPU | head -15

# Context switches (high = problem)
vmstat 1 3 | tail -1 | awk '{print "Context switches/s:", $NF}'

# Per-core CPU usage
mpstat -P ALL 1 3
```

### **Memory Analysis**

```bash
# Detailed memory info
free -h
cat /proc/meminfo

# Swap usage (if non-zero, memory is tight)
free -h | grep Swap

# Memory by process
ps aux --sort=-%mem | head -10

# Check for memory leaks (growing RSS)
ps -o pid,rss,vsz,cmd -p <PID> && sleep 60 && ps -o pid,rss,vsz,cmd -p <PID>
```

### **Disk I/O Analysis**

```bash
# Disk utilization and latency
iostat -x 1 3                   # %util, await, svctm

# Top processes by I/O
iotop -o -b -n 1

# Files being accessed
lsof | grep <PID> | grep REG   # Regular files

# Disk queue depth
iostat -x 1 1 | grep -E "sda|sdb"
```

### **Network Analysis**

```bash
# Connection count
netstat -an | wc -l

# Connections by state
netstat -an | awk '{print $6}' | sort | uniq -c

# Active connections to external hosts
netstat -an | grep ESTABLISHED

# Network interface stats
ifstat 1 3
```

### **Advanced: Tracing**

```bash
# What syscalls is a process making?
strace -e trace=read,write -o trace.log -p <PID>

# Sample stack traces (perf)
perf record -F 99 -p <PID> -g -- sleep 30
perf report

# Flamegraph (visualization)
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg
```

---

## 🎬 Real-World Scenario

**Situation:** Application response time degraded over the week (5ms → 50ms)

```bash
# Day 1 - Noticed slow requests
# Day 2 - Still slow, starting investigation
# Day 3 - Full on-call incident

# Investigation begins:
# Check current state
top
# CPU: 30% (fine)
# Memory: 85% used, swapping
# Load: 2 (ok)

# Hypothesis 1: Memory leak
# Check memory growth over time
ps aux | grep myapp
# VSZ: 4GB (was 2GB on Monday)

# Graph memory:
# Day 1: 2GB
# Day 2: 3GB
# Day 3: 4GB
# Pattern: Linearly growing!

# Root cause: Memory leak in new code

# Check git log
git log --oneline -20
# Commit 5 days ago: "Add caching layer"

# Check the commit
git show abc123def
# Looks like: cache never expires, grows forever

# Fix: Add cache eviction
# Option A: Set max cache size
# Option B: Add TTL to cached entries
# Option C: Revert the commit

# Implemented fix:
git revert abc123def
systemctl restart myapp

# Monitor memory
watch -n 5 'ps aux | grep myapp | grep -v grep'

# Memory stabilizes at 2GB ✓
# Response time returns to 5ms ✓

# Post-incident:
# 1. Add memory monitoring alert (> 3GB)
# 2. Add code review for caching patterns
# 3. Document cache eviction policy
```

---

## 📊 Interview Tips

✅ **Show multi-layer thinking:** "Could be CPU, memory, disk, or network..."  
✅ **Use the right metrics:** "Load average, not CPU %. Await time, not utilization."  
✅ **Ask clarifying questions:** "How long has it been slow? Suddenly or gradual?"  
✅ **Think patterns:** "Spikes every 5 min = periodic task, not random."  
✅ **Eliminate systematically:** "Not CPU, check memory. Not memory, check I/O."  

---

## 🔗 Related Files

- [01-process-management-interview.md](./01-process-management-interview.md)
- [04-storage-disk-interview.md](./04-storage-disk-interview.md)
- [05-logs-syslog-interview.md](./05-logs-syslog-interview.md)
- [06-system-troubleshooting-workflow.md](./06-system-troubleshooting-workflow.md)
- [sre-interview-guide.md](./sre-interview-guide.md)

---

**Congratulations!** You've completed all 7 Linux SRE interview guides. 

**Study sequence:**
1. sre-interview-guide.md (overview + study plan)
2. 01-07 guides in order (or focus on your weak areas)
3. Practice real scenarios
4. Mock interviews with a friend

**Good luck! 🎯**
