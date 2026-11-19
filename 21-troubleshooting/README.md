# Linux Troubleshooting

## Troubleshooting Method

For every problem:

1. Identify the affected service, host, and time.
2. Check the simplest system resource and state.
3. Reproduce or capture evidence.
4. Separate network, application, storage, process, and configuration causes.
5. Make one controlled change and test again.

## High CPU Usage

**Possible causes:** Busy application, runaway process, bad loop, malware, or host contention.

```bash
top
uptime
ps -eo pid,user,pcpu,pmem,stat,comm --sort=-pcpu | head
pidstat -p ALL 5
```

Look for the process consuming CPU and whether usage is sustained. Check service logs and application metrics before stopping it.

**Realistic scenario:** A deployment starts a `java` process using 95% CPU. Check recent deployments, thread state, and application logs, then restart only after evidence and impact are understood.

## High Memory Usage

**Possible causes:** Memory leak, cache, large application workload, or under-sized host.

```bash
free -h
top
ps -eo pid,user,rss,pmem,comm --sort=-rss | head
vmstat 1
```

RSS shows resident process memory. Cached memory is reclaimable and should not automatically be interpreted as a leak.

**Fix:** Size the instance or application, inspect the leak, reduce worker count, or restart a known misbehaving process during an approved maintenance window.

## Out of Memory and OOM Killer

The kernel can terminate a process when memory is exhausted.

```bash
dmesg -T | grep -i -E "out of memory|oom|killed process"
journalctl -k --since "1 hour ago" | grep -iE "oom|killed process"
free -h
systemctl status
```

Identify the killed process and whether memory pressure is sustained. Add memory or limits only after understanding the workload.

## Disk Full

**Possible causes:** Application logs, package cache, backups, core dumps, or container data.

```bash
df -h
df -i
sudo du -xh / 2>/dev/null | sort -h | tail -20
sudo du -sh /var/* 2>/dev/null | sort -h
sudo find / -xdev -type f -size +500M -ls
```

Do not delete files until the owner and data importance are known. Clean package caches or old logs only according to policy.

## Inode Exhaustion

```bash
df -i
sudo find / -xdev -printf '%h\n' | sort | uniq -c | sort -nr | head
sudo find /tmp -xdev -type f | wc -l
```

The likely fixes are removing unnecessary many-small-file datasets or increasing filesystem capacity.

## Application Not Starting

```bash
systemctl status myapp --no-pager
journalctl -u myapp -n 100 --no-pager
sudo -u appuser /path/to/app --check
namei -l /srv/app/app
ss -ltnp
```

Check executable path, application configuration, required environment variables, permissions, dependencies, and occupied ports.

## Failed systemd Service

```bash
systemctl status nginx --no-pager
journalctl -u nginx -n 100 --no-pager
sudo systemctl reset-failed nginx
sudo systemctl restart nginx
```

Read the unit and application log. Verify that `daemon-reload` ran after unit-file changes.

## Port Already in Use

```bash
sudo ss -ltnp | grep ':8080'
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN
ps -fp PID
```

Stop or reconfigure one of the competing services rather than repeatedly starting the application.

## Connection Refused

This means a host is reachable but no process is accepting that TCP connection.

```bash
ping 10.0.0.20
nc -vz 10.0.0.20 8080
ss -ltnp
sudo ufw status
```

Check listener address, process state, service logs, and host firewall.

## Connection Timed Out

This usually indicates packet filtering or a route problem.

```bash
traceroute 10.0.0.20
nc -vz 10.0.0.20 8080
ip route
sudo nft list ruleset
```

On EC2, check security groups and network ACLs.

## DNS Resolution Failure

```bash
getent hosts api.example.com
dig api.example.com
cat /etc/resolv.conf
ping 8.8.8.8
```

If public IP ping works but DNS fails, investigate resolvers and DNS records. If IP ping fails, investigate routing.

## Permission Denied

```bash
namei -l /srv/app/file
ls -ld /srv /srv/app
ls -l /srv/app/file
id appuser
sudo -u appuser test -r /srv/app/file
getfacl /srv/app/file
```

Check parent directories, file ownership, mode, ACL, SELinux/AppArmor, and mount options.

## SSH Connection Failure

```bash
ssh -vvv ubuntu@10.0.0.20
sudo ss -ltnp | grep 22
sudo journalctl -u ssh
sudo tail -n 100 /var/log/auth.log
sudo ufw status
```

Verify security groups, SSH listener, key permissions, username, and server logs.

## Zombie Processes

A zombie process is terminated but not reaped by its parent.

```bash
ps -eo pid,ppid,stat,comm | awk '$3 ~ /^Z/'
ps -o pid,ppid,stat,comm -p PID
```

You cannot kill a zombie directly. Restart or fix the parent process that must call `wait()`; on PID 1, investigate the init process implementation.

## Filesystem Mount Failure

```bash
sudo dmesg --level=err,warn
sudo journalctl -b
findmnt
sudo blkid
sudo mount -a
```

Verify the device or NFS source, filesystem type, UUID, network connectivity, credentials, and `/etc/fstab` syntax.

## strace and lsof Basics

`strace` follows system calls:

```bash
strace -f -e trace=file -p PID
strace -f -o /tmp/app.trace ./app
```

Use `strace` on test systems and only with authorized production access because it exposes process activity and can affect performance.

`lsof` finds open files and network sockets:

```bash
lsof /var/log/app.log
sudo lsof -iTCP -sTCP:LISTEN
sudo lsof -p PID
```

## Scenario Checklist

| Symptom | First commands |
|---|---|
| Slow server | `uptime`, `top`, `free -h`, `df -h`, `iostat` |
| Application unavailable | `systemctl status`, `journalctl`, `ss`, `curl` |
| Disk full | `df -h`, `df -i`, `du -sh` |
| Network timeout | `ip route`, `nc`, `traceroute`, firewall status |
| Permission denied | `namei`, `ls`, `getfacl`, `id` |

## Common Mistakes

- Killing a process before collecting evidence.
- Deleting logs on a live system without checking retention.
- Testing only localhost for an externally unavailable application.
- Ignoring the last successful event before the failure.
- Applying unrelated changes together.

## Interview Questions

**Q: What should you check first for a connection timeout?**  
Route, remote listener, firewall, and cloud security rules.

**Q: What does a zombie process mean?**  
The process exited but its parent has not reaped it.

**Q: Why can `df -h` show free space while writes fail?**  
The filesystem may have exhausted inodes.

## Exercises

1. Identify the process using the most CPU.
2. Simulate a disk-full condition in a safe test directory and inspect `df`.
3. Find a listening port and identify the owning process.
4. Review an SSH failure with `-vvv`.

**Related:** [Monitoring](../08-monitoring/) | [Log Management](../15-log-management/) | [Advanced Networking](../16-advanced-networking/)
