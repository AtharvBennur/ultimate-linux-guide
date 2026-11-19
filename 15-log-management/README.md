# Linux Log Management

## What Are Linux Logs?

Logs are timestamped records produced by the Linux kernel, system services, applications, authentication systems, and device drivers.

DevOps engineers use logs to investigate failures, security events, performance problems, and changes in application behavior.

## The `/var/log` Directory

Common locations include:

| File or directory | Typical purpose |
|---|---|
| `/var/log/syslog` | General system messages on Debian/Ubuntu |
| `/var/log/messages` | General system messages on some RHEL systems |
| `/var/log/auth.log` | Login and privilege activity on Debian/Ubuntu |
| `/var/log/secure` | Login and privilege activity on RHEL systems |
| `/var/log/kern.log` | Kernel messages on Debian/Ubuntu |
| `/var/log/dmesg` | Kernel ring-buffer messages |
| `/var/log/nginx/` | Nginx access and error logs |

List log files:

```bash
ls -l /var/log
sudo ls -l /var/log/nginx
```

## Reading Logs

```bash
tail -n 100 /var/log/syslog
tail -f /var/log/syslog
head -n 20 /var/log/syslog
less /var/log/syslog
grep -n "ERROR" /var/log/app.log
grep -nE "ERROR|FATAL" /var/log/app.log
```

- `tail` shows the end of a file.
- `tail -f` follows new lines.
- `head` shows the beginning.
- `less` allows scrolling and searching.
- `grep` filters matching text.

In `less`, press `/` to search forward, `n` for the next match, and `q` to quit.

## Searching by Time

```bash
journalctl --since "2026-11-20 10:00:00"
journalctl --since "30 minutes ago" --until "10 minutes ago"
journalctl --since today
```

Plain text log timestamps vary. A portable approach is to filter with `grep` once the exact format is known.

## journalctl

The systemd journal stores logs in binary journal files.

```bash
journalctl
journalctl -u nginx
journalctl -u nginx -f
journalctl -u nginx -b
journalctl -b
journalctl -p err
journalctl -k
```

| Option | Purpose |
|---|---|
| `-u` | Unit/service filter |
| `-f` | Follow journal output |
| `-b` | Current or selected boot |
| `-p err` | Error level and higher |
| `-k` | Kernel messages |

Useful boot logs:

```bash
journalctl -b -1
journalctl -k -b
```

## Kernel Logs and dmesg

```bash
dmesg
sudo dmesg --level=err,warn
sudo dmesg -T
```

Use `dmesg` for hardware, driver, disk, memory, and kernel events. A boot ID from `journalctl -k -b` provides a more persistent view.

## Log Levels

Common levels from least severe to most severe include:

```text
debug, info, notice, warning, err, crit, alert, emerg
```

`journalctl -p warning` includes warning and more severe levels. A visible warning is not always the root problem; correlate it with the first error and the affected time range.

## Log Rotation

Logs grow continuously. **Log rotation** moves, compresses, deletes, or recreates log files after a size or age limit.

Configuration is commonly stored in `/etc/logrotate.conf` and `/etc/logrotate.d/`.

```bash
sudo logrotate -d /etc/logrotate.conf
sudo logrotate -f /etc/logrotate.d/nginx
ls -l /var/log/nginx
```

The `-d` option performs a debug/dry-run. It does not change logs. The `-f` option forces rotation and is mainly for testing.

Example application entry:

```text
/var/log/myapp/*.log {
    daily
    rotate 14
    missingok
    notifempty
    compress
    delaycompress
    create 0640 app app
}
```

## Practical Project: Investigate a Failed Service

```bash
sudo systemctl status example-app --no-pager
journalctl -u example-app -n 100 --no-pager
sudo journalctl -u example-app --since "30 minutes ago"
sudo systemctl start example-app
```

Example workflow:

1. Start with `systemctl status` for the service state and final failure.
2. Read the unit-specific journal.
3. Search application logs for the first error.
4. Correct configuration or permissions.
5. Restart and follow logs.

```bash
sudo systemctl restart example-app
journalctl -u example-app -f
```

## Troubleshooting Application Logs

```bash
grep -RniE "error|fatal|exception" /var/log/myapp/
tail -n 200 /var/log/myapp/app.log
sudo journalctl -u myapp --since "1 hour ago"
```

Start with timestamps, then correlate application logs with authentication, network, disk, or system logs.

## Common Mistakes

- Running `tail -f` and looking at `/var/log/syslog` only on a systemd machine.
- Ignoring the first error and focusing on later repeated messages.
- Starting logrotate manually in production without a dry-run.
- Storing logs outside `/var/log` without rotation.
- Printing secrets or tokens into application logs.

## Interview Questions

**Q: What is the difference between `journalctl -f` and `tail -f`?**  
`journalctl -f` follows the systemd journal; `tail -f` follows a regular text file.

**Q: What does log rotation solve?**  
It prevents logs from growing forever and controls retention.

**Q: What command helps identify a failed service's logs?**  
Use `systemctl status <service>` and `journalctl -u <service>`.

## Exercises

1. Display the 50 latest lines in `/var/log/syslog` or `/var/log/messages`.
2. Follow journal entries for SSH or Nginx.
3. Search logs for `error` in a real time range.
4. Run a logrotate dry-run.

**Related:** [systemd](../12-systemd-service-management/) | [Troubleshooting](../21-troubleshooting/)
