# Linux Boot Process

## How Linux Starts

The normal boot process is:

```text
Power on
  -> BIOS or UEFI
  -> POST
  -> Boot device selection
  -> GRUB bootloader
  -> Linux kernel
  -> initramfs/initrd
  -> Root filesystem
  -> PID 1 (systemd)
  -> systemd target
  -> Enabled services
```

## BIOS vs UEFI

| BIOS | UEFI |
|---|---|
| Legacy firmware interface | Modern firmware interface |
| Usually uses MBR boot | Uses EFI System Partition |
| Searches configured boot devices | Loads EFI boot applications |

Check whether a Linux system uses UEFI:

```bash
ls /sys/firmware/efi
```

If that directory exists, UEFI mode is likely active.

## POST

POST (Power-On Self-Test) is performed by firmware. It checks essential hardware before the operating system is loaded.

POST failures may show beep codes or firmware messages rather than Linux logs.

## GRUB

GRUB is the bootloader on many Linux distributions. It presents or loads a kernel, passes kernel parameters, and may load an initial root device configuration.

Common files include:

```text
/boot/grub/
/boot/grub2/
/boot/efi/
/etc/default/grub
```

Regenerate GRUB configuration only using distribution-specific tooling:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

On RHEL-family systems:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

UEFI and Secure Boot layouts vary.

## Linux Kernel

The kernel initializes memory management, scheduling, interrupts, drivers, and core subsystems. Kernel images are commonly in `/boot`, for example `vmlinuz-*`.

Inspect installed kernels:

```bash
ls -l /boot
uname -r
```

## initramfs or initrd

The initial RAM filesystem contains temporary drivers and tools needed before the real root filesystem is mounted.

It is used for:

- Loading storage or encryption drivers
- Connecting network storage
- Finding and mounting the root filesystem
- Starting device discovery

Rebuild it using distribution-specific tools, for example `update-initramfs` on Debian/Ubuntu or `dracut` on RHEL-family systems.

```bash
sudo update-initramfs -u
```

## PID 1 and systemd

After the kernel and initial filesystem setup, PID 1 starts. On modern Linux distributions, PID 1 is systemd.

Verify:

```bash
ps -p 1 -o pid,comm,args
systemctl status
```

## systemd Targets

A target is a group of units representing a desired system state. It is similar to the older runlevel concept.

| Runlevel | systemd target | Meaning |
|---|---|---|
| 0 | `poweroff.target` | Shutdown |
| 1 | `rescue.target` | Single-user rescue |
| 3 | `multi-user.target` | Multi-user without GUI |
| 5 | `graphical.target` | Multi-user with GUI |
| 6 | `reboot.target` | Restart |

## Targets

Check the current target:

```bash
systemctl get-default
systemctl list-units --type=target
```

Change the default:

```bash
sudo systemctl set-default multi-user.target
```

Switch targets for the running system:

```bash
sudo systemctl isolate multi-user.target
sudo systemctl isolate rescue.target
sudo systemctl isolate emergency.target
```

- `rescue.target` starts a basic system and single-user environment.
- `emergency.target` starts the minimum system and usually requires root.
- `graphical.target` starts a desktop environment after core services.

Avoid switching to `emergency.target` or `poweroff.target` on a remote server unless you understand console access and recovery.

## Boot Logs

```bash
journalctl -b
journalctl -b -1
journalctl -b -p err
dmesg
sudo dmesg -T
```

- `journalctl -b` shows the current boot.
- `journalctl -b -1` shows the previous boot.
- `dmesg` shows kernel messages.

## Basic Boot Failure Troubleshooting

### Stuck Before GRUB

Check firmware boot order, boot media, disk availability, and whether the system enters POST.

### GRUB Loads but Kernel Fails

Inspect kernel parameters, `/boot` contents, encrypted or network storage setup, and initramfs.

### Kernel Starts but Root Filesystem Fails

Use rescue mode or emergency mode and verify:

```bash
lsblk
blkid
cat /etc/fstab
mount
```

Check for UUID mismatch, filesystem errors, storage disconnection, or an incorrect mount option.

### systemd Starts but Services Fail

```bash
systemctl --failed
journalctl -b -p err
journalctl -b -u ssh
```

Compare the failed service with networking, storage, and kernel errors from the same boot.

## Common Mistakes

- Changing the default target before testing service availability.
- Booting with an old kernel that does not have required drivers.
- Editing `/etc/fstab` without preserving a rescue plan.
- Rebuilding GRUB before identifying the boot mode.
- Replacing `/boot` contents without backups.

## Interview Questions

**Q: What does PID 1 do?**  
It starts and manages the system's first-level user-space process hierarchy. On modern Linux, PID 1 is systemd.

**Q: What is the difference between rescue and emergency targets?**  
Rescue starts a basic operating state with a root shell; emergency starts the minimum possible environment.

**Q: What does initramfs contain?**  
Temporary drivers and tools needed to find and mount the real root filesystem.

## Exercises

1. Check the current and default targets.
2. View journal entries from the current and previous boots.
3. Verify whether the system uses UEFI.
4. In a test VM, boot one old kernel and inspect the difference.

**Related:** [systemd](../12-systemd-service-management/) | [Log Management](../15-log-management/) | [Advanced Storage](../20-advanced-storage/)
