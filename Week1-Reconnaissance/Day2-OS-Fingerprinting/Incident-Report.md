# 🚨 Incident Report — Day 2

---

## 📋 Report Details

| Field | Details |
|-------|---------|
| Incident ID | IR-2026-002 |
| Date | March 22 2026 |
| Time | 22:27 - 22:52 EDT |
| Analyst | Vanitha Challa |
| Attack Type | OS Fingerprinting (Reconnaissance) |
| Severity | MEDIUM ⚠️ |
| Status | RESOLVED ✅ |

---

## 📌 Incident Summary

OS fingerprinting activity was detected against Windows 11 machine at
**192.168.56.103** originating from **192.168.56.102** on March 22 2026
between 22:27 and 22:52 EDT.

The attacker performed multiple OS detection techniques including
service version detection, TTL analysis via ping, aggressive OS
guessing, and full TCP/IP fingerprinting. The target OS was successfully
identified as **Windows 10 1703 with 99% confidence**.

---

## 🖥️ Systems Involved

| Role | IP Address | Hostname | OS |
|------|------------|---------|-----|
| Attacker | 192.168.56.102 | Kali Linux | Kali Linux |
| Victim | 192.168.56.103 | DESKTOP-I5VGKR9 | Windows 11 |

---

## 📅 Attack Timeline

| Time (EDT) | Activity |
|------------|----------|
| 22:27:00 | nmap -O -sV scan started |
| 22:27:45 | OS identified as Windows (45.64 seconds) |
| 22:30:00 | ping started for TTL analysis |
| 22:30:01 | TTL=128 confirmed Windows immediately |
| 22:46:00 | nmap --osscan-guess started |
| 22:46:24 | Windows 10 1703 identified at 99% confidence |
| 22:52:00 | nmap -A full fingerprinting started |
| 22:52:54 | Complete OS profile obtained (54.17 seconds) |

---

## 🔍 Detection Methods

### Method 1 — Raw Log Analysis (Manual)
**Tool:** Windows Event Viewer
**Event ID:** 5158

**Raw evidence found manually:**
```
Security Log - 30,556 events
Date: 3/22/2026 8:52:50 PM
Event ID: 5158
Description: Windows Filtering Platform
             permitted bind to local port
Computer: DESKTOP-I5VGKR9
Process ID: 7316
Keywords: Audit Success
```

**Manual analysis conclusion:**
Rapid Event ID 5158 occurrences indicate automated
port binding activity consistent with OS fingerprinting!

**Manual Conclusion:** OS FINGERPRINTING DETECTED ✅

---

### Method 2 — Wireshark Analysis

**Evidence 1 - TTL Filter:**
```
Filter: ip.ttl == 128
Result: Windows host visible
        "Host Announcement DESKTOP-I5"
        TTL 128 packets confirmed!
```

**Evidence 2 - ICMP Filter:**
```
Filter: icmp
Source: 192.168.56.102 (Kali)
Dest: 192.168.56.103 (Windows)
Protocol: ICMP
Info: Echo (ping) request/reply

20 packets captured
Confirms active ping fingerprinting!
```

**Evidence 3 - SYN Filter:**
```
Filter: tcp.flags.syn == 1
Source: 192.168.56.102
Ports: 111, 256, 143, 53, 8888,
       22, 1723, 443, 23, 21...
Pattern: Sequential TCP/IP fingerprinting!
```

**Wireshark Conclusion:** ALL 3 FINGERPRINTING METHODS CONFIRMED ✅

---

### Method 3 — Splunk SIEM
**Query:**
```
index=* 192.168.56.100
```

**Results:**
```
47 events found
Time: 3/21/26 8:00PM to 3/22/26 8:53PM

Event details:
Source Address: 192.168.56.103
Destination Address: 192.168.56.100
host: DESKTOP-I5VGKR9
source: WinEventLog:Security
```

**Splunk Conclusion:** 47 SECURITY EVENTS CONFIRMED ✅

---

## ⚠️ All 3 Methods Agreed ✅

```
Raw Logs (5158) → OS FINGERPRINTING ✅
Wireshark ICMP  → OS FINGERPRINTING ✅
Splunk 47 events → OS FINGERPRINTING ✅
```

---

## 🔓 Intelligence Gathered by Attacker

