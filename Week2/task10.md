# TASK 10 – LINUX NETWORKING

## OBJECTIVE

The objective of this task is to investigate Linux networking by identifying the private IP address, public IP address, default gateway, DNS server, open ports, and listening services. It also explains the differences between `ss` and `netstat`, `ping` and `traceroute`, and TCP and UDP.

---

# COMMANDS USED

## Display Private IP Address

```bash
ip addr show
hostname -I
```

## Display Public IP Address

```bash
curl ifconfig.me
```

## Display Default Gateway

```bash
ip route
```

## Display DNS Server

```bash
cat /etc/resolv.conf
```

or

```bash
resolvectl status
```

## Display Open Ports

```bash
sudo ss -tuln
```

## Display Listening Services

```bash
sudo ss -tulpn
```

## Install netstat

```bash
sudo apt update
sudo apt install net-tools -y
```

## Display Open Ports using netstat

```bash
netstat -tuln
```

## Test Connectivity

```bash
ping google.com
```

## Install Traceroute

```bash
sudo apt install traceroute -y
```

## Trace Network Route

```bash
traceroute google.com
```

---

# EXPLANATION

- `ip addr show` displays all network interfaces and assigned IP addresses.
- `hostname -I` shows the system's private IP address.
- `curl ifconfig.me` retrieves the public IP address from an external service.
- `ip route` displays the routing table and identifies the default gateway.
- `cat /etc/resolv.conf` and `resolvectl status` display DNS server information.
- `ss -tuln` lists open TCP and UDP ports.
- `ss -tulpn` lists listening services with associated processes.
- `netstat -tuln` displays open ports using the older net-tools package.
- `ping` checks whether a remote host is reachable.
- `traceroute` shows each network hop between the local system and a destination.

---

# DIFFERENCE BETWEEN ss AND netstat

| ss | netstat |
|----|----------|
| Modern tool | Legacy tool |
| Faster | Slower |
| Installed by default | Requires net-tools |
| Recommended | Compatibility only |

---

# DIFFERENCE BETWEEN ping AND traceroute

| ping | traceroute |
|-------|------------|
| Checks connectivity | Displays network path |
| Measures response time | Shows intermediate routers |
| Uses ICMP | Uses UDP/ICMP |

---

# DIFFERENCE BETWEEN TCP AND UDP

| TCP | UDP |
|-----|-----|
| Reliable | Faster |
| Connection-oriented | Connectionless |
| Error checking | Minimal error checking |
| Guarantees delivery | No delivery guarantee |

---

# OUTPUT

Example outputs:

```text
Private IP : 192.168.1.105
Public IP : 103.xxx.xxx.xxx
Default Gateway : 192.168.1.1
DNS Server : 8.8.8.8
```

---

# SCREENSHOTS

Capture:

1. Private IP
2. Public IP
3. Default Gateway
4. DNS Server
5. Open Ports
6. Listening Services
7. netstat
8. ping
9. traceroute

Store screenshots in:

```text
Week2/Screenshots/Task10/
```

---

# ERRORS ENCOUNTERED

## Error

```
netstat: command not found
```

### Resolution

```bash
sudo apt install net-tools
```

---

## Error

```
traceroute: command not found
```

### Resolution

```bash
sudo apt install traceroute
```

---

# LEARNING SUMMARY

Through this task, I learned how to identify important network configuration details such as private and public IP addresses, default gateways, DNS servers, open ports, and listening services. I also learned the differences between `ss` and `netstat`, `ping` and `traceroute`, and TCP and UDP, along with the purpose of each networking command used in Ubuntu.
