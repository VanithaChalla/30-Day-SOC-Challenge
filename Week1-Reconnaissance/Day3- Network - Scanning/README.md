# Day 3 — Network Scanning (ARP/Layer 2)
## 🛡️ 30 Day SOC Challenge | Week 1 — Reconnaissance

---

## 📌 Objective
Perform a full Layer 2 network scan using ARP-based tools from Kali Linux against a Windows 11 target, then detect and analyze the attack using Event Viewer, Wireshark, and Splunk.

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| Attacker | Kali Linux (192.168.56.100) |
| Defender | Windows 11 (192.168.56.103) |
| Network | VirtualBox Host-Only (192.168.56.0/24) |
| Tools Used | Netdiscover, arp-scan, hping3, Wireshark, Event Viewer, Splunk |

---

## ⚔️ Attack Side — Commands Used

### Step 1 — Passive ARP Discovery
```bash
sudo netdiscover -r 192.168.56.0/24
```
**Result:** Discovered 3 live hosts with MAC addresses and vendor names

### Step 2 — Active ARP Scan
```bash
sudo arp-scan 192.168.56.0/24
```
**Result:** Scanned 256 hosts in 2.036 seconds at 125.74 hosts/sec

### Step 3 — ARP Cache Check
```bash
arp -a
```
**Result:** Checked ARP table on both Kali and Windows CMD

### Step 4 — ICMP Flood with Random Source
```bash
sudo hping3 --rand-source -1 192.168.56.103
```
**Result:** 372 packets transmitted, 0 received, 100% packet loss (firewall blocked!)

---

## 🛡️ Defense Side — Detection Methods

### Event Viewer
- **Location:** Windows Logs → Security
- **Event ID:** 5156 (Windows Filtering Platform)
- **Finding:** Multiple connection attempts from 192.168.56.100

### Wireshark
- **Filter:** `arp`
- **Finding:** ARP request packets — "Who has 192.168.56.103?" flooding from Kali

### Splunk
- **Query:** `index=* sourcetype=WinEventLog:Security EventCode=5156`
- **Finding:** Complete attack timeline with source IP isolated

---

## 📊 Key Findings

| Tool | Evidence Found |
|------|---------------|
| Netdiscover | 3 hosts, MAC addresses, vendors exposed |
| arp-scan | 256 hosts scanned in 2.036 seconds |
| Windows arp -a | Kali MAC 08-00-27-c3-0f-95 cached dynamically |
| hping3 | 372 packets sent, all blocked by firewall |
| Wireshark | ARP flood packets captured |
| Event Viewer | Event ID 5156 repeated from same source |
| Splunk | Full attack timeline confirmed |

---

## 💡 Key Lessons

1. **ARP operates at Layer 2** — reveals MAC addresses that nmap misses
2. **256 hosts scanned in 2 seconds** — network fully mapped instantly
3. **Firewall blocks packets but logs catch everything** — 372 blocked = 372 logged
4. **Windows ARP cache stores attacker MAC** — evidence left behind automatically

---

## 📸 Screenshots

| # | Filename | Description |
|---|----------|-------------|
| 1 | 01-netdiscover-results.png | Passive ARP discovery — 3 hosts found |
| 2 | 02-arp-scan-results.png | Active ARP scan — MAC + vendor exposed |
| 3 | 03-kali-arp-table.png | Kali ARP cache |
| 4 | 04-windows-arp-table.png | Windows ARP cache showing Kali MAC |
| 5 | 05-hping3-flood.png | 372 packets sent, 100% blocked |
| 6 | 06-wireshark-arp.png | ARP packets captured |
| 7 | 07-event-viewer-5156.png | Event ID 5156 repeating |
| 8 | 08-splunk-results.png | Splunk attack timeline |
| 9 | 09-splunk-chart.png | Splunk visualization |

---

---

*Day 3 of 30 | Week 1 — Reconnaissance & Scanning*

