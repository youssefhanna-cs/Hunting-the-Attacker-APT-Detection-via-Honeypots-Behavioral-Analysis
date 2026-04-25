# 🔍 Detection Methodology

## Philosophy

> Detection was based entirely on **behavioral correlation**, not signatures or alerts.

The attacker's traffic appeared legitimate at multiple points — the compromised machine was permitted to scan the network, and a single HTTP connection could be accidental. What confirmed the compromise was **connecting the dots across multiple events over time.**

---

## Detection Stages

### Stage 1 — Network Scan (Suspicious, Not Confirmed)

**What TH saw:**
- `192.168.1.12` performing `nmap -sS 192.168.1.0/24`
- Wireshark captured the SYN packets across the subnet

**TH Reasoning:**
- Machine is permitted to scan → suspicion cleared
- But machine is added to watchlist

---

### Stage 2 — Honeypot Access (Suspicious, Not Confirmed)

**What TH saw:**
- HTTP GET request from `192.168.1.12` to honeypot `192.168.1.11:80`
- Honeypot logs: `192.168.1.12 - GET / HTTP/1.1 200 OK`

**TH Reasoning:**
- Any honeypot access = high suspicion
- But could be an employee who accidentally reached the wrong server
- Not yet confirmed — need more evidence

---

### Stage 3 — Beaconing (Confirmed)

**What TH saw:**
- Repeated HTTP requests from `192.168.1.12` → `192.168.1.11` every 10 seconds
- Wireshark shows fixed-interval pattern — not human behavior
- Pattern continued with no user activity on the machine

**TH Reasoning:**
- Fixed interval = scripted/automated
- No legitimate application communicates with this server
- Combined with previous events: **confirmed attacker**

---

## Correlation = Confirmation

| Event | Alone | Combined |
|-------|-------|---------|
| nmap scan | Suspicious but explainable | Part of attacker pattern |
| Honeypot access | Could be accidental | Confirms intent |
| Beaconing | Unusual traffic | **Seals the case** |

> The Threat Hunter's job is not to react to one event — it's to **build a behavioral picture** until the pattern becomes undeniable.

---

## Tools & Evidence

### Wireshark — Traffic Capture
- Captured full network traffic on the lab segment
- Identified nmap SYN scan pattern
- Identified fixed-interval HTTP beaconing

**Key observations in Wireshark:**
```
192.168.1.12 → 192.168.1.11  TCP   SYN (nmap scan)
192.168.1.12 → 192.168.1.11  HTTP  GET / HTTP/1.1
192.168.1.11 → 192.168.1.12  HTTP  200 OK
[10 seconds later]
192.168.1.12 → 192.168.1.11  HTTP  GET / HTTP/1.1
192.168.1.11 → 192.168.1.12  HTTP  200 OK
[repeating...]
```

### Honeypot Server Logs
- Apache access log showed access from `192.168.1.12`
- Timestamp analysis confirmed fixed 10-second interval

### MITRE ATT&CK Framework
- Mapped each observed behavior to known techniques
- Provided structured, repeatable incident context

---

## Detection Results

| Indicator | Status |
|-----------|--------|
| nmap scan detected | ✅ |
| Honeypot access detected | ✅ |
| Beaconing pattern identified | ✅ |
| Events correlated to single source | ✅ |
| Lateral movement | ❌ Not achieved — detected first |
| False positives | ✅ Zero |
