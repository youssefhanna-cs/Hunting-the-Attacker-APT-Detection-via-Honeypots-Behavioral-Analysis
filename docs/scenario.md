# 📋 Attack Scenario

## 🏢 Business Context

**TechCorp Egypt** is an enterprise with an internal network connecting multiple departments. The security team suspects the network may be compromised but has no concrete evidence.

As the **Threat Hunter**, your job is to proactively investigate — not wait for alerts.

---

## ❗ Assumed Breach

> Initial access is assumed via a **phishing email** that compromised an employee's endpoint.

The attacker has:
- Gained a foothold on an internal machine
- Established **Remote Code Execution (RCE)**
- Begun internal reconnaissance

The security team has **no alerts** — the attacker is operating stealthily.

---

## 🎯 Mission

Detect the attacker **before** they achieve their objective — using behavioral analysis and deception technology.

---

## 🪤 The Trap

A **Honeypot Server** (fake internal web service) is deployed inside the network:
- Looks like a legitimate internal web server
- Port 80 open — appears as a normal service
- **No legitimate user should ever access it**
- Any interaction = guaranteed malicious activity

---

## 🔐 Security Policy Simulated

> **Rule:** Any connection to the honeypot server is treated as a confirmed security incident.

---

## 📍 Lab Summary

| Element | Details |
|---------|---------|
| Attack Vector | Phishing (assumed) |
| Attacker Goal | Maintain persistence, internal reconnaissance |
| Detection Method | Honeypot + behavioral analysis |
| Tools | Wireshark, nmap, VirtualBox, Web Server, kali linux  |
| Outcome | Attacker detected early — before lateral movement |
