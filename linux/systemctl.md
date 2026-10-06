# systemctl - Service Management

`systemctl` manages systemd services on modern Linux systems. Essential for starting/stopping services, checking status, and understanding service dependencies.

## Quick Reference

```bash
systemctl start service_name              # Start service
systemctl stop service_name               # Stop service
systemctl restart service_name            # Restart service
systemctl status service_name             # Check status
systemctl enable service_name             # Enable at boot
systemctl disable service_name            # Disable at boot
systemctl is-active service_name          # Check if running
systemctl list-units --type=service       # List all services
systemctl --failed                        # Show failed services
```

---

## Understanding Status Output

```bash
$ systemctl status nginx

● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2026-10-05 10:20:00 UTC; 2h 30min ago
   Docs: man:nginx(8)
  Process: 2847 ExecStart=/usr/sbin/nginx -g daemon off; (code=exited, status=0/SUCCESS)
 Main PID: 2848 (nginx)
    Tasks: 10
   Memory: 25.3M
   CGroup: /system.slice/nginx.service
           ├─2848 nginx: master process /usr/sbin/nginx -g daemon off;
           ├─2849 nginx: worker process
           └─2850 nginx: worker process
```

| Section | Meaning |
|---------|---------|
| **Loaded** | Unit file path and enabled/disabled status |
| **Active** | Running, failed, inactive, etc. |
| **Docs** | Documentation reference |
| **Process** | Last executed process and exit code |
| **Main PID** | Process ID of the service |
| **Memory** | Memory used |
| **CGroup** | Child processes in control group |

---

## Critical Interview Scenarios

### 1. "Service won't start. Debug the issue."

```bash
# Step 1: Check status
systemctl status servicename

# Step 2: Look for error messages (last few lines of output)

# Step 3: Check logs
journalctl -u servicename -n 50

# Step 4: Check if it's a dependency issue
systemctl list-dependencies servicename

# Step 5: Try starting manually with verbose output
systemctl start servicename -v
```

### 2. "Service keeps crashing. Find why."

```bash
# Check recent logs
journalctl -u servicename --since "10 minutes ago"

# Or more detail
journalctl -u servicename -n 100

# Check for repeated failures
systemctl status servicename

# Look at restart count and timestamps
```

### 3. "Enable service to start at boot"

```bash
# Enable
systemctl enable servicename

# Verify
systemctl is-enabled servicename

# Check if it starts on reboot
systemctl list-unit-files | grep servicename
```

### 4. "Find failed services"

```bash
# Show all failed
systemctl --failed

# Or specific
systemctl status --state=failed

# Then debug each
systemctl status failed_service
journalctl -u failed_service -n 50
```

---

## Service Lifecycle

### Starting a Service

```bash
# Simple start
systemctl start nginx

# Start and enable (at boot)
systemctl enable --now nginx

# Start with verbose output
systemctl start nginx -v
```

### Stopping a Service

```bash
# Graceful stop
systemctl stop nginx

# Forceful stop (after timeout)
systemctl kill -s KILL nginx

# Disable at boot
systemctl disable nginx
```

### Restarting

```bash
# Full restart (stop + start)
systemctl restart nginx

# Reload config without stopping
systemctl reload nginx

# Reload or restart (try reload first)
systemctl reload-or-restart nginx
```

---

## Checking Status

### Single Service

```bash
systemctl status nginx

# Just yes/no (exit code)
systemctl is-active nginx

# Exit codes: 0=active, 3=inactive, 4=error
```

### All Services

```bash
# Running services
systemctl list-units --state=running

# Failed services
systemctl --failed

# Services by type
systemctl list-units --type=service
```

### Enable/Disable at Boot

```bash
# Enable
systemctl enable nginx

# Disable
systemctl disable nginx

# Check status
systemctl is-enabled nginx

# List all enabled
systemctl list-unit-files --state=enabled
```

---

## Dependency Management

### Find Dependencies

```bash
# What this service depends on
systemctl list-dependencies nginx

# What depends on this service
systemctl list-dependencies --reverse nginx
```

### Example Output

```
nginx.service
├─system-getty.slice
├─systemd-journald.socket
└─network-online.target
  └─cloud-final.service
```

