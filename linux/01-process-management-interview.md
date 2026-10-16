# Process Management - SRE Interview Guide

**Time to master:** 60 minutes  
**Difficulty:** ⭐⭐⭐ (Medium)  
**Interview frequency:** Very High (appears in ~80% of SRE interviews)

---

## ⚡ Quick Summary

Processes are running programs. SREs need to:
- **Find** problematic processes (high CPU, memory, zombie)
- **Understand** process lifecycle (fork → exec → exit)
- **Control** processes (signals: SIGTERM, SIGKILL)
- **Monitor** resource usage (ps, top, htop)
- **Manage** priorities (nice, renice)

---

## 🎯 Key Concepts to Know

### 1. **What is a Process?**
- Running instance of a program
- Has own memory space (isolated)
- Identified by PID (Process ID)
- Has parent process (PPID)
- Process state: Running, Sleeping, Zombie, Stopped

### 2. **Process Lifecycle**
```
Parent Process
    ↓
fork() → Create child (copy of parent)
    ↓
exec() → Load new program into memory
    ↓
Process runs...
    ↓
exit() → Process terminates
    ↓
Parent reaps child (waits for exit status)
```

### 3. **Process States**
- **R** (Running) = Currently using CPU
- **S** (Sleeping) = Waiting for I/O or timer
- **Z** (Zombie) = Dead, but parent hasn't reaped it yet
- **T** (Stopped) = Paused by signal
- **D** (Uninterruptible Sleep) = Waiting for disk I/O (dangerous state!)

### 4. **Signals (How to Control Processes)**
- **SIGTERM (15)** = Graceful shutdown (process can catch & clean up)
- **SIGKILL (9)** = Forceful kill (cannot be caught, use as last resort)
- **SIGSTOP (19)** = Pause process (can be resumed)
- **SIGCONT (18)** = Resume paused process
- **SIGHUP (1)** = Reload config (some daemons)
- **SIGUSR1/USR2 (10/12)** = Application-specific

### 5. **Process Priority**
- **nice** = -20 (highest priority) to +19 (lowest priority)
- **Default nice** = 0
- Negative nice = higher priority (CPU gets it first)
- Positive nice = lower priority (yields to others)

---

## ❓ Typical Interview Questions

### **Q1: "A process is consuming 90% CPU. How do you find and kill it?"**

**Answer:**
```bash
# Step 1: Find the process
top                              # See top CPU consumers (sorted by CPU)
# OR
ps aux | sort -k3 -rn | head -5  # Top 5 by CPU usage

# Step 2: Get the PID
ps aux | grep processname        # Find specific process & PID

# Step 3: Send graceful kill first
kill -TERM <PID>                 # Ask nicely, give it 5-10 seconds

# Step 4: If it doesn't die, force it
sleep 10
kill -9 <PID>                    # Nuclear option (no cleanup possible)

# Step 5: Verify it's gone
ps aux | grep <PID>              # Should show nothing
```

**Why this matters:** Shows understanding of signal handling and graceful shutdown. Never jump to `-9` without trying `-TERM` first.

---

### **Q2: "You see a 'Z' (zombie) process in `ps`. What does it mean? How do you fix it?"**

**Answer:**

A **zombie** process is one that:
- Has finished executing (dead)
- Parent process hasn't reaped it yet (called wait())
- Takes up a PID slot but no resources
- Appears as `<defunct>` in ps output

**Why it happens:**
- Parent forked child but didn't wait()
- Parent exited before reaping child
- Child becomes "orphaned" to init (PID 1) which should reap it

**How to fix:**
```bash
# Find the zombie
ps aux | grep Z                  # Look for 'Z' state

# Find the parent process
ps -o ppid= -p <ZOMBIE_PID>      # Get parent PID

# Option 1: Kill the parent (if it's stuck)
kill -9 <PARENT_PID>             # Zombie dies when parent reaps it

# Option 2: Just leave it (kernel will recycle PID eventually)
# Zombies don't consume memory, just a PID slot
```

**Red flag:** If you have many zombies, something is wrong with the parent process (bug in code).

---

### **Q3: "How does a shell execute a command? Walk me through it."**

**Answer:**

When you type `ls -la /home`:

1. **Shell reads the command** (ls -la /home)
2. **Shell forks** → Creates child process (copy of shell)
3. **Child process execs** → Replaces itself with `ls` binary
4. **ls runs** with args (-la /home)
5. **ls exits** → Child process terminates
6. **Parent (shell) waits** → Blocked until child exits
7. **Shell resumes** → Ready for next command (if foreground) OR returns prompt immediately (if background with &)

**Code flow:**
```c
pid = fork();      // Shell creates child
if (pid == 0) {    // In child
  exec("/bin/ls", "-la", "/home");  // Replace with ls
}
// Parent code
wait();            // Wait for child to exit
```

**Why SREs care:** 
- Understand why background jobs (`&`) let shell continue
- Know why killing parent sometimes kills child (process groups)
- Understand resource usage (each fork = new memory space)

---

### **Q4: "Explain the difference between processes and threads. Why does an SRE care?"**

