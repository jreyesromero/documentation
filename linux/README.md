# Linux Commands & SRE Interview Prep

Welcome to the Linux reference guide covering essential commands and SRE interview preparation for DevOps, Platform Engineers, and SRE engineers.

## 🎯 SRE Interview Study Guides (1-Hour Prep)

**New to SRE interviews?** Start here for focused, 1-week interview preparation.

- **[SRE Interview Master Guide](./sre-interview-guide.md)** ← **START HERE**
  - Overview of typical questions
  - Study plan for 1 week
  - Quick command reference
  - What interviewers are looking for

**Detailed Study Guides (by topic):**
1. [Process Management Interview Guide](./01-process-management-interview.md) - Processes, signals, resource management
2. [Filesystem & Permissions Interview Guide](./02-filesystem-permissions-interview.md) - Inodes, permissions, file types
3. [User & Group Management Interview Guide](./03-user-group-management-interview.md) - passwd, shadow, user administration
4. [Storage & Disk Interview Guide](./04-storage-disk-interview.md) - Disk space, RAID, LVM, filesystems
5. [Logs & Syslog Interview Guide](./05-logs-syslog-interview.md) - Log analysis, journalctl, troubleshooting
6. [System Troubleshooting Workflow Interview Guide](./06-system-troubleshooting-workflow.md) - Systematic debugging, common scenarios
7. [Performance Analysis Interview Guide](./07-performance-analysis-interview.md) - CPU, memory, disk I/O, network bottlenecks

---

## 📚 Command Reference

Welcome to the Linux Commands reference guide. This section covers essential commands used by DevOps, Platform Engineers, and SRE engineers.

## Table of Contents

### Process & System Monitoring
- [ps](./ps.md) - Process Status
- [top](./top.md) - Table of Processes (real-time monitoring)
- [lsof](./lsof.md) - List Open Files (process/port debugging)

### Network Troubleshooting
- [netstat](./netstat.md) - Network Statistics (connections, ports)
- [dig](./dig-nslookup.md) - DNS Queries
- [curl](./curl-wget.md) - HTTP Testing & API calls

### Disk & Storage
- [df & du](./df-du.md) - Disk Space Analysis
- [mount & umount](./mount-umount.md) - Filesystem Management

### Log Analysis & Text Processing
- [tail & grep](./tail-grep.md) - Log viewing & pattern searching
- [awk & sed](./awk-sed.md) - Text processing and transformation
- [journalctl](./journalctl.md) - Systemd journal logs

### System Administration
- [systemctl](./systemctl.md) - Service Management
- [crontab](./crontab.md) - Scheduled Tasks
- [find](./find.md) - File searching

### Advanced Debugging
- [strace](./strace.md) - System call tracing

---

**Note:** Each command file includes syntax, common flags, practical examples, and real-world use cases.

**Interview Focus:** 
- For **concept-based questions** (what is inode? explain fork/exec?) → Use Study Guides above
- For **command usage & syntax** → Use this reference section below
- **Top 8 essential commands** (ps, top, netstat, df, tail, grep, find, curl) cover ~90% of interview scenarios
