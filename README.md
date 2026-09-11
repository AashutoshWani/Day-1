# Fortune Cloud Technologies — Day 1 Tasks

Linux networking fundamentals and AWS EC2 network troubleshooting — 5 practical tasks covering IP configuration, IPv4 analysis, dynamic IP behavior, cloud server connectivity, and systematic network troubleshooting.

## 📋 Overview

| Task | Title | Environment |
|---|---|---|
| 1 | Linux IP Investigation | Linux (EC2) |
| 2 | IPv4 Address Analysis | Linux (EC2) |
| 3 | Dynamic IP Investigation | AWS EC2 |
| 4 | Cloud Linux Server & IP | AWS EC2 |
| 5 | Cloud Network Troubleshooting | AWS EC2 |

All tasks were completed on an AWS EC2 Linux instance, accessed via SSH.

## 🗂️ Repository Structure

```
├── README.md
└── screenshots/
    ├── task1/
    ├── task2/
    ├── task3/
    ├── task4/
    └── task5/
```

---

## Task 1 — Linux IP Investigation

**Objective:** Identify the system's IP address, active network interface, default gateway, DNS configuration, and full network configuration.

### Commands Used
```bash
# IP address
ip addr show
hostname -I

# Network interface
ip link show
ip route show default

# Default gateway
ip route show default

# DNS information
cat /etc/resolv.conf
resolvectl status

# Full network configuration
nmcli device show <interface>
```

### Screenshots
| # | Description | File |
|---|---|---|
| 1.1 | `ip addr show` output — IP address | `screenshots/task1/01-ip-addr.png` |
| 1.2 | `ip link show` — active interface | `screenshots/task1/02-ip-link.png` |
| 1.3 | `ip route show default` — gateway | `screenshots/task1/03-gateway.png` |
| 1.4 | `cat /etc/resolv.conf` — DNS config | `screenshots/task1/04-dns.png` |
| 1.5 | `nmcli device show` — full config | `screenshots/task1/05-full-config.png` |

### Result
Successfully identified the instance's private IP, active interface (`eth0`), default gateway, and DNS resolver configuration.

---

## Task 2 — IPv4 Address Analysis

**Objective:** Break the IPv4 address into its four octets, determine private vs. public status, and identify other reachable devices.

### Commands Used
```bash
# IPv4 address
ip -4 addr show

# Octet / subnet breakdown
ipcalc 172.31.20.15/20

# Public IP
curl ifconfig.me
curl -s ipinfo.io

# Other devices on the network
ip neigh
arp -a
```

### IP Address Breakdown

Example address: `172.31.20.15`

| Octet Position | Value | Binary |
|---|---|---|
| 1st | 172 | 10101100 |
| 2nd | 31 | 00011111 |
| 3rd | 20 | 00010100 |
| 4th | 15 | 00001111 |

### Private vs. Public
- **Private IP (inside Linux):** `172.31.20.15` — falls within the `172.16.0.0 – 172.31.255.255` RFC 1918 private range.
- **Public IP (via `curl ifconfig.me`):** the instance's internet-facing address, confirmed against AWS's NAT/Internet Gateway.
- `ipcalc` confirmed this directly, labeling the address space as **Private Use**.

### Screenshots
| # | Description | File |
|---|---|---|
| 2.1 | `ipcalc` output — octets, netmask, network/broadcast | `screenshots/task2/01-ipcalc.png` |
| 2.2 | `curl ifconfig.me` — public IP | `screenshots/task2/02-public-ip.png` |
| 2.3 | `ip neigh` / `arp -a` — other devices | `screenshots/task2/03-other-devices.png` |

### Result
Confirmed the private IP `172.31.20.15/20` (network `172.31.16.0/20`, broadcast `172.31.31.255`, 4094 usable hosts) and its corresponding public IP.

---

## Task 3 — Dynamic IP Investigation

**Objective:** Observe how IP addressing behaves across an instance stop/start cycle and record a before/after comparison.

> **Note:** Disconnecting the network interface directly over an active SSH session would sever the connection being used to run the command. Instead, this was demonstrated safely by stopping and starting the EC2 instance via the AWS Console — a more realistic simulation of dynamic IP behavior in a cloud environment.

### Commands Used
```bash
# Before — record current state
echo "BEFORE - Private IP: $(hostname -I) | Public IP: $(curl -s ifconfig.me)" > ~/ip_record.txt

# Check connection status
nmcli device status

# [Instance stopped and restarted via AWS Console]

# After — reconnect via SSH using new public IP, then record
echo "AFTER  - Private IP: $(hostname -I) | Public IP: $(curl -s ifconfig.me)" >> ~/ip_record.txt

# Display combined record
cat ~/ip_record.txt
```

### Terminal-Based Record

| State | Private IP | Public IP |
|---|---|---|
| Before stop | 172.31.20.15 | *(old public IP)* |
| After start | 172.31.20.15 | *(new public IP)* |

### Screenshots
| # | Description | File |
|---|---|---|
| 3.1 | `nmcli device status` — connection state | `screenshots/task3/01-device-status.png` |
| 3.2 | Instance **Stopped** in AWS Console | `screenshots/task3/02-instance-stopped.png` |
| 3.3 | Instance **Running** with new IP in AWS Console | `screenshots/task3/03-instance-restarted.png` |
| 3.4 | `cat ~/ip_record.txt` — before/after record | `screenshots/task3/04-ip-record.png` |

