# tail & grep - Log Analysis

`tail` displays the end of files (perfect for watching logs), and `grep` searches for text patterns. Together, they're essential for troubleshooting and log analysis—arguably the most-used commands in SRE work.

## Quick Reference

```bash
# tail - view end of file
tail -f logfile              # Follow (stream) new lines
tail -n 50 logfile           # Last 50 lines
tail -100 logfile            # Last 100 lines (shorthand)
tail -f logfile | grep ERROR # Stream and filter

# grep - search for patterns
grep "ERROR" logfile         # Find lines containing ERROR
grep -i "error" logfile      # Case-insensitive search
grep -c "ERROR" logfile      # Count matches
grep -n "ERROR" logfile      # Show line numbers
grep -v "DEBUG" logfile      # Exclude matches (inverse)
grep -A 5 "ERROR" logfile    # Show 5 lines after match
grep -B 5 "ERROR" logfile    # Show 5 lines before match
```

---

## tail - Follow File Changes

### Syntax

```bash
tail [options] [file]
```

### Common Options

| Option | Meaning |
|--------|---------|
| **-f** | Follow (keep reading as new data is appended) |
| **-F** | Follow with retry (reopen if file rotated) |
| **-n N** | Show last N lines (default 10) |
| **-c N** | Show last N bytes |
| **-s SEC** | Sleep N seconds between checks (with -f) |

### Real Examples

**Watch application logs live:**
```bash
tail -f /var/log/application.log
```

**Monitor multiple logs at once:**
```bash
tail -f /var/log/app.log /var/log/error.log
```

**Last 50 lines of a large logfile:**
```bash
tail -50 /var/log/syslog
```

**Watch logs from 5 minutes ago to now:**
```bash
tail -f /var/log/app.log --since "5 minutes ago"
```

---

## grep - Pattern Searching

### Syntax

```bash
grep [options] pattern [file]
```

### Critical Options

| Option | Meaning |
|--------|---------|
| **-i** | Case-insensitive |
| **-n** | Show line numbers |
| **-c** | Count matches |
| **-v** | Invert (show non-matching lines) |
| **-E** | Extended regex (use \|, +, ?, etc.) |
| **-A N** | Show N lines after match |
| **-B N** | Show N lines before match |
| **-C N** | Show N lines before and after (context) |
| **-l** | Show only filenames (not content) |
| **-r** | Recursive (search directories) |
| **-w** | Match whole words only |

### Real Examples

**Count ERROR lines:**
```bash
grep -c "ERROR" /var/log/app.log
```

**Find exceptions with context:**
```bash
grep -B 2 -A 10 "Exception" /var/log/app.log
```

**Case-insensitive search across multiple files:**
```bash
grep -ri "connection refused" /var/log/
```

**Find lines NOT containing a pattern:**
```bash
grep -v "DEBUG" /var/log/app.log
```

---

## Interview Scenarios

### 1. "The application crashed. Find the error in logs."

```bash
# Step 1: Check most recent logs
tail -100 /var/log/app.log

# Step 2: Search for errors
tail -100 /var/log/app.log | grep -i error

# Step 3: See context around error
tail -500 /var/log/app.log | grep -B 5 -A 10 "Exception"

# Step 4: Count how many errors
grep -c "ERROR" /var/log/app.log
```

**Expected answer:** "I'd use tail to see recent logs, grep to find errors, and -A/-B to see context around the error."

### 2. "Service keeps restarting. Show me the restart logs."

```bash
# Find restart messages
grep "Starting service" /var/log/app.log

# Count restarts in last hour
grep -c "Starting service" /var/log/app.log

# Find pattern that causes restart
grep -B 20 "Starting service" /var/log/app.log | grep "ERROR"
```

### 3. "Performance degradation. Find slow queries."

```bash
# Find queries taking >1 second
grep "duration:.*[0-9]\{4,\}ms" /var/log/database.log

# Or more clearly:
grep -E "duration:.*[5-9][0-9]{3}ms|duration:.*[0-9]{5}ms" /var/log/database.log

# Count slow queries
grep -c "slow_query" /var/log/database.log
```

---

## Common Combinations

### Tail + Grep (Real-time filtering)

```bash
tail -f /var/log/app.log | grep "ERROR"
```

Watch logs and highlight only errors.

### Grep + Wc (Count)

```bash
grep "OutOfMemory" /var/log/app.log | wc -l
```

Count how many OutOfMemory errors.

### Grep + Cut + Sort (Extract and analyze)

```bash
grep "Failed login" /var/log/auth.log | cut -d' ' -f5 | sort | uniq -c | sort -rn
```

Find which IPs have most failed logins.

### Tail + Grep + Awk (Real-time analysis)

```bash
tail -f /var/log/app.log | grep "request_duration" | awk -F'=' '{print $2}' | sort -n | tail -10
```

Show slowest 10 requests in real-time.

---

## Advanced Patterns

### Find errors in logs from today

```bash
grep "^$(date +%Y-%m-%d)" /var/log/app.log | grep -i error
```

### Find patterns with regex

```bash
grep -E "ERROR|WARN|CRITICAL" /var/log/app.log

grep -E "failed|denied|refused" /var/log/auth.log
```

### Extract specific fields

```bash
grep "request:" /var/log/app.log | grep -o "duration:[0-9]*ms"
```

### Find lines matching multiple patterns

```bash
grep "ERROR" /var/log/app.log | grep "database"

# Or with grep -E:
grep -E "ERROR.*database" /var/log/app.log
```

---

## Performance Tips

### With Large Files

- **Use grep first, then tail:** Grep is fast for filtering large files
  ```bash
  grep "ERROR" logfile | tail -50
  ```

- **Use -c to count before diving deep:**
  ```bash
  grep -c "ERROR" logfile  # See if it's worth investigating
  ```

- **Use -n for context, not -A/-B on huge results:**
  ```bash
  grep -n "ERROR" logfile | head -20
  ```

### Watching Logs Live

- **Limit output with grep:**
  ```bash
  tail -f /var/log/app.log | grep -v "DEBUG" | grep -v "INFO"
  ```

- **Use tail with sleep interval:**
  ```bash
  tail -f /var/log/app.log -s 5  # Check every 5 seconds instead of every line
  ```

---

## Interview Tips

1. **Always use -n with grep** to see line numbers for context
2. **Combine tail -f with grep** for real-time log monitoring
3. **Know the difference:** tail shows recent entries, grep searches all of it
4. **Remember context flags:** -A (after), -B (before), -C (both)
5. **Use grep -c for quick statistics** before deep diving
6. **Know basic regex:** grep -E for extended regex patterns

