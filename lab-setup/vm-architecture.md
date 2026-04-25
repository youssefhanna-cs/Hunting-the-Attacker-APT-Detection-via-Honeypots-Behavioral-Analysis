# 🏗️ VM Architecture

## Overview

The lab runs 3 virtual machines on an **isolated internal network** using VirtualBox. No internet access — fully contained environment.

---

## VM Configuration

| VM | Role | OS | IP Address |
|----|------|----|------------|
| Victim VM / Attacker(RCE) | Compromised employee endpoint — attacker operates from inside this machine | Kali Linux | 192.168.1.12 |
| Honeypot Server | Fake internal web server | Linux (Ubuntu/Debian) | 192.168.1.11 |
| Threat Hunter VM | Monitoring & detection workstation | Kali Linux | 192.168.1.10 |


---

## Network Design

```
                                                 
[Victim VM/RCE] ─────────────────────────────────────┤
                                                 │
[Honeypot Server] ─────────── [VirtualBox Internal] ─┤
                                                 │
[Threat Hunter VM] ─────────────────────────────┘
```

All VMs on the same VirtualBox **Host-Only** or **Internal** network segment.

---

## VirtualBox Network Settings

- **Network Type:** Host-Only (or Internal Network)
- **DHCP:** Disabled — static IPs assigned manually
- **Internet Access:** None — fully isolated
- **Promiscuous Mode:** Enabled on Threat Hunter VM (required for Wireshark to capture all traffic)

---

## Key Configuration Notes

### Victim VM (Compromised — RCE)
- Treated as already compromised via phishing (assumed breach)
- Attacker operates **from inside this machine** after gaining RCE
- Runs nmap scans and beaconing script from this endpoint
- IP: `192.168.1.12`

### Honeypot Server
- Web server (Apache or Nginx) installed and running
- Port 80 open and accessible from internal network
- Logging enabled — all access logged with timestamp, source IP, request
- **No legitimate service runs on this machine**
- IP: `192.168.1.11`

### Threat Hunter VM
- Wireshark installed and running in promiscuous mode
- Monitoring the full network segment
- Filter configured for honeypot destination IP
- IP: `192.168.1.10`
