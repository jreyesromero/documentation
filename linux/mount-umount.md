# mount & umount - Filesystem Management

`mount` attaches filesystems to the filesystem tree, and `umount` detaches them. Essential for working with storage, USB drives, network shares, and disk troubleshooting.

## Quick Reference

```bash
mount                                   # Show all mounted filesystems
mount /dev/sdb1 /mnt/data               # Mount device to path
mount -t nfs server:/share /mnt/nfs     # Mount NFS share
umount /mnt/data                        # Unmount filesystem
mount -o ro /dev/sda1 /mnt/data         # Mount read-only
mount -L label /mnt/data                # Mount by label
mount -U uuid /mnt/data                 # Mount by UUID
```

---

## Understanding mount Output

```bash
$ mount

/dev/disk3s1s1 on / (apfs, local, journaled)
devfs on /dev (devfs, local, nobrowse)
/dev/disk3s5 on /System/Volumes/Data (apfs, local, journaled)
/dev/sdb1 on /mnt/data (ext4, rw, relatime)
server.nfs:/export on /mnt/nfs (nfs, rw, hard, intr)
```

| Component | Meaning |
|-----------|---------|
| **Device** | Physical device or remote share |
| **Mountpoint** | Where it's accessible in filesystem |
| **Type** | Filesystem type (ext4, nfs, tmpfs, etc.) |
| **Options** | Mount flags (rw=read-write, ro=read-only) |

---

## Critical Interview Scenarios

### 1. "Can't delete a file. Filesystem might be full."

```bash
# Check what's mounted where
mount | grep /var/log

# Check disk usage
df -h /var/log

# If it's separate filesystem, might need to mount with different options
```

### 2. "New disk added. Mount and use it."

```bash
# Step 1: Identify the device
lsblk
# or
fdisk -l

# Step 2: Create mount point
mkdir -p /mnt/data

# Step 3: Mount it
mount /dev/sdb1 /mnt/data

# Step 4: Verify
mount | grep /mnt/data
df -h /mnt/data

# Step 5: Make persistent (edit /etc/fstab)
```

### 3. "Can't unmount filesystem (still in use)"

```bash
# Step 1: Find what's using it
lsof +D /mnt/data

# Step 2: Kill the process or close files
kill PID

# Step 3: Try unmount again
umount /mnt/data

# Step 4: Force unmount (caution - might cause data loss)
umount -f /mnt/data
```

### 4. "NFS share won't mount"

```bash
# Step 1: Verify network connectivity
ping server.nfs

# Step 2: Check if NFS service is available
showmount -e server.nfs

# Step 3: Try mounting
mount -t nfs server:/export /mnt/nfs

# Step 4: Check if it's mounted
mount | grep nfs

# Step 5: For persistent, add to /etc/fstab
```

---

## Mount Options

| Option | Meaning |
|--------|---------|
| **rw** | Read-write |
| **ro** | Read-only |
| **noexec** | Don't allow executing files |
| **nodev** | Don't interpret character/block devices |
| **nosuid** | Don't honor SUID bits |
| **noatime** | Don't update access times (faster) |
| **relatime** | Update access times relatively (default) |
| **sync** | Synchronous writes (slow but safe) |
| **async** | Asynchronous writes (fast but risky) |
| **hard** | Hang on NFS errors (default) |
| **soft** | Return error on NFS failure |
| **intr** | Allow interrupting NFS operations |

---

## Real-World Scenarios

### Mount external drive

```bash
# List available devices
lsblk

# Identify USB drive (usually /dev/sdb or /dev/sdc)
lsblk /dev/sdb

# Create mount point
sudo mkdir -p /mnt/usb_backup

# Mount it
sudo mount /dev/sdb1 /mnt/usb_backup

# Use it
cp -r /important/data /mnt/usb_backup/

# Unmount when done
sudo umount /mnt/usb_backup
```

### Mount NFS network share

```bash
# Check what server is exporting
showmount -e nfs.server.com

# Create mount point
mkdir -p /mnt/shared_data

# Mount with options
mount -t nfs -o hard,intr,noatime nfs.server.com:/data /mnt/shared_data

# Verify
mount | grep nfs

# For persistent, add to /etc/fstab:
# nfs.server.com:/data /mnt/shared_data nfs hard,intr,noatime 0 0
```

### Make mount permanent

Edit `/etc/fstab`:

```
# Device          Mountpoint      Type  Options                 Dump Pass
/dev/sdb1         /mnt/data       ext4  defaults,noatime        0    2
server:/export    /mnt/nfs        nfs   hard,intr,noatime       0    0
UUID=abc123...    /mnt/by_uuid    ext4  defaults                0    2
LABEL=backup      /mnt/backup     ext4  defaults                0    2
```

Then mount all:
```bash
mount -a
```

### Troubleshooting mount failures

```bash
# Check filesystem integrity
fsck /dev/sdb1  # Warning: unmount first!

# Check mount permissions
ls -la /mnt/data  # Must be accessible

# Check for stale NFS mounts
mount | grep -E "stale|hang"

# Use verbose to see what's happening
mount -v -t nfs server:/export /mnt/nfs

# Check system logs
journalctl | grep -i mount
```

---

## Common Filesystem Types

| Type | Purpose | Read-only | Network |
|------|---------|-----------|---------|
| **ext4** | Linux native | No | No |
| **XFS** | High performance | No | No |
| **NTFS** | Windows | Yes (mostly) | No |
| **VFAT** | USB drives, old | No | No |
| **NFS** | Network | No | Yes |
| **SMB/CIFS** | Windows shares | Depends | Yes |
| **tmpfs** | Memory-based | No | No |
| **ISO 9660** | CD/DVD | Yes | No |

---

## Important Flags

| Flag | Meaning |
|------|---------|
| **-t TYPE** | Filesystem type |
| **-o OPTIONS** | Mount options (comma-separated) |
| **-L LABEL** | Mount by partition label |
| **-U UUID** | Mount by UUID |
| **-r** | Mount read-only |
| **-w** | Mount read-write |
| **-v** | Verbose |
| **-a** | Mount all in /etc/fstab |
| **-f** | Fake mount (show what would happen) |

---

## Interview Tips

1. **Know the basic syntax:** `mount DEVICE MOUNTPOINT`
2. **Know how to check:** `mount` shows all, `mount | grep pattern` finds specific
3. **Know common issues:** device in use, already mounted, permission denied
4. **Know NFS syntax:** `-t nfs server:/path /mnt/point`
5. **Know how to verify:** `df -h /mnt/point` and `ls /mnt/point`
6. **Know to check logs on failure:** `journalctl` shows mount errors

---

## Real-World Checklist

When mounting a new filesystem:

- [ ] Identify correct device (`lsblk`, `fdisk -l`)
- [ ] Create mount point (`mkdir /mnt/path`)
- [ ] Mount it (`mount /dev/xxx /mnt/path`)
- [ ] Verify it's mounted (`mount | grep`, `df -h`)
- [ ] Check permissions (`ls -la /mnt/path`)
- [ ] Test read/write (`touch /mnt/path/test`)
- [ ] Make permanent if needed (edit `/etc/fstab`)

