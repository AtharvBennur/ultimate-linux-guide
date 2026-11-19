# Linux Container Fundamentals

## What Is a Container?

A container is an isolated process or group of processes running on a Linux host. It shares the host kernel while using namespaces, control groups, filesystem layers, capabilities, and security policies.

Containers are lighter than virtual machines because they do not boot an independent operating system kernel.

| Virtual machine | Container |
|---|---|
| Has a guest kernel | Shares the host kernel |
| Hardware virtualization or a hypervisor | Linux kernel isolation features |
| Stronger machine boundary | Strong process isolation, but not a hardware boundary |
| Larger startup cost | Faster startup in many cases |

## Linux Namespaces

A namespace gives a process a private view of a system resource. Containers combine namespaces to appear isolated.

List namespaces for a process:

```bash
lsns
ls -l /proc/PID/ns
```

### PID Namespace

A PID namespace gives processes an isolated process-number view.

```bash
unshare --pid --fork --mount-proc ps -ef
```

Inside the namespace, the shell or initial process may appear as PID 1.

### Network Namespace

A network namespace has its own interfaces, routes, firewall rules, and sockets.

```bash
sudo unshare --net bash
ip addr
ip route
```

Docker creates or joins network namespaces for container networking.

### Mount Namespace

A mount namespace controls which filesystems and mounts a process can see.

```bash
sudo unshare --mount bash
mount --bind /tmp /mnt
```

The mount behavior can remain isolated when created inside the namespace.

### UTS Namespace

A UTS namespace isolates hostname and domain name.

```bash
sudo unshare --uts bash
hostname container-test
hostname
```

### IPC Namespace

An IPC namespace isolates System V semaphores, message queues, and shared memory.

### User Namespace

A user namespace maps user and group IDs to a different namespace view. It can allow a user in the container to map to a privileged ID inside, while remaining non-root outside.

Not every container runtime or host configuration enables user namespaces.

## cgroups

Control groups, or cgroups, limit and account resources for process groups.

Linux has cgroup v1 and cgroup v2:

| Cgroup v1 | Cgroup v2 |
|---|---|
| Multiple separate controller hierarchies | Unified hierarchy |
| Legacy distributions and systems | Modern default on many distributions |
| Controller paths vary | Controllers appear in a common mount |

Inspect:

```bash
mount | grep cgroup
cat /proc/self/cgroup
```

On cgroup v2:

```bash
stat -fc %T /sys/fs/cgroup
```

## CPU Limits

Common container runtimes represent CPU limits through cgroup controller values.

With Docker:

```bash
docker run --cpus="0.5" nginx
docker run --cpu-shares=512 nginx
```

- `--cpus` limits CPU quota.
- `--cpu-shares` sets relative scheduling weight.

Inspect a running container's cgroup with Docker-specific commands and `/sys/fs/cgroup`.

## Memory Limits

```bash
docker run --memory="256m" nginx
docker run --memory="256m" --memory-swap="512m" nginx
```

The container may receive OOM-killed by the runtime or kernel when it exceeds limits.

Check host kernel logs:

```bash
dmesg -T | grep -iE "oom|cgroup"
journalctl -k --since "10 minutes ago"
```

## OverlayFS

OverlayFS combines a read-only lower filesystem and a writable upper filesystem into a merged view.

Container runtimes use a storage implementation based on layers. A container image's read-only layers are below the container's writable layer.

Conceptually:

```text
Merged view
Upper writable layer
Lower read-only image layers
```

Writes in a container go to the upper layer. Deleting a lower-layer file records a whiteout in the upper layer.

## Linux Capabilities

Capabilities divide root privileges. Containers normally drop unnecessary capabilities.

Inspect container runtime capabilities with Docker:

```bash
docker run --rm alpine cat /proc/self/status | grep -i '^Cap'
docker run --rm alpine sh -c 'grep Cap /proc/self/status'
docker run --rm --cap-drop=ALL --cap-add=CHOWN alpine id
```

Avoid `--privileged`. It adds broad host-level capabilities and device access.

## Seccomp

Seccomp is a system-call filter. A container runtime can prevent a container process from invoking dangerous or unnecessary system calls.

Docker commonly applies a default seccomp profile.

```bash
docker run --rm alpine grep Seccomp /proc/self/status
```

Seccomp is complementary to namespaces, cgroups, capabilities, and MAC policies.

## How Docker Uses Linux Isolation

Docker and similar runtimes use Linux kernel features:

1. Namespaces isolate process, network, mount, hostname, IPC, and user views.
2. Cgroups limit CPU, memory, PIDs, and I/O.
3. Capabilities limit privileges.
4. Seccomp filters system calls.
5. OverlayFS or another storage driver manages image and container layers.
6. Networking creates interfaces and routes between namespaces.

Docker does not run a new kernel for every container. Containers share the host kernel.

## Simple Hands-On Demonstration

Using a Linux VM with Docker:

```bash
docker run --rm -d --name demo nginx:alpine
docker exec demo cat /proc/1/cgroup
docker exec demo cat /proc/self/status | grep -E "NSpid|Cap"
docker inspect demo
docker stop demo
```

Observe the container process from the host:

```bash
docker inspect --format '{{.State.Pid}}' demo
lsns
```

The exact PID and namespace output depend on the host and runtime.

## Common Mistakes

- Calling containers a virtual-machine replacement.
- Running containers as `--privileged` for convenience.
- Assuming container isolation eliminates kernel vulnerabilities.
- Confusing CPU shares with strict CPU quotas.
- Storing important mutable data only in the container writable layer.

## Interview Questions

**Q: What is a namespace?**  
It gives a process an isolated view of a system resource.

**Q: What is a cgroup?**  
It applies resource limits and accounting to process groups.

**Q: How do containers differ from virtual machines?**  
Containers share the host kernel; virtual machines usually run guest kernels.

**Q: How does Docker isolate container processes?**  
Docker combines namespaces, cgroups, capabilities, seccomp, filesystems, and storage drivers.

## Exercises

1. Run `lsns` and identify PID, network, and mount namespaces.
2. Create a network namespace and view its interfaces.
3. Run a container with a memory limit and inspect behavior.
4. Inspect a container process's capabilities.
5. Explain why a container may not be a security boundary for a vulnerable host kernel.

**Related:** [Kernel Management](../23-kernel-management/) | [Linux Security Hardening](../26-linux-security-hardening/) | [Troubleshooting](../21-troubleshooting/)
