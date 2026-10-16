# Storage & Disk Management - SRE Interview Guide

**Time to master:** 60 minutes  
**Difficulty:** ⭐⭐⭐ (Medium)  
**Interview frequency:** Very High (appears in ~80% of SRE interviews)

---

## ⚡ Quick Summary

Disk and storage is critical for system uptime. SREs need to:
- **Monitor disk space** (df -h, du -sh)
- **Find disk hogs** (what's using 50GB?)
- **Understand RAID** (redundancy and performance)
- **Know LVM** (dynamic resizing without reboot)
- **Recognize the gotcha:** Disk full != out of inodes

---

## 🎯 Key Concepts to Know

### 1. **Disk Space vs Inodes (Common Confusion)**

```bash
# Disk space = actual data (bytes)
df -h                           # Shows in GB/TB

# Inodes = file metadata count
df -i                           # Shows inode count (NOT bytes)
```

| Aspect | Disk Space | Inodes |
|--------|-----------|--------|
| Measures | Data bytes (GB) | File count (metadata) |
| Can run out? | Yes (most common) | Yes (common with temp files) |
| Check with | `df -h` | `df -i` |
| Fix | Delete files OR add disk | Delete many small files |

**Classic interview question:**
> "I have 50GB free but can't create files. Why?"

**Answer:** Out of inodes! Check `df -i`. Solution: Delete old files/logs.

### 2. **RAID: Redundant Array of Disks**

| RAID Level | How It Works | Pros | Cons | Use Case |
|-----------|-------------|------|------|----------|
| **0** (Striping) | Data split across disks | Fast (parallel) | No redundancy | Temporary data, cache |
| **1** (Mirroring) | Identical copies on 2 disks | Simple, readable redundancy | Slower writes, 50% capacity | Boot drives, small critical data |
| **5** (Striping + Parity) | Data + parity across 3+ disks | Good balance | Slower writes than RAID 0 | **Standard for production** |
| **6** (Striping + 2x Parity) | Data + 2 parity across 4+ disks | Survives 2 disk failures | Slower | Large arrays, important data |
| **10** (RAID 1+0) | Mirrored pairs, striped | Fast, redundant | Need 4+ disks | Performance + redundancy |

**SRE perspective:**
- RAID 5 is most common (good balance)
- RAID 6 for critical data (survives 2 failures)
- RAID 0 never in production (one disk = total failure)
- Rebuild time matters (12TB RAID 5 can take days to rebuild)

### 3. **LVM: Logical Volume Manager**

**Problem without LVM:**
```
Physical disk: /dev/sda (100GB)
├── Partition 1: /boot (2GB) - stuck with 2GB
├── Partition 2: /home (50GB) - stuck with 50GB
└── Partition 3: /var (48GB) - stuck with 48GB

If /home fills up, can't borrow from /var without reinstalling!
```

**LVM solution:**
```
Physical disks: /dev/sda, /dev/sdb
    ↓
Volume Group (pool of all disks, 200GB)
    ├── Logical Volume: home (100GB) - can resize!
    ├── Logical Volume: var (75GB) - can resize!
    └── Logical Volume: boot (25GB)

Resize without reboot:
# Grow /home from 100GB to 150GB
lvresize -L +50G /dev/vg0/home
resize2fs /dev/vg0/home        # Grow filesystem
```

**Benefits:**
- ✅ Resize without reboot
- ✅ Snapshots for backups
- ✅ Abstract physical layout
- ❌ More complex than raw partitions

### 4. **Filesystems**

| Filesystem | Max Size | Max Files | Features | When to Use |
|-----------|----------|-----------|----------|------------|
| **ext4** | 16TB | 4 billion | Journaling, fast | Standard Linux |
| **XFS** | 8 exabytes | Huge | Scalable, fast | Large data, RAID |
| **ZFS** | Massive | Unlimited | Checksums, snapshots | Enterprise, NAS |
| **btrfs** | 16 exabytes | Unlimited | Modern, snapshots | Next-gen Linux |

**SRE care:** ext4 is standard. Know it exists, don't overthink.

---

## ❓ Typical Interview Questions

### **Q1: "Disk is 90% full. Walk me through how you'd free up space."**

**Answer:**
```bash
# Step 1: See what's using space
df -h                           # Filesystem overview
df -h /                         # Root specifically

# Step 2: Find the culprits
du -sh /*                       # Size of each top-level directory
# Output:
# 45G  /var
# 20G  /home
# 15G  /usr
# 10G  /

# Step 3: Drill down into largest
du -sh /var/*
# 40G  /var/log               ← Found it!

# Step 4: Check log sizes
du -sh /var/log/*
# 35G  /var/log/app.log
# 4G   /var/log/syslog

# Step 5: Delete old logs
rm /var/log/app.log.1*          # Delete rotated logs
gzip /var/log/app.log.1         # Compress if keeping
journalctl --vacuum=30d         # Keep only 30 days of journal

# Step 6: Verify space recovered
df -h /

# Step 7: Prevention - set up logrotate
cat > /etc/logrotate.d/app <<EOF
/var/log/app.log {
  daily
  rotate 7
  compress
  delaycompress
  missingok
  notifempty
}
EOF
```

**Common culprits:**
- `/var/log/` - Old logs (40-50% of time)
- `/tmp/` - Old temp files
- `/home/` - User data, downloads
- Core dumps in `/var/crash/`

---

### **Q2: "You have 50GB free but 'disk full' error. How do you debug?"**

**Answer:**
```bash
# Check disk space
df -h
# Filesystem     Size  Used Avail Use%
# /dev/sda1      100G   95G    5G  95% ← Space is 95% full!

# So NOT an inode issue, disk is just full
# But you said you have 50GB free elsewhere...

# Check if it's a different filesystem
df -h | grep -v ^/dev          # Logical volumes or mounts?

# Different filesystem example:
# /dev/sda1  100G  95G  5G  95% /
# /dev/sdb1  100G  50G  50G 50% /data

# To fix: Move data to /data filesystem
mv largefile /data/

# Or: Add more space
# For LVM:
lvextend -L +50G /dev/vg0/root
resize2fs /dev/vg0/root

# Reboot to apply, or use online resize (with ext4/XFS)
```

**Check for reserved space:**
```bash
# ext4 reserves 5% for root by default
tune2fs -m 1 /dev/sda1          # Reduce to 1% (risky!)

# Shows current reserve
tune2fs -l /dev/sda1 | grep "Reserved block"
```

---

### **Q3: "Explain RAID 5 vs RAID 6. When would you use each?"**

**Answer:**
```
RAID 5 (Striping + Single Parity):
├── Uses N disks (minimum 3)
├── Capacity = (N-1) disks (1 disk = parity)
├── Survives 1 disk failure
├── Rebuild time: hours to days
└── Use: Most production servers (cost-effective)

RAID 6 (Striping + Dual Parity):
├── Uses N disks (minimum 4)
├── Capacity = (N-2) disks (2 disks = parity)
├── Survives 2 disk failures
├── Rebuild time: longer (more parity to compute)
└── Use: Large arrays, critical data, long rebuild times

Example scenario:
- 4x 2TB disks
- RAID 5: 6TB usable, survives 1 failure
- RAID 6: 4TB usable, survives 2 failures

When to use RAID 6:
- Array > 10 disks (rebuild risk)
- Critical data (can't tolerate outages)
- Long rebuild times (large disks)
```

**Gotcha:** RAID is NOT backup!
```bash
# RAID = redundancy (protects against hardware failure)
# Backup = protection against data loss (accidental delete, corruption)

# You need BOTH:
# RAID 5 for redundancy + daily backups
```

---

### **Q4: "How would you resize a filesystem without downtime?"**

**Answer:**

**With LVM (online, no downtime):**
```bash
# Check current size
df -h /home
# /dev/vg0/home  100G  80G  20G  80%

# Extend logical volume
lvextend -L +50G /dev/vg0/home

# Resize filesystem (online, no unmount needed!)
resize2fs /dev/vg0/home         # For ext4
# OR
xfs_growfs /home                # For XFS

# Verify
df -h /home
# /dev/vg0/home  150G  80G  70G  53%
```

**Without LVM (requires downtime):**
```bash
# Requires offline filesystem
# Usually requires adding disk, repartitioning, backup/restore
# Not practical in production
```

**Gotcha:** Order matters!
```bash
# WRONG:
resize2fs /dev/vg0/home         # Filesystem thinks it's still 100GB
lvextend -L +50G /dev/vg0/home

# RIGHT:
lvextend -L +50G /dev/vg0/home  # Grow logical volume first
resize2fs /dev/vg0/home         # Then grow filesystem
```

---

## 🚨 Common Mistakes

❌ **"Disk full, I'll just delete /var/log"**  
✅ Check what's in there first! Might have important data.

❌ **"RAID is a backup"**  
✅ RAID = hardware redundancy. Backup = data protection (different things).

❌ **"RAID 5 is always the best choice"**  
✅ For large arrays (>10 disks), RAID 6 is safer during rebuild.

❌ **"I'll format with RAID 0 for speed"**  
✅ One disk failure = total data loss. Only for temp data.

❌ **"Running out of disk space, better disable compression"**  
✅ Compression helps! Enable it instead (ZFS, Btrfs).

---

## 🛠️ Essential Commands with Examples

### **Check Disk Space**

```bash
# Filesystem usage
df -h                           # Human-readable
df -h /                         # Specific filesystem
df -H                           # Base-10 (1000) vs base-2 (1024)

# Directory usage
du -sh /var                     # Total size of /var
du -sh /var/*                   # Size of subdirectories
du -sh /* | sort -h             # All top-level, sorted

# Find large files
find / -size +1G                # Files > 1GB
find / -size +100M -type f      # Files > 100MB

# Real-time monitoring
watch -n 1 'df -h'              # Update every second
```

### **Find Disk Hogs**

```bash
# What's consuming most space?
du -sh /* | sort -rh | head -10 # Top 10 directories

# Recursive view
du -sh /var/* | sort -rh

# Large files specifically
find / -type f -size +500M -exec ls -lh {} \; | sort -k5 -rh
```

### **RAID Management**

```bash
# Check RAID status
cat /proc/mdstat                # Software RAID status
mdadm --query --scan            # All RAID arrays

# Hardware RAID (depends on controller)
# Check with: lsscsi, lshw, or vendor tool
```

### **LVM Operations**

```bash
# List volumes
lvs                             # Logical volumes
vgs                             # Volume groups
pvs                             # Physical volumes

# Create logical volume
lvcreate -L 50G -n mydata vg0   # 50GB volume named mydata

# Extend volume
lvextend -L +20G /dev/vg0/mydata  # Add 20GB

# Resize filesystem after extend
resize2fs /dev/vg0/mydata       # ext4
xfs_growfs /mydata              # XFS

# Snapshots (backup without downtime)
lvcreate -L 10G -s -n snap /dev/vg0/mydata
# Creates snapshot for backup
```

### **Filesystem Operations**

```bash
# Check filesystem type
df -T                           # Type column

# Filesystem info
tune2fs -l /dev/sda1            # ext4 details
xfs_info /mount                 # XFS details

# Repair (WARNING: requires unmount)
fsck /dev/sda1                  # File system check

# Adjust reserved space
tune2fs -m 1 /dev/sda1          # Set reserved to 1% (vs default 5%)
```

---

## 🎬 Real-World Scenario

**Situation:** Production system disk is 85% full.

```bash
# 1. Quick assessment
df -h
# /dev/vg0/root  500G  425G  75G  85%

# 2. Find what's using space
du -sh /*
# 350G  /var
# 100G  /home

# 3. Check /var in detail
du -sh /var/*
# 300G  /var/log
# 40G   /var/cache

# 4. Clear logs
du -sh /var/log/*
# 250G  /var/log/app.log.1
# 40G   /var/log/app.log

rm /var/log/app.log.1.*
journalctl --vacuum=7d

# 5. Verify space freed
df -h
# /dev/vg0/root  500G  200G  300G  40% ✓

# 6. Permanent fix: Set up logrotate + extend disk
# Add new disk, extend LVM:
pvcreate /dev/sdb
vgextend vg0 /dev/sdb
lvextend -L +200G /dev/vg0/root
resize2fs /dev/vg0/root
```

---

## 📊 Interview Tips

✅ **Show diagnostic methodology:** "First I'd check df -h, then find culprits..."  
✅ **Understand trade-offs:** "RAID 5 vs 6 depends on rebuild time..."  
✅ **Know the gotchas:** "Disk full doesn't always mean inodes full!"  
✅ **Plan for growth:** "I'd set up logrotate to prevent this..."  

---

## 🔗 Related Files

- [df-du.md](./df-du.md) - Detailed disk usage commands
- [mount-umount.md](./mount-umount.md) - Filesystem mounting
- [sre-interview-guide.md](./sre-interview-guide.md) - Master interview guide

---

**Next:** Read [05-logs-syslog-interview.md](./05-logs-syslog-interview.md)
