# Time and Host Management

## Why System Time Matters

Correct system time is critical for:

- Distributed application ordering
- Log correlation
- TLS certificate validation
- Authentication tickets and tokens
- Database replication
- Cloud API signatures
- Monitoring and alert timestamps

Time drift can produce valid-looking but misleading logs and authentication failures.

## System Time vs Hardware Time

Linux has:

- **System clock:** maintained by the kernel
- **Hardware clock:** maintained by device firmware, also called RTC

```bash
timedatectl
timedatectl status
timedatectl show
```

UTC means coordinated universal time. Local time includes the timezone offset.

## Timezones

```bash
timedatectl
timedatectl list-timezones
timedatectl set-timezone Asia/Kolkata
timedatectl status
date
```

Recommended practice for servers is to operate in UTC and let applications convert for users.

Verify timezone files:

```bash
ls -l /etc/localtime
cat /etc/timezone
```

## UTC vs Local Time

```bash
date
date -u
TZ=America/New_York date
```

- `date` shows local time.
- `date -u` shows UTC.

Cloud servers commonly use UTC while application teams view local timezone reports.

## NTP Fundamentals

NTP synchronizes system clocks from trusted time sources. Correctly synchronized clocks prevent gradual drift.

Traditional `ntpd` and modern `chronyd` implementations differ. Modern systemd distributions commonly use systemd-timesyncd or Chrony.

Check the active service:

```bash
timedatectl
systemctl status systemd-timesyncd
systemctl status chronyd
systemctl status ntpd
```

Do not manage an unused time service based only on package availability.

## Chrony

Chrony is common on RHEL-family systems and can also be used on Ubuntu/Debian.

```bash
# Ubuntu/Debian
sudo apt install chrony

# RHEL/Fedora
sudo dnf install chrony

sudo systemctl enable --now chronyd
sudo systemctl enable --now chrony
```

The service name is commonly `chronyd` on RHEL and `chrony` on Debian/Ubuntu.

Configuration is commonly:

- `/etc/chrony.conf` on RHEL-family
- `/etc/chrony/chrony.conf` on Debian/Ubuntu

## chronyc

```bash
chronyc tracking
chronyc sources
chronyc sources -v
chronyc activity
```

`chronyc tracking` shows whether the clock is synchronized and the current offset.

Example interpretation:

```text
Reference ID    : 52.47.68.170
System time     : 0.000123 seconds slow of NTP
Last offset     : -0.000020 seconds
Leap status     : Normal
```

Small offsets are normal. Persistent high offset or no sources indicates a problem.

## Manual Testing

Check network access to an NTP source and logs:

```bash
timedatectl timesync-status
journalctl -u systemd-timesyncd
journalctl -u chronyd
journalctl -u chrony
```

Cloud hosts often block direct NTP traffic or provide an internal metadata time source. A virtualized host clock may already be synchronized by the hypervisor.

## Hostnames

The hostname identifies the machine locally and in logs.

```bash
hostname
hostnamectl
hostnamectl status
sudo hostnamectl set-hostname app-server-01
```

Check:

```bash
cat /etc/hostname
hostnamectl
```

`hostnamectl set-hostname` updates systemd-managed hostname state, but cloud-init or provisioning tools may overwrite it.

Do not include sensitive or personally identifying details in hostnames.

## `/etc/hosts`

`/etc/hosts` is a local static name mapping.

```text
127.0.0.1 localhost
10.0.0.20 app-server-01
```

Commands:

```bash
cat /etc/hosts
getent hosts app-server-01
```

Local host mappings may override DNS depending on NSS configuration.

## DNS Hostname Resolution

```bash
getent hosts example.com
host app-server-01
dig app-server-01
resolvectl status
```

DNS and `/etc/hosts` resolve names differently. `/etc/nsswitch.conf` determines lookup order.

Check:

```bash
grep '^hosts:' /etc/nsswitch.conf
```

Example:

```text
hosts: files dns
```

This checks `/etc/hosts` before DNS.

## Checking Time Synchronization Problems

```bash
timedatectl
timedatectl show-timesync
chronyc tracking
chronyc sources -v
journalctl -u chronyd
journalctl -u chrony
journalctl -u systemd-timesyncd
```

Possible causes include:

- No network access to NTP
- Wrong timezone rather than wrong clock
- Cloud security rule blocking UDP 123
- Incorrect DNS or clock source
- A virtual machine suspended or migrated
- Chrony configuration source names

Correct the source and network path before repeatedly setting the clock manually.

## Common Mistakes

- Setting timezone while the clock is actually unsynchronized.
- Comparing log timestamps in UTC and local time without conversion.
- Assuming `/etc/hosts` changes are used before DNS.
- Changing hostname without updating cloud inventory or applications.
- Manually setting time instead of fixing synchronization.

## Interview Questions

**Q: Why use UTC on servers?**  
UTC provides a consistent reference across applications, regions, and distributed systems.

**Q: What is the difference between system time and hardware time?**  
System time is maintained by the kernel; hardware time is maintained by the RTC.

**Q: Where does `/etc/hosts` fit in DNS resolution?**  
It is a local static mapping consulted according to `/etc/nsswitch.conf`.

**Q: What command checks Chrony synchronization?**  
`chronyc tracking` and `chronyc sources -v`.

## Exercises

1. Display the current UTC and local time.
2. Change the timezone in a test VM and verify it.
3. Check `chronyc tracking` or systemd-timesyncd status.
4. Set a temporary hostname and verify `/etc/hostname`.
5. Add a host mapping to `/etc/hosts` and test it with `getent`.

**Related:** [Boot Process](../22-boot-process/) | [Networking](../16-advanced-networking/) | [Troubleshooting](../21-troubleshooting/)
