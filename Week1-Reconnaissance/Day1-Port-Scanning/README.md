# Day 1 — Port Scanning 🔍

## 📅 Date: March 22 2026
## 🎯 Category: Reconnaissance
## ⚠️ Severity: Medium
## 🕐 Duration: 53 seconds (nmap -A)

---

## 🤔 What is Port Scanning?

Port scanning is the FIRST step every attacker takes before launching
any real attack. Like a burglar walking down a street checking every door
and window of a building before breaking in.

Every computer has 65,535 ports. Each port is like a door.
Port scanning checks which doors are open!

---

## 🛠️ Tools Used

| Tool | Purpose | Machine |
|------|---------|---------|
| Nmap 7.94SVN | Perform port scan | Kali Linux |
| Wireshark | Capture live traffic | Windows 11 |
| Splunk 10.2.1 | SIEM log detection | Windows 11 |
| Windows Event Viewer | Raw log analysis | Windows 11 |

---

## 🖥️ Lab Environment

| Role | IP Address | OS |
|------|------------|-----|
| Attacker | 192.168.56.100 (Kali) | Kali Linux |
| Attacker | 192.168.56.102 (VM) | VirtualBox VM |
| Defender/Target | 192.168.56.103 | Windows 11 Enterprise |

---

## 😈 Attack Side (Kali Linux)

### Step 1 — Network Discovery
```bash
nmap -sn 192.168.56.0/24
```

What this does:
Scans the entire /24 network to find all alive devices.
Like knocking on every door in the neighborhood!

Real Results:
```
Nmap scan at 2026-03-22 13:37 EDT
192.168.56.1   → Host is up (Router)
192.168.56.100 → Host is up (Kali Linux - Me!)
192.168.56.103 → Host is up (Windows 11 - TARGET!)
192.168.56.102 → Host is up (Another VM)

4 hosts found in 27.94 seconds
```

Finding: Target identified at 192.168.56.103 ✅

---

### Step 2 — Basic Port Scan
```bash
nmap 192.168.56.103
```

Real Results:
```
Nmap scan at 2026-03-22 13:39 EDT
Host is up (0.00046s latency)

PORT     STATE  SERVICE
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
8000/tcp open   http-alt
8089/tcp open   unknown

Nmap done in 14.43 seconds
```

Finding: 5 open ports discovered in just 14 seconds! ⚠️

---

### Step 3 — Service Version Detection
```bash
nmap -sV 192.168.56.103
```

Note: Running nmap -sv showed help menu (lowercase v issue)
Used nmap -A for full service detection instead.

---

### Step 4 — Full Aggressive Scan
```bash
nmap -A 192.168.56.103
```

Real Results:
```
Nmap scan at 2026-03-22 13:50 EDT
Host is up (0.0016s latency)

PORT     STATE  SERVICE    VERSION
135/tcp  open   msrpc      Microsoft Windows RPC
139/tcp  open   netbios-ssn Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds?
8000/tcp open   http       Splunkd httpd 🔴
8089/tcp open   ssl/http   Splunkd httpd 🔴

Computer Name: DESKTOP-I5VGKR9
OS: Windows (CPE: microsoft:windows)
Network Distance: 1 hop
Traceroute: 1.57ms to 192.168.56.103

Nmap done in 53.21 seconds
```

Shocking Finding: Attacker can see Splunk SIEM is running! 😱

---

## 👮 Defense Side (Windows 11)

### Detection Method 1 — Raw Log Analysis (Manual FIRST!)

I opened Windows Event Viewer WITHOUT any tools first!**

Went to: Windows Logs → Security

What I saw in raw logs:
```
Security Log - Number of events: 31,179
(!) New events available

Date/Time         Event ID  Description
3/22/2026 12:28   5156      Filtering Platform Connection
3/22/2026 12:28   5156      Filtering Platform Connection
3/22/2026 12:28   5156      Filtering Platform Connection
3/22/2026 12:28   5156      Filtering Platform Connection
```

Event 5156 details:
```
Event: Windows Filtering Platform permitted a connection
Process ID: 4536
Log: Security
Computer: DESKTOP-I5VGKR9
Event ID: 5156
Level: Information
Keywords: Audit Success
```

What I noticed reading RAW logs manually:
- ✅ Event ID 5156 repeating rapidly
- ✅ All events at exact same timestamp (12:28:21)
- ✅ Same second = automated tool!
- ✅ Pattern = sequential port connections

Manual Conclusion: PORT SCAN DETECTED! ✅

---

