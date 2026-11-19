# Package Management

## What Is Package Management?

Package management installs, updates, configures, verifies, and removes software. A **package** is an archive containing files, metadata, dependencies, and installation instructions.

A **package manager** downloads packages from repositories and tracks them consistently. This prevents manual library copying and makes updates and removals predictable.

## APT and DPKG on Ubuntu/Debian

- `dpkg` installs and manages locally downloaded `.deb` packages.
- `apt` resolves repositories and dependencies and provides a higher-level interface.

```bash
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
sudo apt install nginx
sudo apt remove nginx
sudo apt purge nginx
sudo apt autoremove
```

`apt update` refreshes package indexes. It does not upgrade installed software. `apt upgrade` installs available upgrades for already installed packages.

### Search and Inspect Packages

```bash
apt search nginx
apt show nginx
apt-cache policy nginx
dpkg -l nginx
dpkg -L nginx
dpkg -S /usr/sbin/nginx
```

- `apt search` searches names and descriptions.
- `apt show` displays package metadata.
- `apt-cache policy` shows available and installed versions.
- `dpkg -S` shows which package owns a file.

## Install and Remove Software

```bash
sudo apt install curl wget
sudo apt remove curl
sudo apt purge curl
sudo apt autoremove
```

Use `-y` only in automation where confirmation is acceptable:

```bash
sudo apt install -y nginx
```

## DNF and YUM on RHEL Systems

Modern RHEL, Rocky Linux, AlmaLinux, and Fedora systems use DNF. YUM is a legacy command and, on some systems, is linked to DNF.

| Ubuntu/Debian | RHEL/Fedora |
|---|---|
| `apt update` | `dnf check-update` |
| `apt upgrade` | `dnf upgrade` |
| `apt install nginx` | `sudo dnf install nginx` |
| `apt remove nginx` | `sudo dnf remove nginx` |
| `apt search nginx` | `dnf search nginx` |
| `apt show nginx` | `dnf info nginx` |
| `dpkg -l` | `rpm -qa` |
| `dpkg -S file` | `rpm -qf file` |

DNF commonly uses RPM packages:

```bash
sudo dnf install nginx
sudo dnf remove nginx
sudo dnf upgrade
dnf info nginx
rpm -qa | grep nginx
```

## Repositories and Dependencies

A repository is a configured server from which package metadata and package files are downloaded.

Ubuntu repository configuration is commonly found in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/`.

```bash
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
```

A dependency is another package required for software to run. Package managers resolve dependencies automatically. A package can also have conflicts or optional recommended packages.

## Practical Project: Install and Verify Nginx

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
systemctl status nginx
curl http://localhost
apt-cache policy nginx
```

Expected status should show `active (running)`, and `curl` should return Nginx's HTML page.

## Updating Safely

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

For scripts and CI automation:

```bash
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y
```

Production servers should use a maintenance process, backups, and testing before upgrades.

## Troubleshooting

- **Package is unavailable:** Run `apt update` and check repository configuration.
- **Lock is busy:** Another package operation is running. Do not delete lock files while an operation is active.
- **Broken dependencies:** Run `sudo apt --fix-broken install`.
- **Package held back:** Inspect the reason with `apt list --upgradable` and `apt-cache policy`.
- **Local `.deb` will not install:** `sudo apt install ./package.deb` also resolves dependencies.

## Common Mistakes

- Running `apt upgrade` without `apt update`.
- Confusing package removal with repository removal.
- Installing random third-party repositories.
- Running upgrades without reviewing the package plan.
- Running `rm` to remove software instead of the package manager.

## Interview Questions

**Q: What is the difference between `apt update` and `apt upgrade`?**  
`update` refreshes package indexes; `upgrade` updates already installed packages.

**Q: What is a dependency?**  
A package or library required for another package to install or operate.

**Q: Which tool is lower-level, APT or DPKG?**  
DPKG is lower-level and works with `.deb` packages. APT resolves repositories and dependencies.

## Exercises

1. Search for `tree`, install it, and verify its package.
2. Check which package owns `/bin/bash`.
3. Compare `apt update` with `apt upgrade`.
4. Install Nginx and confirm the service is active.

**Related:** [Getting Started Package Management](../01-getting-started/05-package-manager.md) | [systemd](../12-systemd-service-management/)
