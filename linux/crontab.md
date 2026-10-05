# crontab - Scheduled Tasks

`crontab` schedules jobs to run at specified times (backups, reports, maintenance tasks). Essential for understanding recurring tasks and automation.

## Quick Reference

```bash
crontab -l                              # List current cron jobs
crontab -e                              # Edit cron jobs
crontab -r                              # Remove all cron jobs
crontab -i                              # Prompt before removing
crontab file                            # Install crontab from file
```

---

## Crontab Format

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 7) (0 = Sunday, 7 = Sunday)
# │ │ │ │ │
# │ │ │ │ │
* * * * * /path/to/command
```

### Examples

```bash
# Every minute
* * * * * command

# Every hour at :00
0 * * * * command

# Every day at 2:30 AM
30 2 * * * command

# Every Monday at 9 AM
0 9 * * 1 command

# Every 1st day of month at midnight
0 0 1 * * command

# Every weekday (Mon-Fri) at 6 PM
0 18 * * 1-5 command

# Every 15 minutes
*/15 * * * * command

# Multiple times: 9 AM, 12 PM, 3 PM, 6 PM
0 9,12,15,18 * * * command
```

---

## Critical Interview Scenarios

### 1. "Backup job isn't running. Debug."

```bash
# Step 1: Check if job exists
crontab -l

# Step 2: Verify cron service is running
systemctl status cron  # or crond

# Step 3: Check cron logs
journalctl | grep -i cron

# Step 4: Check permissions
ls -l /etc/cron.d/backup_job

# Step 5: Test command manually
/path/to/backup/script.sh
```

### 2. "Job is running but not producing output"

```bash
# Edit crontab to capture output
crontab -e

# Add this to see errors:
0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1

# Check what happened
tail -f /var/log/backup.log
```

### 3. "Check what jobs are scheduled"

```bash
# Current user's jobs
crontab -l

# All users' jobs
for user in $(cut -f1 -d: /etc/passwd); do 
  echo "Crontab for $user:"
  crontab -u $user -l 2>/dev/null
done

# System-wide cron jobs
ls -la /etc/cron.d/
cat /etc/crontab
```

### 4. "Job runs but doesn't have permissions it needs"

```bash
# Problem: cron runs as the user, not with elevated privileges
# Solution 1: Use sudo in command
0 2 * * * sudo /path/to/backup.sh

# Solution 2: Run as root (add to root's crontab)
sudo crontab -e

# Solution 3: Use specific user (requires sudo)
0 2 * * * sudo -u backupuser /path/to/backup.sh
```

---

## Crontab Fields

| Field | Range | Meaning |
|-------|-------|---------|
| **Minute** | 0-59 | Minute of hour |
| **Hour** | 0-23 | Hour of day (24-hour) |
| **Day of Month** | 1-31 | Day |
| **Month** | 1-12 | Month (1=Jan, 12=Dec) |
| **Day of Week** | 0-7 | Day (0 and 7 = Sunday) |

---

## Special Syntax

| Syntax | Meaning |
|--------|---------|
| **\*** | Any value |
| **,** | List (e.g., 1,3,5) |
| **-** | Range (e.g., 1-5) |
| **/** | Step (e.g., */15 = every 15 mins) |
| **@yearly** | Once a year (Jan 1, 00:00) |
| **@monthly** | Once a month (1st, 00:00) |
| **@weekly** | Once a week (Sun, 00:00) |
| **@daily** | Once a day (00:00) |
| **@hourly** | Once an hour (:00) |
| **@reboot** | At system boot |

---

## Real-World Examples

### Daily backup at 2 AM

```bash
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

### Check database integrity every Monday at 10 PM

```bash
0 22 * * 1 /opt/db/integrity_check.sh
```

### Cleanup old logs every 1st of month

```bash
0 3 1 * * find /var/log -type f -mtime +30 -delete
```

### Run every 15 minutes

```bash
*/15 * * * * /opt/monitor/check_health.sh
```

### Restart service every Sunday at midnight

```bash
0 0 * * 0 systemctl restart myservice
```

### Multiple times per day

```bash
# Run at 9 AM, 12 PM, and 6 PM
0 9,12,18 * * * /path/to/task.sh
```

### Only on weekdays

```bash
# Run at 8 AM Monday-Friday
0 8 * * 1-5 /path/to/weekday_task.sh
```

---

## Troubleshooting

### Job not running

```bash
# 1. Verify job exists
crontab -l

# 2. Check cron service is running
systemctl status crond  # or cron

# 3. Check logs
journalctl -u cron  # or /var/log/cron

# 4. Verify command has full path
# ❌ DON'T: * * * * * backup.sh
# ✅ DO: * * * * * /opt/scripts/backup.sh

# 5. Check for syntax errors (edit and save)
crontab -e
```

### Job runs but fails silently

```bash
# Add logging to command
0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1

# Check output
tail /var/log/backup.log

# Or check for email (cron sends to MAILTO)
mail  # Read system mail
```

### Job doesn't have required environment

```bash
# Cron has minimal environment, add to crontab:
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@example.com

0 2 * * * /path/to/backup.sh
```

---

## Permissions and Locations

### User Crontabs

```bash
# User-specific crontabs
~/.crontab       # User's personal crontab
/var/spool/cron/crontabs/username
```

### System Crontabs

```bash
# System-wide jobs
/etc/crontab
/etc/cron.d/     # Directory of system cron jobs
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

---

## Interview Tips

1. **Know the format:** Minute, Hour, Day, Month, DayOfWeek
2. **Know `*/X` syntax:** `*/15` = every 15 minutes
3. **Know special strings:** `@hourly`, `@daily`, `@reboot`
4. **Know to capture output:** Use `>> /path/to/log.txt 2>&1`
5. **Know debugging:** `crontab -l` to verify, logs for troubleshooting
6. **Know permissions:** Cron runs as the user who owns the crontab

---

## Common Mistakes

❌ **Don't:** Use relative paths
```bash
* * * * * backup.sh  # WRONG
```

✅ **Do:** Use absolute paths
```bash
* * * * * /opt/scripts/backup.sh  # CORRECT
```

---

❌ **Don't:** Forget redirects (output lost)
```bash
0 2 * * * /path/to/backup.sh  # Output disappears
```

✅ **Do:** Capture output
```bash
0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1
```

---

❌ **Don't:** Forget PATH in system crontabs
```bash
* * * * * find /var/log -delete  # Might fail
```

✅ **Do:** Use full path
```bash
* * * * * /usr/bin/find /var/log -delete  # Always works
```

