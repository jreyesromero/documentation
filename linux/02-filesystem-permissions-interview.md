# Filesystem & Permissions - SRE Interview Guide

**Time to master:** 60 minutes  
**Difficulty:** ⭐⭐⭐ (Medium)  
**Interview frequency:** Very High (appears in ~75% of SRE interviews)

---

## ⚡ Quick Summary

Filesystems are how Linux stores and organizes files. SREs need to:
- **Understand inodes** (file metadata) and why limits matter
- **Master permissions** (rwx bits, chmod, chown)
- **Know file types** (regular, directory, symlink, device)
- **Troubleshoot access issues** (why can't user write?)
- **Monitor inode usage** (df -i) — a common gotcha

---

## 🎯 Key Concepts to Know

### 1. **What is an Inode?**

An **inode** (index node) is metadata about a file, NOT the filename:

```
Inode #12345 contains:
├─ Owner (UID)
├─ Group (GID)
├─ Permissions (rwx bits)
├─ File type (file, dir, symlink, etc)
├─ Size (bytes)
├─ Timestamps (access, modify, change)
├─ Link count (hard links)
├─ Pointers to data blocks (where file content lives)
└─ [NOT the filename!]
```

**Why SREs care:**
- **Filesystem has limited inodes** (e.g., ext4 = 2^32 - 1 inodes)
- **Running out of inodes** = can't create files, even with free disk space
- **Check inodes:** `df -i` (not `df -h`)
- **Classic issue:** Millions of small files (temp logs, cache) fills inode table

### 2. **Permissions: rwx Bits**

```
-rw-r--r-- 1 user group 1024 Oct 15 10:30 file.txt
│││││││││
│││││││└─ Others (world): r (read)
││││││└─  Others: - (no write)
│││││└─   Others: - (no execute)
││││└─    Group: r (read)
│││└─     Group: - (no write)
││└─      Group: - (no execute)
│└─       Owner: w (write)
└─        Owner: r (read)

r (read)    = 4
w (write)   = 2
x (execute) = 1
```

**For files:**
- `r` = read file contents
- `w` = modify/delete file
- `x` = execute as program (or script)

**For directories:**
- `r` = list contents (ls)
- `w` = create/delete files inside
- `x` = enter directory (cd), traverse through

**Common permissions:**
- `644` (rw-r--r--) = Owner can read/write, others read-only
- `755` (rwxr-xr-x) = Owner full access, others execute only (typical for scripts)
- `700` (rwx------) = Owner only (secure home dir)
- `777` (rwxrwxrwx) = Everyone can do everything (security risk!)

### 3. **File Types**

```bash
- Regular file
d Directory
l Symbolic link (shortcut)
c Character device (e.g., /dev/tty)
b Block device (e.g., /dev/sda)
p Named pipe (FIFO)
s Socket
```

### 4. **Symlinks vs Hard Links**

| Aspect | Symlink (soft link) | Hard link |
|--------|---------------------|-----------|
| Points to | Filename | Inode |
| Can cross filesystems | ✅ Yes | ❌ No |
| Size | Small (filename string) | Same as original |
| Breaks if original deleted | ✅ Yes (dangling) | ❌ No (still works) |
| Create | `ln -s` | `ln` |
| Command | `ln -s target linkname` | `ln target linkname` |

---

## ❓ Typical Interview Questions

### **Q1: "A user says they can't write to a directory they own. How do you fix it?"**

**Answer:**
```bash
# Step 1: Check directory permissions
ls -ld /home/username            # -d = directory, not contents

# Sample output:
# drwxr-xr-x 5 username group 4096 Oct 15 10:00 /home/username
#
# Owner (username) has rwx ✓ (should work!)
# Group has r-x (no write)
# Others have r-x (no write)

# Step 2: Check parent directory permissions
ls -ld /home                     # User needs 'x' to enter /home

# Sample output:
# drwxr-xr-x 10 root root 4096 Oct 15 09:00 /home
#
# Others have r-x (can read and enter) ✓

# Step 3: If user can't write, check actual issue
cd /home/username
touch testfile.txt               # Try to create file

# If it fails with "Permission denied":
# Likely cause: Directory doesn't have write bit!

# Step 4: Fix it
chmod u+w /home/username         # Add write for owner
# OR
chmod 755 /home/username         # Standard home dir permissions

# Step 5: Verify
ls -ld /home/username            # Should now show rwx for owner
```

**Why it happens:**
- User can list (`r`) and enter (`x`) but not create files (no `w`)
- Parent dir might need `x` for traversal
- Accidentally set wrong permissions during setup

---

### **Q2: "You're out of disk space but have inodes available. Why can't you create files?"**

**Answer:**
```bash
# Check both space AND inodes
df -h                            # Disk space (GB)
# Filesystem      Size  Used Avail Use%
# /dev/sda1       100G   95G    5G  95% ← Space is 95% full!

df -i                            # Inode count
# Filesystem     Inodes IUsed IFree IUse%
# /dev/sda1   2097152 500K  1.5M   24% ← Plenty of inodes

# Since disk space is full, can't create files
# Solution: Delete large files or add disk

# But what if the opposite?
df -h                            # Plenty of space
df -i                            # 99% inodes used!

# Find what's consuming inodes (usually many small files)
find / -type f | wc -l           # Count all files (slow!)
du --files0-from=<(find / -type f -print0) # Better count

# Or check by directory
for dir in /*; do echo -n "$dir: "; find $dir -type f | wc -l; done

# Solution: Delete temp files, cache, or logs
rm -rf /tmp/*                    # Clear temp
find /var/log -type f -delete    # Clear old logs
```

**Why this matters:**
- Inodes != disk space, check both!
- Many small files (cache, temp, old logs) fill inode table
- This is a classic gotcha question

---

### **Q3: "What's a symlink? When would you use it? What happens if you delete the original file?"**

**Answer:**
```bash
# Create a symlink (soft link)
ln -s /usr/local/bin/myapp /opt/bin/myapp

# It's like a shortcut:
ls -l /opt/bin/myapp
# lrwxrwxrwx 1 root root 20 Oct 15 /opt/bin/myapp -> /usr/local/bin/myapp
#                                                      ↑ Points to this

# Using the symlink works
/opt/bin/myapp                   # Runs /usr/local/bin/myapp

# If you delete the original:
rm /usr/local/bin/myapp

# Symlink is now "broken" (dangling)
ls -l /opt/bin/myapp             # File disappears, symlink remains
/opt/bin/myapp                   # Error: No such file or directory
```

**Use cases:**
- **Version management:** `/usr/bin/python` → `/usr/bin/python3.9`
- **Cross-filesystem links:** Only symlinks work across filesystems
- **Configuration:** `/etc/app/config` → `/mnt/data/config`

**Gotchas:**
- Don't create symlinks to symlinks (confusing)
- Use absolute paths for clarity: `ln -s /full/path` not `ln -s ../relative`
- Check symlink target: `readlink -f /path`

---

### **Q4: "Explain the difference between changing ownership with chown vs changing permissions with chmod."**

**Answer:**

```bash
# chmod = change permissions (rwx bits)
chmod 755 /var/app              # Owner=rwx, Group=r-x, Others=r-x

# Who can do what hasn't changed, just the what they can do
# Original: -rw-r--r-- → New: -rwxr-xr-x

# chown = change owner/group
chown newuser:newgroup /var/app # Ownership changes

# Who owns it changes (ownership), not their permissions
# Original: user1 group1 → New: newuser newgroup
```

**Real scenario:**
```bash
# Service runs as 'appuser', but file is owned by 'root'
ls -l /var/app/data.db
# -rw-r--r-- 1 root root 1024 ...

# appuser can't write to it (not owner, not in group, others can't write)
# Fix: Change owner to appuser
chown appuser:appgroup /var/app/data.db

# OR give group write permission
chmod g+w /var/app/data.db
```

---

## 🚨 Common Mistakes

❌ **"I'll just `chmod 777` to fix permission issues"**  
✅ Security risk! Use minimal permissions: 755 for dirs, 644 for files

❌ **"We're out of disk, better delete some files"** (without checking inodes)  
✅ Check `df -i` first! Might be inode problem, need to delete many small files

❌ **"Symlinks are the same as hard links"**  
✅ Different: symlinks point to filename, hard links point to inode

❌ **"File has 644 permissions, so I can't execute it"**  
✅ Only if you're the owner! Others can't execute, but owner can't without execute bit

❌ **"My directory has write bit, so I can delete other people's files"**  
✅ No! You can only delete files YOU own (or if you own the directory)

---

## 🛠️ Essential Commands with Examples

### **View Permissions**

```bash
# Show full permissions
ls -l /var/app                   # Long format
ls -ld /var/app                  # -d = directory itself, not contents

# Recursive
ls -lR /var/app                  # Show all files inside

# Understand octal notation
stat /var/app                    # Shows Access: (0755) format
```

### **Change Permissions**

```bash
# Symbolic notation (readable)
chmod u+x script.sh              # Add execute for owner
chmod g+w file.txt               # Add write for group
chmod o-r file.txt               # Remove read for others
chmod a-rwx file.txt             # Remove all permissions

# Octal notation (quick)
chmod 755 /var/app               # Standard for app dirs
chmod 644 file.txt               # Standard for files
chmod 700 ~/.ssh                 # Home dir (owner only)
chmod 600 ~/.ssh/id_rsa          # Private key (owner only!)

# Recursive
chmod -R 755 /var/app            # All files and dirs inside
```

### **Change Ownership**

```bash
# Change owner
chown newuser /var/app           # Change owner only

# Change owner and group
chown newuser:newgroup /var/app  # Change both

# Recursive
chown -R appuser:appgroup /var/app  # All files inside
```

### **Check Inode Usage**

```bash
# Filesystem inode usage
df -i                            # Summary per filesystem
df -i /                          # Root filesystem only

# Find what's using inodes
find / -type f | wc -l           # Count all files (slow!)

# Count files per directory
for d in /*; do echo -n "$d: "; find "$d" -type f 2>/dev/null | wc -l; done
```

### **Symlinks**

```bash
# Create symlink
ln -s /usr/local/bin/app /usr/bin/app

# Check where it points
readlink /usr/bin/app            # Shows target path
readlink -f /usr/bin/app         # Absolute path (follow all links)

# List symlinks
ls -l | grep '^l'                # Show only symlinks
find . -type l                   # Recursive symlink search
```

---

## 🎬 Real-World Scenario

**Situation:** Web app can't write logs.

```bash
# 1. Check error
tail -f /var/app/logs/app.log
# "Permission denied"

# 2. Check directory permissions
ls -ld /var/app/logs
# drwxr-xr-x 2 root root 4096 Oct 15 10:00 /var/app/logs

# Problem: Owned by root, app user can't write!

# 3. Fix: Change owner to app user
chown appuser:appgroup /var/app/logs

# 4. Verify
ls -ld /var/app/logs
# drwxr-xr-x 2 appuser appgroup 4096 Oct 15 10:00 /var/app/logs

# 5. Test
/var/app/bin/runapp
# Logs now write successfully!
```

---

## 📊 Interview Tips

✅ **Ask for context:** "What error is the user getting? Permission denied?"  
✅ **Check both:** "Let me verify both disk space (`df -h`) and inodes (`df -i`)"  
✅ **Show debugging:** "I'd use `ls -ld` to check the directory itself..."  
✅ **Know the principle:** "rwx for directories means r=list, w=create, x=enter"  

---

## 🔗 Related Files

- [find.md](./find.md) - Finding files and directories
- [mount-umount.md](./mount-umount.md) - Filesystem mounting
- [sre-interview-guide.md](./sre-interview-guide.md) - Master interview guide

---

**Next:** Read [03-user-group-management-interview.md](./03-user-group-management-interview.md)
