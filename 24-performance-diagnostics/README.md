# Linux Performance Diagnostics

## System Load

`uptime` and `top` show load averages for 1, 5, and 15 minutes.

```bash
uptime
top
```

Example:

```text
load average: 4.20, 3.10, 1.80
```

Load average represents processes runnable or waiting on uninterruptible I/O, averaged over time. CPU utilization describes how busy CPU cores are.

A load average of 4 on a 4-core system may indicate full utilization, but a host with 16 cores has more available capacity. Cloud burst, queueing, and I/O wait must be considered.

## Basic Investigation Method

1. Record time, host, load, CPU, memory, disk, and network symptoms.
2. Use one-second or short intervals to observe trends.
3. Identify the busiest resource.
4. Identify the process or kernel subsystem responsible.
5. Compare application metrics and logs.
6. Make one controlled change and compare performance.

## top and htop

```bash
top
htop
```

In `top`, important fields include:

| Field | Meaning |
|---|---|
| `load average` | 1, 5, and 15-minute scheduling demand |
| `%CPU` | CPU used by a process |
| `%MEM` | RAM used by a process |
| `RES` | Resident physical memory |
| `VIRT` | Virtual address space |
| `S` or `STAT` | Process state |

Use `top -H` to inspect threads. Press `P` to sort by CPU, `M` to sort by memory, and `q` to quit.

## vmstat

```bash
vmstat 1 5
```

```text
procs -----------memory---------- ---swap-- -----io----
 r  swpd   free  buff  cache   si   so    bi    bo
 2     0 120000   5000 800000    0    0     10    20
```

- `r` is the run queue.
- `b` represents processes blocked in uninterruptible state.
- `si`/`so` show swap in/out.
- `bi`/`bo` show block I/O in and out.
- `wa` in CPU fields shows I/O wait.

## iostat

Install `sysstat` when unavailable:

```bash
# Ubuntu/Debian
sudo apt install sysstat

# RHEL/Fedora
sudo dnf install sysstat
```

```bash
iostat
iostat -xz 1
iostat -p xvda 1
```

Important fields include:

| Field | Meaning |
|---|---|
| `r/s`, `w/s` | Reads and writes per second |
| `rkB/s`, `wkB/s` | Read/write throughput |
| `await` | Average I/O completion wait |
| `%util` | Percentage of time the device was busy |
| `svctm` | Legacy service-time estimate; not always reliable |

High `await`, high queue, or high `%util` can indicate storage pressure. Compare with application workload before increasing capacity.

## mpstat

```bash
mpstat
mpstat -P ALL 1
```

Use per-CPU statistics to find one busy core, uneven scheduling, or CPU affinity problems.

Fields include:

- `%usr`: user-space CPU
- `%sys`: kernel CPU
- `%iowait`: waiting for I/O
- `%idle`: idle CPU

High `%usr` usually points to application usage. High `%sys` may indicate kernel-heavy work. High `%iowait` may indicate storage bottlenecks, but CPU can also be idle while wait events occur.

## pidstat

```bash
pidstat 1
pidstat -p ALL 1
pidstat -d 1
pidstat -r 1
pidstat -u 1
```

| Option | Measures |
|---|---|
| Default | CPU per process |
| `-d` | Disk I/O per process |
| `-r` | Memory faults and resident memory |
| `-u` | CPU usage |

Use `pidstat -d 1` to identify a process generating disk operations.

## sar

Enable or configure `sar` through the `sysstat` package.

```bash
sar
sar -u 1 5
sar -r 1 5
sar -d 1 5
ls /var/log/sa
sar -u -f /var/log/sa/SA_DATE
```

Replace `SA_DATE` with the archive file name. `sar` is useful for historical comparison after an incident.

## iotop

```bash
sudo iotop
sudo iotop -o
```

`iotop -o` shows processes doing I/O at that moment. It requires sufficient kernel and system privileges.

## free -h

```bash
free -h
```

```text
              total        used        free      shared  buff/cache   available
Mem:           7.7G        2.1G        1.1G        200M        4.5G        5.2G
Swap:          2.0G          0B        2.0G
```

Use `available`, not just `free`. Linux can reclaim cache memory for applications.

## Practical Scenarios

### EC2 CPU Usage Reaches 100%

```bash
uptime
mpstat -P ALL 1
top
pidstat -u 1 10
journalctl --since "15 minutes ago"
```

Check whether the CPU is high in user or system mode, identify the process, review deployments or traffic spikes, and inspect application metrics.

### Application Response Becomes Slow

Check CPU, memory, disk, network, and application logs:

```bash
uptime
free -h
vmstat 1
iostat -xz 1
ss -s
pidstat -d 1
curl -v http://localhost:8080/health
```

Then compare database latency and external API dependencies.

### Disk I/O Is Extremely High

```bash
iostat -xz 1
sudo iotop -o
pidstat -d 1
df -h
df -i
dmesg -T | tail
```

A process generating writes may be logging excessively or performing a backup.

### High Load Average

```bash
uptime
top
mpstat -P ALL 1
vmstat 1
iostat -xz 1
free -h
```

Compare CPU queueing with I/O wait and memory pressure. Do not immediately restart or scale without identifying a cause.

## Common Mistakes

- Treating load average as CPU percentage.
- Treating `free` memory as the only memory-health indicator.
- Ignoring I/O wait.
- Comparing one-second spikes to hourly data.
- Restarting services before identifying the resource consumer.

## Interview Questions

**Q: What is the difference between load average and CPU usage?**  
Load average includes runnable and certain waiting work over time; CPU usage is processor utilization over an interval.

**Q: What does `%iowait` indicate?**  
The CPU was idle while waiting for outstanding I/O work, which can indicate storage pressure but is not itself a complete diagnosis.

**Q: Which tool helps show disk activity by process?**  
`pidstat -d` and `iotop`.

## Exercises

1. Run `uptime` and explain three load-average values.
2. Capture five seconds of `vmstat`.
3. Use `pidstat -u` and identify the busiest process.
4. Use `iostat -xz` and find an unhealthy await or utilization value.
5. Use `sar` to compare a period before and after a problem.

**Related:** [Monitoring](../08-monitoring/) | [Troubleshooting](../21-troubleshooting/) | [Kernel Management](../23-kernel-management/)