---

## Logging (journalctl Integration)

### View Service Logs

```bash
# Last 50 lines
journalctl -u servicename -n 50

# Follow (like tail -f)
journalctl -u servicename -f

# Last hour
journalctl -u servicename --since "1 hour ago"

# Last boot
journalctl -u servicename -b

# Errors only
journalctl -u servicename -p err
```

---

## Real-World Scenarios

### Application won't start on boot (enabled but not running)

```bash
# Check if it's actually enabled
systemctl is-enabled myapp

# Check boot logs
journalctl -u myapp -b

# Check if dependencies are met
systemctl list-dependencies myapp

# Check service file
systemctl cat myapp

# Start manually for debugging
systemctl start myapp -v
systemctl status myapp
```

### Service crashes frequently

```bash
# Check restart count
systemctl status myapp

# See recent logs
journalctl -u myapp -n 100

# Check for resource constraints
systemctl status myapp | grep Memory

# See if it's hitting limits
journalctl -u myapp | grep -i "killed\|oom\|limit"
```

### Port already in use error

```bash
# Service fails to bind
journalctl -u myapp | grep -i "address in use"

# Find what's using the port
lsof -i :8080

# Kill or reconfigure
```

---

## Important Flags

| Flag | Meaning |
|------|---------|
| **-u SERVICE** | Show logs for specific service |
| **-n N** | Show last N lines |
| **-f** | Follow (continuous) |
| **-b** | Since last boot |
| **-p LEVEL** | Priority: err, warn, info, debug |
| **--since TIME** | Since timestamp |
| **--until TIME** | Until timestamp |
| **--failed** | Show failed units |
| **--state=STATE** | Filter by state |

---

## Service States

| State | Meaning |
|-------|---------|
| **active (running)** | Service is running |
| **active (exited)** | Service ran once and exited normally |
| **inactive** | Service is stopped |
| **failed** | Service failed to start or crashed |
| **activating** | Service is starting |
| **deactivating** | Service is stopping |
| **reloading** | Service is reloading configuration |

---

## Interview Tips

1. **Know the basic workflow:** start → stop → restart → enable → disable
2. **Know how to check logs:** `journalctl -u servicename -n 50`
3. **Know how to debug failures:** status → logs → dependencies
4. **Know the difference:** start (now) vs enable (at boot)
5. **Know exit codes:** 0=active, 3=inactive (useful for scripting)

---

## SRE Troubleshooting Scenarios

### Scenario 1: Service Failed on Boot - Production Outage

**Problem:** Server rebooted, service didn't start. Users report app down.

**Emergency Debugging Steps:**
```bash
# 1. Check service status
systemctl status problematic_service

# 2. Check last boot logs
journalctl -u problematic_service -b

# 3. Look for specific errors (last 20 lines)
journalctl -u problematic_service -n 20 --no-pager

# 4. Check if it's a dependency issue
systemctl list-dependencies problematic_service

# 5. Check if dependencies are running
systemctl status $(systemctl list-dependencies --plain problematic_service)

# 6. Try starting manually for detailed errors
systemctl start problematic_service -v
systemctl status problematic_service
```

**Common Root Causes:**
- Missing dependency: check `systemctl list-dependencies`
- Config file missing/broken: check `systemctl cat service | grep ExecStart`
- Port already in use: `lsof -i :PORT_NUMBER`
- Filesystem not mounted: check `/etc/systemd/system/*.mount` files

**Fix Strategy:**
1. Start a single dependency at a time to isolate
2. Check service ExecStart command for errors
3. Run the command manually to see real error
4. Fix and enable: `systemctl enable --now service_name`

---

### Scenario 2: Service Keeps Crashing - Restart Loop

**Problem:** Service shows "Active (exited)" or keeps restarting. Check `journalctl` shows repeated failures.

**Investigation:**
```bash
# 1. Check restart count and state
systemctl status service_name

# 2. Check restart policy
systemctl cat service_name | grep -i restart

# 3. Check recent failures
journalctl -u service_name --since "1 hour ago" | head -50

# 4. Look for specific error patterns
journalctl -u service_name | grep -i "error\|fail\|exit"

# 5. Check if resource limits are causing crashes
journalctl -u service_name | grep -i "memory\|oom\|killed"

# 6. Check exit code
systemctl status service_name | grep "ExecStart\|code="
```

