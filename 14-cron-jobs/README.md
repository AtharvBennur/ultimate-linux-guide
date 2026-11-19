# Cron Jobs

## What Is Cron?

Cron is a Linux time-based job scheduler. A cron daemon reads schedules and runs commands at the configured times.

- On Debian/Ubuntu the daemon is commonly named `cron`.
- On RHEL-family systems the daemon is commonly named `crond`.

A **crontab** is a file containing schedule expressions and commands for a user or system.

## Cron Syntax

```text
minute hour day-of-month month day-of-week command
```

| Field | Allowed values |
|---|---|
| Minute | 0-59 |
| Hour | 0-23 |
| Day of month | 1-31 |
| Month | 1-12 or names |
| Day of week | 0-7, where Sunday is 0 or 7 |

Special characters include `*` (every), `,` (list), `-` (range), and `/` (step).

Examples:

```text
* * * * * echo "runs every minute"
*/5 * * * * echo "runs every 5 minutes"
0 * * * * echo "runs every hour"
0 2 * * * echo "runs at 2 AM every day"
0 2 * * 1 echo "runs at 2 AM every Monday"
30 3 1 * * echo "runs at 3:30 AM on the first day of each month"
```

## User Crontab

```bash
crontab -e
crontab -l
crontab -r
```

- `-e` creates or edits the current user's crontab.
- `-l` lists the current user's jobs.
- `-r` removes the entire current user's crontab.

Example entry:

```text
0 2 * * * /usr/bin/tar -czf /backup/app-$(date +\%F).tar.gz /srv/app  >> /var/log/app-backup.log 2>&1
```

The `%` character has special meaning in cron, so escape it as `\%` unless it is being used intentionally.

## System-Wide Cron

Edit `/etc/crontab` or place scripts in `/etc/cron.d/`, `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/`, or `/etc/cron.monthly/`.

`/etc/crontab` includes a user field:

```text
minute hour day month weekday user command
```

Example:

```text
30 1 * * * root /usr/local/bin/nightly-check
```

## Environment Variables

Cron jobs may have a limited environment and a different `PATH` than an interactive shell. Use absolute command paths and set required variables.

```text
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""

0 1 * * * root /usr/bin/find /tmp -type f -mtime +7 -delete
```

## Redirecting Output

```text
0 * * * * /usr/bin/df -h > /var/log/disk-usage.log 2>&1
```

- `>` overwrites the log.
- `>>` appends to it.
- `2>&1` sends standard error to the same place as standard output.

## Practical Projects

### Daily Backup

```text
0 2 * * * /usr/bin/tar -czf /backup/app-$(date +\%F).tar.gz /srv/app >> /var/log/app-backup.log 2>&1
```

Create `/backup` first and ensure it has enough free space.

### Hourly Disk Check

```text
0 * * * * /usr/bin/df -h >> /var/log/disk-usage.log 2>&1
```

### Health Check Every 5 Minutes

```text
*/5 * * * * /usr/bin/curl --fail --silent http://localhost/health >> /var/log/health.log 2>&1
```

## Troubleshooting Cron

```bash
sudo systemctl status cron
sudo systemctl status crond
journalctl -u cron
journalctl -u crond
crontab -l
ls -l /var/spool/cron/
tail -f /var/log/syslog
```

1. Verify the cron daemon is running.
2. Verify the job appears in `crontab -l`.
3. Use absolute paths.
4. Check file and directory permissions.
5. Capture output to a log.
6. Test the command manually.

Common causes include incorrect schedule values, a missing newline at the end of the crontab, shell-specific assumptions, and environment variables missing from cron.

## Cron vs systemd timers

| Cron | systemd timer |
|---|---|
| Simple recurring schedules | Integrates with systemd dependencies and logging |
| User and system crontabs | Unit-based configuration |
| Limited missed-job handling | Can support catch-up behavior |
| Widely available on Linux | Available on systemd systems |

Cron is generally simpler for small administrative jobs.

## Interview Questions

**Q: What does `*/15 * * * *` mean?**  
Run every 15 minutes.

**Q: Why use absolute paths in cron?**  
Cron often has a minimal `PATH`, so a command such as `tar` may not be found.

**Q: Does `crontab -r` remove one job?**  
No. It removes the entire crontab for the current user.

## Exercises

1. Schedule `date >> /tmp/cron-test.log` every minute.
2. Wait two minutes and inspect the file.
3. Redirect both output and errors.
4. Remove the test crontab.

**Related:** [systemd](../12-systemd-service-management/) | [Log Management](../15-log-management/)
