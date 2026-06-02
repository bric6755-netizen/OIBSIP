# Task 1 — Basic Network Scanning with Nmap

**Intern:** Richard Boakye Danquah
**Program:** AICTE Oasis Infobyte Security Analyst Internship
**Task Level:** Beginner
**Date Completed:** June 2, 2026

---

## Objective

Perform a network scan to identify open ports and active services on a local machine and subnet using **Nmap**, and document findings with security analysis.

---

## Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | 7.99 | Network scanning and service detection |
| Windows CMD | — | Running scans and saving output |
| Npcap | 1.87 | Packet capture driver required by Nmap on Windows |

---

## Environment

| Detail | Value |
|--------|-------|
| Operating System | Microsoft Windows 11 (Build 10.0.26200.8457) — 24H2/25H2 |
| Hostname | Richard |
| WiFi IP Address | 192.168.43.78 |
| Hyper-V Virtual Switch IP | 172.22.144.1 |
| Gateway (Hotspot) | 192.168.43.1 |
| Subnet Scanned | 192.168.43.0/24 |

---

## Scans Performed

### Scan 1 — Basic Port Scan (Localhost)
```bash
nmap 127.0.0.1
```
Identifies open TCP ports on the local machine using Nmap's default top-1000-port scan.

---

### Scan 2 — Service & Version Detection (Localhost)
```bash
nmap -sV 127.0.0.1
```
Probes each open port to determine the running service name and version.

---

### Scan 3 — Aggressive Scan (This Machine via WiFi IP)
```bash
nmap -A 192.168.43.78
```
Enables OS detection, version detection, NSE script scanning, and traceroute.
This scan revealed the OS as **Windows 11 24H2–25H2** and confirmed **SMB 3.1.1 with message signing required**.

---

### Scan 4 — Full Subnet Scan
```bash
nmap 192.168.43.0/24
```
Scans all 256 IPs on the /24 subnet to discover active hosts and their open ports.

---

### Scan 5 — Save Output to File
```bash
nmap -sV 127.0.0.1 -oN nmap_scan_results.txt
```
Saves scan results in normal readable format for documentation.

---

## Key Findings

### Open Ports — Local Machine (192.168.43.78 / 127.0.0.1)

| Port | State | Service | Risk Level | Notes |
|------|-------|---------|------------|-------|
| 135/tcp | Open | Microsoft RPC | ⚠️ Medium | Windows core service; block at perimeter |
| 139/tcp | Open | NetBIOS Session Service | ⚠️ Medium | Legacy protocol; disable if unused |
| 445/tcp | Open | SMB (microsoft-ds) | 🔴 High | Patch MS17-010; SMB signing is ON ✅ |
| 2179/tcp | Open | Hyper-V RDP (vmrdp) | 🟡 Low | Disable if VMs not in active use |

### Open Ports — Gateway / Android Hotspot (192.168.43.1)

| Port | State | Service | Risk Level | Notes |
|------|-------|---------|------------|-------|
| 53/tcp | Open | DNS (dnsmasq 2.51) | 🟡 Low-Medium | Outdated version; acceptable on personal hotspot |

### Active Hosts Discovered on Subnet (192.168.43.0/24)

| IP Address | Hostname | Device Type |
|------------|----------|-------------|
| 192.168.43.1 | — | Android Mobile Hotspot (Gateway) |
| 192.168.43.78 | Richard | Windows 11 PC (This Machine) |

Only **2 out of 256** IPs were active — a clean, minimal network.

---

## Port Significance Explained

### Port 135 — Microsoft RPC
Windows uses Remote Procedure Call for inter-process communication between services like DCOM, WMI, and Task Scheduler. While essential for Windows internals, it should never be exposed to the internet. Historically exploited by the **Blaster worm (MS03-026, 2003)**.

### Port 139 — NetBIOS Session Service
A legacy protocol enabling Windows file and printer sharing over TCP/IP. It can expose NetBIOS names and workgroup details, which attackers use during **network enumeration** to map internal infrastructure. Best practice is to disable it when not required.

### Port 445 — SMB (Server Message Block)
The core Windows file-sharing protocol. Exploited by the **EternalBlue vulnerability (MS17-010)**, which was weaponised by the **WannaCry** ransomware (May 2017) causing an estimated $4–8 billion in global damages. 

