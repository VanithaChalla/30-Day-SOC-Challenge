
# 🚨 Incident Report — Day 1

---

## 📋 Report Details

| Field | Details |
|-------|---------|
| Incident ID | IR-2026-001 |
| Date | March 22 2026 |
| Time | 13:39 EDT |
| Analyst | Vanitha Challa |
| Attack Type | Port Scanning (Reconnaissance) |
| Severity | MEDIUM ⚠️ |
| Status | RESOLVED ✅ |

---

## 📌 Incident Summary

A port scan was detected against Windows 11 machine at **192.168.56.103**
originating from **192.168.56.100** and **192.168.56.102** on March 22 2026
between 13:37 and 13:50 EDT.

The attacker performed a full network reconnaissance including host discovery,
port scanning, service detection, and aggressive fingerprinting. All 5 open
ports were mapped and sensitive service information including Splunk SIEM
running on default ports 8000 and 8089 was successfully identified.

---

## 🖥️ Systems Involved

| Role | IP Address | Hostname | OS |
|------|------------|---------|-----|
| Attacker 1 | 192.168.56.100 | Kali Linux | Kali Linux |
| Attacker 2 | 192.168.56.102 | VirtualBox VM | Unknown |
| Victim | 192.168.56.103 | DESKTOP-I5VGKR9 | Windows 11 Enterprise |

---

## 📅 Attack Timeline

| Time (EDT) | Activity |
|------------|----------|
| 13:37:00 | Host discovery scan started (nmap -sn) |
| 13:37:27 | 4 hosts discovered including target 192.168.56.103 |
| 13:39:00 | Basic port scan started (nmap) |
| 13:39:14 | 5 open ports discovered in 14.43 seconds |
| 13:50:00 | Full aggressive scan started (nmap -A) |
| 13:50:53 | Complete service fingerprinting done in 53 seconds |
| 13:51:00 | Splunk SIEM identified on ports 8000 and 8089 |

---

## 🔍 Detection Methods

### Method 1 — Raw Log Analysis (Manual)
**Tool:** Windows Event Viewer
**Event ID:** 5156 (Windows Filtering Platform Connection)

**Raw evidence found manually:**
- Event ID 5156 appearing 31,179+ times in Security log
- Multiple events at exact timestamp: 3/22/2026 12:28:21 PM
- Computer: DESKTOP-I5VGKR9
- Process ID: 4536
- Keywords: Audit Success

**Pattern identified without tools:**
- Same Event ID repeating at same second
- Indicates automated rapid connection attempts
- Sequential port targeting pattern visible

**Manual Conclusion:** PORT SCAN DETECTED ✅

---

### Method 2 — Wireshark Network Analysis
**Tool:** Wireshark
**Interface:** Ethernet 2 (VirtualBox Host-Only)
**Filter:** tcp.flags.syn == 1

**Raw packet evidence:**
```
Source          Destination     Protocol  Info
192.168.56.102  192.168.56.103  TCP       33698→8200 [SYN]
192.168.56.102  192.168.56.103  TCP       33698→5001 [SYN]
192.168.56.102  192.168.56.103  TCP       33698→3351 [SYN]
192.168.56.103  192.168.56.102  TCP       [RST, ACK] responses
```

**Analysis:**
- SYN packets sent to multiple ports rapidly
- No complete TCP handshakes (RST responses)
- This is the classic signature of a SYN port scan
- Source IP 192.168.56.102 confirmed as attacker

**Wireshark Conclusion:** SYN SCAN CONFIRMED ✅

---

### Method 3 — Splunk SIEM Analysis
**Tool:** Splunk Enterprise 10.2.1
**Query:**
```
index=* 192.168.56.100
```
**Time Range:** Last 24 hours

**Results:**
```
12 events found
Time: 03/22/2026 12:35:22 PM
Network Information:
  Direction: Inbound
  Source Address: 192.168.56.100
```

**Splunk Conclusion:** ATTACK SOURCE CONFIRMED ✅

---

## ⚠️ All 3 Detection Methods Agreed ✅

```
Raw Logs → PORT SCAN ✅
Wireshark → PORT SCAN ✅
Splunk   → PORT SCAN ✅

Manual analysis confirmed BEFORE tools!
```

---

## 🔓 Open Ports Discovered

| Port | State | Service | Version | Risk |
|------|-------|---------|---------|------|
| 135/tcp | Open | msrpc | Microsoft Windows RPC | Medium |
| 139/tcp | Open | netbios-ssn | Microsoft Windows netbios-ssn | Medium ⚠️ |
| 445/tcp | Open | microsoft-ds | Microsoft-DS | HIGH ⚠️ |
| 8000/tcp | Open | http | **Splunkd httpd** | HIGH 🔴 |
| 8089/tcp | Open | ssl/http | **Splunkd httpd SSL** | HIGH 🔴 |