### Detection Method 2 — Wireshark Traffic Analysis

Filter applied:
```
tcp.flags.syn == 1
```

What Wireshark showed:
```
Capturing from: Ethernet 2 (Host-Only network)

Source          Destination     Protocol  Info
192.168.56.102  192.168.56.103  TCP       33698→8200 [SYN]
192.168.56.102  192.168.56.103  TCP       33698→5001 [SYN]
192.168.56.102  192.168.56.103  TCP       33698→3351 [SYN]
192.168.56.103  192.168.56.102  TCP       [RST, ACK] responses
```

What the packets revealed:
- ✅ SYN packets going to multiple ports
- ✅ RST/ACK responses (port scan signature!)
- ✅ Same source IP attacking multiple ports
- ✅ Sequential port targeting pattern

Wireshark Conclusion: SYN PORT SCAN CONFIRMED! ✅

---

### Detection Method 3 — Splunk SIEM

SPL Query used:
```
index=* 192.168.56.100
```

Time range: Last 24 hours

Splunk Results:
```
12 events found
Time range: 3/21/26 - 3/22/26

Event details:
Time: 03/22/2026 12:35:22 PM
Network Information:
  Direction: Inbound
  Source Address: 192.168.56.100 🔴
```

Splunk Conclusion: Attack source 192.168.56.100 CONFIRMED! ✅

---

## 🔓 Open Ports Discovered

| Port | Service | Version | Risk Level |
|------|---------|---------|------------|
| 135/tcp | Windows RPC | Microsoft Windows RPC | Medium |
| 139/tcp | NetBIOS | Microsoft Windows netbios-ssn | Medium ⚠️ |
| 445/tcp | SMB | Microsoft-DS | HIGH ⚠️ |
| 8000/tcp | HTTP | Splunkd httpd| HIGH 🔴 |
| 8089/tcp | HTTPS | Splunkd httpd SSL | HIGH 🔴 |

---

## 🚨 Indicators of Compromise (IOCs)

| IOC | Value | Significance |
|-----|-------|-------------|
| Source IP | 192.168.56.100 / .102 | Attack origin |
| Scan speed | 1000 ports in 14 sec | Automated tool (Nmap) |
| Port pattern | Sequential scanning | Nmap signature |
| SYN packets | No handshake completion | SYN scan type |
| Service detected | Splunkd on 8000/8089 | SIEM exposed! |
| Computer name | DESKTOP-I5VGKR9 | Identity disclosed |

---

## 💡 Key Insight — SIEM Exposed!

The most alarming finding today:

> Running Splunk on default ports 8000 and 8089 tells every attacker
> that this machine is being monitored by a SOC!

This means an attacker would:
- Know a SIEM is running and logging everything
- Target the Splunk service directly
- Try to delete logs to cover tracks
- Adjust attack behavior to evade detection

Fix: Change Splunk to custom non-standard ports!

---

## 🛡️ Recommendations

| Priority | Action |
|----------|--------|
| 🔴 HIGH | Move Splunk to custom port (not 8000/8089) |
| 🔴 HIGH | Block source IPs in Windows Firewall |
| ⚠️ MEDIUM | Disable NetBIOS if not needed |
| ⚠️ MEDIUM | Enable Windows Firewall rules |
| 🟡 LOW | Monitor for follow-up exploitation attempts |

---

## 🧠 What I Learned Today

1. Port scanning takes only 14 seconds — attacker maps everything instantly!
2. Splunk on default ports = SIEM exposed to attackers!
3. Raw log analysis (Event ID 5156) reveals attacks before any tool alerts!
4. SMB port 445 is HIGH RISK — common ransomware entry point!
5. Computer name is visible to attacker from Nmap scan!
6. Always start Wireshark BEFORE the attack — live capture only!
7. All 3 detection methods agreed — raw logs, Wireshark, and Splunk!

---

## 📸 Screenshots

| File | Description |
|------|-------------|
| 1-nmap-sn-discovery.png | Network host discovery |
| 2-nmap-basic-scan.png | 5 open ports found |
| 3-wireshark-syn-filter.png | SYN packet capture |
| 4-event-viewer-5156.png | Raw Event ID 5156 logs |
| 5-splunk-search.png | SIEM detection results |
| 6-nmap-A-results.png | Full aggressive scan |
| 7-nmap-A-splunk-found.png | Splunk detected by Nmap! |

---

## 🔗 References
- Nmap documentation: https://nmap.org
- Windows Event ID 5156 reference
- Splunk SPL query documentation

