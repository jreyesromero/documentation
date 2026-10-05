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

