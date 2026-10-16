# User & Group Management - SRE Interview Guide

**Time to master:** 45 minutes  
**Difficulty:** ⭐⭐ (Easy-Medium)  
**Interview frequency:** High (appears in ~70% of SRE interviews)

---

## ⚡ Quick Summary

User and group management is how Linux controls who can access what. SREs need to:
- **Know the files:** /etc/passwd, /etc/shadow, /etc/group
- **Understand UIDs & GIDs:** How system identifies users
- **Add/remove users:** useradd, userdel, usermod
- **Know what NOT to do:** Never recycle UIDs, never edit /etc/passwd by hand
- **Understand security:** Why passwords are in shadow, not passwd

---

## 🎯 Key Concepts to Know

### 1. **The Three Files**

| File | Readable by | Contains | Fields |
|------|------------|----------|--------|
| `/etc/passwd` | Everyone | User account info | 7 (login, uid, gid, home, shell, etc.) |
| `/etc/shadow` | Root only | Encrypted passwords | 9 (login, password, aging info) |
| `/etc/group` | Everyone | Group membership | 4 (name, gid, members) |

### 2. **/etc/passwd Structure** (7 fields)

```
login:x:UID:GID:GECOS:home_dir:login_shell
```

Example:
```
root:x:0:0:The System::/root:/bin/bash
jsmith:x:1000:1000:John Smith:/home/jsmith:/bin/bash
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

**Key points:**
- `x` = password placeholder (actual password in /etc/shadow)
- `!` or `*` = account disabled
- UID 0 = root (never duplicate!)
- Regular users: UID >= 1000 (or 500 on older systems)
- System users: UID < 1000 (bin, daemon, syslog, etc.)

### 3. **/etc/shadow Structure** (9 fields)

```
login:encrypted_password:last_change:min_days:max_days:warn_days:grace_days:expiration:reserved
```

Example:
```
jsmith:$6$salt$hash:19000:0:180:14:::
```

**Key points:**
- Readable only by root
- `$1$` = MD5, `$2a$` = Blowfish, `$5$` = SHA256
- Never leave password field empty (security hole!)
- Password aging: min/max days between changes

### 4. **/etc/group Structure** (4 fields)

```
group_name:x:GID:member1,member2,member3
```

Example:
```
wheel:x:10:root,admin
developers:x:1005:alice,bob,charlie
```

**Key points:**
- GID 0 = root/system
- Members = comma-separated (NO SPACES!)
- Personal group = one group per user (recommended)
- Default GID = user's login group for new files

### 5. **UID & GID Management**

**Why UIDs matter:**
- System uses UID internally (not login name)
- File ownership stored by UID, not name
- Must be unique across organization
- **Never recycle UIDs** (backup files still reference old UID)

**Default ranges:**
- UID 0 = root
- UIDs 1-999 = system users
- UIDs 1000+ = regular users

---

## ❓ Typical Interview Questions

### **Q1: "Walk me through adding a new user account."**

**Answer:**
```bash
# Step 1: Use useradd (not manual editing!)
useradd -m -s /bin/bash newuser
# -m = create home directory
# -s = specify shell

# With more options:
useradd -m -s /bin/bash -u 1005 -g wheel -d /home/newuser newuser
# -u = specify UID (1005)
# -g = specify primary group (wheel)
# -d = specify home directory

# Step 2: Set initial password
passwd newuser
# Prompts for password (set securely)

# Step 3: Verify
id newuser
# Shows: uid=1005(newuser) gid=10(wheel) groups=10(wheel)

# Step 4: Check home directory
ls -la /home/newuser/
# Should show ownership: newuser:wheel

# Step 5: Verify login works
su - newuser
# Should work without errors
```

**Why useradd?**
- Automatically updates both /etc/passwd and /etc/shadow
- Handles UID allocation
- Creates home directory
- Sets up defaults from /etc/default/useradd

**Red flag:** Never edit /etc/passwd by hand (unless you use `vipw`)

---

### **Q2: "Why should you never recycle UIDs?"**

**Answer:**
```bash
# Scenario: User alice (UID 1000) leaves company
# Admin deletes alice: userdel -r alice

# A few months later, new user bob joins
# Admin assigns bob UID 1000 (reusing)

# Problem: Old backup has alice's files with UID 1000
# When backup is restored:
# - Files now belong to bob (security issue!)
# - bob can read alice's private data
# - If bob leaves, new user gets alice's files (privacy issue)

# NFS also fails:
# - NFS maps UID to user
# - File UID 1000 from server A = different user on server B
```

**Best practice:**
```bash
# Never reuse UIDs, even when deleting users
userdel -r alice                 # Delete but don't recycle UID 1000

# Assign new user a fresh UID
useradd -m -u 1010 bob           # Use 1010, not 1000
```

---

### **Q3: "What's the difference between /etc/passwd and /etc/shadow? Why are they separate?"**

**Answer:**

**Historical reason:**
- Old Unix stored passwords in /etc/passwd
- Anyone could read /etc/passwd (to look up UIDs)
- Hackers could steal encrypted passwords, crack them offline
- **Solution:** Move passwords to /etc/shadow (root-only readable)

**Modern setup:**
```bash
# /etc/passwd (world-readable)
cat /etc/passwd | grep john
john:x:1000:1000:John Smith:/home/john:/bin/bash
#    ↑ "x" = password in shadow, not here

