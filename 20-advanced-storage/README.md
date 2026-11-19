# Advanced Storage

## Block Storage vs File Storage

**Block storage** exposes disk blocks to a Linux system. The system creates partitions or logical volumes, formats them with a filesystem, and mounts them. AWS EBS is block storage.

**File storage** exposes directories and files through a network protocol such as NFS or SMB. AWS EFS is file storage.

| Type | Example | Linux concept |
|---|---|---|
| Block | AWS EBS | Disk, partition, filesystem, mount |
| File | AWS EFS | Shared directory mounted over NFS |

## Filesystems

A filesystem organizes files and directories inside a formatted volume.

Common Linux choices:

| Filesystem | Characteristics |
|---|---|
| ext4 | Mature, general-purpose default on many distributions |
| XFS | High performance and scalability; common on RHEL |

Use the distribution-supported filesystem unless there is a documented performance or application requirement.

## Inspecting Storage

```bash
lsblk
lsblk -f
blkid
sudo fdisk -l
df -h
df -i
du -sh /var/*
mount
findmnt
```

- `lsblk` shows disks and partitions.
- `blkid` shows filesystem types and UUIDs.
- `df -h` shows filesystem capacity.
- `df -i` shows inode use.
- `du -sh` estimates directory disk use.

## Partitions

Use `fdisk` for MBR or GPT partition work on interactive systems:

```bash
sudo fdisk /dev/nvme1n1
sudo fdisk -l
```

Inside `fdisk`, `n` creates a partition, `p` prints it, `w` writes changes, and `q` exits without writing.

`parted` supports both MBR and GPT:

```bash
sudo parted /dev/nvme1n1 print
sudo parted /dev/nvme1n1 mklabel gpt
sudo parted -a optimal /dev/nvme1n1 mkpart primary ext4 0% 100%
```

> **WARNING:** Formatting or repartitioning can destroy existing data. Back up important volumes and verify the device name before writing.

## Creating and Mounting Filesystems

```bash
sudo mkfs.ext4 /dev/nvme1n1
sudo mkfs.xfs /dev/nvme1n1
sudo mkdir -p /data
sudo mount /dev/nvme1n1 /data
df -h /data
sudo umount /data
```

`mkfs` formats a partition or volume. Mounting connects it to a directory in the Linux filesystem hierarchy. **Attaching** an AWS EBS volume makes the device available to the instance; **mounting** makes a filesystem in it available at a directory.

## /etc/fstab and UUID Mounts

Do not use device names only when stability matters. Identify the UUID:

```bash
sudo blkid /dev/nvme1n1
```

Example `/etc/fstab` entry:

```text
UUID=12345678-abcd-1234-abcd-1234567890ab /data ext4 defaults,nofail 0 2
```

Mount and verify:

```bash
sudo mount -a
findmnt /data
df -h /data
```

Test `/etc/fstab` carefully. An incorrect entry can delay or prevent boot. The `nofail` option can let the system continue when a volume is unavailable.

## Inodes

An inode stores metadata and pointers for a file. Disk space can appear available while files cannot be created if every inode is consumed.

```bash
df -i
find / -xdev -type f | wc -l
find /var -xdev -type f -links +1
```

## Symbolic Links and Hard Links

```bash
ln -s /data/current /srv/app
ln original.txt hard.txt
ls -li original.txt hard.txt
```

| Symbolic link | Hard link |
|---|---|
| Separate directory entry pointing to another path | Another directory entry for the same inode |
| Can span filesystems | Usually limited to one filesystem |
| A broken target may remain | Target exists independently by inode until final link is removed |

## LVM Fundamentals

LVM adds a logical storage layer:

- Physical volume (PV): a disk or partition used by LVM.
- Volume group (VG): storage pool made of physical volumes.
- Logical volume (LV): usable block device created from a volume group.

```bash
sudo pvcreate /dev/nvme1n1
sudo vgcreate appvg /dev/nvme1n1
sudo lvcreate -L 10G -n applv appvg
sudo mkfs.xfs /dev/appvg/applv
sudo mkdir -p /srv/app
sudo mount /dev/appvg/applv /srv/app
```

Inspect:

```bash
sudo pvs
sudo vgs
sudo lvs
lsblk
```

## NFS Fundamentals

NFS exposes directories from a server over the network. Clients mount the exported path like a network filesystem.

Server-side exports include:

```text
/srv/share 10.0.0.0/24(rw,sync,no_subtree_check)
```

Client-side:

```bash
sudo apt install nfs-common
sudo mkdir -p /mnt/nfs
sudo mount -t nfs 10.0.0.20:/srv/share /mnt/nfs
df -h /mnt/nfs
```

Persistent entry:

```text
10.0.0.20:/srv/share /mnt/nfs nfs defaults,_netdev 0 0
```

## AWS EBS Project

1. In the AWS console or CLI, create or identify an EBS volume in the same availability zone as the EC2 instance.
2. Attach the volume to the instance.
3. Inspect the attached device:

```bash
lsblk
sudo dmesg --level=notice,warn,err
```

4. Verify the device has no existing filesystem or data before formatting:

```bash
sudo blkid /dev/nvme1n1
```

5. Format only if it is intentionally blank:

```bash
sudo mkfs.ext4 /dev/nvme1n1
sudo mkdir -p /data
sudo mount /dev/nvme1n1 /data
```

6. Add a UUID entry to `/etc/fstab`.

## AWS EFS Project

Create a mount target in the appropriate subnet and allow NFS traffic from the EC2 security group.

On Amazon Linux:

```bash
sudo dnf install -y amazon-efs-utils
sudo mkdir -p /mnt/efs
sudo mount -t efs fs-12345678 /mnt/efs
```

Persistent entry:

```text
fs-12345678:/ /mnt/efs efs _netdev,tls 0 0
```

## Common Mistakes

- Formatting a device that contains existing data.
- Assuming AWS device names remain stable.
- Forgetting to update `/etc/fstab`.
- Mounting EBS at a directory that already contains important files.
- Testing EFS without security-group or subnet routing.

## Interview Questions

**Q: What is the difference between attaching and mounting?**  
Attaching makes a block device available to the instance. Mounting makes a filesystem available at a directory.

**Q: Why use UUID in `/etc/fstab`?**  
It identifies the filesystem independently of device names that may change.

**Q: What is an inode exhaustion problem?**  
The filesystem has no free inodes even if some byte capacity remains.

## Exercises

1. Identify a disk and its partitions with `lsblk`.
2. Create a temporary directory and bind or mount a test filesystem.
3. Calculate disk and inode use.
4. Add a UUID-based entry to a test copy of `/etc/fstab`.

**Related:** [Disk Management](../10-disk-management/) | [Troubleshooting](../21-troubleshooting/)
