# 🚀 Setup Guide

## Prerequisites

- VirtualBox Workstation / VirtualBox Player installed
- Kali Linux ISO
- Windows ISO (any version)
- Ubuntu/Debian ISO (for honeypot)
- Minimum: 6GB RAM, 50GB free disk

---

## Step 1 — Create the VMs

Create 3 VMs in VirtualBox:

| VM | RAM | Disk | OS |
|----|-----|------|----|
| Victim / Attacker(RCE) | 2GB | 20GB | Kali Linux |
| Honeypot | 1GB | 10GB | Ubuntu/Debian |
| Threat Hunter | 2GB | 20GB | Kali Linux |

---

## Step 2 — Configure the Network (VirtualBox)

In VirtualBox:
1. Go to **File → Tools → Network Manager**
2. Create a new **Host-Only** network (e.g., vboxnet0)
3. **Disable DHCP** on this network
4. Assign all 3 VMs to **vboxnet0**

---

## Step 3 — Assign Static IPs

On each VM, set a static IP:

```bash
# Example for Ubuntu/Kali (adjust interface name)
sudo ip addr add 192.168.100.10/24 dev eth0
sudo ip link set eth0 up
```

| VM | IP |
|----|----|
| Victim / Attacker | 192.168.1.12 |
| Honeypot | 192.168.1.11 |
| Threat Hunter | 192.168.1.10 |

---

## Step 4 — Deploy the Honeypot

On the Honeypot VM:

```bash
# Install Apache
sudo apt update
sudo apt install apache2 -y

# Start the service
sudo systemctl start apache2
sudo systemctl enable apache2

# Verify it's running
sudo systemctl status apache2

# Monitor access logs in real-time
sudo tail -f /var/log/apache2/access.log
```

The honeypot is now running on `http://192.168.1.11` — looks like a real web server.

---

## Step 5 — Enable Promiscuous Mode (Threat Hunter VM)

In VirtualBox, for the Threat Hunter VM:
1. VM Settings → Network Adapter
2. Advanced → Enable **Promiscuous Mode**

Then in the VM:
```bash
# Enable promiscuous mode on the interface
sudo ip link set eth0 promisc on

# Start Wireshark
sudo wireshark
```

---

## Step 6 — Simulate the Attack

On the **Attacker VM**, simulate beaconing from the compromised machine:

```bash
# Simulate C2 beaconing (periodic requests every 10 seconds)
while true; do
  curl http://192.168.1.11/status
  sleep 10
done
```

This simulates an attacker maintaining persistence by checking in with an internal server.

---

## Step 7 — Detect with Wireshark

On the **Threat Hunter VM**, in Wireshark:

1. Select the network interface
2. Apply filter:
```
ip.dst == 192.168.1.11 && http
```
3. Observe the periodic GET requests
4. Note the fixed time interval — this is beaconing behavior

---

## Step 8 — Check Honeypot Logs

On the **Honeypot VM**:
```bash
sudo cat /var/log/apache2/access.log
```

You should see entries like:
```
192.168.1.11 - - [date] "GET /status HTTP/1.1" 200 -
192.168.1.11 - - [date] "GET /status HTTP/1.1" 200 -
```

Same source IP, regular intervals — confirmed attacker activity.
