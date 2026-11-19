# SSH Administration

## What Is SSH?

SSH (Secure Shell) is a protocol for encrypted remote administration and file transfer. It uses a client, server, and authenticated identity.

- The **SSH client** is the `ssh` command used to connect.
- The **SSH server** listens for connections.
- **sshd** is the server process on Linux.

SSH normally uses TCP port 22.

## How SSH Works

1. The client connects to the server.
2. The server presents a host identity.
3. The client verifies the host key.
4. The client authenticates with a password or key pair.
5. An encrypted session is created.

The client stores accepted server identities in `~/.ssh/known_hosts`. The server stores authorized public keys in `~/.ssh/authorized_keys`.

## Password and Key Authentication

| Password authentication | Key authentication |
|---|---|
| User enters a secret | Private key signs an authentication challenge |
| Server verifies password | Server verifies against the public key |
| Easier initially | Better for automation and hardening |

The public key may be shared. The private key must remain secret and should be protected by a passphrase.

## Creating SSH Keys

```bash
ssh-keygen -t ed25519 -C "devops-workstation"
```

Files are commonly:

- `~/.ssh/id_ed25519` for the private key
- `~/.ssh/id_ed25519.pub` for the public key

For RSA compatibility:

```bash
ssh-keygen -t rsa -b 4096 -C "devops-workstation"
```

## Copying a Public Key

```bash
ssh-copy-id ubuntu@server.example.com
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@server.example.com
```

Manual alternative:

```bash
cat ~/.ssh/id_ed25519.pub
ssh ubuntu@server.example.com
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## Connecting

```bash
ssh ubuntu@10.0.0.10
ssh -i ~/.ssh/my-key.pem ubuntu@10.0.0.10
ssh -p 2222 ubuntu@10.0.0.10
```

The PEM file used for AWS EC2 must be private and secure:

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ubuntu@ec2-public-ip
```

Use the EC2 user name for the distribution: Ubuntu commonly uses `ubuntu`, Amazon Linux commonly uses `ec2-user`, and RHEL commonly uses `ec2-user`.

## SCP and SFTP

```bash
scp file.txt ubuntu@10.0.0.10:/tmp/
scp ubuntu@10.0.0.10:/var/log/syslog ./syslog-copy
sftp ubuntu@10.0.0.10
```

Inside SFTP:

```text
ls
put file.txt
get remote.log
exit
```

Modern alternatives include `rsync -e ssh`, but the course-level objective is to understand SCP and SFTP.

## SSH Configuration

Client configuration is under `~/.ssh/config`:

```sshconfig
Host web1
    HostName 10.0.0.10
    User ubuntu
    IdentityFile ~/.ssh/devops.pem
```

Use it with:

```bash
ssh web1
```

The server configuration is `/etc/ssh/sshd_config` and commonly includes:

```text
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
```

Before disabling password authentication, verify key login in a separate session.

## Permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config
```

Incorrect ownership or permissions can cause SSH to reject keys.

## Changing the SSH Port

Edit `/etc/ssh/sshd_config`, choose an unused port, and open it in the local firewall and cloud security group.

```bash
sudo ss -tulpn | grep ssh
sudo systemctl restart ssh
sudo systemctl status ssh
```

On RHEL-family systems the service may be named `sshd`.

Before applying changes, keep an existing session open. Test access on the new port before closing it.

## Practical Project: Passwordless SSH

On server A:

```bash
ssh-keygen -t ed25519
ssh-copy-id ubuntu@SERVER_B_IP
ssh ubuntu@SERVER_B_IP
```

Do not copy the private key from server A. Keep it in `~/.ssh` with mode `600`.

## Troubleshooting

```bash
ssh -vvv ubuntu@10.0.0.10
sudo tail -f /var/log/auth.log
sudo journalctl -u ssh
sudo ss -tulpn | grep 22
```

On RHEL:

```bash
sudo journalctl -u sshd
sudo tail -f /var/log/secure
```

Common checks:

- Security group allows TCP 22 or the custom port.
- Server firewall allows SSH.
- Server is listening.
- Client uses the correct private key.
- Public key is in `authorized_keys`.
- `~/.ssh` and key files have correct permissions.
- Host key changed or was regenerated.

When a host key warning appears:

```bash
ssh-keygen -R 10.0.0.10
```

Only remove the old entry after verifying that the server identity changed legitimately.

## Common Mistakes

- Setting `PasswordAuthentication no` before testing key access.
- Distributing private keys.
- Using a permission of `777` on `.ssh`.
- Closing the only SSH session after changing the port.
- Forgetting security groups when changing ports.

## Interview Questions

**Q: What is `sshd`?**  
The SSH server daemon that accepts remote connections.

**Q: Which key is kept secret?**  
The private key.

**Q: Which file contains server-authorized public keys?**  
`~/.ssh/authorized_keys`.

## Exercises

1. Generate an ED25519 key pair.
2. Configure SSH access to an EC2 instance with a PEM file.
3. Create an SSH config alias.
4. Copy a file using SCP and inspect server authentication logs.

**Related:** [Advanced Networking](../16-advanced-networking/) | [Linux Security](../19-linux-security/)
