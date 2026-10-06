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

---

## SRE Troubleshooting Scenarios

### Scenario 1: Disk 95% Full - Emergency Cleanup

**Problem:** Production alert: disk usage at 95%. Service might crash if it fills. Need immediate action.

**Emergency Investigation & Cleanup:**
```bash
# 1. What's using all the space?
df -h

# 2. Get the critical filesystem
FILESYSTEM=/dev/sda1
MOUNTPOINT=/

# 3. What directory is the culprit?
du -sh /* | sort -h | tail -10

# 4. Drill down into largest
du -sh /var/* | sort -h | tail -10
du -sh /var/log/* | sort -h | tail -10
du -sh /home/* | sort -h | tail -10

# 5. Most likely culprit: logs or temp files
ls -lh /var/log/*.log | sort -k5 -h | tail -10

# 6. Check if logs are old and can be deleted
find /var/log -name "*.log" -mtime +30 -ls | awk '{print $11, $7}' | sort

# 7. IMMEDIATE CLEANUP (if safe):
# Delete old rotated logs
find /var/log -name "*.gz" -mtime +7 -delete
find /var/log -name "*.1" -mtime +7 -delete

# Or delete old application logs
find /var/log/myapp -name "*.log" -mtime +30 -delete

# 8. Check temp directories
du -sh /tmp /var/tmp /dev/shm

# 9. Clean old temp files (> 7 days, owned by nobody/root)
find /tmp -type f -mtime +7 -delete

# 10. Verify space is freed
df -h
```

**Important Safety Checks:**
```bash
# NEVER delete files without checking:
# 1. Who owns them?
ls -la /var/log/*.log | head -5

# 2. Are they currently in use?
lsof | grep -E "\.log|\.tmp" | head -10

# 3. Check file modification time
find /var/log -name "*.log" -mtime +30  # Files older than 30 days

# 4. In production, rotate instead of delete
systemctl restart rsyslog  # Force log rotation
logrotate -f /etc/logrotate.d/rsyslog  # Force immediate rotation
```

**Permanent Fix:**
```bash
# Enable log rotation
cat /etc/logrotate.conf | grep -i "daily\|weekly"

# Or configure disk usage limits
df -h /var
# If /var is separate filesystem and filling, mount a larger one
```

---

### Scenario 2: Disk Shows Full But du Doesn't Account For It

**Problem:** `df -h /` shows 85% full but `du -sh /*` only adds up to 70%. Where's the missing 15%?

**Investigation:**
```bash
# 1. Get exact numbers
DF_SIZE=$(df /var | tail -1 | awk '{print $2}')
DF_USED=$(df /var | tail -1 | awk '{print $3}')
DU_SIZE=$(du -sh /var | awk '{print $1}')

echo "df shows: $DF_USED used out of $DF_SIZE"
echo "du shows: $DU_SIZE"

# 2. Difference = something not visible to du
# Common causes:
#   - Deleted files still held open by processes
#   - Snapshots or copy-on-write
#   - Swap or temporary allocations
#   - Sparse files

# 3. Check for deleted files still in use
lsof +D /var | grep deleted | head -10

# Example output:
# nginx    2847  www-data   10r  REG  253,0   1024000  1234567 /var/log/access.log (deleted)

# 4. If found, restart the service to close the file
systemctl restart service_name

# 5. Recheck df
df -h /var

# 6. Check for open files by any process
lsof | grep /var | grep -v "cwd\|rtd\|txt" | wc -l

# 7. Look for hidden snapshot/compression
ls -la /var/.snapshot  # snapshots
stat /var | grep -i block

# 8. Check inode usage separately from block usage
df -i /var  # Inode usage
```

**Most Common Culprit:**
```bash
# Log files deleted but still held open by running process
# Example:
# 1. Application writes to /var/log/app.log
# 2. Administrator deletes: rm /var/log/app.log
# 3. Application still has file open, keeps writing
# 4. Disk space not freed until process closes the file

# Fix:
ps aux | grep -i app  # Find the process
systemctl restart app  # Restart it
# or
kill -HUP PID  # If it handles SIGHUP for log rotation
```

---

### Scenario 3: Inode Exhaustion - "No Space Left" But Disk has Space

**Problem:** Getting "No space left on device" error but `df -h` shows space available. Inodes are exhausted.

**Investigation:**
```bash
# 1. Check block usage (space)
df -h /var
# Shows: 40% full, plenty of space

# 2. Check inode usage
df -i /var
# Shows: 95% full (inodes used)

# Expected output:
# Filesystem    Inodes   IUsed   IFree IUse% Mounted on
# /dev/sda2     2000000  1900000  100000  95%  /var

# 3. Find directories with most inodes
find /var -type f | wc -l  # Total files

# 4. Count by directory
for dir in /var/*/; do echo "$(find "$dir" -type f | wc -l) $dir"; done | sort -rn | head -10

# 5. Most likely culprits: temp files, cache, old uploads
ls -la /var/cache | head -10
ls -la /var/spool | head -10
du --inodes /var/* | sort -rn | head -10  # If available
```

**Fix (Clean Up Inodes):**
```bash
# Delete unnecessary files
find /var/cache -type f -mtime +30 -delete
find /var/spool -type f -empty -delete
find /var/tmp -type f -mtime +7 -delete

# Or if you have many small files:
# Consolidate them (e.g., gzip log files)
find /var/log -name "*.log" -size -1M -mtime +30 -exec gzip {} \;

# Recheck
df -i /var
```

