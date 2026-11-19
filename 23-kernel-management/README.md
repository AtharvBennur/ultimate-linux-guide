# Kernel Management and Configuration

## What Is the Linux Kernel?

The Linux kernel is the core of the operating system. It manages CPU scheduling, memory, processes, devices, filesystems, networking, and access to hardware.

Applications do not normally talk directly to hardware. They use system calls, and the kernel provides the implementation and safety checks.

The kernel includes:

- Process and thread scheduling
- Virtual memory
- Device and filesystem support
- Network protocol handling
- Security controls
- Hardware device drivers

## Kernel Architecture Basics

Linux is mainly a monolithic kernel. Many features are compiled into the kernel or loaded as modules. Modules add code for a device, filesystem, network protocol, or other feature without rebuilding the complete kernel.

User space contains applications, shells, systemd, package managers, and libraries. Kernel space contains privileged kernel code.

## Kernel Versions

```bash
uname -a
uname -r
uname -m
cat /proc/version
```

- `uname -r` shows the running kernel release.
- `uname -m` shows the architecture.
- `/proc/version` shows kernel and build details.
- `uname -a` displays a complete summary.

Example:

```text
5.15.0-101-generic
```

The first package changes vary by distribution and kernel branch.

## Kernel Modules

A kernel module is optional kernel code loaded while the system is running. Modules are commonly used for drivers.

```bash
lsmod
modinfo e1000
sudo modprobe e1000
sudo rmmod e1000
sudo insmod ./e1000.ko
```

| Command | Purpose |
|---|---|
| `lsmod` | List loaded modules |
| `modinfo` | Display module metadata |
| `modprobe` | Load a module and its dependencies |
| `rmmod` | Remove a module |
| `insmod` | Load a module file directly |

Prefer `modprobe` because it resolves dependencies and follows module configuration. `insmod` does not normally resolve dependencies.

View module dependencies:

```bash
cat /proc/modules
sudo modprobe --show-depends tcp_bbr
```

## `/proc` and `/sys`

`/proc` exposes process and kernel runtime information:

```bash
ls /proc
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/swaps
cat /proc/sys/net/ipv4/ip_forward
```

`/sys` exposes kernel objects and device/hardware configuration:

```bash
ls /sys/class/net
ls /sys/block
cat /sys/class/net/eth0/mtu
```

Neither directory should be treated as a normal persistent configuration directory.

## sysctl

`sysctl` reads and changes kernel parameters.

```bash
sysctl -a
sudo sysctl -a | grep ip_forward
sudo sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
```

The `-w` option changes a live parameter. It may not survive reboot.

DevOps examples include:

- Increasing open file limits
- Enabling IP forwarding for routing or containers
- Configuring TCP networking behavior
- Changing memory or swap policy
- Tuning virtual memory settings

Persistent configuration belongs in `/etc/sysctl.conf` or `/etc/sysctl.d/`.

Example `/etc/sysctl.d/99-network.conf`:

```text
net.ipv4.ip_forward=1
net.core.somaxconn=1024
```

Load it:

```bash
sudo sysctl --system
```

The loading order and distribution defaults must be considered. A later file can override an earlier value.

## dmesg and Kernel Messages

```bash
dmesg
sudo dmesg -T
sudo dmesg --level=err,warn
journalctl -k
journalctl -k -b
```

- `dmesg -T` adds human-readable timestamps.
- `-b` limits journal kernel logs to the selected boot.
- `journalctl -k` reads persistent kernel journal entries where configured.

Kernel messages can reveal:

- Missing or failing hardware
- Disk and filesystem errors
- Driver failures
- Network interface changes
- OOM-killer decisions
- Module loading failures

## Practical DevOps Scenarios

### Container or Router Cannot Forward Network Traffic

Check forwarding:

```bash
sudo sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
```

Also verify firewall and cloud routing rules.

### Disk Device Is Not Detected

```bash
lsblk
sudo dmesg -T | tail -100
journalctl -k -b
lspci
lsusb
```

Look for device attachment, driver, or I/O errors.

### Application Is Killed for Memory Pressure

```bash
sudo dmesg -T | grep -iE "out of memory|oom|killed process"
free -h
sudo sysctl vm.swappiness
```

This is a kernel memory-pressure investigation, not proof that changing `vm.swappiness` will fix the leak.

## Basic Kernel Troubleshooting

1. Record `uname -r` and the system distribution.
2. Check kernel logs around the incident time.
3. Check loaded modules and hardware state.
4. Review live `sysctl` values.
5. Compare configuration in `/etc/sysctl.d/`.
6. Test one change on a non-production host.
7. Reboot only if required and test the persistent value.

## Common Mistakes

- Changing a `/proc` value and expecting it to survive reboot.
- Copying tuning values without understanding workload and kernel version.
- Installing a module while ignoring its package and signature.
- Assuming every kernel warning is related to the latest application change.
- Using `dmesg` only and ignoring application logs.

## Interview Questions

**Q: What is the difference between `/proc` and `/sys`?**  
`/proc` primarily exposes process and kernel runtime information. `/sys` exposes kernel objects and hardware or device configuration.

**Q: What is the difference between `modprobe` and `insmod`?**  
`modprobe` understands module dependencies; `insmod` loads a specific module file directly.

**Q: Why may a `sysctl -w` change disappear after reboot?**  
It changes only the live kernel parameter unless a persistent sysctl configuration is also created.

## Exercises

1. Record `uname -r` and the active architecture.
2. List loaded modules with `lsmod`.
3. Inspect one module with `modinfo`.
4. Read `net.ipv4.ip_forward` through `sysctl`.
5. Check kernel errors from the current boot.

**Related:** [Boot Process](../22-boot-process/) | [Performance Diagnostics](../24-performance-diagnostics/) | [Troubleshooting](../21-troubleshooting/)