# /etc/shadow (root-only)
sudo cat /etc/shadow | grep john
john:$6$salt$hash:19000:0:180:14:::
#    ↑ Actual encrypted password (only root can read)
```

**Why separate files?**
- **Principle of least privilege:** Users need /etc/passwd for name lookup, not passwords
- **Security:** Encrypted password in shadow can't be cracked offline
- **Both files must stay in sync** (use useradd, not manual editing)

---

### **Q4: "A user is in the 'wheel' group but can't execute wheel-only commands. Why?"**

**Answer:**
```bash
# Check group membership
id username
# uid=1000(john) gid=1000(john) groups=1000(john),10(wheel)
# Shows john is in wheel group ✓

# But changes don't take effect until...
# Option 1: User logs out and logs back in
exit
ssh username@hostname           # New login = re-read groups

# Option 2: Use newgrp to switch groups
newgrp wheel                    # Activate wheel group for this session

# Why this happens:
# When user logs in, shell reads /etc/group
# If /etc/group changes AFTER login, groups aren't refreshed
# Need new login session to pick up changes
```

**Prevention:**
```bash
# When adding user to group
usermod -aG wheel username      # -a = append, -G = group list

# User must log out/in for changes to take effect
# Tell them: "Log out and back in"
```

---

## 🚨 Common Mistakes

❌ **"I'll just edit /etc/passwd by hand"**  
✅ Use `vipw` (safe) or better yet, use `useradd`

❌ **"I can reuse UIDs when users leave"**  
✅ Never! Causes backup restoration issues and security problems

❌ **"Password is empty in /etc/shadow, so the account is disabled"**  
✅ Empty password = no password required (security hole!). Use `!` or `*` for disabled.

❌ **"User is in group, so command should work immediately"**  
✅ User must log out/in for group changes to take effect

❌ **"I'll set the password field to 'x' to disable the account"**  
✅ Use `!` or `*` to disable. `x` is just a placeholder.

---

## 🛠️ Essential Commands with Examples

### **Add/Remove Users**

```bash
# Add user with home directory
useradd -m username             # -m = create home

# Add with specific UID and group
useradd -m -u 1005 -g wheel username

# Set shell (default is /bin/sh)
useradd -m -s /bin/bash username

# Remove user
userdel username                # Keep home directory

# Remove user and home directory
userdel -r username             # -r = remove home

# Modify user
usermod -aG wheel username      # Add to group (append)
usermod -s /bin/tcsh username   # Change shell
usermod -d /home/newlocation username  # Change home dir
```

### **Change Passwords**

```bash
# User changes own password
passwd                          # No args = current user

# Admin sets password for user
passwd username

# Force password change on next login
passwd -e username              # (Linux)
# or
sudo usermod -p $(date +%s) username  # Set to epoch (very old)
```

### **View Users/Groups**

```bash
# Get user info
id username                     # Show UID, GID, groups
id -u username                  # Just UID
id -g username                  # Just primary GID

# List all users
cat /etc/passwd | cut -d: -f1

# Get users with UID >= 1000 (regular users only)
awk -F: '$3 >= 1000' /etc/passwd

# Check groups
groups username                 # Groups user belongs to
cat /etc/group                  # All groups
getent group wheel              # Single group details
```

### **Edit Files Safely**

```bash
# Safe passwd/shadow editing (auto-locks)
vipw                            # Edit /etc/passwd

# Edit shadow file (Linux specific)
vipw -s                         # Edit /etc/shadow

# Safe group editing
vipw -g                         # Edit /etc/group
```

### **Validate Files**

```bash
# Check passwd/shadow consistency
pwck                            # Check /etc/passwd & /etc/shadow
pwck -r                         # Read-only (don't modify)

# Check group file
grpck                           # Check /etc/group
```

---

## 🎬 Real-World Scenario

**Situation:** New team member joins, needs access to dev tools.

```bash
# 1. Create user
sudo useradd -m -s /bin/bash -u 1010 alice

# 2. Set password
sudo passwd alice
# Prompts for password twice

# 3. Add to developer group
sudo usermod -aG developers alice

# 4. Verify
id alice
# uid=1010(alice) gid=1010(alice) groups=1010(alice),1005(developers)

# 5. Check home directory
ls -ld /home/alice
# drwxr-xr-x 2 alice alice 4096 Oct 15 /home/alice

# 6. Alice logs in
# (Must log out of old session first!)
# su - alice
# Then can use dev tools

# Later: Alice leaves company
sudo userdel -r alice            # Delete user and home
# UID 1010 is NOT reused (good practice)
```

---

## 📊 Interview Tips

✅ **Show understanding of security:** "We keep passwords in shadow because..."  
✅ **Know the tools:** "I use useradd, not manual editing"  
✅ **Think about long-term:** "Never recycle UIDs because of backups..."  
✅ **Understand the system:** "Groups don't take effect until re-login"  

---

## 🔗 Related Files

- [sre-interview-guide.md](./sre-interview-guide.md) - Master interview guide

---

**Next:** Read [04-storage-disk-interview.md](./04-storage-disk-interview.md)
