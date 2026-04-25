# 🔎 Attacker Activity

## VM Details

| Machine | IP | Role |
|---------|-----|------|
| Attacker / Compromised | 192.168.1.12 | Post-RCE endpoint |
| Honeypot Server | 192.168.1.11 | Fake web server (Port 80) |
| Other internal host | 192.168.1.10 | No open ports |

---

## Observed Behavior — Chronological

### 1. Network Reconnaissance
```bash
sudo nmap -sS 192.168.1.0/24
```

**Output:**
- `192.168.1.10` → All 1000 ports closed
- `192.168.1.11` → Port 80/tcp OPEN — HTTP ← Target identified
- `192.168.1.12` → All 1000 ports closed (attacker's own machine)

---

### 2. Honeypot Interaction
- Attacker connects to `192.168.1.11:80`
- HTTP GET request sent
- Server responds — attacker believes it's a real internal service

---

### 3. C2 Beaconing Script
```bash
while true; do
  curl http://192.168.1.11/
  sleep 10
done
```

- Sends HTTP request every 10 seconds
- Confirms attacker is still inside the network
- Creates the beaconing pattern that exposes the compromise

---

## MITRE ATT&CK Mapping

| Observed Activity | Technique | ID |
|------------------|-----------|-----|
| Phishing compromise (assumed) | Phishing | T1566 |
| RCE on endpoint | Command & Scripting Interpreter | T1059 |
| nmap internal scan | Network Service Discovery | T1046 |
| HTTP connection to honeypot | Application Layer Protocol: Web | T1071.001 |
| Beaconing every 10 seconds | Application Layer Protocol: Web | T1071.001 |
