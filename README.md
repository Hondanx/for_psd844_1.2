
# Firewall and NAT Configuration on Ubuntu Router (R1)

## 📌 Overview

This project implements a secure firewall using `iptables` on a Linux-based router (R1). It uses **packet filtering**, **stateful inspection**, and **NAT (DNAT)** to control ICMP and TCP traffic across a simulated network with multiple subnets.

## 🧱 Topology Summary

- **R1** – Acts as the main router and firewall
- **R2 & R3** – Internal routers connecting to FTP (172.31.0.1) and Telnet (172.16.0.1) servers
- **Client (LAN)** – Connected via `192.168.56.1`, routed through `192.168.56.254` (R1)

📁 Full lab setup instructions available at:  
➡️ https://github.com/Hondanx/for_psd844

## 🔧 Firewall Configuration Highlights

### ✅ Base Configuration
```bash
# Loopback traffic
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established sessions
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

### 🚫 ICMP Filtering

| Action | Source | Destination | Result |
|--------|--------|-------------|--------|
| Ping to firewall | Any | R1 | **Blocked** |
| LAN → FTP server | 192.168.56.0/24 | 172.31.0.1 | **Rejected** |
| LAN → Telnet server | 192.168.56.0/24 | 172.16.0.1 | **Allowed** |

```bash
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
iptables -A FORWARD -s 192.168.56.0/24 -d 172.31.0.1 -p icmp --icmp-type echo-request -j REJECT
iptables -A FORWARD -d 172.16.0.1 -p icmp --icmp-type echo-request -j ACCEPT
```

### 📡 Service Rules

```bash
# FTP
iptables -A FORWARD -p tcp -d 172.31.0.1 --dport 21 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Telnet
iptables -A FORWARD -p tcp -d 172.16.0.1 --dport 23 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```

### 🔁 NAT Redirection

```bash
# FTP Control + Data
iptables -t nat -A PREROUTING -s 192.168.56.0/24 -p tcp --dport 21 -j DNAT --to-destination 172.31.0.1:21
iptables -t nat -A PREROUTING -s 192.168.56.0/24 -p tcp --dport 20 -j DNAT --to-destination 172.31.0.1:20

# Telnet
iptables -t nat -A PREROUTING -p tcp --dport 23 -j DNAT --to-destination 172.16.0.1:23
```

## 🧪 Testing & Verification

### ✅ Test Client: `192.168.56.1`

1. **Ping before rules** → success
2. **Ping after rules** → FTP server unreachable (REJECT)
3. **Telnet to R1** (23) → successfully redirected to internal server
4. **iptables -L -v -n** confirms counters on allowed/rejected packets

📸 Screenshots and video recording included in `/test_results`.

## 📂 File Structure

```
.
├── firewall_rules.sh      # iptables commands
├── test_results/
│   ├── ping_before.png
│   ├── ping_after.png
│   └── telnet_test.mp4
├── Firewall Implementation and TCP_IP Network Monitoring.pptx
├── README.md
```

## 👨‍💻 Author

Created by [HONDANX]