| Information | Value | Risk Level |
|------------|-------|-----------|
| OS Type | Windows | HIGH 🔴 |
| OS Version | Windows 10 1703 | HIGH 🔴 |
| Confidence | 99% | HIGH 🔴 |
| TTL Value | 128 (Windows confirmed) | MEDIUM ⚠️ |
| Computer Name | DESKTOP-I5VGKR9 | MEDIUM ⚠️ |
| Services Running | Splunk, RPC, SMB, NetBIOS | HIGH 🔴 |
| Network Distance | 1 hop (1.23ms) | LOW |

---

## 🚨 Indicators of Compromise (IOCs)

| IOC | Description | Severity |
|-----|-------------|---------|
| IP: 192.168.56.102 | Attack source | HIGH 🔴 |
| TTL=128 in all packets | Windows OS fingerprint | MEDIUM ⚠️ |
| ICMP echo flood | Ping-based fingerprinting | MEDIUM ⚠️ |
| Event ID 5158 rapid | Port binding fingerprint | MEDIUM ⚠️ |
| 99% OS confidence | Exact version identified | HIGH 🔴 |
| TCP/IP fingerprint | Nmap signature detected | MEDIUM ⚠️ |

---

## ⚠️ Severity Assessment

```
Overall Severity: MEDIUM ⚠️

Reasoning:
✅ Reconnaissance phase only
✅ No exploitation confirmed
✅ No data accessed

BUT serious concerns:
🔴 OS version identified at 99%
🔴 Attacker can find version-specific CVEs
🔴 Complete OS profile obtained
🔴 Computer name disclosed
🔴 All services identified again
```

---

## 🛡️ Recommendations

| Priority | Action | Reason |
|----------|--------|--------|
| 🔴 HIGH | Keep Windows fully updated | Eliminate known CVEs |
| 🔴 HIGH | Block ICMP at firewall | Prevent TTL fingerprinting |
| 🔴 HIGH | Block source IP 192.168.56.102 | Stop further recon |
| ⚠️ MEDIUM | Enable IDS rules for Nmap patterns | Detect future scans |
| ⚠️ MEDIUM | Monitor Event ID 5158 spikes | Early warning system |
| ⚠️ MEDIUM | Restrict SMB and NetBIOS | Reduce info disclosure |
| 🟡 LOW | Consider OS obfuscation | Make fingerprinting harder |

---

## 📸 Evidence Screenshots

| Screenshot | Description | Tool |
|------------|-------------|------|
| 01-nmap-O-wrong-command.png | Lowercase o mistake | Kali |
| 02-nmap-O-sV-results.png | OS + service detection | Nmap |
| 03-wireshark-ttl128-filter.png | TTL 128 Windows filter | Wireshark |
| 04-wireshark-icmp-packets.png | ICMP ping captured | Wireshark |
| 05-ping-ttl128-kali.png | TTL=128 in ping output | Kali |
| 06-osscan-guess-99percent.png | 99% Windows 10 1703! | Nmap |
| 07-nmap-A-results.png | Full fingerprint results | Nmap |
| 08-nmap-A-computer-name.png | DESKTOP-I5VGKR9 exposed | Nmap |
| 09-wireshark-syn-packets.png | TCP fingerprinting | Wireshark |
| 10-eventviewer-5158.png | Raw Event 5158 logs | Event Viewer |
| 11-splunk-47-events.png | 47 SIEM events | Splunk |
| 12-splunk-source-address.png | Source address details | Splunk |

---

## 🏁 Conclusion

A complete OS fingerprinting reconnaissance was performed against
Windows 11 host DESKTOP-I5VGKR9 at 192.168.56.103.

The attacker successfully:
- Confirmed Windows OS from TTL=128 via single ping
- Identified Windows 10 version 1703 with **99% confidence**
- Obtained complete TCP/IP fingerprint
- Mapped all running services
- Identified Splunk SIEM again on default ports

The most critical finding is the **99% confidence OS version
identification**. This allows an attacker to search CVE databases
for vulnerabilities specific to Windows 10 build 1703 and launch
highly targeted exploits.

**Immediate action required**: Update Windows and block ICMP!

---

## ✅ Incident Status

```
Status: RESOLVED ✅

Actions Taken:
- Full incident documented
- All 3 detection methods confirmed
- OS fingerprinting techniques identified
- Recommendations provided

Lessons Learned:
- TTL=128 immediately reveals Windows
- One ping = OS identified!
- 99% confidence = targeted attack possible
- Keeping OS updated is CRITICAL
```

---

*Report by: Vanitha Challa*
*Date: March 22 2026*
*30 Day SOC Challenge — Day 2*
*GitHub: github.com/VanithaChalla/30-Day-SOC-Challenge*
