# Swap and Resource Limits

## What Is Swap?

Swap is disk-backed virtual memory used when the system moves less-used memory pages out of RAM or has no available RAM.

RAM is fast physical memory. Swap is usually much slower storage. Swap may be a partition or a file.

Linux uses swap to:

- Extend total virtual memory
- Move inactive pages away from RAM
- Support memory pressure cleanup
- Reduce immediate OOM termination in some workloads

Swap is not a replacement for RAM.

## Checking Swap

```bash
free -h
swapon --show
cat /proc/swaps
grep -i swap /proc/meminfo
```

Example:

```text
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2
```

## Creating a 2 GB Swap File

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show
free -h
```

If `fallocate` is unsupported on the target filesystem:

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Make it persistent in `/etc/fstab`:

```text
/swapfile none swap sw 0 0
```

Test the mount configuration with a careful, controlled reboot or:

```bash
sudo swapoff /swapfile
sudo swapon -a
```

Using swap on SSDs is commonly acceptable, but active swapping can shorten latency-sensitive performance and increase I/O.

## swapon and swapoff

```bash
sudo swapon --show
sudo swapon /swapfile
sudo swapoff /swapfile
```

Do not disable swap on a memory-constrained production host without confirming available memory and application limits.

## swappiness

`vm.swappiness` controls the kernel's preference for swapping versus reclaiming file cache. Values generally range from `0` to `100`, with distribution defaults around `60`.

Check:

```bash
sudo sysctl vm.swappiness
```

Change temporarily:

```bash
sudo sysctl -w vm.swappiness=10
```

Persist in `/etc/sysctl.d/99-swappiness.conf`:

```text
vm.swappiness=10
```

Then:

```bash
sudo sysctl --system
```

Lower swappiness does not allocate more RAM or guarantee no swap use. Workload behavior and kernel policy determine whether the setting helps.

## File Descriptors

A file descriptor is an integer reference to an open file, socket, pipe, device, or other resource.

Check process limits:

```bash
ulimit -n
ulimit -Sn
ulimit -Hn
cat /proc/PID/limits
lsof -p PID | wc -l
sudo ls /proc/PID/fd | wc -l
```

Too many open files can produce:

```text
Too many open files
```

Common causes include application leaks, long-running connection pools, missing socket cleanup, and an excessive request load.

## ulimit

```bash
ulimit -a
ulimit -n
ulimit -Sn
ulimit -Hn
```

- Soft limits are the current limits.
- Hard limits are the maximum values an allowed process may raise.
- Only authorized root or policy mechanisms can raise hard limits.

Shell limits do not apply to every service. systemd and PAM services have separate controls.

## `/etc/security/limits.conf`

Example:

```text
appuser soft nofile 8192
appuser hard nofile 65535
appuser soft nproc 4096
appuser hard nproc 8192
```

Verify the distribution and PAM integration. This file may not control a systemd service directly.

## systemd Resource Controls

Add properties to a unit or an override:

```bash
sudo systemctl edit myapp
```

```ini
[Service]
LimitNOFILE=65535
MemoryMax=1G
TasksMax=512
```

Apply and restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
systemctl show myapp -p LimitNOFILE
systemctl show myapp -p MemoryMax
systemctl show myapp -p TasksMax
```

`MemoryMax` is a memory limit, not a recommendation. An application that exceeds it may be killed or denied allocations depending on version and workload.

## Investigating Too Many Open Files

```bash
sudo lsof -p PID
sudo ls -l /proc/PID/fd
cat /proc/PID/limits
sudo systemctl show myapp -p LimitNOFILE
journalctl -u myapp --since "30 minutes ago"
```

Identify whether FDs are increasing over time:

```bash
watch -n 2 'ls /proc/PID/fd | wc -l'
```

Fix the application leak or connection handling if possible. Increasing limits is appropriate only after the root cause and capacity are understood.

## Common Mistakes

- Adding swap to `/etc/fstab` with a non-root-owned file.
- Granting a mode other than `600` to a swap file.
- Increasing `ulimit` for the wrong shell or process.
- Treating a swapped process as the sole memory problem.
- Raising `LimitNOFILE` without finding a leaked descriptor source.

## Risks

- Excess swap can conceal memory shortages and add severe I/O latency.
- Lower or higher swappiness can change workload behavior unpredictably.
- Increasing process limits may allow a runaway process to consume more resources.
- Hard limits can expose or protect against resource exhaustion but do not replace monitoring.

## Interview Questions

**Q: What is swap memory?**  
Disk-backed virtual memory used when RAM is under pressure or inactive memory should be moved.

**Q: What is the difference between soft and hard limits?**  
The soft limit is the effective current limit; the hard limit is the maximum to which an authorized process can raise it.

**Q: Does adding swap always improve performance?**  
No. Swap can prevent some OOM failures, but severe swapping may make an application slower.

## Exercises

1. Check available RAM and swap.
2. Create and enable a 2 GB swap file in a test VM.
3. View `/proc/swaps`.
4. Change swappiness temporarily and persistently.
5. Inspect a process's open descriptor count and limits.

**Related:** [Performance Diagnostics](../24-performance-diagnostics/) | [systemd](../12-systemd-service-management/) | [Troubleshooting](../21-troubleshooting/)
