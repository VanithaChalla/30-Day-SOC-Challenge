# 🚨 Incident Report — Day 1

---

## 📋 Report Details
| Field | Details |
|-------|---------|
| Incident ID | IR-2026-001 |
| Date | March 13 2026 |
| Time | 22:14 EDT |
| Analyst | Vanitha Reddy |
| Attack Type | Port Scanning |
| Severity | MEDIUM ⚠️ |

---

## 📌 Incident Summary
A port scan was detected against
Windows 11 machine at 192.168.56.103
originating from 192.168.56.100
at 22:14 EDT on March 13 2026.
The attacker successfully mapped
all open ports and services.

---

## 🖥️ Systems Involved
| Role | IP Address | OS |
|------|------------|-----|
| Attacker | 192.168.56.100 | Kali Linux |
| Victim | 192.168.56.103 | Windows 11 |

---

## 🔍 Detection Methods

### Method 1 — Raw Log Analysis
Tool: Windows Event Viewer
Event ID: 5156

What I found manually:
- Same IP 192.168.56.100 repeating
- Different destination ports
- All within same second
- Sequential port pattern

Manual Conclusion: PORT SCAN DETECTED ✅

### Method 2 — Wireshark
Filter: tcp.flags.syn == 1

What packets showed:
- Hundreds of SYN packets
- All from 192.168.56.100
- Going to sequential ports
- No complete TCP handshakes

Wireshark Conclusion: SYN SCAN DETECTED ✅

### Method 3 — Splunk
SPL Query:
index=* EventCode=5156
| stats count by Source_Address
| sort -count

What Splunk showed:
- 500+ events from single IP
- Sequential ports targeted
- All within 2 seconds

Splunk Conclusion: PORT SCAN CONFIRMED ✅

---

## 🔓 Open Ports Discovered
| Port | Service | Risk Level |
|------|---------|------------|
| 135 | Windows RPC | Medium |
| 139 | NetBIOS | Medium ⚠️ |
| 445 | SMB | HIGH ⚠️ |
| 8000 | Splunk HTTP | HIGH 🔴 |
| 8089 | Splunk HTTPS | HIGH 🔴 |

---

## 🚨 Indicators of Compromise
| IOC | Description |
|-----|-------------|
| IP: 192.168.56.100 | Attack source |
| 1000 ports in 14 sec | Automated scan |
| Sequential ports | Nmap signature |
| SYN without ACK | SYN scan type |
| Splunk detected | SIEM exposed! |

---

## ⚠️ Severity Assessment
```
Severity: MEDIUM

Reason:
- Reconnaissance only
- No exploitation yet
- BUT sensitive info exposed!
- Splunk port visible to attacker!
- Computer name revealed!
```

---

## 🛡️ Recommendations
| Priority | Action |
|----------|--------|
| HIGH | Block IP 192.168.56.100 |
| HIGH | Move Splunk to custom port |
| MEDIUM | Disable NetBIOS if unused |
| MEDIUM | Enable Windows Firewall |
| LOW | Monitor for follow up attacks |

---

## 📸 Evidence
| Screenshot | Description |
|------------|-------------|
| 1-network-discovery.png | Host discovery |
| 2-basic-port-scan.png | Open ports found |
| 3-service-detection.png | Services identified |
| 4-aggressive-scan.png | Full scan results |
| 5-wireshark-capture.png | Traffic captured |
| 6-event-viewer-logs.png | Raw Windows logs |
| 7-splunk-detection.png | SIEM detection |

---

## 🏁 Conclusion
Full port scan reconnaissance
was performed against Windows 11.

Attacker mapped all open ports
and identified Splunk SIEM running
on default ports 8000 and 8089.

No exploitation confirmed yet
but exposed information could
enable further targeted attacks.

Immediate hardening required!

---

## ✅ Status
```
Status: RESOLVED ✅
Actions Taken:
- Documented findings
- Firewall rules recommended
- Splunk port change recommended
```
