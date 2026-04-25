# ⚔️ Attack Flow

## Overview

The attack follows the **Cyber Kill Chain** model, starting from assumed initial access through to confirmed detection. Every step reflects real attacker behavior — and how a Threat Hunter thinks one step ahead.

---

## Phase 1 — Initial Access (Assumed)

**MITRE Technique:** T1566 — Phishing

- Employee receives a phishing email and opens a malicious attachment
- Attacker gains a foothold on the internal endpoint
- **RCE (Remote Code Execution)** established on the victim machine
- **Assumed in this lab** — we start post-compromise

> IP of compromised machine: `192.168.1.12`

---

## Phase 2 — Reconnaissance

**MITRE Technique:** T1046 — Network Service Discovery

The attacker performs an internal network scan using **Nmap** to map the environment:

```bash
sudo nmap -sS 192.168.1.0/24
```

**Scan Results:**
- `192.168.1.10` → All 1000 ports closed
- `192.168.1.11` → **Port 80/tcp OPEN (HTTP)** ← This catches the attacker's eye
- `192.168.1.12` → All 1000 ports closed (the attacker's own machine)

> From the **Threat Hunter's perspective:**
> Wireshark logs show `192.168.1.12` performing a network scan.
> Initial suspicion raised — but not yet confirmed.
> The TH verifies: this machine **is permitted** to scan the network.
> Suspicion drops — but the machine stays on the watchlist.

---

## Phase 3 — Discovery (The Trap Is Set)

The attacker notices `192.168.1.11` has **Port 80 open**.

From the attacker's perspective — this looks like a legitimate internal web server.

**What the attacker doesn't know:**
This is a **Honeypot** — a fake web server deployed by the Threat Hunter as a deception trap. No legitimate user or system should ever communicate with it.

---

## Phase 4 — Honeypot Interaction

**MITRE Technique:** T1071.001 — Application Layer Protocol: Web

The attacker connects to `192.168.1.11:80`.

```
GET / HTTP/1.1
Host: 192.168.1.11
```

The honeypot responds — the attacker believes it's a real service.

> From the **Threat Hunter's perspective:**
> Honeypot logs show access from `192.168.1.12`.
> Suspicion raised again — but still not 100% confirmed.
> Could be an employee who accidentally reached the wrong server.

---

## Phase 5 — C2 Beaconing (The Confirmation)

**MITRE Technique:** T1071.001 — Application Layer Protocol: Web

To confirm he's still inside the network, the attacker runs a simple script that sends a **beacon every 10 seconds**:

```bash
while true; do
  curl http://192.168.1.11/
  sleep 10
done
```

This is a **C2 beaconing** behavior — the attacker checks in at fixed intervals to confirm persistence.

> From the **Threat Hunter's perspective:**
> This is the confirmation.
> Wireshark shows **repeated HTTP requests from `192.168.1.12` to `192.168.1.11` every 10 seconds**.
> No legitimate user generates this pattern.
> The TH connects the dots:
> - Same machine that did the nmap scan
> - Now sending automated requests to the honeypot
> - Fixed interval = scripted behavior = attacker confirmed

---

## Phase 6 — Detection & Confirmation

The Threat Hunter correlates the events:

| Event | Source | Assessment |
|-------|--------|------------|
| nmap scan on 192.168.1.0/24 | 192.168.1.12 | Suspicious — but permitted |
| HTTP connection to honeypot | 192.168.1.12 | Suspicious — could be accidental |
| Beaconing every 10 seconds to honeypot | 192.168.1.12 | **Confirmed — attacker behavior** |

**Connecting the dots = confirmed compromise.**

---

## Timeline Summary

```
[T+0]      RCE established on 192.168.1.12 (assumed via phishing)
[T+5min]   Attacker runs: nmap -sS 192.168.1.0/24
[T+6min]   TH sees scan in Wireshark — suspects, then clears suspicion (permitted)
[T+8min]   Attacker discovers 192.168.1.11:80 open
[T+9min]   Attacker connects to honeypot → logs generated
[T+9min]   TH sees honeypot access — new suspicion raised
[T+10min]  Attacker starts beaconing script (every 10 seconds)
[T+11min]  TH sees repeated fixed-interval requests → pattern identified
[T+15min]  TH correlates: scan + honeypot + beaconing = same source
[T+16min]  Incident confirmed — 192.168.1.12 is compromised
```
