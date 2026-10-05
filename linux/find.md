# find - Search for Files

`find` searches for files and directories based on various criteria (name, type, size, modification time, permissions). Essential for locating config files, logs, and troubleshooting missing data.

## Quick Reference

```bash
find /path -name "*.log"              # Find by name pattern
find /path -type f                    # Find files (f=file, d=directory)
find /path -size +100M                # Files larger than 100MB
find /path -mtime -7                  # Modified in last 7 days
find /path -user username             # Files owned by user
find /path -name "*.log" -delete       # Delete matching files
find /path -name "*.log" -exec rm {} \;  # Execute command on results
```

---

## Core Concepts

### Syntax

```bash
find [starting-point] [options] [action]
```

### Common Search Criteria

| Criteria | Example | Meaning |
|----------|---------|---------|
| **-name** | -name "*.log" | Filename (glob pattern) |
| **-type** | -type f | File type (f=file, d=dir, l=link) |
| **-size** | -size +100M | File size (+ bigger, - smaller) |
| **-mtime** | -mtime -7 | Modified in last N days |
| **-user** | -user root | Files owned by user |
| **-perm** | -perm 777 | File permissions |
| **-iname** | -iname "*.LOG" | Case-insensitive name |
| **-newer** | -newer /tmp/marker | Modified after another file |

### Common Actions

| Action | Effect |
|--------|--------|
| **-print** | Print results (default) |
| **-delete** | Delete matching files |
| **-exec** | Execute command on each result |
| **-ok** | Execute command (prompt first) |

---

## Real Interview Scenarios

### 1. "Find all .log files modified in last 7 days"

```bash
find /var/log -name "*.log" -mtime -7
```

Breaking it down:
- `/var/log` — start in this directory
- `-name "*.log"` — filename ends with .log
- `-mtime -7` — modified in last 7 days (- means within, + means older)

### 2. "Clean up old logs to free disk space"

```bash
# First, see what would be deleted
find /var/log -name "*.log" -mtime +30

# Then delete
find /var/log -name "*.log" -mtime +30 -delete

# Or with exec (more control):
find /var/log -name "*.log" -mtime +30 -exec rm {} \;
```

### 3. "Find large files consuming disk space"

```bash
find / -type f -size +100M

# More practical (exclude system directories):
find /home /var /opt -type f -size +500M 2>/dev/null

# Find and show sizes
find / -type f -size +1G -exec du -h {} \; 2>/dev/null | sort -h
```

### 4. "Find files modified by specific user"

```bash
find /home -user julianre -type f -mtime -7
```

### 5. "Find and count files in a directory"

```bash
find /var/log -type f -name "*.log" | wc -l
```

---

## Advanced Scenarios

### Complex Pattern Matching

**Multiple extensions:**
```bash
find /var/log -type f \( -name "*.log" -o -name "*.txt" \)
```

**Exclude directories:**
```bash
find / -name node_modules -prune -o -name "*.js" -type f -print
```

**Find files with specific pattern in name:**
```bash
find . -name "*test*" -type f
```

### Permission-Based Searches

**Files with weak permissions (world-writable):**
```bash
find / -perm -002 -type f
```

**Files with specific owner and permissions:**
```bash
find / -user root -perm 644 -type f
```

### Time-Based Operations

**Files not accessed in 6 months:**
```bash
find /var -type f -atime +180
```

**Files modified TODAY:**
```bash
find /var/log -type f -mtime 0
```

**Files modified between two dates:**
```bash
find /var -type f -newer /tmp/start_date -o -newer /tmp/end_date
```

### Bulk Operations

**Delete files matching pattern (use carefully!):**
```bash
# Test first
find /tmp -name "*.tmp" -type f

# Then delete
find /tmp -name "*.tmp" -type f -delete
```

**Move files to archive:**
```bash
find /var/log -name "*.log" -mtime +30 -exec mv {} /archive/ \;
```

**Compress old files:**
```bash
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;
```

---

## Common Interview Questions

**Q: How would you find files larger than 1GB?**
A: `find / -type f -size +1G`

**Q: How would you find files NOT modified in 30 days?**
A: `find / -type f -mtime +30`

**Q: How would you find and delete all .tmp files safely?**
A: First preview with `find /path -name "*.tmp"`, then use `find /path -name "*.tmp" -delete`

**Q: Find files owned by user AND larger than 100MB?**
A: `find /home -user username -type f -size +100M`

---

## Important Flags

| Flag | Meaning | Example |
|------|---------|---------|
| **-print** | Print results (default) | find /path -name "*.log" |
| **-type f** | Files only (not directories) | find / -type f -size +1G |
| **-type d** | Directories only | find / -type d -name backup |
| **-mtime** | Modified time (days) | find / -mtime -7 (last 7 days) |
| **-atime** | Access time (days) | find / -atime +30 (30+ days) |
| **-ctime** | Change time (days) | find / -ctime -1 (last 24 hours) |
| **-prune** | Don't descend into directory | find / -name node_modules -prune -o ... |
| **-exec** | Execute command on result | find / -name "*.log" -exec rm {} \; |

---

## Performance Tips

1. **Start with narrow paths:** 
   ```bash
   find /var/log -name "*.log"  # Better than find / -name "*.log"
   ```

2. **Use -type early:**
   ```bash
   find /path -type f -name "*.log"  # Faster than find /path -name "*.log" -type f
   ```

3. **Limit recursion depth:**
   ```bash
   find /path -maxdepth 3 -name "*.log"  # Only 3 levels deep
   ```

4. **Exclude slow filesystems:**
   ```bash
   find /path -name "*.log" 2>/dev/null  # Suppress permission errors
   ```

5. **Use -print0 with xargs for special characters:**
   ```bash
   find /path -name "*.log" -print0 | xargs -0 rm
   ```

---

## Real-World Examples

### Clean up application logs older than 30 days
```bash
find /var/log/myapp -name "*.log" -mtime +30 -delete
```

### Find recently modified config files
```bash
find /etc -type f -mtime -1
```

### Find all temporary files
```bash
find /tmp /var/tmp -type f -atime +7 -delete
```

### Locate configuration files by pattern
```bash
find /etc -name "*nginx*" -o -name "*apache*"
```

### Find and fix permissions
```bash
find /var/www -type f -perm 777 -exec chmod 644 {} \;
```

