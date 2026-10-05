# strace - System Call Tracing

`strace` traces system calls and signals made by a process. Advanced debugging tool for understanding what a process is doing at the kernel level.

## Quick Reference

```bash
strace command                          # Trace a command
strace -p PID                           # Trace running process
strace -e trace=open,read,write cmd     # Trace specific syscalls
strace -o output.txt command            # Save output to file
strace -c command                       # Summary statistics
strace -f command                       # Follow child processes
strace -e trace=network command         # Network calls only
```

---

## Understanding Output

```
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=38693, ...}) = 0
mmap(NULL, 38693, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f8b8c6e8000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
```

| Component | Meaning |
|-----------|---------|
| **Syscall** | System call name (openat, fstat, mmap, etc.) |
| **Arguments** | Parameters passed to the syscall |
| **Return value** | Result (file descriptor, 0 for success, -1 for error) |

---

## Critical Interview Scenarios

### 1. "Application hangs. Find what it's waiting for."

```bash
# Trace running process
strace -p PID

# Watch what syscalls it's making
# Look for blocked syscalls (no immediate return)
# e.g., select, poll, read, write stuck without returning

# Common signs:
# - Stuck on read() from socket/pipe
# - Stuck on semaphore operation
# - Stuck on file lock
```

### 2. "File permission denied but permissions look correct. Debug."

```bash
# Trace the app
strace -e trace=open,openat,access command

# Look for:
# openat(...) = -1 EACCES (Permission denied)

# See which files it's trying to access and failing
```

### 3. "Performance bottleneck in application. Find what's slow."

```bash
# Trace with timing
strace -c command

# Output shows:
# % time    seconds usecs/call     calls    syscall
# 45.32    0.123456      123       1000    write
# 30.21    0.082103       82        1001    read
# 20.45    0.055678        55       1000    futex

# Tells you which syscalls take most time
```

### 4. "Child processes aren't being created. Debug."

```bash
# Follow child processes
strace -f command

# Watch for:
# clone(), fork(), or vfork() syscalls
# Check their return values and subsequent syscalls
```

### 5. "File not found but definitely exists. Debug."

```bash
# Trace file operations
strace -e trace=open,openat,stat,lstat command 2>&1 | grep filename

# See exactly what path it's trying to open
# Check if it's looking in wrong directory
```

---

## Common Syscalls

| Syscall | Meaning |
|---------|---------|
| **open/openat** | Open file |
| **read/write** | Read from/write to file descriptor |
| **stat/lstat** | Get file metadata |
| **mmap** | Memory map file |
| **fork/clone** | Create process |
| **execve** | Execute program |
| **socket** | Create network socket |
| **connect** | Establish connection |
| **accept** | Accept connection |
| **close** | Close file descriptor |
| **select/poll/epoll** | Wait for I/O |
| **mutex/futex** | Synchronization primitive |

---

## Useful Flags

| Flag | Meaning |
|------|---------|
| **-p PID** | Attach to running process |
| **-f** | Follow child processes |
| **-e trace=NAME** | Trace specific syscalls |
| **-e trace=!NAME** | Exclude syscalls |
| **-c** | Summary statistics (time, call counts) |
| **-o FILE** | Write output to file |
| **-t** | Print timestamps |
| **-T** | Print syscall duration |
| **-s N** | Limit string output to N bytes |
| **-q** | Quiet (suppress some messages) |

---

## Syscall Groups

```bash
# File operations
strace -e trace=file command

# Network operations
strace -e trace=network command

# Process operations
strace -e trace=process command

# Memory operations
strace -e trace=memory command

# Signals
strace -e signal=all command
```

---

## Real-World Scenarios

### Application won't start

```bash
# Trace startup
strace -f -e trace=execve,exit command

# Watch for:
# - execve() calls showing program loading
# - exit_group() call showing where it exits
# - Check return values for errors
```

### Database connection failing

```bash
# Trace network syscalls
strace -e trace=network command

# Look for:
# socket() - create socket
# connect() - attempt connection
# -1 ECONNREFUSED - connection refused

# This tells you if connection is even being attempted
```

### File locking issue

```bash
# Trace file operations
strace -e trace=fcntl,flock command

# Look for:
# fcntl() with F_SETLK
# Return -1 EAGAIN (already locked)

# Shows if lock is being requested and what happens
```

### Memory leak debugging (advanced)

```bash
# Trace memory syscalls
strace -e trace=mmap,munmap,brk command

# Count mmap calls that aren't matched by munmap
# Suggests memory not being freed
```

---

## Performance Tips

1. **Use `-c` for summary:** Much faster than full output
   ```bash
   strace -c command
   ```

2. **Filter syscalls:** Reduces output volume
   ```bash
   strace -e trace=open,read,write command
   ```

3. **Write to file:** Don't print to terminal (slow)
   ```bash
   strace -o output.txt command
   ```

4. **Use `-q` for quiet mode:** Less verbose
   ```bash
   strace -q command
   ```

---

## Interpreting Errors

| Error | Meaning | Action |
|-------|---------|--------|
| **ENOENT** | File not found | Check file exists, path correct |
| **EACCES** | Permission denied | Check file permissions |
| **EAGAIN** | Resource temporarily unavailable | Retry or wait |
| **ECONNREFUSED** | Connection refused | Check service is running |
| **ETIMEDOUT** | Operation timed out | Network issue or slow service |
| **EINTR** | Interrupted system call | Signal received, usually ok |
| **EINVAL** | Invalid argument | Check syscall parameters |

---

## Interview Tips

1. **Know basic syntax:** `strace command` or `strace -p PID`
2. **Know `-c` for statistics:** Shows which syscalls take time
3. **Know `-e trace=` for filtering:** Reduces output noise
4. **Know `-f` for child processes:** Important for multi-process apps
5. **Know to write to file:** Full output is huge, save to file
6. **Know common syscalls:** open, read, write, connect, fork

---

## When to Use strace

✅ **Good for:**
- Debugging permission issues
- Finding what file an app is looking for
- Finding which service a connection attempt is going to
- Understanding why startup is slow
- Debugging why child process isn't created

❌ **Not ideal for:**
- Performance profiling (use `perf` instead)
- Application logic debugging (use debugger instead)
- Long-running processes (output becomes huge)

---

## Advanced Example: Debugging DNS Issue

```bash
# Trace DNS resolution
strace -e trace=network,connect dig google.com

# Look for:
# socket(AF_INET, SOCK_DGRAM, ...) = 3    # Create UDP socket
# connect(3, {sa_family=AF_INET, sin_port=htons(53), ...}) # Connect to nameserver
# sendto(3, ..., MSG_NOSIGNAL, ...) = X   # Send query
# recvfrom(3, ...) = Y                    # Receive response

# If connect fails or recvfrom times out, DNS is the problem
```

