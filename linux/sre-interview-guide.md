# SRE Interview Guide - Linux Focus (1 Hour Prep)

> **Goal:** Master the core Linux concepts & troubleshooting skills for a 1-hour SRE screening interview.

**Time allocation:** 50 min studying + 10 min review before interview.

---

## 📌 Interview Structure (What to Expect)

Most SRE interviews follow this pattern:
1. **10 min** - Introductions & warm-up questions
2. **30 min** - Core Linux concepts + troubleshooting scenarios
3. **15 min** - Practical problem-solving (live or scenario-based)
4. **5 min** - Questions for interviewer

---

## 🎯 Typical SRE Interview Questions by Topic

### **1. PROCESS MANAGEMENT** (10 min - ~2 questions)

**Q1: "How do you find and kill a runaway process consuming 90% CPU?"**
- Check with `top` or `ps aux | grep processname`
- Understand `SIGTERM` (graceful) vs `SIGKILL` (forceful)
- Know when to use `kill`, `kill -9`, or `systemctl stop`
- **Red flag:** Using `kill -9` as first option

**Q2: "What's the difference between a process and a thread? Why does it matter?"**
- Process = isolated memory space; Thread = shared memory
- Processes are safer but heavier; threads are lighter but risky
- SRE cares about resource usage & stability

**Q3: "Explain what happens when you type a command in the shell"**
- Shell forks a child process
- Child process execs the command binary
- Parent waits for child to finish (or runs in background with &)
- Interviewers want to see you understand fork() + exec()

---

### **2. FILESYSTEM & PERMISSIONS** (10 min - ~2 questions)

**Q1: "A user says they can't write to a directory even though they own it. How do you troubleshoot?"**
- Check permissions with `ls -ld dirname`
- Understand rwx bits: r(4) w(2) x(1)
- Owner needs `w` on directory to create files, `x` to enter
- Check parent dir permissions too (need `x` to traverse)
- Use `chmod` to fix

**Q2: "What's an inode? Why do we care about the inode limit?"**
- Inode = metadata for a file (permissions, owner, size, pointers to data blocks)
- Filesystem has max inodes
- Running out of inodes = can't create files even with free space
- Check with `df -i`
- **This is a gotcha interview question**

**Q3: "I added a user but they can't write to /home/username. Why?"**
- Home dir must be owned by user with rwx permissions
- Check `ls -la /home/ | grep username`
- Permission bits: owner needs 700 (rwx)

---

### **3. USER & GROUP MANAGEMENT** (8 min - ~2 questions)

**Q1: "You need to add a user account. Walk me through the steps."**
- Use `useradd -m -s /bin/bash username` (creates home dir)
- Set password with `passwd username`
- Verify with `id username`
- Check home dir ownership
- **Red flag:** Manual editing of /etc/passwd instead of useradd

**Q2: "Why should you never recycle or reuse UIDs?"**
- Old files from backups still have UID as owner
- If you reuse UID, new user owns old files (security risk!)
- Also causes confusion with NFS
- **This tests understanding of UID's role in the system**

**Q3: "What's the difference between /etc/passwd and /etc/shadow?"**
- /etc/passwd = readable by all, contains login name & UID
- /etc/shadow = root-only, contains encrypted passwords
- Both must stay in sync (use `useradd`, not manual editing)

---

### **4. STORAGE & DISK MANAGEMENT** (10 min - ~2-3 questions)

**Q1: "Disk is 90% full. What do you check first? How do you find what's using space?"**
- `df -h` shows filesystem usage
- `du -sh /*` shows directory sizes
- `du -sh /var/log/*` often finds the culprit (old logs)
- `find / -size +1G` finds large files
- **Practical scenario - this is common in real SRE work**

**Q2: "You have 50% disk free but 'disk full' errors. Why?"**
- Running out of inodes (not space)
- Check with `df -i`
- Solution: delete files or increase inode count (usually need reinstall)
- **This is a classic gotcha**

**Q3: "What's RAID and why do we use it in production?"**
- RAID = Redundant Array of Independent Disks
- RAID 0 = striping (fast, no redundancy)
- RAID 1 = mirroring (redundant, slower writes)
- RAID 5/6 = striping + parity (balance of speed & redundancy)
- SREs care about failure tolerance

**Q4: "What's LVM and how does it help with storage management?"**
- LVM = Logical Volume Manager
- Allows dynamic resizing of partitions without reboot
- Snapshots for backups
- Better abstraction than raw partitions

---

### **5. LOGS & SYSLOG** (8 min - ~2 questions)

**Q1: "A service is failing. Walk me through how you'd debug it using logs."**
- Check service status: `systemctl status servicename`
- View logs: `journalctl -u servicename -n 50` (last 50 lines)
- Or: `tail -f /var/log/servicename.log`
- Look for errors, timestamps, stack traces
- Cross-reference with system logs: `/var/log/syslog` or `/var/log/messages`
- **This is a real SRE daily task**

