# lsof - List Open Files

`lsof` lists all open files and network connections by running processes. Critical for troubleshooting port conflicts, finding which process is using a file, and network debugging.

## Quick Reference

```bash
lsof                       # All open files (very long)
lsof -i                    # All network connections
lsof -i :8080              # Process using port 8080
lsof -u username           # Files opened by user
lsof -p PID                # Files opened by process
lsof +D /path              # All files in directory
lsof | grep filename       # Find who has file open
```

---

## Understanding Output

```
COMMAND     PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
rapportd   2475 julianre   10u  IPv4 0x77e5c9b2   0t0  TCP *:49157 (LISTEN)
rapportd   2475 julianre   11u  IPv6 0xeaf447ee   0t0  TCP *:49157 (LISTEN)
ControlCe  2490 julianre    9u  IPv4 0xb611ac58   0t0  TCP *:7000 (LISTEN)
Microsoft  2762 julianre   20u  IPv4 0x74f49789   0t0  TCP 10.41.56.74:50650->23.103.239.193:443 (ESTABLISHED)
```

| Column | Meaning |
|--------|---------|
| **COMMAND** | Process name |
| **PID** | Process ID |
| **USER** | User running the process |
| **FD** | File descriptor (0=stdin, 1=stdout, 2=stderr, 3+=files/sockets) |
| **TYPE** | IPv4, IPv6, CHR (character), REG (regular file), DIR (directory) |
| **DEVICE** | Device identifier |
| **SIZE/OFF** | File size or offset |
| **NODE** | Socket protocol or file inode |
| **NAME** | File path or network address |

---

## Critical Interview Scenarios

### 1. "Port 8080 is already in use. Find what's using it."

```bash
lsof -i :8080
```

Output:
```
COMMAND   PID     USER   FD  TYPE DEVICE SIZE/OFF NODE NAME
apache2  2490 www-data   10u  IPv4 12345   0t0  TCP *:8080 (LISTEN)
```

**Answer:** "I'd use `lsof -i :8080` to find the process using that port, then decide whether to kill it or change my port."

### 2. "Can't delete a file. Which process is using it?"

```bash
lsof | grep filename
# or
lsof +D /path/to/directory
```

Then kill or close the process holding the file.

### 3. "Why can't I umount the filesystem?"

```bash
# Find what's using files in that filesystem
lsof +D /path/to/mount

# Kill the process or close files
```

### 4. "Show me all network connections from a process"

```bash
lsof -p PID | grep -i TCP
```

### 5. "Find all files opened by a user"

```bash
lsof -u username
```

---

## Network Troubleshooting

### Find listening ports

```bash
lsof -i -P -n | grep LISTEN
```

Breaking it down:
- `-i` — network connections only
- `-P` — don't convert port numbers to names (faster)
- `-n` — don't convert IP addresses to hostnames (faster)
- `LISTEN` — only listening sockets

### Check specific port

```bash
lsof -i :3306

# More detailed (shows IP and port clearly)
lsof -i -P -n | grep :3306
```

### Find all IPv4 connections to specific IP

```bash
lsof -i @10.0.0.5
```

### Find all UDP traffic

```bash
lsof -i UDP
```

### Find all TCP traffic

```bash
lsof -i TCP
```

---

## Common Scenarios

### Application won't start (address already in use)

```bash
# Find what's using the port
lsof -i :5000

# Kill the process (if safe)
kill -9 PID

# Or find and kill in one command
kill -9 $(lsof -t -i :5000)
```

### Memory leak investigation

```bash
# Show all files opened by process
lsof -p PID

# Watch for increasing file descriptors
watch 'lsof -p PID | wc -l'
```

### Database connection issues

```bash
# Find all connections to database port
lsof -i :3306

# Count connections from specific client
lsof -i @192.168.1.100 | wc -l
```

### Debugging hung process

```bash
# See what files/sockets it has open
lsof -p PID

# Look for unusual open files or connections
```

---

## Useful Flags

| Flag | Meaning |
|------|---------|
| **-i** | Show network connections |
| **-P** | Numeric port numbers (no service names) |
| **-n** | Numeric IP addresses (no DNS) |
| **-u user** | Show files for specific user |
| **-p PID** | Show files for specific process |
| **-t** | Show only PIDs (terse) |
| **+D path** | Show files in directory (all subdirs) |
| **-a** | AND conditions (more specific) |
| **-s** | Show socket state (doesn't work on all systems) |

---

## Advanced Scenarios

### Find all processes with files in current directory

```bash
lsof +D .
```

### Kill all processes using a port (caution!)

```bash
kill -9 $(lsof -t -i :8080)
```

### Monitor file descriptors for a process

```bash
watch 'lsof -p PID | grep -E "REG|IPv4|IPv6" | wc -l'
```

### Find zombie processes (that still have open files)

```bash
lsof | grep -i zombie
```

### Find processes with many open files (potential leak)

```bash
lsof -a -u username | cut -d' ' -f1 | sort | uniq -c | sort -rn
```

---

## Performance Tips

- **Always use `-P` and `-n`:** Avoids DNS/service lookups, much faster
  ```bash
  lsof -i -P -n     # Fast
  lsof -i            # Slow (DNS lookups)
  ```

- **Use `-t` for scripting:** Returns only PIDs, useful for commands like `kill -9`
  ```bash
  kill -9 $(lsof -t -i :8080)
  ```

- **Combine with grep for filtering:**
  ```bash
  lsof -i -P -n | grep LISTEN
  ```

---

## Real-World Examples

### Find and kill hung process using a port

```bash
# Find the process
lsof -i :9000

# Kill it
kill -9 $(lsof -t -i :9000)
```

### Monitor if a service is leaking file descriptors

```bash
# Check baseline
lsof -p $(pgrep nginx) | wc -l

# Check again later
lsof -p $(pgrep nginx) | wc -l

# If growing, investigate for leak
```

### Find all database connections from app server

```bash
lsof -i @db-server.internal
```

### Debug permission denied errors

```bash
# See all files a process is trying to access
lsof -p PID | grep denied
```

---

## Interview Tips

1. **Know `-i` for network troubleshooting** — most common use in interviews
2. **Remember port syntax:** `lsof -i :PORT`
3. **Know how to extract PID:** `lsof -t -i :PORT`
4. **Use `-P -n` for speed** — shows you understand performance
5. **Combine with kill for real scenarios:** `kill $(lsof -t -i :8080)`

