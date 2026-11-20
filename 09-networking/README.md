# Beginner Networking Commands

Linux networking tools help inspect interfaces, test connectivity, look up
names, and retrieve content. This chapter introduces common commands; see
[Advanced Networking](../16-advanced-networking/) for deeper coverage of IP
addressing, routing, DNS, ports, and firewalls.

## Interfaces and Routes

`ip` from the iproute2 suite displays and manages network configuration:

```bash
ip addr show
ip route
```

`ip addr show` displays interface addresses and state. `ip route` displays the
kernel's routing table, including the default route when one is configured.

## Listening Sockets

`ss` displays socket information. To list listening TCP and UDP sockets with
numeric addresses and ports:

```bash
ss -tuln
```

`ss` is the modern replacement for many `netstat` uses. Add `-p` to include
process information when you have permission to see it:

```bash
sudo ss -tulpn
```

## Test Connectivity

`ping` sends ICMP echo requests and reports replies:

```bash
ping -c 4 example.com
```

Some networks block ICMP, so a failed ping does not by itself prove a service
is unavailable.

`traceroute` attempts to show the network hops toward a destination:

```bash
traceroute example.com
```

It may require installation and some hops may not respond. Install the command
when needed:

```bash
# Debian/Ubuntu
sudo apt install traceroute

# RHEL-family
sudo dnf install traceroute
```

## HTTP Requests and Downloads

`curl` can make a request and display response headers:

```bash
curl -I https://example.com
```

Save a response body to a named file with `-o`:

```bash
curl -o page.html https://example.com
```

`wget` is commonly used to download a file using its URL:

```bash
wget https://example.com/file.zip
```

## DNS Lookups

`dig` queries DNS. For a basic lookup:

```bash
dig example.com
dig example.com A
dig example.com AAAA
```

If `dig` is not installed, install the DNS utilities package:

```bash
# Debian/Ubuntu
sudo apt install dnsutils

# RHEL-family
sudo dnf install bind-utils
```

## Practical Scenario: A Website Is Not Loading

Check the local interface and route, test name resolution, then make an HTTP
request:

```bash
ip addr show
ip route
dig example.com
curl -I https://example.com
```

If DNS resolves but the request fails, use `ping` or `traceroute` for additional
connectivity clues, while remembering that ICMP traffic may be filtered. Use
the advanced networking chapter to investigate routes, ports, and firewalls.

## Common Mistakes

- Treating `ping` failure as proof that a web service is down.
- Confusing an IP address or route problem with a DNS lookup problem.
- Assuming every hop will reply to `traceroute`.
- Using `curl -I` when you need to save the response body; use `curl -o` for that.
- Assuming `ss` can show details about every process without sufficient permissions.

## Interview Questions

**Q: What does `ip route` show?**

It displays the system's routing table, including routes used to reach other
networks.

**Q: What does `ss -tuln` list?**

Listening TCP and UDP sockets, with numeric addresses and port numbers.

**Q: Does a failed `ping` prove that a website is unreachable?**

No. ICMP may be blocked even when the website's service is reachable.

## Exercises

1. Display the addresses assigned to your network interfaces.
2. Find the default route with `ip route`.
3. List listening TCP and UDP sockets with `ss`.
4. Look up `example.com` with `dig`.
5. Request headers with `curl -I`, then save a page with `curl -o`.
6. Run `traceroute` to a host you are authorized to test and note any
   nonresponsive hops.

**Related:** [Advanced Networking](../16-advanced-networking/) | [SSH Administration](../17-ssh-administration/)