**Analysis Examples:**
```bash
# Exit code 1 usually means application error
# Exit code 127 means command not found
# Exit code 137 means killed (likely OOMKilled)
# Exit code 143 means SIGTERM (graceful shutdown)

# Example: check which app is causing issue
journalctl -u myapp | grep -E "core dump|segmentation|signal"
```

**Fix:**
```bash
# 1. Check app logs for root cause
systemctl status myapp | grep "ExecStart"
# Run that command manually to see error

# 2. Check if it's a resource limit
systemctl show -p MemoryMax myapp

# 3. Increase limits if needed
systemctl edit myapp
# Add: MemoryMax=2G (or adjust)

# 4. Restart and monitor
systemctl restart myapp
journalctl -u myapp -f  # Follow logs
```

---

### Scenario 3: Service Running But Not Responding

**Problem:** `systemctl status` shows "active (running)" but application doesn't respond to requests.

**Diagnosis Steps:**
```bash
# 1. Verify it's actually running
systemctl is-active service_name
echo $?  # Should be 0

# 2. Check the actual process
systemctl status service_name | grep "Main PID"

# 3. Verify process exists
ps -p PID_FROM_ABOVE

# 4. Check if listening on expected port
lsof -i -p PID_FROM_ABOVE

# 5. Test connectivity
curl localhost:PORT  # If HTTP
nc -zv localhost:PORT  # Generic port test

# 6. Check application logs
journalctl -u service_name -f  # Follow in real-time

# 7. Check for resource issues
systemctl status service_name | grep -E "Memory|CPU"
```

**Likely Causes:**
- Deadlocked thread: process running but hung
- Port not listening: check app logs
- Configuration error: app starts but fails silently
- Waiting for external service: check dependencies

**Recovery:**
```bash
# If deadlocked, restart
systemctl restart service_name

# If port conflict, find what's using it
lsof -i :PORT_NUMBER | grep -v service_name
kill PID_OF_OTHER_SERVICE

# If config issue, check:
systemctl cat service_name
# Look at ExecStart command, working directory, environment
```

---

### Scenario 4: Can't Stop Service - Hangs on Stop

**Problem:** `systemctl stop service_name` takes forever and times out.

**Investigation:**
```bash
# 1. Check if it's actually stopping or stuck
systemctl status service_name &
sleep 15
systemctl status service_name | grep Active

# 2. Check the stop timeout
systemctl cat service_name | grep TimeoutStop

# 3. Check what processes are in the service
ps -ef | grep service_name

# 4. Check if process is in unusual state
ps aux | grep service_name | awk '{print "State:", $8}'

# 5. Check if it's waiting for I/O
lsof -p PID_OF_SERVICE | grep -i disk

# 6. Check systemd logs for why it won't stop
journalctl -u service_name | grep -i "stop\|terminate"
```

