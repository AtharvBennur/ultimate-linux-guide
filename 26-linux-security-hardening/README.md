# Linux Security Hardening

This section covers mandatory access controls, Linux capabilities, and security auditing. Basic file permissions, ACLs, sudo, and SSH hardening are already covered in [Linux Security](../19-linux-security/).

## SELinux

SELinux is a Linux Security Module that applies mandatory access control policies. It is commonly enabled on RHEL, CentOS Stream, AlmaLinux, Rocky Linux, Fedora, and related systems.

SELinux labels processes and resources. A policy decides whether the process domain may access a file, port, socket, or system resource.

Check mode:

```bash
getenforce
sestatus
```

Modes:

| Mode | Behavior |
|---|---|
| Enforcing | Policy violations are denied and logged |
| Permissive | Policy violations are logged but not denied |
| Disabled | SELinux policy is not active |

Temporarily change mode:

```bash
sudo setenforce 1
sudo setenforce 0
```

Permissive mode is useful for controlled troubleshooting, but it should not replace policy configuration in production.

Persistent configuration is commonly in `/etc/selinux/config`:

```text
SELINUX=enforcing
SELINUXTYPE=targeted
```

Changing `disabled` requires an appropriate reboot and can complicate existing labels. Re-enable only as planned and tested by the distribution.

## SELinux Contexts

```bash
ls -Z /srv/app
ps -eZ
id -Z
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/app(/.*)?"
sudo restorecon -Rv /srv/app
```

- `-Z` shows contexts.
- `semanage fcontext` configures persistent file-label rules.
- `restorecon` applies labels based on policy rules.

Common troubleshooting output includes `Permission denied` plus an AVC denial:

```bash
sudo ausearch -m AVC,USER_AVC -ts recent
sudo journalctl -t setroubleshoot
sudo sealert -a /var/log/audit/audit.log
```

Do not automatically disable SELinux. Check the correct label, boolean, port, or service policy before changing policy.

Example port configuration:

```bash
sudo semanage port -l | grep http
sudo semanage port -a -t http_port_t -p tcp 8080
```

## AppArmor

AppArmor is a mandatory access control system commonly used by Ubuntu and Debian. It confines applications with profiles.

Check status:

```bash
sudo aa-status
sudo apt install apparmor-utils
```

Profiles may be in `/etc/apparmor.d/`.

```bash
sudo aa-status
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

- Enforce mode denies violations.
- Complain mode logs violations without denying them.
- `aa-status` shows active profiles and modes.

Inspect logs:

```bash
dmesg | grep -i apparmor
journalctl -k | grep -i apparmor
sudo journalctl --since "30 minutes ago" | grep -i apparmor
```

## SELinux vs AppArmor

| SELinux | AppArmor |
|---|---|
| Label-based mandatory access control | Path/profile-based mandatory access control |
| Strong integration with RHEL policy tools | Common on Ubuntu/Debian |
| Policies include domains, types, labels, and booleans | Profiles restrict named application behavior |
| Managed through contexts and policy modules | Managed through profile files and aa commands |

Use the security framework provided and supported by the distribution. Do not run both as if they were interchangeable.

## Linux Capabilities

Traditional root privilege is broad. Linux capabilities divide privileged operations into smaller units.

Examples include `CAP_NET_BIND_SERVICE`, `CAP_NET_ADMIN`, and `CAP_SYS_ADMIN`.

Inspect file capabilities:

```bash
getcap -r / 2>/dev/null
getcap /usr/bin/ping
```

Set a file capability:

```bash
sudo setcap 'cap_net_bind_service=+ep' /srv/app/server
getcap /srv/app/server
sudo setcap -r /srv/app/server
```

This example allows a program to bind low ports without full root access.

Process capabilities are shown in `/proc/PID/status`:

```bash
grep -i '^Cap' /proc/PID/status
sudo capsh --decode=HEX_VALUE
```

Keep file capabilities minimal. Avoid `CAP_SYS_ADMIN`, `CAP_DAC_OVERRIDE`, or `CAP_SYS_PTRACE` unless an application has an audited requirement.

## Security Auditing

`auditd` records security events configured in audit rules.

Ubuntu/Debian:

```bash
sudo apt install auditd audispd-plugins
sudo systemctl enable --now auditd
sudo auditctl -l
```

RHEL-family:

```bash
sudo dnf install audit
sudo systemctl enable --now auditd
sudo auditctl -l
```

Search audit logs:

```bash
sudo ausearch -k auth_changes
sudo ausearch -m USER_AUTH
sudo aureport --summary
sudo aureport -a
```

Example temporary rule:

```bash
sudo auditctl -w /etc/passwd -p wa -k auth_changes
```

Persistent rules belong in `/etc/audit/rules.d/`.

## Practical Hardening Scenario

An application started by systemd is denied by SELinux while writing to `/srv/app`:

1. Check service status and the application log.
2. Inspect AVC denials with `ausearch`.
3. Confirm the file context with `ls -Z`.
4. Restore or assign the correct context.
5. Retest without weakening policy globally.

On AppArmor:

```bash
sudo aa-status
sudo journalctl -k | grep apparmor
sudo aa-logprof
```

Update the profile only after identifying repeated legitimate access.

## Common Mistakes

- Disabling SELinux instead of fixing labels or policy.
- Changing AppArmor from enforce to complain and leaving it there.
- Granting broad Linux capabilities.
- Starting auditd with rules but never reviewing audit logs.
- Using only package examples without checking distribution support.

## Interview Questions

**Q: What is the difference between SELinux and AppArmor?**  
Both provide mandatory access control, but SELinux uses labels and policy domains, while AppArmor commonly uses path-based application profiles.

**Q: What does `getenforce` do?**  
It reports whether SELinux is enforcing, permissive, or not active.

**Q: What is a Linux capability?**  
A capability divides privileges that were previously associated with root.

## Exercises

1. Identify whether your system uses SELinux or AppArmor.
2. Run `getenforce` or `aa-status`.
3. Inspect file contexts or one AppArmor profile.
4. View audit events for a configuration file.
5. Explain why disabling SELinux is a poor first response.

**Related:** [Linux Security](../19-linux-security/) | [SSH Administration](../17-ssh-administration/) | [Troubleshooting](../21-troubleshooting/)