### Result
- **Public IP changed** after stop/start — AWS releases the public IP back to its pool on stop and assigns a new one on start (unless an Elastic IP is attached).
- **Private IP remained the same** — it is tied to the instance's network interface (ENI), not to its power state.

---

## Task 4 — Cloud Linux Server & IP

**Objective:** Launch an AWS EC2 Linux instance, connect via SSH, and compare the private/public IP shown inside Linux against the AWS Console.

### Commands Used
```bash
# Connect via SSH
chmod 400 your-key.pem
ssh -i "your-key.pem" ec2-user@<public-ip>

# Private IP (inside Linux)
hostname -I
ip -4 addr show

# Private IP (via instance metadata)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/local-ipv4

# Public IP
curl -s ifconfig.me
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/public-ipv4
```

### IP Comparison

| Source | Private IP | Public IP |
|---|---|---|
| AWS Console (Details tab) | 172.31.20.15 | *(matches below)* |
| Inside Linux (`hostname -I` / `curl ifconfig.me`) | 172.31.20.15 | *(matches above)* |

### Screenshots
| # | Description | File |
|---|---|---|
| 4.1 | Instance **Running** in AWS Console | `screenshots/task4/01-instance-running.png` |
| 4.2 | Successful SSH login | `screenshots/task4/02-ssh-login.png` |
| 4.3 | Private IP from inside Linux | `screenshots/task4/03-private-ip.png` |
| 4.4 | Public IP from inside Linux | `screenshots/task4/04-public-ip.png` |
| 4.5 | AWS Console Details tab — both IPs | `screenshots/task4/05-console-details.png` |

### Result
The private and public IPs retrieved from inside the Linux instance matched exactly with the values shown in the AWS Console, confirming consistency between the OS-level and cloud-provider-level network views.

---

## Task 5 — Cloud Network Troubleshooting

**Objective:** Systematically diagnose why a service running on a Linux cloud server is unreachable, identify the root cause, fix it, and demonstrate the working result.

### Scenario
Nginx was installed and running on the EC2 instance, but the web page was unreachable from a browser at `http://<public-ip>`.

### Commands Used
```bash
# 1. IP address
ip -4 addr show

# 2. Network interface
ip link show

# 3. Default route
ip route show

# 4. Connectivity
ping -c 4 8.8.8.8
curl -I https://www.google.com

# 5. Running services
systemctl status nginx

# 6. Listening ports
sudo ss -tulnp

# 7. Firewall / security rules
sudo firewall-cmd --list-all      # or: sudo ufw status
# + check Security Group inbound rules in AWS Console

# 8. Fix — add inbound rule for port 80 in the Security Group (via Console)

# 9. Verify the fix
curl -I http://localhost
curl -I http://<public-ip>
```

### Troubleshooting Documentation

| Field | Detail |
|---|---|
| **Problem** | Web service on port 80 unreachable from outside the instance |
| **Command Used** | `ip addr show`, `ip link show`, `ip route show`, `ping 8.8.8.8`, `systemctl status nginx`, `sudo ss -tulnp` |
| **Output** | Interface UP, routing correct, outbound connectivity fine, nginx `active (running)`, port 80 listening on `0.0.0.0` — OS-level networking fully healthy |
| **Cause** | Security Group had no inbound rule allowing port 80 (HTTP) |
| **Solution** | Added inbound rule: HTTP, port 80, source `0.0.0.0/0` |
| **Final Result** | `curl -I http://<public-ip>` returns `HTTP/1.1 200 OK`; nginx welcome page loads in browser |

### Screenshots
| # | Description | File |
|---|---|---|
| 5.1 | Failed browser load (before fix) | `screenshots/task5/01-before-fix.png` |
| 5.2 | `ip addr show` / `ip link show` / `ip route show` | `screenshots/task5/02-network-checks.png` |
| 5.3 | `ping` / `curl` — connectivity check | `screenshots/task5/03-connectivity.png` |
| 5.4 | `systemctl status nginx` — service status | `screenshots/task5/04-service-status.png` |
| 5.5 | `sudo ss -tulnp` — listening ports | `screenshots/task5/05-listening-ports.png` |
| 5.6 | Security Group — port 80 **missing** | `screenshots/task5/06-sg-before.png` |
| 5.7 | Security Group — port 80 **added** | `screenshots/task5/07-sg-after.png` |
| 5.8 | `curl -I http://<public-ip>` → `200 OK` | `screenshots/task5/08-curl-success.png` |
| 5.9 | Nginx welcome page in browser (after fix) | `screenshots/task5/09-browser-success.png` |

### Result
Root cause was an AWS Security Group missing an inbound rule for port 80. After adding the rule, the service became reachable, confirmed both via `curl` and a browser load of the nginx default page.

---

## 🧾 Key Learnings

- `ip` and `nmcli` are the modern standard for Linux network diagnostics; `ifconfig`/`route`/`netstat` are legacy but still common in exam/reference material.
- A cloud instance's **private IP** is tied to its network interface (ENI) and typically persists across stop/start, while its **public IP** is drawn from AWS's pool and usually changes — unless an Elastic IP is attached.
- Cloud network troubleshooting requires checking **both layers**: the OS (interface, routing, service, listening ports) and the cloud provider's controls (Security Groups) — a fully healthy OS-level stack can still be unreachable due to a missing cloud firewall rule.

## 🛠️ Environment

- **Cloud Platform:** AWS EC2
- **OS:** Amazon Linux 2023 / Ubuntu 22.04
- **Access:** SSH (key pair authentication)