**Process States Explained:**
- **S** = sleeping (responsive, will stop)
- **D** = uninterruptible sleep (waiting for disk, will NOT respond to signals)
- **T** = stopped (paused, should respond)
- **Z** = zombie (dead but parent hasn't cleaned up)

**Force Stop (if needed):**
```bash
# Wait for graceful timeout first
systemctl stop service_name

# If hangs > 90 seconds, force kill
systemctl kill -s SIGKILL service_name

# Verify it's gone
systemctl status service_name

# Check if zombie processes remain
ps aux | grep service_name | grep Z

# If zombies, restart parent
systemctl restart parent_service_if_applicable
```

**Prevention:**
Increase timeout in service file:
```bash
systemctl edit service_name
# Add: TimeoutStopSec=300  (increased from default)
```

---

### Scenario 5: Service Won't Start - Port Already in Use

**Problem:** Service fails to start with "Address already in use" or similar error.

**Quick Diagnosis:**
```bash
# 1. Get the error
systemctl status service_name
journalctl -u service_name -n 10

# 2. Find what port it wants
systemctl cat service_name | grep -i port
# Or check app logs:
journalctl -u service_name | grep -i "bind\|port"

# 3. Find what's using that port
PORT=8080  # From error above
lsof -i :$PORT

# 4. Check if it's the old service still running
ps aux | grep service_name

# 5. Check if socket file exists (for unix sockets)
ls -la /var/run/service_name.sock
```

**Resolution:**
```bash
# Option 1: Stop the conflicting service
systemctl stop conflicting_service

# Option 2: Kill the process using the port
PID=$(lsof -t -i :PORT)
kill -9 $PID

# Option 3: Reconfigure service to use different port
systemctl edit service_name
# Update port configuration

# Then restart
systemctl restart service_name

# Verify it's listening
lsof -i :PORT
```

---

### Scenario 6: Service Fails at Specific Time Daily

**Problem:** Service crashes every day at 2 AM, comes back automatically.

**Investigation Pattern:**
```bash
# 1. Check what happens at that time
journalctl -u service_name --since "2026-10-06 01:50" --until "2026-10-06 02:10"

# 2. Look for patterns (cron jobs, backups, other services stopping)
journalctl --since "2026-10-06 01:50" --until "2026-10-06 02:10" | grep -E "RESTART|backup|cron|stopped"

# 3. Check if another service is stopping this one
systemctl list-dependencies service_name

# 4. Check for resource spikes at that time
journalctl --since "2 days ago" | grep "2 AM" | head -50

# 5. Check system logs for OOM killer
journalctl --since "2 days ago" | grep -i oom

# 6. Check if log rotation is killing the service
ls -la /etc/logrotate.d/service_name
```

**Common Causes:**
- Backup job consuming all resources → service OOMKilled
- Log rotation killing process
- Cron job running heavy task → starves service
- Database maintenance/vacuum at fixed time

**Fix:**
```bash
# If backup job: reschedule to off-peak
# If log rotation: add 'copytruncate' to avoid restart
# If cron job: move to different time or nice it

# Monitor to confirm fix:
journalctl -u service_name -f
```

---

### Scenario 7: Enable Service at Boot - Verify It Works

**Problem:** You enable a service but it doesn't start on next boot. Need to verify before reboot.

**Verification Steps:**
```bash
# 1. Confirm it's enabled
systemctl is-enabled service_name
echo $?  # 0 = enabled, 1 = disabled

# 2. Check enable status with list-unit-files
systemctl list-unit-files | grep service_name

# 3. Simulate boot (start the service cold)
systemctl stop service_name
systemctl start service_name

# 4. Check if it started successfully
systemctl is-active service_name

# 5. Check logs for boot startup
journalctl -u service_name -b

# 6. Verify all dependencies would be available at boot
systemctl list-dependencies service_name
systemctl status $(systemctl list-dependencies --plain service_name)
```

**Test Boot Sequence:**
```bash
# Schedule for next reboot
systemctl enable service_name
systemctl set-default multi-user.target  # Ensure it boots headless

# Create test check
echo "systemctl is-active service_name && echo 'OK' || echo 'FAILED'" > /etc/init.d/test_service.sh

# After reboot, check if service is running
systemctl status service_name
```

---

### Scenario 8: Interview Scenario - "Debug Service Start Failure in 10 Minutes"

**Time: 10 minutes - Here's what you'd do:**

```bash
# MINUTE 1-2: Understand the problem
systemctl status failing_service
journalctl -u failing_service -n 20 --no-pager

# MINUTE 3-4: Check dependencies
systemctl list-dependencies failing_service
systemctl status $(systemctl list-dependencies --plain failing_service)

# MINUTE 5-6: Check service file configuration
systemctl cat failing_service

# MINUTE 7-8: Try manual start with verbose output
systemctl start failing_service -v
journalctl -u failing_service -n 5

# MINUTE 9-10: Propose solution
# "The issue is [specific error]. Solution: [restart dependency / fix config / check logs]"
```

**Key Interview Points:**
- **Don't panic:** systemctl gives clear error messages
- **Follow the chain:** status → logs → dependencies → manual test
- **Show your thinking:** "I'm checking X because Y"
- **Be specific:** quote the actual error message

