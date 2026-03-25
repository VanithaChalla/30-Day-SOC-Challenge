
# Incident Report — Day 3
## Network Scanning / ARP Layer 2 Attack
### 30 Day SOC Challenge | Week 1 — Reconnaissance

---

## 1. INCIDENT SUMMARY

| Field | Details |
|-------|---------|
| Date | March 2026 |
| Attack Type | Network Scanning — ARP Layer 2 |
| Attacker IP | 192.168.56.100 (Kali Linux) |
| Target IP | 192.168.56.103 (Windows 11) |
| Network | 192.168.56.0/24 (VirtualBox Host-Only) |
| Severity | Medium |
| Status | Detected ✅ |

---

## 2. ATTACK TIMELINE

| Time | Action | Tool |
|------|--------|------|
| T+0:00 | Passive ARP scan initiated | Netdiscover |
| T+0:30 | 3 hosts discovered with MAC addresses | Netdiscover |
| T+1:00 | Active ARP scan launched | arp-scan |
| T+1:02 | 256 hosts scanned in 2.036 seconds | arp-scan |
| T+2:00 | ARP cache checked on both machines | arp -a |
| T+3:00 | ICMP flood launched with random source IPs | hping3 |
| T+3:30 | 372 packets transmitted, 0 received | hping3 |
| T+4:00 | Attack detected in Wireshark | Wireshark |
| T+4:30 | Event ID 5156 found in Event Viewer | Event Viewer |
| T+5:00 | Full timeline confirmed in Splunk | Splunk |

---

## 3. ATTACK DETAILS

### 3.1 Passive ARP Discovery (Netdiscover)
- **Command:** `sudo netdiscover -r 192.168.56.0/24`
- **Captured:** 6 ARP Req/Rep packets from 3 hosts
- **Hosts Discovered:**

| IP Address | MAC Address | Vendor |
|------------|-------------|--------|
| 192.168.56.1 | 0a:00:27:00:00:11 | Unknown (VirtualBox Gateway) |
| 192.168.56.100 | 08:00:27:c3:0f:95 | PCS Systemtechnik GmbH (Kali) |
| 192.168.56.103 | 08:00:27:84:a8:71 | PCS Systemtechnik GmbH (Windows) |

### 3.2 Active ARP Scan (arp-scan)
- **Command:** `sudo arp-scan 192.168.56.0/24`
- **Scan Speed:** 125.74 hosts/sec
- **Time to Complete:** 2.036 seconds
- **Hosts Found:** 3 responded out of 256 scanned
- **Packets:** 3 received, 0 dropped by kernel

### 3.3 ARP Cache Analysis
- **Kali arp -a:** Showed only own MAC (192.168.56.100)
- **Windows arp -a:** Cached Kali's MAC as dynamic entry
  - Entry: `192.168.56.100 → 08-00-27-c3-0f-95 → dynamic`
  - This confirms Windows received and stored ARP packets from attacker

### 3.4 ICMP Flood (hping3)
- **Command:** `sudo hping3 --rand-source -1 192.168.56.103`
- **Packets Transmitted:** 372
- **Packets Received:** 0
- **Packet Loss:** 100%
- **Conclusion:** Windows Firewall successfully blocked all ICMP packets
- **Key Insight:** Despite 100% block rate, all 372 attempts were logged

---

## 4. DETECTION EVIDENCE

### 4.1 Wireshark
- **Filter Applied:** `arp`
- **Interface:** Ethernet 2 (192.168.56.x Host-Only)
- **Evidence:** ARP request packets captured — "Who has 192.168.56.103?" repeated from 192.168.56.100
- **Conclusion:** ✅ ARP flood confirmed at packet level

### 4.2 Windows Event Viewer
- **Location:** Windows Logs → Security
- **Event ID:** 5156 (Windows Filtering Platform Connection Permitted/Blocked)
- **Evidence:** Multiple Event ID 5156 entries from source IP 192.168.56.100
- **Pattern:** Repeated events within same timestamp = automated tool
- **Conclusion:** ✅ Network scanning activity confirmed

### 4.3 Splunk
- **Query:** `index=* sourcetype=WinEventLog:Security EventCode=5156`
- **Evidence:** Events clustered from single source IP
- **Conclusion:** ✅ Full attack timeline reconstructed

---

## 5. MITRE ATT&CK MAPPING

| Tactic | Technique | ID |
|--------|-----------|-----|
| Reconnaissance | Active Scanning | T1595 |
| Reconnaissance | Gather Victim Network Info | T1590 |
| Discovery | Network Service Discovery | T1046 |
| Discovery | Remote System Discovery | T1018 |

---

## 6. KEY FINDINGS

1. **Layer 2 ARP scanning exposes MAC addresses** that port scanning alone cannot reveal
2. **Network fully mapped in 2.036 seconds** — attacker speed is devastating
3. **Windows Firewall blocked 100% of ICMP packets** but could not prevent ARP reconnaissance
4. **ARP cache on Windows stored attacker MAC address** automatically — evidence persists
5. **Raw log analysis (Event ID 5156) detected attack** before Splunk alert triggered
6. **Three independent tools** (Wireshark, Event Viewer, Splunk) all confirmed same attack

---

## 7. RECOMMENDATIONS

| # | Recommendation | Priority |
|---|---------------|----------|
| 1 | Enable Dynamic ARP Inspection (DAI) on network switches | High |
| 2 | Monitor for unusual ARP traffic volume in Splunk | High |
| 3 | Alert on Event ID 5156 spikes from single source IP | High |
| 4 | Implement network segmentation to limit ARP broadcast domains | Medium |
| 5 | Use static ARP entries for critical systems | Medium |

---

## 8. CONCLUSION

The Day 3 network scanning simulation successfully demonstrated how an attacker can perform complete Layer 2 network reconnaissance using ARP-based tools. The entire network was mapped in under 3 seconds, exposing IP addresses, MAC addresses, and vendor information. Despite the Windows Firewall blocking 100% of ICMP flood packets, the ARP reconnaissance phase was undetected by the firewall — highlighting why Layer 2 monitoring is critical in a SOC environment.

All three detection methods (Wireshark, Event Viewer, Splunk) independently confirmed the attack, validating the importance of multi-layer detection strategy.

**Analyst:** Vanitha Challa
**Challenge:** 30 Day SOC Challenge — Day 3 of 30
**GitHub:** https://github.com/VanithaChalla/30-Day-SOC-Challenge

---
*End of Incident Report*