**Answer:**

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Isolated (own space) | Shared (same space) |
| Creation | Heavy (fork() is expensive) | Light (cheap to create) |
| Context switch | Slower (full memory swap) | Faster (minimal swap) |
| Communication | Need IPC (pipes, sockets) | Direct memory access |
| Crash impact | Only itself | Can crash entire process |
| Security | Better isolation | Risky (one thread = all threads) |

**SRE implications:**
- **Processes** = safer (isolation) but slower (fork overhead)
- **Threads** = faster but risk (one bug = whole process crashes)
- **Resource limits** matter more for threaded apps (memory limits affect all threads)
- **Debugging** is harder with threads (shared memory = race conditions)

---

## 🚨 Common Mistakes

❌ **"I always use `kill -9`"**  
✅ Use `kill -TERM` first, only `-9` if needed

❌ **"Zombies are bad, I need to kill them"**  
✅ Zombies don't harm system, they indicate a parent process bug

❌ **"Top shows 100% CPU usage"**  
✅ If load < CPU count, system is fine. Top % is % of that CPU core.

❌ **"I'll just increase CPU priority to fix slowness"**  
✅ Usually the problem is I/O, disk, or memory — not CPU priority

❌ **"Process owns X GB of memory (from ps)"**  
✅ ps shows virtual memory, not real (RSS is better); use `top` for clarity

---

## 🛠️ Essential Commands with Examples

### **View Processes**

```bash
# Show all processes
ps aux                           # Standard snapshot
ps aux | head -20                # First 20 processes
ps aux | grep servicename        # Find specific process

# Sort by resource usage
ps aux --sort=-%cpu | head -10   # Top 10 by CPU
ps aux --sort=-%mem | head -10   # Top 10 by memory

# Show process tree (hierarchy)
pstree                           # Tree view of processes
pstree -p PID                    # Tree for specific process

# Get specific info
ps -o pid,ppid,cmd -p <PID>     # Show PID, parent, command
ps -o %cpu,%mem,cmd -p <PID>    # Show resource usage
```

### **Real-Time Monitoring**

```bash
# Interactive monitoring (best for troubleshooting)
top                              # Press 'q' to quit
# In top: press 'Shift+P' for CPU sort, 'Shift+M' for memory sort

# Simpler alternative
htop                             # Better UI than top (if available)

# Watch a process
watch -n 1 'ps aux | grep servicename'  # Refresh every 1 sec
```

### **Control Processes**

```bash
# Send signals
kill -TERM <PID>                 # Graceful shutdown
kill -9 <PID>                    # Force kill
kill -STOP <PID>                 # Pause process
kill -CONT <PID>                 # Resume process

# Change priority
nice -n 10 command               # Start with low priority (+10)
renice -n -5 -p <PID>            # Change running process to high priority (-5)

# Background/Foreground
command &                        # Run in background
jobs                             # List background jobs
fg %1                            # Bring job 1 to foreground
bg %1                            # Resume job 1 in background
```

### **Check Zombie Processes**

```bash
# Find zombies
ps aux | grep Z                  # Look for 'Z' state
ps aux | grep '<defunct>'        # Alternate search

# Find zombie's parent
ps -o ppid= -p <ZOMBIE_PID>      # Get parent PID
ps -p <PARENT_PID>               # Check parent status
```

### **System-wide Monitoring**

```bash
# Process count
ps aux | wc -l                   # Total number of processes

# Check load average
uptime                           # Shows 1, 5, 15 min load average
cat /proc/loadavg                # Raw load data

# Find stuck processes (uninterruptible sleep)
ps aux | grep ' D '              # 'D' state = dangerous I/O wait
```

---

## 🎬 Real-World Scenario

**Situation:** Service is slow, you have 30 seconds to fix it.

**Approach:**
```bash
# 1. Check overall load
uptime                           # Is system overloaded?

# 2. Find culprit process
top                              # Top CPU/memory consumers
# Spot: Python process using 98% CPU

# 3. Identify it more
ps aux | grep python             # Get PID: 2847

# 4. Try graceful shutdown
kill -TERM 2847
sleep 5

# 5. Check if it died
ps -p 2847                       # Still running?

# 6. Force kill if needed
kill -9 2847

# 7. Service probably restarts automatically (systemd)
systemctl status myservice       # Verify it restarted
```

**Key insight:** Know when to stop investigating and just kill it. In production, time matters more than perfection.

---

## 📊 Interview Tips

✅ **Mention the problem-solving approach:** "I'd first check with top to see the culprit..."  
✅ **Show understanding of signals:** "SIGTERM is graceful, SIGKILL is forceful..."  
✅ **Ask clarifying questions:** "Is this a production or dev system? How urgent?"  
✅ **Know when to escalate:** "If I can't identify the process, I'd check recent changes..."  

---

## 🔗 Related Files

- [ps.md](./ps.md) - Detailed `ps` command reference
- [top.md](./top.md) - Detailed `top` command reference
- [sre-interview-guide.md](./sre-interview-guide.md) - Master interview guide

---

**Next:** Read [02-filesystem-permissions-interview.md](./02-filesystem-permissions-interview.md)
