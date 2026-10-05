# top - Table of Processes

`top` displays a dynamic, real-time view of running processes and system statistics. It's the go-to command for monitoring overall system health.

## Quick Reference

```bash
top                    # Start interactive mode
top -l 1              # Show once and exit (macOS)
top -b -n 1           # Batch mode, one iteration (Linux)
top -p PID            # Monitor specific process
top -u username       # Show processes for user
top -o %CPU           # Sort by CPU (macOS)
top -o %MEM           # Sort by memory (macOS)
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
top -p 12345
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
top -p PID -l 100 -s 1
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
A: Run `top -p PID` and watch the MEM column over 10+ minutes. If it keeps increasing, there's a leak.

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

- **Batch mode for scripting (Linux):**
  ```bash
  top -b -n 1 | tail -n +8
  ```

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

