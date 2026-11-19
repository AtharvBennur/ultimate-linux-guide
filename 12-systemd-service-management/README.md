# systemd Service Management

## What is systemd?

`systemd` is Linux's system and service manager. It starts with process ID 1 (PID 1), starts the operating system services, tracks their state, and manages dependencies between services.

A **service** is a long-running process managed as a system resource. Examples include SSH, Nginx, PostgreSQL, or a custom application.

`systemctl` is the command used to manage systemd units. A unit is a configuration object managed by systemd; service units usually end in `.service`.

## Viewing and Controlling Services

```bash
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
sudo systemctl disable nginx
```

| Command | Meaning |
|---|---|
| `status` | Show whether a service is active and recent log information |
| `start` | Start the service now |
| `stop` | Stop the service now |
| `restart` | Stop and start the service |
| `reload` | Ask the running service to reload configuration without fully restarting |
| `enable` | Start the service automatically at boot |
| `disable` | Do not start the service automatically at boot |

`enable` does not start a stopped service immediately. `start` does not enable a service for future boots. Use both when both behaviors are required.

## Unit File Basics

Unit files are commonly stored in:

- `/usr/lib/systemd/system/` or `/lib/systemd/system/` for distribution packages
- `/etc/systemd/system/` for administrator-created or overridden units
- `/run/systemd/system/` for runtime units

Inspect a unit file:

```bash
systemctl cat nginx
systemctl show nginx
```

A basic service file:

```ini
[Unit]
Description=Example Python API
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/example-app
ExecStart=/usr/bin/python3 /opt/example-app/app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

- `[Unit]` describes dependencies and metadata.
- `[Service]` defines the process, user, working directory, and restart behavior.
- `[Install]` defines how the unit is enabled.

## Practical Project: Run a Python Application as a Service

```bash
sudo useradd --system --home /opt/example-app appuser
sudo mkdir -p /opt/example-app
sudo tee /opt/example-app/app.py <<'EOF'
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"OK")

HTTPServer(("0.0.0.0", 8080), Handler).serve_forever()
EOF
```

Create `/etc/systemd/system/example-app.service` with the unit file shown above. Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable example-app
sudo systemctl start example-app
systemctl status example-app
curl http://localhost:8080
```

Run `daemon-reload` after adding or editing unit files so systemd reads the changes.

## journalctl Basics

```bash
journalctl -u nginx
journalctl -u nginx -f
journalctl -u nginx --since "10 minutes ago"
journalctl -b
```

- `-u` filters by unit.
- `-f` follows new entries.
- `-b` shows logs from the current boot.

## Troubleshooting Failed Services

```bash
systemctl status example-app
sudo systemctl status example-app --no-pager
journalctl -u example-app -n 100 --no-pager
sudo systemctl reset-failed
```

Common causes include an incorrect executable path, missing permissions, a missing environment variable, or a required port already in use.

1. Read the status and the final error in the journal.
2. Test the `ExecStart` command as the configured user.
3. Correct the unit file or application issue.
4. Run `daemon-reload` if the unit file changed.
5. Restart the service.

## Common Mistakes

- Using `enable` when only a one-time start is needed.
- Forgetting `daemon-reload`.
- Running a network application as `root` unnecessarily.
- Assuming `reload` exists for every service.
- Leaving relative paths or environment assumptions in `ExecStart`.

## Interview Questions

**Q: What is the difference between `start` and `enable`?**  
`start` starts a service now; `enable` configures it to start at boot.

**Q: Why is `daemon-reload` required?**  
Systemd has a loaded copy of unit definitions, so it must reload after unit files are added or changed.

**Q: What does `Restart=on-failure` do?**  
Systemd restarts the service only when the process exits unsuccessfully.

## Exercises

1. Check the status of SSH or Nginx.
2. Enable and start a custom service.
3. Stop it, verify it does not stop on a previously open connection, and review the journal.
4. Change the unit file and reload systemd.

**Related:** [Process Management](../07-process-management/) | [Log Management](../15-log-management/) | [Boot Process](../22-boot-process/)
