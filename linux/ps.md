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

