# Advanced Networking

## IP Addressing

An **IPv4 address** identifies a device on an IP network. An address has a network portion and a host portion.

- `10.0.0.10/24` is in the `10.0.0.0/24` network.
- Private addresses are used inside networks and are not routed directly over the public internet.
- Public addresses are reachable on the internet where network rules permit.

Examples:

```text
Private: 10.0.0.10
Public:  203.0.113.10
```

Documentation addresses such as `203.0.113.10` are not necessarily real public hosts.

## Subnet and CIDR

CIDR notation identifies how many leading address bits belong to the network:

| CIDR | Subnet mask | Usable hosts in a conventional IPv4 subnet |
|---|---|---|
| `/24` | 255.255.255.0 | 254 |
| `/25` | 255.255.255.128 | 126 |
| `/27` | 255.255.255.224 | 30 |
| `/30` | 255.255.255.252 | 2 |

The default gateway sends traffic to networks outside the local subnet.

## Network Commands

```bash
ip addr
ip link
ip route
ip neigh
```

- `ip addr` shows addresses.
- `ip link` shows interface state.
- `ip route` shows the routing table.
- `ip neigh` shows neighboring IP and MAC addresses.

Inspect TCP and UDP listeners:

```bash
ss -tulpn
netstat -tulpn
sudo lsof -iTCP -sTCP:LISTEN
```

`ss` is the modern replacement for many `netstat` uses.

## TCP and UDP

- TCP provides ordered, reliable connections and is used by HTTP, SSH, and databases.
- UDP provides connectionless datagrams and is used by DNS, streaming, and some monitoring traffic.

Port numbers identify applications or services:

| Port | Protocol |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 53 | DNS |
| 8080 | Application |

## DNS

DNS maps names to records.

- `A` maps a name to IPv4.
- `AAAA` maps a name to IPv6.
- `CNAME` maps a name to another name.

```bash
dig example.com A
dig example.com AAAA
dig example.com CNAME
nslookup example.com
host example.com
getent hosts example.com
```

If DNS fails, inspect `/etc/resolv.conf` and the resolver response:

```bash
cat /etc/resolv.conf
systemd-resolve --status
```

## Connectivity Testing

```bash
ping example.com
traceroute example.com
curl -I https://example.com
curl -v http://10.0.0.10:8080/health
wget https://example.com/file
```

- `ping` tests ICMP connectivity.
- `traceroute` shows hops.
- `curl` tests HTTP requests and headers.
- `wget` downloads files.

## Netcat and Telnet

```bash
nc -vz 10.0.0.20 8080
telnet 10.0.0.20 22
```

`nc -vz` tests whether a TCP port accepts connections. `telnet` is also useful for basic protocol checks, but SSH or encrypted tools should be used for remote administration.

## Firewall Basics

UFW is a simple firewall interface on Ubuntu:

```bash
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw delete allow 80/tcp
```

NFTables is the modern Linux packet-filter framework. Iptables is still common on older systems and is often backed by nftables compatibility.

```bash
sudo nft list ruleset
sudo iptables -L -n -v
```

AWS also uses security groups and network ACLs. On EC2, Linux firewall rules and AWS rules must both allow traffic.

## Real-World Troubleshooting

### Website Is Not Opening

```bash
ip addr
ip route
ss -tulpn | grep 80
curl -I http://localhost
curl -I http://PUBLIC_IP
sudo ufw status
```

Check process state, listener, local curl, host firewall, cloud firewall, DNS, and public reachability.

### Port 8080 Is Not Accessible

```bash
ss -ltnp | grep 8080
nc -vz 127.0.0.1 8080
nc -vz PRIVATE_IP 8080
```

Confirm the application listens on `0.0.0.0` or the expected interface, not only `127.0.0.1`.

### DNS Is Not Resolving

```bash
getent hosts example.com
dig example.com
cat /etc/resolv.conf
```

Distinguish DNS failure from routing failure. Check TTL and record type as well.

### EC2 Instances Cannot Communicate

1. Verify private IP addresses and subnet routes.
2. Verify both instances are in the same route path or appropriate network.
3. Check security groups and network ACLs.
4. Check Linux firewalls.
5. Test with `ping` or `nc`.

## Common Mistakes

- Testing only from the remote side without testing locally.
- Leaving an application bound only to localhost.
- Configuring a security group but forgetting the subnet ACL or OS firewall.
- Using `/24` assumptions across AWS subnets.
- Confusing DNS failure with a TCP connection failure.

## Interview Questions

**Q: What is the default gateway?**  
The router used when sending traffic outside the local network.

**Q: Why can a service be running but unreachable?**  
It may bind only to localhost, its port may be blocked, or cloud firewall rules may deny traffic.

**Q: What is the difference between TCP and UDP?**  
TCP provides reliable ordered streams; UDP sends connectionless datagrams.

## Exercises

1. Display the active interface and default route.
2. Find all listening TCP ports.
3. Resolve `example.com` using `dig`.
4. Test `localhost` on a known listening port.

**Related:** [Basic Networking](../09-networking/) | [SSH](../17-ssh-administration/) | [Troubleshooting](../21-troubleshooting/)