✅ **Good news:** The aggressive scan confirmed this machine uses **SMB dialect 3.1.1** with **message signing enabled and required** — a strong security posture that defends against NTLM relay and man-in-the-middle attacks.

### Port 2179 — Hyper-V Remote Desktop (vmrdp)
Allows remote desktop connections into Hyper-V virtual machines. Confirmed active by the presence of the `vEthernet (Default Switch)` adapter (172.22.144.1) visible in `ipconfig`. If remote VM access is not required, disabling Hyper-V removes this exposure entirely.

### Port 53 — DNS (dnsmasq on Hotspot)
The Android hotspot at 192.168.43.1 acts as a mini-router using **dnsmasq** to provide DNS resolution and DHCP for connected devices. The version (2.51) is old and has known CVEs (e.g., **CVE-2017-14491** — heap buffer overflow), but the risk is low in a personal hotspot context.

---

## Notable Security Observations

### ✅ SMB Message Signing Enabled
The aggressive scan returned:
```
smb2-security-mode:
  3.1.1:
    Message signing enabled and required
```
This is a positive security finding. SMB signing prevents attackers from intercepting and relaying authentication tokens on the network.

### ℹ️ Hyper-V Confirmed Active
The `vEthernet (Default Switch)` adapter with IP `172.22.144.1` and the open port `2179` together confirm Windows Hyper-V is installed and running. This is expected for a development machine but worth noting in a security audit.

### ℹ️ OS Fingerprinting Successful
Nmap successfully identified the OS as **Windows 11 24H2–25H2** purely from TCP/IP stack behaviour. This demonstrates why minimising open ports matters — fewer open ports means less data for an attacker's reconnaissance.

---

## Security Recommendations

1. **Keep Windows 11 fully patched** — apply all cumulative updates, especially those addressing SMB vulnerabilities (KB4012212 and later).
2. **Enable Windows Firewall rules** — restrict inbound connections on ports 135, 139, and 445 to trusted networks only.
3. **Disable NetBIOS over TCP/IP** — via *Network Adapter Settings → IPv4 → Advanced → WINS tab* if legacy file sharing is not needed.
4. **Disable Hyper-V** if virtual machines are not actively in use — removes both the vmrdp port and the virtual network adapter.
5. **Maintain SMB signing** — the current configuration (signing required) is already best practice; do not downgrade this setting.
6. **Use a VPN on mobile hotspots** — DNS queries are visible to the hotspot operator; a VPN or DNS-over-HTTPS mitigates this.

---

## Files in This Repository

```
OIBSIP/
└── Task1_NetworkScanning/
    ├── README.md                  ← This file
    ├── nmap_scan_results.txt      ← Full documented scan output
    └── screenshots/               ← Terminal screenshots of each scan
```

---

## What I Learned

- How to install and verify Nmap on Windows (with Npcap driver)
- How to use `ipconfig` to map the network topology before scanning
- How to perform basic, service-version, aggressive, and subnet scans
- How to interpret open ports and assess their security risk
- How SMB signing protects against NTLM relay attacks
- How OS fingerprinting works and why reducing open ports matters
- The real-world impact of unpatched SMB (EternalBlue / WannaCry)

---

## References

- [Nmap Official Documentation](https://nmap.org/docs.html)
- [CVE-2017-0144 — EternalBlue / MS17-010](https://nvd.nist.gov/vuln/detail/CVE-2017-0144)
- [CVE-2017-14491 — dnsmasq heap overflow](https://nvd.nist.gov/vuln/detail/CVE-2017-14491)
- [Microsoft Security Bulletin MS03-026 (RPC)](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2003/ms03-026)
- [WannaCry Ransomware Analysis — CISA](https://www.cisa.gov/news-events/alerts/2017/05/12/indicators-associated-wannacry-ransomware)
- [SMB Security Best Practices — Microsoft](https://docs.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3)

  ## Demo Video
[Watch Task 1 Demo](https://1drv.ms/v/c/18646ff513db828c/IQAgnbisTy8WTJPecFJF2rdTAX-S6b4FHi1Te6I_Q08u6WA?e=zPn8Yy)


---

*Submitted as part of the AICTE Oasis Infobyte Security Analyst Internship Program — Task 1*
