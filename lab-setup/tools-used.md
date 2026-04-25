# 🛠️ Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| VirtualBox Workstation | Latest | Lab environment isolation |
| Kali Linux | 2024.x | Attacker simulation & Threat Hunter workstation |
| Kali Linux | 2024.x | Victim machine (compromised endpoint) |
| Ubuntu/Debian | Latest | Honeypot server OS |
| Apache Web Server | 2.4.x | Honeypot service simulation |
| Wireshark | 4.x | Network traffic capture and analysis |
| MITRE ATT&CK Navigator | Web | Behavior mapping & threat profiling |

---

## Why These Tools

### VirtualBox
Provides full network isolation — no risk of real traffic contamination. Snapshots allow quick reset between tests.

### Kali Linux
Industry-standard penetration testing and security research platform. Includes Wireshark and network analysis tools out of the box.

### Apache Web Server (Honeypot)
Simple to deploy, generates detailed access logs automatically. Looks like a real service to an attacker performing reconnaissance.

### Wireshark
The most widely used network protocol analyzer. Allows real-time traffic capture, filtering, and behavioral pattern identification — essential for detecting beaconing.

### MITRE ATT&CK
Provides a common language for describing attacker behavior. Maps observed activity to known techniques for structured incident response.