---

## 🔍 Additional Intelligence Gathered by Attacker

| Information | Value | Risk |
|------------|-------|------|
| Computer Name | DESKTOP-I5VGKR9 | Identity exposed |
| OS | Windows (CPE: microsoft:windows) | OS fingerprinted |
| Splunk Login URL | http://192.168.56.103:8000 | SIEM accessible |
| SSL Certificate | SplunkServerDefaultCert | Default cert used |
| Cert Valid Until | 2029-03-10 | Certificate info exposed |
| SMB Signing | Enabled and Required | SMB configured |
| Network Distance | 1 hop | Direct network access |
| Traceroute RTT | 1.57ms | Network mapping done |

---

## 🚨 Indicators of Compromise (IOCs)

| IOC | Description | Severity |
|-----|-------------|---------|
| IP: 192.168.56.100 | Primary attack source | HIGH 🔴 |
| IP: 192.168.56.102 | Secondary attack source | HIGH 🔴 |
| 1000 ports in 14 sec | Automated scanning tool | HIGH 🔴 |
| Sequential ports | Nmap tool signature | MEDIUM ⚠️ |
| SYN without ACK | SYN scan technique | MEDIUM ⚠️ |
| Port 8000 probed | SIEM targeted | HIGH 🔴 |
| Splunk identified | SIEM tool exposed | HIGH 🔴 |
| Computer name exposed | DESKTOP-I5VGKR9 | MEDIUM ⚠️ |

---

## ⚠️ Severity Assessment

```
Overall Severity: MEDIUM ⚠️

Reasoning:
✅ Reconnaissance phase only
✅ No exploitation confirmed
✅ No data accessed

BUT serious concerns:
🔴 Splunk SIEM fully exposed
🔴 Attacker knows monitoring exists
🔴 Computer name disclosed
🔴 All services fingerprinted
🔴 SMB port 445 open (ransomware risk!)
🔴 Default Splunk certificate detected
```

---

## 🛡️ Recommendations

| Priority | Action | Reason |
|----------|--------|--------|
| 🔴 HIGH | Block IPs 192.168.56.100 and .102 in firewall | Stop further attacks |
| 🔴 HIGH | Move Splunk to non-standard port | Hide SIEM from attackers |
| 🔴 HIGH | Change Splunk default SSL certificate | Remove tool fingerprint |
| ⚠️ MEDIUM | Disable NetBIOS (port 139) if unused | Reduce attack surface |
| ⚠️ MEDIUM | Restrict SMB (port 445) access | Prevent ransomware entry |
| ⚠️ MEDIUM | Enable Windows Firewall | Block unauthorized scans |
| 🟡 LOW | Monitor source IPs for follow-up attacks | Detect exploitation attempts |
| 🟡 LOW | Set up Splunk alert for rapid port connections | Automated detection |

---

## 📸 Evidence Screenshots

| Screenshot | Description | Tool |
|------------|-------------|------|
| 1-nmap-sn-discovery.png | 4 hosts found on network | Nmap |
| 2-nmap-basic-scan.png | 5 open ports in 14 seconds | Nmap |
| 3-wireshark-syn-filter.png | SYN packets from attacker | Wireshark |
| 4-event-viewer-5156.png | Event ID 5156 raw logs | Event Viewer |
| 5-splunk-search.png | Attack source in SIEM | Splunk |
| 6-nmap-A-full-results.png | Complete service fingerprint | Nmap |
| 7-splunk-exposed.png | Splunk detected by attacker | Nmap |

---

## 🏁 Conclusion

A complete network reconnaissance was performed against Windows 11 host
DESKTOP-I5VGKR9 at 192.168.56.103. The attacker successfully:

- Discovered all 4 hosts on the network
- Mapped all 5 open ports in 14 seconds
- Identified the Splunk SIEM tool running on default ports
- Obtained the computer name and OS details
- Gathered SSL certificate information

The most critical finding is that **Splunk SIEM is visible on default
ports 8000 and 8089**. This tells any attacker that monitoring is active
and could lead to targeted attacks against the SIEM itself to delete logs
and cover tracks.

No exploitation was confirmed. This was reconnaissance only. However,
the information gathered provides a complete roadmap for further attacks.

**Immediate action required** especially changing Splunk default ports!

---

## ✅ Incident Status

```
Status: RESOLVED ✅

Actions Taken:
- Full incident documented
- All 3 detection methods confirmed
- IOCs identified and listed
- Recommendations provided

Next Steps:
- Monitor 192.168.56.100 and .102
- Implement firewall rules
- Change Splunk to custom port
- Watch for Day 2 attack patterns
```

---

*Incident Report prepared by: Vanitha Challa*
*Date: March 22 2026*
*Part of 30 Day SOC Challenge*
*GitHub: github.com/VanithaChalla/30-Day-SOC-Challenge*
