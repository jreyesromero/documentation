# awk & sed - Text Processing

`awk` and `sed` are powerful tools for parsing and transforming text. Critical for extracting data from logs, processing structured text, and data analysis.

## Quick Reference

```bash
# sed - stream editor (find and replace)
sed 's/old/new/' file.txt                    # Replace first occurrence per line
sed 's/old/new/g' file.txt                   # Replace all occurrences
sed '10d' file.txt                           # Delete line 10
sed -n '5,10p' file.txt                      # Print lines 5-10 only
tail -f log.txt | sed 's/ERROR/[ERROR]/g'    # Real-time log highlighting

# awk - text processing (columns and patterns)
awk '{print $1, $3}' file.txt                # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd             # Use colon as delimiter
awk '$3 > 100 {print $0}' file.txt           # Print if column 3 > 100
awk '{sum += $1} END {print sum}' file.txt   # Sum a column
```

---

## sed - Stream Editor

### Syntax

```bash
sed [options] 'command' [file]
```

### Common Commands

| Command | Meaning |
|---------|---------|
| **s/old/new/** | Substitute (first per line) |
| **s/old/new/g** | Substitute (all) |
| **s/old/new/i** | Substitute (case-insensitive) |
| **d** | Delete |
| **p** | Print |
| **-i** | Edit in-place (modify file) |
| **-e** | Multiple expressions |

### Real Examples

**Replace string in file:**
```bash
sed 's/localhost/127.0.0.1/' config.txt

# Save changes (dangerous! use carefully)
sed -i 's/localhost/127.0.0.1/' config.txt
```

**Delete lines:**
```bash
# Delete line 5
sed '5d' file.txt

# Delete lines matching pattern
sed '/debug/d' file.txt

# Delete lines 10-20
sed '10,20d' file.txt
```

**Print specific lines:**
```bash
# Print lines 5-10 only
sed -n '5,10p' file.txt

# Print lines matching pattern
sed -n '/ERROR/p' file.txt
```

---

## awk - Text Processing

### Syntax

```bash
awk 'pattern { action }' file
```

### Core Concepts

- **Input:** Reads file line by line, splits into fields (columns)
- **Fields:** `$1` = first column, `$2` = second, `$0` = entire line
- **Separators:** Default is whitespace, change with `-F`

### Real Examples

**Extract specific columns:**
```bash
# Print 1st and 3rd columns
awk '{print $1, $3}' file.txt

# Print 2nd column from colon-separated file
awk -F: '{print $2}' /etc/passwd
```

**Filter by condition:**
```bash
# Print lines where column 3 > 100
awk '$3 > 100 {print $0}' data.txt

# Print if column 1 equals "ERROR"
awk '$1 == "ERROR" {print $0}' log.txt

# Print if column 2 contains "active"
awk '$2 ~ /active/ {print $0}' status.txt
```

**Calculate statistics:**
```bash
# Sum column 1
awk '{sum += $1} END {print sum}' numbers.txt

# Average column 1
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# Count lines
awk 'END {print NR}' file.txt

# Count matching lines
awk '/ERROR/ {count++} END {print count}' log.txt
```

---

## Critical Interview Scenarios

### 1. "Parse access logs and find top IPs"

```bash
# Extract IP (first column) and count
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Output:
#  150 192.168.1.100
#  120 192.168.1.101
#   90 192.168.1.102
```

### 2. "Extract data from CSV file"

```bash
# CSV with comma delimiter
awk -F, '{print $1, $3}' data.csv

# Filter rows
awk -F, '$2 > 1000 {print $1, $2}' data.csv
```

### 3. "Parse log file and find errors"

```bash
# Find ERROR lines and extract timestamp + message
awk '/ERROR/ {print $1, $2, $3, $0}' app.log

# Find ERROR lines and count
awk '/ERROR/ {count++} END {print count}' app.log
```

### 4. "Extract usernames from /etc/passwd"

```bash
# Colon-separated: user:password:uid:gid:...
awk -F: '{print $1}' /etc/passwd

# Get specific users with UID > 1000 (regular users)
awk -F: '$3 > 1000 {print $1}' /etc/passwd
```

### 5. "Combine awk with grep in a pipeline"

```bash
# Find ERROR lines, extract just the message (last field)
grep ERROR app.log | awk '{print $NF}'

# Find ERROR lines, extract timestamp and count
grep ERROR app.log | awk '{print $1}' | sort | uniq -c

# Real-time: tail + grep + awk
tail -f app.log | grep ERROR | awk '{print $0}'
```

---

## Advanced Patterns

### Multiple conditions

```bash
# AND condition
awk '$1 == "ERROR" && $2 > 100 {print $0}' log.txt

# OR condition
awk '$1 == "ERROR" || $1 == "WARN" {print $0}' log.txt
```

### String matching

```bash
# Contains (regex ~)
awk '$2 ~ /error/ {print $0}' file.txt

# Doesn't contain (!~)
awk '$2 !~ /debug/ {print $0}' file.txt
```

### Field manipulation

```bash
# Replace field value
awk '{$2 = "REDACTED"; print}' file.txt

# Add computed field
awk '{print $0, $1 * $2}' file.txt
```

### Useful Variables

| Variable | Meaning |
|----------|---------|
| **NR** | Number of records (line number) |
| **NF** | Number of fields (columns) |
| **$0** | Entire line |
| **$1, $2, ...** | Individual fields |
| **-F** | Field separator |
| **BEGIN** | Execute before reading file |
| **END** | Execute after reading file |

---

## Real-World Examples

### Analyze web server logs

```bash
# Get top 10 most requested URLs
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -10

# Get average response time
awk '{sum += $10; count++} END {print sum/count}' access.log

# Get status code distribution
awk '{print $9}' access.log | sort | uniq -c
```

### System monitoring

```bash
# Get processes using > 10% CPU
ps aux | awk '$3 > 10 {print $0}'

# Get users with > 1000 UID (non-system)
awk -F: '$3 > 1000 {print $1}' /etc/passwd

# Get mounted filesystems and usage
df -h | awk 'NR > 1 {print $1, $3, $4}'
```

### Log analysis

```bash
# Count log entries by hour
awk '{print $1}' access.log | cut -d: -f1 | sort | uniq -c

# Extract error messages and count
grep ERROR log.txt | awk '{print $(NF-1), $NF}' | sort | uniq -c

# Real-time error highlighting
tail -f app.log | sed 's/ERROR/\x1b[91mERROR\x1b[0m/g'
```

---

## Interview Tips

1. **Know the difference:** sed = find/replace, awk = column processing
2. **Know field extraction:** `awk '{print $1}'` for first column
3. **Know filtering:** `awk '$1 == "ERROR"'` for conditions
4. **Know piping:** Combine with grep, sort, uniq, wc for data analysis
5. **Know END block:** `awk 'END {print NR}'` for summaries
6. **Know -F flag:** `-F:` for different delimiters

---

## Comparison: sed vs awk

| Aspect | sed | awk |
|--------|-----|-----|
| **Purpose** | Find/replace | Text processing, columns |
| **Strength** | Simple substitutions | Data analysis, filtering |
| **Complexity** | Simple patterns | More powerful |
| **Use case** | Config file edits | Log analysis, data extraction |
| **Learning curve** | Easy | Moderate |

