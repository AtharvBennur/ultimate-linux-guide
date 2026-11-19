# Linux Security

## Security Fundamentals

Linux security depends on identity, permissions, privilege boundaries, network controls, patching, logs, and secure configuration.

The **principle of least privilege** means giving a user or process only the access required for its task.

## Permissions Revision

Permission categories are:

- `u` owner
- `g` group
- `o` others
- `rwx` read, write, and execute

```bash
ls -l
chmod 750 /srv/app
chown app:app /srv/app/app.log
chgrp developers /srv/app
```

- `chmod` changes permissions.
- `chown` changes owner and optionally group.
- `chgrp` changes group.

Symbolic and numeric examples:

```bash
chmod u+x deploy.sh
chmod g-w app.log
chmod 640 app.log
```

## Special Permission Bits

| Bit | Effect |
|---|---|
| SUID | Execute a program with file owner's effective privileges |
| SGID | Execute with group privileges or inherit group in directories |
| Sticky | Only file owner, directory owner, or root can remove entries in a shared directory |

Examples:

```bash
chmod u+s /usr/local/bin/app
chmod g+s /srv/shared
chmod 1777 /srv/public
```

Sticky-bit directories are useful for shared temporary directories. SUID and SGID should be reviewed carefully because they expand effective privilege.

Find special permissions:

```bash
find / -perm /6000 2>/dev/null
find / -perm -1000 2>/dev/null
```

## umask

`umask` defines permission bits removed when files and directories are created.

```bash
umask
umask 027
```

With `umask 027`, directories commonly receive `750` and files receive `640`.

Set a process-level default interactively for testing, and a persistent default in shell or application configuration when required.

## ACL Fundamentals

ACLs allow permissions for specific users or groups beyond standard owner/group/other permissions.

```bash
sudo apt install acl
getfacl /srv/app
sudo setfacl -m u:alice:rwx /srv/app
sudo setfacl -m g:developers:rx /srv/app
getfacl /srv/app
sudo setfacl -x u:alice /srv/app
sudo setfacl -b /srv/app
```

`setfacl -b` removes extended ACL entries.

## sudo and sudoers

`sudo` executes authorized administrative commands.

```bash
sudo whoami
sudo systemctl restart nginx
sudo -l
sudo visudo
sudo -u appuser whoami
```

`visudo` safely checks `/etc/sudoers` syntax before saving.

Example entry:

```text
deploy ALL=(root) /bin/systemctl restart nginx, /bin/systemctl status nginx
```

Place site-specific configuration in `/etc/sudoers.d/deploy` when supported.

```bash
sudo visudo -f /etc/sudoers.d/deploy
```

Never grant `ALL=(ALL) ALL` unless the user fully requires root administrative access.

## User Account Security

```bash
sudo passwd -l alice
sudo passwd -u alice
sudo usermod -L alice
sudo usermod -U alice
sudo chage -l alice
```

- `-l`/`usermod -L` lock password authentication.
- `-u`/`usermod -U` unlock it.
- `chage -l` shows password aging.

Basic password policy configuration is commonly in `/etc/login.defs` and PAM files, but the exact setup varies by distribution.

## SSH Hardening

In `/etc/ssh/sshd_config`:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowUsers ubuntu deploy
```

Restart only after testing configuration:

```bash
sudo sshd -t
sudo systemctl restart ssh
```

Key permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys ~/.ssh/id_ed25519
chmod 400 deploy.pem
```

## Firewall Basics

```bash
sudo ufw status
sudo ufw default deny incoming
sudo ufw allow from 10.0.0.0/24 to any port 22
sudo ufw enable
```

For remote servers, allow SSH before enabling the firewall. AWS security groups must also permit required traffic.

## Practical Scenarios

### Give a User Access to a Specific Directory

```bash
sudo groupadd developers
sudo usermod -aG developers alice
sudo chown -R root:developers /srv/project
sudo chmod -R 2770 /srv/project
```

The SGID bit preserves the directory's group on new files.

### Allow a User to Restart Nginx

Use a narrow sudoers entry instead of granting unrestricted sudo.

### Troubleshoot Permission Denied

```bash
namei -l /srv/app/file
ls -ld /srv /srv/app
ls -l /srv/app/file
getfacl /srv/app/file
id alice
sudo -u alice test -r /srv/app/file
```

Check every parent directory, active groups, ACLs, mount options, SELinux/AppArmor, and file ownership.

## Security Best Practices

- Patch systems and applications promptly.
- Use SSH keys and disable direct root login.
- Limit open ports and inbound sources.
- Use fail2ban or equivalent controls where appropriate.
- Apply least privilege to users, sudo rules, cloud roles, and service accounts.
- Review audit and authentication logs.
- Never store credentials in shell history or source control.
- Back up and test recovery.

## Common Mistakes

- Using `chmod 777`.
- Adding users to broad sudo groups instead of narrow roles.
- Forgetting that parent-directory permissions affect access.
- Disabling password login before confirming key access.
- Changing SSH ports without updating firewalls.

## Interview Questions

**Q: What does least privilege mean?**  
Grant the minimum access needed and remove it when no longer needed.

**Q: What is the purpose of `visudo`?**  
It edits sudoers configuration safely and validates syntax.

**Q: Why are private SSH key permissions important?**  
Unprotected private keys can be used by another user or process to impersonate the key owner.

## Exercises

1. Create a group and grant it read/write access to a project directory.
2. Create an ACL for one user.
3. Configure a sudo rule that permits only Nginx restart/status commands.
4. Check and reduce permissions on an SSH key.

**Related:** [File Permissions](../06-file-permissions/) | [SSH](../17-ssh-administration/) | [User Management](../03-user-management/)