**Permanent Fix:**
```bash
# Reduce number of retained files
# For mail server:
find /var/mail -type f -mtime +90 -delete

# For web server:
find /var/www -name "*.tmp" -delete

# Use log rotation more aggressively
systemctl restart logrotate
```

---

### Scenario 4: Growing Directory - Identify & Stop The Growth

**Problem:** `/home` directory grows 5GB per day. Need to find and stop the culprit.

**Investigation:**
```bash
# 1. Baseline measurement
DU_BASELINE=$(du -sh /home | awk '{print $1}')
echo "Current size: $DU_BASELINE"

# 2. Monitor overnight (cron job)
# Create a script:
echo "Date: $(date)" >> /tmp/home_growth.log
du -sh /home >> /tmp/home_growth.log

# 3. Next day, check growth rate
tail /tmp/home_growth.log

# 4. Find what's growing fastest
# Method 1: Recent files
find /home -type f -mtime -1 -size +100M | sort -k7 -rn | head -10

# Method 2: Recent directories
for user in /home/*/; do
  echo "$(du -sh "$user" 2>/dev/null) $(basename $user)"
done | sort -h | tail -5

# Method 3: Which file types
find /home -type f -mtime -1 | xargs file | awk -F: '{print $2}' | sort | uniq -c | sort -rn

# 5. Check if it's a backup or cache
du -sh /home/*/Downloads
du -sh /home/*/.cache
du -sh /home/*/snap  # snap package cache
```

**Common Culprits:**
```bash
# Backups
ls -la /home/backups/
find /home/backups -mtime -1 -ls

# Package managers
ls -la /home/*/snap
du -sh /home/*/snap/*

# Caches
find /home -name ".cache" -type d -exec du -sh {} \;

# Application data
du -sh /home/*/.config
du -sh /home/*/.local

# Log files
find /home -name "*.log" -mtime -1 -ls
```

**Stop The Growth:**
```bash
# Identify the culprit process/user
ps aux | grep -i backup  # Check if backup running
systemctl --user status cache_service  # Check user services

# If it's a legitimate backup:
# Redirect to different filesystem with more space
# Or add exclusions to backup script

# If it's runaway process:
kill -9 PID
systemctl restart service_name
```

---

### Scenario 5: Database Files Consuming All Space

**Problem:** PostgreSQL/MySQL database directory is consuming 80% of disk. Emergency action needed.

**Investigation:**
```bash
# 1. Confirm database location
du -sh /var/lib/postgresql/*  # PostgreSQL
du -sh /var/lib/mysql/*        # MySQL

# 2. Check individual databases
sudo -u postgres psql -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;"

# MySQL:
mysql -e "SELECT table_schema, SUM(data_length + index_length) AS size FROM information_schema.tables GROUP BY table_schema ORDER BY size DESC;"

# 3. Check for large temp files
ls -lha /var/lib/postgresql/pg_tblspc/

# 4. Check transaction logs (WAL) taking space
du -sh /var/lib/postgresql/wal  # Or pg_wal

# 5. Check if bloat from deleted data
# PostgreSQL: check table bloat
select schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) from pg_tables where schemaname != 'information_schema' order by pg_total_relation_size(schemaname||'.'||tablename) desc;
```

**Emergency Actions (Careful!):**
```bash
# Option 1: Vacuum (reclaim space from deleted rows)
sudo -u postgres vacuumdb -d production_db

# Option 2: Archive old data
# First, identify old data
psql -d production_db -c "SELECT COUNT(*) FROM events WHERE created_at < NOW() - INTERVAL '90 days';"

# Then archive or delete
psql -d production_db -c "DELETE FROM events WHERE created_at < NOW() - INTERVAL '90 days';"

# Option 3: Add more disk space (long-term)
# Mount new filesystem, move database
```

**Monitoring to Prevent:**
```bash
# Add cron job
0 2 * * * du -sh /var/lib/postgresql >> /var/log/db_size.log

# Alert if growing > 1GB/day
0 3 * * * [ $(stat -c%s /var/lib/postgresql) -gt $THRESHOLD ] && send_alert
```

---

### Scenario 6: Interview Scenario - "Disk Full - What's Your Debugging Process?"

**Ideal Answer (step-by-step):**

```bash
# Step 1: Understand the scope
df -h  # See which filesystem is full

# Step 2: Get directory breakdown
du -sh /* | sort -h | tail -10

# Step 3: Drill into largest
du -sh /var/* | sort -h | tail -10

# Step 4: Drill deeper
du -sh /var/log/* | sort -h | tail -10

# Step 5: Find recent large files
find /var/log -type f -mtime -1 -size +100M

# Step 6: Check for deleted but open files
lsof | grep deleted | head -5

# Step 7: Take action
# If logs: rotate or delete old ones
# If deleted but open: restart the service
# If legitimate: add more disk space

# Step 8: Verify fix
df -h
```

**Interview Tips:**
- "Start broad (df -h) then narrow down"
- "Always check for deleted files held open"
- "Distinguish between space and inode exhaustion"
- "Test with find/ls before deleting in bulk"
- "Document what you deleted"