**Q2: "Logs are growing too fast and filling up disk. How do you handle it?"**
- Check log size: `du -sh /var/log/*`
- Compress old logs: `gzip /var/log/app.log.1`
- Set up logrotate to automate: `/etc/logrotate.d/myapp`
- Or: increase disk, use log aggregation (ELK, Splunk)
- **Understanding log rotation is critical**

**Q3: "How would you monitor a log file for errors in real-time?"**
- `tail -f /var/log/myapp.log` (follow mode)
- `journalctl -u myapp -f` (follow systemd logs)
- `tail -f /var/log/myapp.log | grep ERROR`
- For alerting: use tools like Splunk, ELK, or grep with alerts

---

## 🚨 Red Flags (What NOT to Say)

❌ "I don't know, I'd just Google it"  
✅ "I'd start with X, then check Y if that doesn't work"

❌ "I always use `kill -9` to stop processes"  
✅ "I use SIGTERM first (graceful), then SIGKILL only if needed"

❌ "UID is just a number, doesn't matter"  
✅ "UID is how the system identifies users; reusing them causes security issues"

❌ "I just edit `/etc/passwd` by hand"  
✅ "I use tools like `useradd` to keep passwd and shadow in sync"

❌ "Disk is full, so we're out of space"  
✅ "We need to check both disk space AND inode usage with `df -h` and `df -i`"

---

## 🎓 Study Plan (1-2 Weeks)

| Day | Topic | Files to Read | Time |
|-----|-------|---------------|------|
| 1 | Process Management | `01-process-management-interview.md` + `ps.md`, `top.md` | 60 min |
| 2 | Filesystem & Permissions | `02-filesystem-permissions-interview.md` + `find.md` | 60 min |
| 3 | User/Group Management | `03-user-group-management-interview.md` | 45 min |
| 4 | Storage & Disk | `04-storage-disk-interview.md` + `df-du.md` | 60 min |
| 5 | Logs & Syslog | `05-logs-syslog-interview.md` + `journalctl.md` | 60 min |
| 6 | System Troubleshooting | `06-system-troubleshooting-workflow.md` | 45 min |
| 7 | Performance Analysis | `07-performance-analysis-interview.md` | 60 min |
| 8 | Review + Practice Scenarios | All guides, focus on weak areas | 90 min |
| 9 | Mock interview + rest | - | 60 min |

---

## 📚 Quick Reference: Most Important Commands

```bash
# Process Management
ps aux | grep processname          # Find process
top                                 # Real-time monitoring
kill -TERM PID                      # Graceful kill
kill -9 PID                         # Forceful kill (last resort)

# Filesystem
ls -ld dirname                      # Check directory permissions
chmod 755 dirname                   # Set permissions (rwxr-xr-x)
df -h                               # Disk space by filesystem
df -i                               # Inode usage
du -sh /*                           # Directory sizes

# User Management
useradd -m -s /bin/bash username    # Add user with home dir
passwd username                     # Set password
id username                         # Check user info
chown -R user:group dirname         # Change ownership

# Logs
systemctl status servicename        # Check service status
journalctl -u servicename -n 50     # View service logs
tail -f /var/log/app.log            # Follow log file
grep ERROR /var/log/app.log         # Search for errors

# Storage
lsblk                               # List block devices
fdisk -l                            # Partition table
mount | grep -E '^/dev'             # Mounted filesystems
```

---

## 🎯 What Interviewers Are Looking For

✅ **Problem-solving approach:** Can you ask clarifying questions?  
✅ **Practical knowledge:** Have you actually done this before?  
✅ **Troubleshooting methodology:** Do you check logs? Try to isolate?  
✅ **Understanding root causes:** Not just symptoms, but why?  
✅ **Automation mindset:** How would you prevent this happening again?  
✅ **Communication:** Can you explain technical concepts clearly?

---

## 📖 Next Steps

1. **Read this guide** (15 min)
2. **Pick ONE topic** (start with Process Management)
3. **Read the detailed interview guide** for that topic (30 min)
4. **Review the command reference files** (ps.md, top.md, etc.) (20 min)
5. **Practice the scenario** (write down how you'd troubleshoot) (15 min)
6. **Repeat for other topics**

---

## 🔗 Related Files

**Core Interview Guides:**
- [01-process-management-interview.md](./01-process-management-interview.md)
- [02-filesystem-permissions-interview.md](./02-filesystem-permissions-interview.md)
- [03-user-group-management-interview.md](./03-user-group-management-interview.md)
- [04-storage-disk-interview.md](./04-storage-disk-interview.md)
- [05-logs-syslog-interview.md](./05-logs-syslog-interview.md)
- [06-system-troubleshooting-workflow.md](./06-system-troubleshooting-workflow.md)
- [07-performance-analysis-interview.md](./07-performance-analysis-interview.md)

**Command Reference Files:**
- [ps.md](./ps.md) - Process status
- [top.md](./top.md) - Real-time monitoring
- [df-du.md](./df-du.md) - Disk usage
- [journalctl.md](./journalctl.md) - System logs
- [systemctl.md](./systemctl.md) - Service management

---

**Last updated:** 2026-10-16  
**Confidence level:** High (based on actual SRE interview patterns)
