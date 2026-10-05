# df & du - Disk Space Analysis

`df` shows filesystem disk usage (how much space is used/available on mounted filesystems), while `du` shows directory disk usage (how much space each directory/file is using).

## Quick Reference

```bash
# df - filesystem overview
df -h                    # Human-readable, all filesystems
df -h /path              # Check specific mount point
df -i                    # Inode usage instead of blocks

# du - directory breakdown
du -sh *                 # Size of each item in current directory
du -sh directory         # Total size of directory
du -sh ./*  | sort -h    # Size each item, human-readable, sorted
du -h --max-depth=2      # Limit depth of recursion
```

---

## df - Disk Free

### Output Format

```
Filesystem        Size    Used   Avail Capacity iused ifree %iused  Mounted on
/dev/disk3s1s1   460Gi    12Gi   292Gi     4%    459k  3.1G    0%   /
devfs            200Ki   200Ki     0Bi   100%     692     0  100%   /dev
/dev/disk3s5     460Gi   148Gi   292Gi    34%    1.5M  3.1G    0%   /System/Volumes/Data
```

| Column | Meaning |
|--------|---------|
| **Filesystem** | Device name or mount point |
| **Size** | Total capacity of filesystem |
| **Used** | Space currently used |
| **Avail** | Available space remaining |
| **Capacity** | Percentage used (85%+ is concerning) |
| **iused/ifree** | Inode usage (each file is an inode) |
| **Mounted on** | Where filesystem is mounted |

### Common Scenarios

**Check if disk is full:**
```bash
df -h | grep -E "9[0-9]%|100%"
```

**Monitor specific partition:**
```bash
df -h /var/log
```

**Check inode exhaustion (space shows available, but can't create files):**
```bash
df -i /
# If %iused is >95%, you're out of inodes despite free blocks
```

---

## du - Disk Usage

### Output Format

```
584K   /Users/julianre/Music
5.2M   /Users/julianre/Desktop
 15M   /Users/julianre/tmp
 22M   /Users/julianre/Pictures
250M   /Users/julianre/go
1.2G   /Users/julianre/Downloads
1.9G   /Users/julianre/java_error_in_idea.hprof
2.3G   /Users/julianre/Documents
3.6G   /Users/julianre/workspace
 42G   /Users/julianre/Library
```

| Column | Meaning |
|--------|---------|
| **Size** | Total disk usage (includes all subdirectories) |
| **Path** | Directory or file path |

### Common Scenarios

**Find largest directories:**
```bash
du -sh /* | sort -h | tail -10
```

**Find largest files (not directories):**
```bash
find / -type f -exec du -h {} + | sort -h | tail -10
```

**Deep analysis of a directory:**
```bash
du -h --max-depth=3 /var
```

**What's taking up space in /var/log:**
```bash
du -h /var/log/* | sort -h
```

---

## Critical Interview Scenarios

### "The disk is 95% full, find what's consuming space"

```bash
# Step 1: Check overall usage
df -h

# Step 2: Find the full filesystem (e.g., /var)
df -h | grep -E "9[0-9]%|100%"

# Step 3: Dive into that filesystem
du -h /var --max-depth=1 | sort -h

# Step 4: Go deeper into largest directory
du -h /var/log --max-depth=2 | sort -h

# Step 5: Find and remove old logs
find /var/log -name "*.log" -mtime +30 -delete
```

### "Filesystem shows 0% available but reports free blocks"

This is an inode exhaustion issue:

```bash
# Check inode usage
df -i /

# If %iused is 100%, you can't create new files
# Find what's using inodes (usually many small files)
find / -type f | wc -l

# Or find directories with many files:
find / -type d -exec sh -c 'echo $(ls -1 "$1" | wc -l) "$1"' _ {} \; 2>/dev/null | sort -rn | head -10
```

### "Logs filled up disk overnight"

```bash
# Find recently modified large files
find /var/log -type f -mtime -1 -exec du -h {} + | sort -rh | head -10

# Clean up old logs
find /var/log -name "*.log" -mtime +7 -delete
find /var/log -name "*.gz" -mtime +30 -delete

# Check if application is logging excessively
tail -f /var/log/application.log | grep ERROR | wc -l
```

---

## Real-World Troubleshooting

### Application Crashed Due to Disk Full

```bash
# Identify the problem
df -h

# Locate culprit
du -sh /* | sort -h

# Find most recent large files
find / -type f -newer /tmp/marker_file -exec du -h {} + | sort -rh | head -20

# Clean up
rm -rf /tmp/large_temp_files
# or
find /var/log -type f -mtime +30 -delete
```

### Running Out of Inodes Despite Free Space

```bash
# Verify the problem
df -i /

# Find directory with most files
for i in /var/*; do echo -n "$i: "; find "$i" -type f | wc -l; done | sort -t: -k2 -rn

# Solutions:
# 1. Delete old files
find /var/log -type f -mtime +30 -delete

# 2. Archive instead of delete
tar czf old_logs.tar.gz /var/log/old_*
rm /var/log/old_*
```

---

## Comparison: df vs du

| Aspect | df | du |
|--------|----|----|
| **Shows** | Filesystem-level usage | Directory/file usage |
| **Speed** | Very fast | Slower (recursion) |
| **Use case** | Overall disk health | Find space hogs |
| **Output** | High-level summary | Detailed breakdown |

**Rule:** If `df` shows 80% full but total `du -sh /*` is only 60%, something is using space not visible (deleted files still open by processes, snapshots, compression).

---

## Tips & Tricks

- **Combine df + du for investigation:**
  ```bash
  df -h /var
  du -sh /var/*
  du -sh /var/log/*
  du -sh /var/log/archive/*
  ```

- **Watch disk growth over time:**
  ```bash
  date >> disk_usage.log
  du -sh / >> disk_usage.log
  ```

- **Find large files changed recently:**
  ```bash
  find /var -type f -size +100M -mtime -7
  ```

- **Safe cleanup (test first):**
  ```bash
  # See what would be deleted
  find /var/log -name "*.log" -mtime +30

  # Then delete
  find /var/log -name "*.log" -mtime +30 -delete
  ```

---

## Interview Tips

1. Always start with `df -h` to see the overall picture
2. Then use `du -sh` to find culprits
3. Mention checking for `deleted files still held open` (lsof +D /path)
4. Know the difference between inode and block exhaustion
5. Be familiar with log rotation (`logrotate`) as a solution

