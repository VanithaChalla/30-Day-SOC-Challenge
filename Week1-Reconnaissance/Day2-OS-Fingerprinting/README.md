
# Day 2 — OS Fingerprinting 🖥️

## 📅 Date: March 24 2026
## 🎯 Category: Reconnaissance
## ⚠️ Severity: Medium
## 🕐 Duration: Multiple scans

---

## 🤔 What is OS Fingerprinting?

OS Fingerprinting is the technique of identifying what **Operating System**
is running on a target machine WITHOUT logging in!

Every OS leaves unique "fingerprints" in network traffic:

```
Windows  → TTL = 128
Linux    → TTL = 64
Mac      → TTL = 64
```

Just from ONE ping command an attacker knows exactly
what OS you are running! One number reveals everything! 🔥

---

## 🛠️ Tools Used

| Tool | Purpose | Machine |
|------|---------|---------|
| Nmap 7.94SVN | OS detection scanning | Kali Linux |
| Ping | TTL value check | Kali Linux |
| Wireshark | Capture ICMP + TTL traffic | Windows 11 |
| Splunk 10.2.1 | SIEM log detection | Windows 11 |
| Windows Event Viewer | Raw log analysis | Windows 11 |

---

## 🖥️ Lab Environment

| Role | IP Address | Hostname | OS |
|------|------------|---------|-----|
| Attacker | 192.168.56.102 | Kali Linux | Kali Linux |
| Defender/Target | 192.168.56.103 | DESKTOP-I5VGKR9 | Windows 11 |

---

## 😈 Attack Side (Kali Linux)

### Step 1 — OS + Service Detection
```bash
nmap -O -sV 192.168.56.103
```

**What this does:**
Detects the Operating System AND all running services!

**Real Results:**
```
Nmap scan at 2026-03-22 22:27 EDT
Host is up (0.0013s latency)

PORT     STATE  SERVICE    VERSION
135/tcp  open   msrpc      Microsoft Windows RPC
139/tcp  open   netbios-ssn Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds?
8000/tcp open   http       Splunkd httpd
8089/tcp open   ssl/http   Splunkd httpd

TCP/IP fingerprint detected!
Service Info: OS: Windows
CPE: cpe:/o:microsoft:windows

Nmap done in 45.64 seconds
```

**Finding:** Target confirmed as **Windows OS** ✅

---

### Step 2 — TTL Check (Ping)
```bash
ping 192.168.56.103
```

**What this does:**
Sends ICMP packets and checks TTL value to identify OS!

**Real Results:**
```
PING 192.168.56.103 (192.168.56.103) 56(84) bytes
64 bytes from 192.168.56.103: icmp_seq=1 ttl=128 time=2.10ms
64 bytes from 192.168.56.103: icmp_seq=2 ttl=128 time=2.20ms
64 bytes from 192.168.56.103: icmp_seq=3 ttl=128 time=1.60ms
...every single packet ttl=128!
```

**KEY FINDING:**
```
TTL = 128 = WINDOWS! ✅

This single number tells attacker:
→ Target is Windows machine!
→ No doubt! 100% certain!
→ Now picks Windows-specific exploits!
```

---

### Step 3 — Aggressive OS Guess
```bash
nmap -O --osscan-guess 192.168.56.103
```

**What this does:**
Forces Nmap to make its best guess at OS version even
if not 100% certain!

**Real Results:**
```
Nmap scan at 2026-03-22 22:46 EDT
Host is up (0.0023s latency)

Aggressive OS guesses:
→ Microsoft Windows 10 1703 (99%) 🔴
→ Microsoft Windows 10 1507-1607 (97%)
→ Microsoft Windows 10 1511 (97%)
→ Microsoft Windows Longhorn (95%)
→ Microsoft Windows 10 10586-14393 (94%)
→ Microsoft Windows Server 2008 (94%)

Network Distance: 1 hop
Nmap done in 24.07 seconds
```

**SHOCKING FINDING:**
```
Attacker gets 99% confidence
on specific Windows version!

Windows 10 1703 (99%) means:
→ Attacker knows EXACT build!
→ Can search CVE database for
  vulnerabilities specific to
  Windows 10 version 1703!
→ Targeted attack is now possible!
```

---

### Step 4 — Full TCP/IP Fingerprinting
```bash
nmap -A 192.168.56.103
```

**Real Results:**
```
Nmap scan at 2026-03-22 22:52 EDT
Host is up (0.0012s latency)

PORT     STATE SERVICE    VERSION
135/tcp  open  msrpc      Microsoft Windows RPC
139/tcp  open  netbios-ssn Microsoft Windows
445/tcp  open  microsoft-ds?
8000/tcp open  http       Splunkd httpd
8089/tcp open  ssl/http   Splunkd httpd

Service Info: OS: Windows
CPE: cpe:/o:microsoft:windows

Host script results:
NetBIOS name: DESKTOP-I5VGKR9
smb2-security-mode: Message signing enabled
smb2-time: 2026-03-23T03:52:17

TRACEROUTE:
HOP RTT     ADDRESS
1   1.23ms  192.168.56.103

Nmap done in 54.17 seconds
```

---

## 👮 Defense Side (Windows 11)

### Detection Method 1 — Raw Log Analysis (Manual FIRST!)

**Opened Event Viewer WITHOUT any tools!**

Went to: Windows Logs → Security

**What I saw in raw logs:**
```
Security Log - 30,556 events
(!) New events available

Date/Time           Event ID  Description
3/22/2026 8:52:50 PM  5158    Filtering Platform Connection
3/22/2026 8:52:50 PM  5158    Filtering Platform Connection
3/22/2026 8:52:45 PM  5158    Filtering Platform Connection
3/22/2026 8:52:45 PM  5158    Filtering Platform Connection
```

