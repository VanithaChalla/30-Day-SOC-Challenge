# Day 1 — Port Scanning 🔍

## 📅 Date: March 13 2026
## 🎯 Category: Reconnaissance
## ⚠️ Severity: Medium

---

## 🤔 What is Port Scanning?
Port scanning is the FIRST step
attackers take before any attack.
Like checking every door and window
of a building before breaking in.

Every computer has 65,535 ports.
Each port is like a door.
Port scanning checks which
doors are open!

---

## 🛠️ Tools Used
- Nmap (attack tool)
- Wireshark (traffic capture)
- Splunk (SIEM detection)
- Windows Event Viewer (raw logs)
- TCPdump (raw packet capture)

---

## 😈 Attack Side (Kali Linux)

### Step 1 - Network Discovery
```
nmap -sn 192.168.56.0/24
```
Result:
- 192.168.56.1 → Router
- 192.168.56.100 → Kali Linux
- 192.168.56.103 → Windows 11 TARGET!
- 192.168.56.102 → Another VM

### Step 2 - Basic Port Scan
```
nmap 192.168.56.103
```

### Step 3 - Service Detection
```
nmap -sV 192.168.56.103
```

### Step 4 - Full Aggressive Scan
```
nmap -A 192.168.56.103
```

---

## 🔓 Open Ports Found
| Port | Service | Risk Level |
|------|---------|------------|
| 135 | Windows RPC | Medium |
| 139 | NetBIOS | Medium ⚠️ |
| 445 | SMB | HIGH ⚠️ |
| 8000 | Splunk HTTP | HIGH 🔴 |
| 8089 | Splunk HTTPS | HIGH 🔴 |

---

## 👮 Defense Side (Windows 11)

### Method 1 — Raw Log Analysis
Opened Windows Event Viewer
WITHOUT any tools first!

Manually read Security logs
and noticed:
- Same IP appearing 500+ times
- Different ports each time
- All within same second
- Sequential port pattern

Manual Conclusion: PORT SCAN! ✅

### Method 2 — Wireshark
Filter used:
```
tcp.flags.syn == 1
```
Saw hundreds of SYN packets
from same source IP
going to sequential ports!

Wireshark Conclusion: PORT SCAN! ✅

### Method 3 — Splunk
SPL Query used:
```
index=* EventCode=5156
| stats count by Source_Address
| sort -count
```
500+ connection events
from single source IP confirmed!

Splunk Conclusion: PORT SCAN! ✅

---

## 🚨 Indicators of Compromise
- Single IP scanning 1000 ports
- All occurring in 14 seconds
- Sequential port targeting pattern
- SYN packets without completion
- Service fingerprinting detected
- Splunk SIEM exposed on network!

---

## 🧠 What I Learned Today
1. Port scanning takes only 14 seconds!
2. Splunk on default port exposes SIEM!
3. Raw logs reveal attacks before tools!
4. SMB port 445 = ransomware entry risk!
5. Computer name visible to attacker!

---

## 📸 Screenshots
See Screenshots folder!
