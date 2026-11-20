# Linux Fundamentals

## What Is Linux?

Linux is an open-source operating-system kernel. In everyday use, "Linux" often
refers to a complete operating system built around that kernel, together with
system utilities, libraries, and applications.

## Kernel, Shell, and Distribution

- **Kernel:** The core of the operating system. It manages hardware, memory,
  processes, and communication between software and devices.
- **Shell:** A program that reads commands and starts other programs. Bash is a
  common shell; it is separate from the kernel.
- **Distribution (distro):** An operating system built around the Linux
  kernel, with user-space tools, package management, defaults, and a release
  process. Ubuntu, Debian, Fedora, and Red Hat Enterprise Linux are examples.

Use these commands to see kernel and distribution information:

```bash
uname -r
cat /etc/os-release
```

## Linux over Windows

### Cost-Effectiveness
- **Free and Open Source**: Linux does not require expensive licensing fees, making it a cost-effective choice for companies.
- **Lower Maintenance Costs**: Linux is stable and requires minimal maintenance, reducing operational expenses.

### Performance and Efficiency
- **Better Resource Utilization**: Linux is lightweight and consumes fewer system resources compared to Windows.
- **High Scalability**: Linux efficiently scales from small embedded systems to enterprise data centers without performance degradation.

### Security and Reliability
- **Less Vulnerable to Malware**: Linux has strong user privilege separation, making it more secure against viruses and malware.
- **Frequent and Transparent Updates**: Regular security patches ensure system stability without requiring frequent reboots.
- **High Stability**: Linux systems can run for years without crashes, ensuring better uptime and reliability.