**Event 5158 details:**
```
"Windows Filtering Platform has
permitted a bind to a local port"

Event ID: 5158
Computer: DESKTOP-I5VGKR9
Process ID: 7316
Level: Information
Keywords: Audit Success
```

**What I noticed manually:**
- ✅ Event ID 5158 appearing rapidly
- ✅ Multiple events same timestamp
- ✅ Automated activity pattern!
- ✅ OS fingerprinting in progress!

**Manual Conclusion:** OS FINGERPRINTING DETECTED! ✅

---

### Detection Method 2 — Wireshark Analysis

**Filter 1 - TTL Detection:**
```
ip.ttl == 128
```
**Showed:** Windows machine visible with
TTL 128 packets and "Host Announcement DESKTOP-I5"!

**Filter 2 - ICMP Ping Traffic:**
```
icmp
```
**Real Results:**
```
Capturing from: Ethernet 2

Source          Destination     Protocol  Info
192.168.56.102  192.168.56.103  ICMP      Echo (ping) request
192.168.56.103  192.168.56.102  ICMP      Echo (ping) reply
192.168.56.102  192.168.56.103  ICMP      Echo (ping) request
192.168.56.103  192.168.56.102  ICMP      Echo (ping) reply
```

**Wireshark Conclusion:** ICMP PING FINGERPRINTING CONFIRMED! ✅

**Filter 3 - SYN Scan Traffic:**
```
tcp.flags.syn == 1
```
**Showed:** Sequential SYN packets from
192.168.56.102 to 192.168.56.103
Ports: 111, 256, 143, 53, 8888, 22, 1723, 443...

**Wireshark Conclusion:** TCP/IP FINGERPRINTING CONFIRMED! ✅

---

### Detection Method 3 — Splunk SIEM

**SPL Query:**
```
index=* 192.168.56.100
```
**Time Range:** Last 24 hours

**Real Splunk Results:**
```
47 events found!
Time range: 3/21/26 - 3/22/26

Event details:
Time: 03/22/2026 08:49:19 PM
Source Address: 192.168.56.103
Destination Address: 192.168.56.100
Source Port: 68
Destination Port: 67
host: DESKTOP-I5VGKR9
source: WinEventLog:Security
```

**Splunk Conclusion:** 47 EVENTS CONFIRMED! ✅

---

## 🔑 Key Technical Findings

### TTL Value Analysis:
| OS | Default TTL | Detected |
|----|------------|---------|
| Windows | 128 | ✅ TTL=128 confirmed! |
| Linux | 64 | N/A |
| Mac | 64 | N/A |

### OS Confidence Levels (osscan-guess):
| OS Version | Confidence |
|------------|-----------|
| Windows 10 1703 | 99% 🔴 |
| Windows 10 1507-1607 | 97% |
| Windows 10 1511 | 97% |
| Windows Longhorn | 95% |
| Windows Server 2008 | 94% |

---

## 🚨 Indicators of Compromise (IOCs)

| IOC | Value | Significance |
|-----|-------|-------------|
| TTL value | 128 | Windows OS confirmed |
| ICMP packets | Rapid requests | Ping fingerprinting |
| TCP/IP fingerprint | Windows pattern | OS confirmed |
| Event ID 5158 | Rapid occurrence | Port binding activity |
| OS guess confidence | 99% | Specific version identified! |
| Computer name | DESKTOP-I5VGKR9 | Identity exposed |

---

## 💡 Key Insight — Version Specific Attacks!

> **Getting 99% confidence on Windows 10 1703 is dangerous because
> attackers can now search CVE databases for vulnerabilities SPECIFIC
> to that exact Windows build and launch targeted exploits!**

This is why keeping Windows updated is CRITICAL for security!

---

## 🛡️ Recommendations

| Priority | Action |
|----------|--------|
| 🔴 HIGH | Keep Windows fully updated always |
| 🔴 HIGH | Enable firewall to block ICMP ping requests |
| ⚠️ MEDIUM | Block external Nmap scans at perimeter |
| ⚠️ MEDIUM | Monitor for rapid Event ID 5158 |
| 🟡 LOW | Consider OS obfuscation techniques |

---

## 🧠 What I Learned Today

1. **TTL=128 immediately reveals Windows** — one ping = OS identified!
2. **Nmap osscan-guess gives 99% confidence** on specific Windows version!
3. **Specific version = targeted CVE attacks** possible!
4. **Event ID 5158 is OS fingerprinting signature** in raw logs!
5. **Wireshark ICMP filter** perfectly captures ping fingerprinting!
6. **Splunk found 47 events** confirming all attack activity!
7. **Computer name DESKTOP-I5VGKR9 visible** from network alone!

---

## 📸 Screenshots

| File | Description |
|------|-------------|
| 01-nmap-O-wrong-command.png | Wrong lowercase o command |
| 02-nmap-O-sV-results.png | OS + service detection |
| 03-wireshark-ttl128-filter.png | TTL 128 filter |
| 04-wireshark-icmp-packets.png | ICMP ping packets |
| 05-ping-ttl128-kali.png | Ping showing ttl=128 |
| 06-osscan-guess-99percent.png | Windows 10 1703 at 99%! |
| 07-nmap-A-results.png | Full aggressive scan |
| 08-nmap-A-computer-name.png | Computer name revealed |
| 09-wireshark-syn-packets.png | TCP fingerprinting |
| 10-eventviewer-5158.png | Raw Event ID 5158 |
| 11-splunk-47-events.png | 47 events in SIEM |
| 12-splunk-source-address.png | Source address details |
