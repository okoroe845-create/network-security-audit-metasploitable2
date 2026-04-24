# network-security-audit-metasploitable2
Description: Network security audit of Metasploitable 2 — Nmap, Wireshark, CVE analysis
# 🔐 Network Security Audit — Metasploitable 2
### Performed by: Okoro Francis Emmanuel

<div align="center">

![Security Audit](https://img.shields.io/badge/Type-Network%20Security%20Audit-red?style=for-the-badge&logo=shield)
![Target](https://img.shields.io/badge/Target-Metasploitable%202-orange?style=for-the-badge&logo=linux)
![Tools](https://img.shields.io/badge/Tools-Nmap%20%7C%20Wireshark-blue?style=for-the-badge&logo=wireshark)
![Status](https://img.shields.io/badge/Status-Completed-green?style=for-the-badge)
![Class Rank](https://img.shields.io/badge/Class%20Rank-Highest%20Score-gold?style=for-the-badge&logo=trophy)

</div>

---

> **⚠️ Legal Disclaimer**
> This audit was conducted entirely in an isolated, controlled lab environment against a deliberately vulnerable target (Metasploitable 2) deployed on a private local network. No real production systems, live networks, or third-party infrastructure were targeted. All findings documented here are for educational purposes only. Unauthorized penetration testing is illegal and unethical.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Lab Environment](#-lab-environment)
- [Tools Used](#-tools-used)
- [Tool Breakdown](#-tool-breakdown)
- [Methodology & Project Steps](#-methodology--project-steps)
- [Key Findings](#-key-findings)
  - [🔴 Critical Vulnerabilities](#-critical-vulnerabilities)
  - [🟠 Medium Vulnerabilities](#-medium-vulnerabilities)
  - [🟡 Low Vulnerabilities](#-low-vulnerabilities)
- [Full Vulnerability Report](#-full-vulnerability-report)
- [Wireshark Credential Capture](#-wireshark-credential-capture)
- [Lessons Learned](#-lessons-learned)
- [Next Steps](#-next-steps)
- [References](#-references)

---

## 🎯 Project Overview

This project is a structured **black-box network security audit** conducted against a deliberately vulnerable virtual machine (Metasploitable 2). The goal was to simulate the reconnaissance and discovery phases of a real-world penetration test — identifying exposed services, enumerating vulnerabilities, and producing a professional risk-rated findings report exactly as a security consultant would deliver to a client.

The audit followed a systematic methodology: passive host discovery, progressive active scanning, service enumeration, vulnerability identification, traffic analysis, and formal documentation. Every finding was mapped to its CVE identifier, given a CVSS risk rating, and paired with a concrete remediation recommendation.

**This project was awarded the highest score in the class.**

### What This Project Demonstrates
- Ability to plan and execute a structured network security assessment
- Proficiency with industry-standard reconnaissance and analysis tools
- Understanding of real CVEs and how they manifest in live environments
- Professional documentation and risk communication skills
- Practical network traffic analysis using Wireshark

---

## 🖥️ Lab Environment

| Component         | Details                                           |
|-------------------|---------------------------------------------------|
| **Attacker OS**   | Kali Linux (primary machine, mr-robot@kali)       |
| **Target OS**     | Metasploitable 2 (Ubuntu 8.04 LTS — intentionally vulnerable) |
| **Target IP**     | `192.168.122.175`                                 |
| **Attacker IP**   | `192.168.122.1` (KVM/Virt-Manager host network)  |
| **Network Type**  | Isolated private virtual network (KVM/libvirt)   |
| **Virtualisation**| KVM with Virt-Manager — Metasploitable 2 deployed from VMDK (converted to QCOW2) |
| **Internet Access**| Disabled on target — fully air-gapped audit network |

**Architecture Diagram:**
```
┌─────────────────────────────────────────────┐
│            ISOLATED KVM NETWORK             │
│                                             │
│  ┌──────────────────┐   ┌────────────────┐  │
│  │   Kali Linux     │   │ Metasploitable │  │
│  │  (Attacker)      │◄──│       2        │  │
│  │ 192.168.122.1    │   │ 192.168.122.175│  │
│  └──────────────────┘   └────────────────┘  │
│                                             │
│  ← No external internet access →           │
└─────────────────────────────────────────────┘
```

---

## 🛠️ Tools Used

| Tool           | Version | Purpose                                      |
|----------------|---------|----------------------------------------------|
| **Nmap**       | 7.94    | Network scanning, service enumeration, vulnerability detection |
| **Wireshark**  | 4.x     | Packet capture and network traffic analysis  |
| **Netcat (nc)**| 1.10    | Banner grabbing and manual service probing   |
| **Terminal**   | bash    | Command execution, workflow automation       |

---

## 🔬 Tool Breakdown

### Nmap — Network Mapper
Nmap is the industry standard for network reconnaissance. It sends crafted packets to a target and analyses the responses to determine which hosts are online, what ports are open, which services are running, their versions, and known vulnerabilities.

**Why Nmap?**
It is used by professional penetration testers, red teams, and network administrators worldwide. Understanding Nmap is a core skill for any security professional.

**Key Nmap Flags Used in This Audit:**

| Flag | Full Name | What It Does |
|------|-----------|--------------|
| `-sn` | Ping Scan | Host discovery — identifies live hosts without port scanning |
| `-sS` | TCP SYN Scan | "Stealth" half-open scan — does not complete TCP handshake, faster, less noisy |
| `-sV` | Service Version Detection | Probes open ports to determine exact software and version |
| `-O` | OS Detection | Attempts to identify the target's operating system |
| `-A` | Aggressive Scan | Combines -sV, -O, traceroute, and default scripts |
| `-T4` | Timing Template | Aggressive timing — faster scan on reliable networks |
| `--script vuln` | Vulnerability Scripts | Runs Nmap's built-in NSE vulnerability detection scripts |
| `--script ftp-anon` | FTP Anonymous | Tests whether anonymous FTP login is permitted |
| `--script smb-vuln*` | SMB Vulnerability | Tests for known SMB vulnerabilities (MS17-010, CVE-2007-2447 etc.) |
| `-p-` | All Ports | Scans all 65,535 TCP ports instead of top 1,000 |
| `-oN` | Normal Output | Saves scan results to a readable text file |
| `-oX` | XML Output | Saves scan results in XML format for tool integration |

---

### Wireshark — Network Protocol Analyser
Wireshark captures all packets passing through a network interface and allows deep inspection of every protocol layer. In this audit, it was used to demonstrate the critical real-world danger of unencrypted protocols.

**Why Wireshark?**
It is the most widely used network analysis tool in the world — used by security analysts, forensic investigators, and network engineers. The ability to read and interpret packet captures is essential for SOC work and incident response.

**Key Wireshark Features Used:**

| Feature | What It Does |
|---------|--------------|
| **Capture Filter** | Limits captured traffic to a specific host or protocol before capture |
| **Display Filter** | Filters already-captured packets by protocol, IP, port, or content |
| **Follow TCP Stream** | Reconstructs the entire conversation of a TCP session in human-readable form |
| **Protocol Dissection** | Automatically identifies and decodes protocols (HTTP, FTP, Telnet, SMB, DNS) |
| `telnet` filter | Isolates all Telnet protocol packets |
| `http.request.method == "POST"` | Captures form submissions including login credentials |

---

## 📐 Methodology & Project Steps

This audit followed a structured penetration testing methodology aligned with industry standards (PTES — Penetration Testing Execution Standard).

```
Phase 1: Planning & Scope Definition
         ↓
Phase 2: Host Discovery
         ↓
Phase 3: Port & Service Scanning
         ↓
Phase 4: Version & OS Detection
         ↓
Phase 5: Vulnerability Scanning
         ↓
Phase 6: Traffic Analysis (Wireshark)
         ↓
Phase 7: Documentation & Reporting
```

---

### Phase 1 — Planning & Scope Definition

Before any scanning began, the scope was formally defined:

- **Target in scope:** `192.168.122.175` (Metasploitable 2 only)
- **Network in scope:** `192.168.122.0/24` (isolated private network only)
- **Explicitly out of scope:** Any real external IP, any production system, the internet
- **Testing type:** Black-box — no prior knowledge of target configuration assumed
- **Testing window:** Unrestricted (lab environment)
- **Goal:** Complete enumeration and vulnerability identification — no active exploitation in this phase

---

### Phase 2 — Host Discovery

**Objective:** Confirm the target is live and responsive before scanning.

```bash
# Ping scan to discover live hosts on the network
nmap -sn 192.168.122.175/24
```
<img width="1366" height="739" alt="Screenshot From 2026-04-23 10-43-22" src="https://github.com/user-attachments/assets/c59dc772-f2b6-431a-b2d1-c143fda6eed0" />



**Result:** `192.168.122.175` confirmed as live — 2 host up.

---

### Phase 3 — Port Scanning

**Objective:** Identify all open TCP ports on the target.

```bash
# TCP SYN scan of top 1000 ports
nmap -sS 192.168.122.175
```
<img width="1366" height="739" alt="Screenshot From 2026-04-23 10-35-05" src="https://github.com/user-attachments/assets/a270977d-e4c4-4d67-bcaf-977cc0926cb0" />




# Full port scan — all 65,535 ports
nmap -sS -p- 192.168.122.175 -T4
```
```
<img width="1366" height="739" alt="Screenshot From 2026-04-23 10-38-51" src="https://github.com/user-attachments/assets/f6a8d268-19c1-4afc-bf61-a003585f6db8" />



**Result:** 30 open ports discovered including 21 (FTP), 22 (SSH), 23 (Telnet), 25 (SMTP), 80 (HTTP), 139/445 (SMB), 1524 (shell), 3306 (MySQL), 5432 (PostgreSQL), 6667 (IRC), and more.

---

### Phase 4 — Service Version & OS Detection

**Objective:** Identify exact software versions running on each open port.

```bash
# Service version + OS detection
nmap -sV -O 192.168.122.175
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/35085496-3ef7-46cb-a0dd-72915ea7c517" />

# Aggressive scan with all default scripts
nmap -A -T4 192.168.122.175
```
```

<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/9efe34ee-a32e-412d-ad6b-b5891973c1a4" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/2fbda742-4144-47ec-b9a3-52814dcdcfab" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/0643fb9c-79a5-4f3e-8024-bd2af815a529" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/c5358670-bc01-4bd0-9aae-87d2f29af7fc" />








**Key Version Discoveries:**

| Port  | Service     | Version Identified              |
|-------|-------------|----------------------------------|
| 21    | FTP         | vsftpd 2.3.4                    |
| 22    | SSH         | OpenSSH 4.7p1 (Debian)          |
| 23    | Telnet      | Linux telnetd                   |
| 80    | HTTP        | Apache httpd 2.2.8               |
| 139/445| SMB        | Samba 3.0.20-Debian             |
| 3306  | MySQL       | MySQL 5.0.51a-3ubuntu5          |
| 6667  | IRC         | UnrealIRCd                      |
| 1524  | Shell       | Metasploitable root shell       |
| 5432  | PostgreSQL  | PostgreSQL DB 8.3.0 - 8.3.7     |

---

### Phase 5 — Vulnerability Scanning

**Objective:** Run automated vulnerability detection scripts against identified services.

```bash
# Nmap vulnerability scripts — all services
nmap --script vuln 192.168.122.175
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/46196679-8550-4582-adf8-4eaf64cf0835" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/59741ad5-6d37-4e27-afbe-7e03ea4eee72" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/35d842bf-d86f-4043-b113-efbb5ceef809" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/71e54dcc-0092-404f-a6f8-0400da6d6666" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/62e04efb-810f-4e9f-97de-9d4526b9e869" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/d13be7d4-6d25-44c9-a05c-cbb775e0cf2d" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/ac5e3292-8b0b-4975-b979-8083ea4dbad0" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/28f134a9-03c6-40a6-89d3-dcaa23a4d503" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/25df1d70-986b-4fdf-b18c-89422d1a02b8" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/4edf34c8-f329-4edc-bb64-04cace82c260" />



















# FTP anonymous login test
nmap --script ftp-anon 192.168.122.175 -p 21
```
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/9a7e5180-f0de-447b-8cf3-67009804ed8e" />


# SMB vulnerability checks
nmap --script smb-vuln* 192.168.122.175 -p 139,445
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/3d9aa31e-9ab5-44d4-a956-05426218744b" />

---

# Full combined vulnerability scan
nmap -sV --script vuln,ftp-anon,smb-vuln* -T4 192.168.122.175 -oN audit_results.txt
```
---
**Result:** Multiple critical CVEs confirmed. Full findings documented in the vulnerability report section below.

---

### Phase 6 — Traffic Analysis (Wireshark)

**Objective:** Demonstrate the real-world danger of cleartext authentication protocols by capturing credentials in transit.

#### Telnet Credential Capture
```
---

Steps:
1. Start Wireshark → Select network interface → Start capture
2. Open new terminal: telnet 192.168.122.175
3. Log in with credentials: msfadmin / msfadmin
4. In Wireshark: Apply filter → telnet
5. Right-click any Telnet packet → Follow → TCP Stream
```

**Result:** Full username and password captured in plaintext — visible to any observer on the network segment. Every keystroke was visible including the password.

#### What This Proves
Any attacker with access to the network (via ARP poisoning, rogue AP, or direct access) can capture every character typed during a Telnet session. This is why Telnet is completely unacceptable in any environment and must be replaced with SSH.

---

### Phase 7 — Documentation & Reporting

All findings were compiled into a 21-slide professional presentation with:
- Executive summary (non-technical audience)
- Per-finding detail: description, CVE, CVSS score, evidence, remediation
- Prioritised remediation roadmap
- Network diagram and tool overview

---

## 🔍 Key Findings

### Summary Dashboard

| Severity | Count | Description |
|----------|-------|-------------|
| 🔴 Critical | 4 | Remotely exploitable, no authentication required |
| 🟠 Medium   | 3 | Significant risk, requires some conditions |
| 🟡 Low      | 2 | Information disclosure, reduced exposure risk |
| **Total**   | **9** | |

---

### 🔴 Critical Vulnerabilities

---

#### CRIT-01 — vsftpd 2.3.4 Backdoor

| Field | Detail |
|-------|--------|
| **CVE** | CVE-2011-2523 |
| **CVSS Score** | 10.0 (Critical) |
| **Port** | 21/TCP (FTP) |
| **Service** | vsftpd 2.3.4 |
| **Authentication Required** | None |
| **Network Access** | Remote |

**Description:**
vsftpd version 2.3.4 was distributed with a malicious backdoor inserted into the source code by an attacker who compromised the distribution server. When a user connects to the FTP service and sends a username containing a smiley face (`:)`) as part of the login, the backdoor opens a root command shell on TCP port 6200. No valid credentials are required. Any unauthenticated remote attacker can gain a root shell on the system instantly.

**Evidence:**
```
Nmap output:
21/tcp open  ftp  vsftpd 2.3.4
```

**Real-World Impact:**
Complete system compromise. An attacker gains root (administrator) access and full control of the machine. They can read all data, install malware, create new accounts, pivot to other systems, and maintain persistent access.

**Remediation:**
Immediately replace vsftpd 2.3.4 with a patched version (2.3.5+). If FTP is required, configure it with explicit TLS (FTPS) and restrict access via firewall to trusted IPs only. If FTP is not required, disable and remove the service entirely.

---

#### CRIT-02 — Samba 3.0.20 Remote Code Execution

| Field | Detail |
|-------|--------|
| **CVE** | CVE-2007-2447 |
| **CVSS Score** | 9.3 (Critical) |
| **Port** | 139/TCP, 445/TCP (SMB) |
| **Service** | Samba 3.0.20 |
| **Authentication Required** | None |
| **Network Access** | Remote |

**Description:**
Samba versions 3.0.0 through 3.0.25rc3 contain a command injection vulnerability in the MS-RPC functionality used during username handling. When using the non-default "username map script" configuration option, an attacker can inject arbitrary shell commands via the username field by inserting shell metacharacters (backticks or `$(...)`). No authentication is required. This is one of the most well-known Linux remote code execution vulnerabilities.

**Evidence:**
```
Nmap output:
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  microsoft-ds Samba smbd 3.0.20-Debian
```

**Real-World Impact:**
Remote code execution as the 'nobody' user (which can escalate to root with secondary exploitation). Complete server compromise. In a real organisation, this would mean access to file shares, user credentials, and adjacent systems.

**Remediation:**
Upgrade Samba to version 3.0.25rc3 or later. Patch immediately. Restrict SMB ports (139/445) at the perimeter firewall — SMB should never be exposed to the internet. Implement host-based firewall rules limiting SMB access to authorised hosts only.

---

#### CRIT-03 — UnrealIRCd 3.2.8.1 Backdoor

| Field | Detail |
|-------|--------|
| **CVE** | CVE-2010-2075 |
| **CVSS Score** | 10.0 (Critical) |
| **Port** | 6667/TCP, 6697/TCP (IRC) |
| **Service** | UnrealIRCd 3.2.8.1 |
| **Authentication Required** | None |
| **Network Access** | Remote |

**Description:**
UnrealIRCd version 3.2.8.1 was distributed with a backdoor inserted into the source code. The backdoor triggers when an attacker sends a specific string starting with `AB` to the IRC port. This causes the server to execute any command that follows as the user running UnrealIRCd — often root or an elevated account. Similar to the vsftpd backdoor, this was a supply chain attack — the official distribution archive was replaced with a backdoored version.

**Evidence:**
```
Nmap output:
6667/tcp open  irc  UnrealIRCd
```

**Real-World Impact:**
Arbitrary remote command execution. Full control of the server depending on the user context UnrealIRCd runs under. Combined with local privilege escalation, this leads to complete root compromise.

**Remediation:**
Remove UnrealIRCd immediately. If an IRC service is genuinely required, replace with a clean, verified version from a trustworthy source. Verify file integrity by comparing checksums against officially published hashes. Block port 6667 at the perimeter firewall.

---

#### CRIT-04 — Exposed Root Shell on Port 1524

| Field | Detail |
|-------|--------|
| **CVE** | N/A — Intentional Metasploitable Configuration |
| **CVSS Score** | 10.0 (Critical — Custom Assessment) |
| **Port** | 1524/TCP |
| **Service** | Bindshell (root shell) |
| **Authentication Required** | None |
| **Network Access** | Remote |

**Description:**
Metasploitable 2 has a pre-configured bind shell listening on port 1524. Any TCP connection to this port immediately presents a root shell prompt with no authentication required. This is essentially an open backdoor providing instant root access to any attacker who can reach the port.

**Evidence:**
```
Nmap output:
1524/tcp open  bindshell Metasploitable root shell

Manual verification:
nc 192.168.122.175 1524
root@metasploitable:/#
```

**Real-World Impact:**
Instant root compromise. No tools, exploits, or credentials needed. Complete system takeover in under 5 seconds.

**Remediation:**
Close and remove the bind shell immediately. Investigate how it was installed — its presence indicates prior compromise or a severely misconfigured deployment. Perform a full system forensic review. Block all unexpected listening ports at the host-based and network firewall.

---

### 🟠 Medium Vulnerabilities

---

#### MED-01 — Telnet Service Enabling Cleartext Credential Transmission

| Field | Detail |
|-------|--------|
| **CVE** | No specific CVE — Protocol design flaw |
| **CVSS Score** | 7.5 (High — elevated from Medium due to active credential capture) |
| **Port** | 23/TCP (Telnet) |
| **Service** | Linux telnetd |
| **Authentication Required** | Valid credentials (but transmitted in plaintext) |
| **Network Access** | Remote / Local Network |

**Description:**
The Telnet protocol transmits all data — including usernames, passwords, and commands — in unencrypted plaintext. Any attacker with the ability to intercept network traffic (via ARP poisoning, a rogue access point, SPAN port access, or physical network access) can capture the complete authentication session and all subsequent commands.

**Evidence:**
Credentials captured live using Wireshark during the audit:
```
Wireshark TCP Stream (Telnet session):
Username: msfadmin
Password: msfadmin  ← visible in plaintext
```

**Real-World Impact:**
Complete credential theft requiring only passive network access. Captured credentials can be reused immediately for authentication or leveraged for lateral movement across the network.

**Remediation:**
Disable Telnet immediately. Replace with SSH (Secure Shell) which provides encrypted authentication and command transmission. There is no scenario in which Telnet is acceptable in a modern environment.

---

#### MED-02 — Anonymous FTP Access Permitted

| Field | Detail |
|-------|--------|
| **CVE** | N/A — Configuration weakness |
| **CVSS Score** | 6.4 (Medium) |
| **Port** | 21/TCP (FTP) |
| **Service** | vsftpd (anonymous auth enabled) |
| **Authentication Required** | None (anonymous) |
| **Network Access** | Remote |

**Description:**
The FTP service permits anonymous login — any user can authenticate using the username `anonymous` with any email as the password and gain read (and potentially write) access to the FTP directory without valid credentials.

**Evidence:**
```
Nmap output:
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x  2 0  65534    4096 Mar 17  2010 .
```

**Real-World Impact:**
Unauthorised access to all files in the FTP directory. In a real environment, anonymous FTP access can expose sensitive configuration files, credentials, or business documents. Write access (if permitted) could allow an attacker to upload malicious files.

**Remediation:**
Disable anonymous FTP access. Require authentication for all FTP connections. Better still, replace FTP entirely with SFTP (SSH File Transfer Protocol) which provides encrypted file transfer. Restrict FTP access to specific IP addresses if the service must remain.

---

#### MED-03 — MySQL Database Accessible Without Authentication

| Field | Detail |
|-------|--------|
| **CVE** | N/A — Configuration weakness |
| **CVSS Score** | 6.8 (Medium) |
| **Port** | 3306/TCP (MySQL) |
| **Service** | MySQL 5.0.51a |
| **Authentication Required** | None (root account has no password) |
| **Network Access** | Remote |

**Description:**
The MySQL database server is accessible on port 3306 from any host on the network. The root database account has no password set. An attacker can connect directly to the database engine and execute arbitrary SQL commands — including reading all databases, creating new admin accounts, and potentially executing OS commands via `SELECT INTO OUTFILE` or `LOAD_FILE()` functions.

**Evidence:**
```
Nmap output:
3306/tcp open  mysql  MySQL 5.0.51a-3ubuntu5
```

**Real-World Impact:**
Complete database compromise. Full read/write access to all stored data. Potential escalation to OS command execution depending on MySQL configuration and permissions.

**Remediation:**
Set a strong password for the MySQL root account immediately. Bind MySQL to 127.0.0.1 (localhost only) so it is not accessible from the network. Implement firewall rules blocking external access to port 3306. Apply the principle of least privilege to all database accounts.

---

### 🟡 Low Vulnerabilities

---

#### LOW-01 — SSH Running Outdated Version (OpenSSH 4.7p1)

| Field | Detail |
|-------|--------|
| **CVE** | Multiple historical CVEs |
| **CVSS Score** | 3.5 (Low — mitigated by no current active exploit) |
| **Port** | 22/TCP (SSH) |
| **Service** | OpenSSH 4.7p1 |

**Description:**
The SSH service is running OpenSSH 4.7p1, released in 2007. This version is over 15 years out of date and is subject to multiple historical CVEs. While no critical actively-exploited vulnerability exists for this specific version in isolation, running end-of-life software with known historical vulnerabilities represents an unnecessary risk.

**Remediation:**
Update OpenSSH to the latest stable version. Implement SSH key-based authentication and disable password-based SSH login. Restrict SSH access to authorised IP addresses via firewall rules.

---

#### LOW-02 — HTTP Service Exposing Apache Version Information

| Field | Detail |
|-------|--------|
| **CVE** | N/A — Information Disclosure |
| **CVSS Score** | 2.6 (Low) |
| **Port** | 80/TCP (HTTP) |
| **Service** | Apache httpd 2.2.8 |

**Description:**
The HTTP server returns the exact Apache version and OS information in its response headers (`Server: Apache/2.2.8 (Ubuntu) DAV/2`). This is information disclosure — it gives attackers the exact target version to search for known vulnerabilities without needing to fingerprint the system.

**Remediation:**
Configure Apache to suppress version information by setting `ServerTokens Prod` and `ServerSignature Off` in the Apache configuration file. This does not fix vulnerabilities but denies attackers free reconnaissance data.

---

## 📊 Full Vulnerability Report

### Summary Table

| ID | Vulnerability | Port | CVE | CVSS | Severity | Auth Required | Remote Exploit |
|----|---------------|------|-----|------|----------|---------------|----------------|
| CRIT-01 | vsftpd 2.3.4 Backdoor | 21 | CVE-2011-2523 | 10.0 | 🔴 Critical | No | Yes |
| CRIT-02 | Samba 3.0.20 RCE | 139/445 | CVE-2007-2447 | 9.3 | 🔴 Critical | No | Yes |
| CRIT-03 | UnrealIRCd Backdoor | 6667 | CVE-2010-2075 | 10.0 | 🔴 Critical | No | Yes |
| CRIT-04 | Root Bind Shell | 1524 | N/A | 10.0 | 🔴 Critical | No | Yes |
| MED-01 | Telnet Cleartext Auth | 23 | N/A | 7.5 | 🟠 Medium | Yes | Yes |
| MED-02 | Anonymous FTP | 21 | N/A | 6.4 | 🟠 Medium | No | Yes |
| MED-03 | MySQL No Auth | 3306 | N/A | 6.8 | 🟠 Medium | No | Yes |
| LOW-01 | Outdated OpenSSH | 22 | Multiple | 3.5 | 🟡 Low | Yes | Limited |
| LOW-02 | Apache Version Disclosure | 80 | N/A | 2.6 | 🟡 Low | No | Indirect |

### Remediation Priority Roadmap

```
IMMEDIATE (within 24 hours):
├── Isolate the system from all network access
├── CRIT-01: Replace vsftpd 2.3.4 — active backdoor
├── CRIT-02: Patch Samba — active RCE
├── CRIT-03: Remove UnrealIRCd — active backdoor
└── CRIT-04: Kill and remove the bind shell on port 1524

SHORT TERM (within 1 week):
├── MED-01: Disable Telnet, deploy SSH
├── MED-02: Disable anonymous FTP, migrate to SFTP
└── MED-03: Set MySQL root password, bind to localhost

ONGOING:
├── LOW-01: Update OpenSSH, enforce key-based auth
├── LOW-02: Suppress Apache version headers
├── Enable host-based firewall (ufw/iptables) default-deny
└── Implement patch management process for all services
```

---

## 📡 Wireshark Credential Capture
```
<img width="843" height="477" alt="Screenshot From 2026-04-05 20-34-07" src="https://github.com/user-attachments/assets/7f494e75-adf8-400d-9c82-ba6c23e2393a" />
<img width="843" height="477" alt="Screenshot From 2026-04-05 20-34-12" src="https://github.com/user-attachments/assets/5f1cbba3-418b-4a30-bac6-cbc6e16aa78a" />


### What Was Captured
Using Wireshark during a live Telnet session to the target, the following was observed:

```
[Wireshark TCP Stream — Telnet Session to 192.168.122.175:23]

Ubuntu 8.04
TelnetServer ready.

login: msfadmin
Password: Setan get behind me

Last login: [timestamp]
Linux metasploitable 2.6.24-16-server #1 SMP ...

msfadmin@metasploitable:~$
```

Every character of the username and password was transmitted in plaintext and captured in a single TCP stream reassembly — visible to anyone on the same network segment.

### Why This Matters
This demonstration illustrates why protocol selection is a security-critical decision, not just a technical one. Telnet was designed in 1969 when networks were trusted and private. In any environment where an attacker can reach the network — including through a compromised adjacent system, a rogue access point, or ARP poisoning — Telnet provides zero protection for credentials.

**SSH (Secure Shell) vs Telnet:**

| Feature | Telnet | SSH |
|---------|--------|-----|
| Encryption | None — plaintext | AES-256 encrypted |
| Authentication | Password only | Password + public key |
| Integrity | None | HMAC verification |
| Credential protection | Zero | Full |
| Recommended use | Never | Always |

---

## 📚 Lessons Learned

### Technical Lessons

**1. Supply Chain Attacks Are Real and Dangerous**
Both vsftpd 2.3.4 and UnrealIRCd 3.2.8.1 contained backdoors inserted into official distribution packages by attackers who compromised the download servers. This is a real threat pattern — the SolarWinds attack (2020) followed the exact same model at a massive scale. The lesson: always verify file integrity via cryptographic checksums and never blindly trust official-looking packages.

**2. Protocol Age ≠ Protocol Safety**
Telnet was designed in 1969. Its continued use in 2024 is not a technical limitation — it is a choice. Every cleartext protocol (Telnet, FTP, HTTP, SMTP without TLS) was designed before network security was a concern. The existence of these services on any network today represents a deliberate decision to accept unnecessary risk.

**3. Version Information Is Your Attack Surface**
Knowing that a service is running vsftpd 2.3.4 or Samba 3.0.20 is all an attacker needs to find a working exploit. Version enumeration (`-sV` in Nmap) is often the most productive phase of a penetration test. Defenders must treat version information as sensitive and suppress it wherever possible.

**4. Default Configurations Are Almost Always Insecure**
Anonymous FTP, MySQL with no root password, and a bind shell listening on 1524 are all default configurations in Metasploitable 2. They represent a pattern seen constantly in real environments: systems deployed quickly with default settings that were never hardened. Security must be explicit, not assumed.

**5. A Single Unpatched Service Can Compromise an Entire Network**
The vsftpd backdoor, the Samba RCE, and the UnrealIRCd backdoor all provide root access independently. Any one of them is sufficient to compromise the entire machine. In a real network, this machine would become the launchpad for lateral movement to every other system that trusts it.

**6. Network Visibility Is Non-Negotiable**
Before this audit, I did not know what services were running on the target. After Nmap, I had a complete picture of the attack surface in under 5 minutes. Organisations must perform regular internal scanning of their own networks — if you do not know what is exposed, you cannot defend it.

### Process Lessons

**7. Documentation Is as Important as Discovery**
Finding a vulnerability is only half the work. A finding with no CVE reference, no CVSS score, no business impact description, and no remediation recommendation is not actionable. The goal of security work is not to demonstrate skill — it is to enable the organisation to improve its security posture. This requires clear, professional communication.

**8. Methodology Prevents Missed Findings**
Working systematically through phases (discovery → scanning → enumeration → vulnerability identification) ensures nothing is missed. Ad-hoc scanning based on intuition leads to gaps. The structured approach in this audit found 9 distinct findings that a less methodical approach might have missed.

---

## 🚀 Next Steps

### Phase 2 — Exploitation (Planned)

With the reconnaissance and vulnerability identification phase complete, the next stage is active exploitation of the identified vulnerabilities using Metasploit Framework. This will demonstrate the full impact of each vulnerability — moving beyond theoretical risk to demonstrated, evidence-based proof of exploitability.

**Planned Exploitation Targets:**

```
┌─────────────────────────────────────────────────────────────┐
│  Target Vulnerability      │  Tool           │  Goal        │
├────────────────────────────┼─────────────────┼──────────────┤
│  vsftpd 2.3.4 backdoor     │  Metasploit     │  Root shell  │
│  Samba 3.0.20 CVE-2007-2447│  Metasploit     │  RCE shell   │
│  UnrealIRCd backdoor       │  Metasploit     │  Remote shell│
│  Port 1524 bind shell      │  Netcat         │  Root shell  │
│  MySQL no-auth access      │  mysql client   │  DB dump     │
└─────────────────────────────────────────────────────────────┘
```

**Planned Documentation:**
Each exploitation will be documented with:
- Metasploit module used and configuration
- Screenshot evidence of successful exploitation
- Post-exploitation enumeration (whoami, uname -a, id)
- Demonstration of what an attacker could achieve from that access point

---

### Phase 3 — System Hardening

After exploitation is documented, the same Metasploitable system will be hardened against all identified vulnerabilities. This demonstrates the defender's perspective — understanding both how to attack and how to defend is the foundation of professional security practice.

**Planned Hardening Steps:**

- [ ] Replace vsftpd 2.3.4 with a patched version; disable if not needed
- [ ] Upgrade Samba; restrict SMB to LAN only via firewall rules
- [ ] Remove UnrealIRCd; block IRC ports at firewall
- [ ] Kill and remove the port 1524 bind shell; audit all listening ports
- [ ] Disable Telnet (`systemctl disable telnet`); enforce SSH-only remote access
- [ ] Configure SSH: disable password auth, enforce public key only, change default port
- [ ] Disable anonymous FTP; require authentication; migrate to SFTP
- [ ] Set MySQL root password; bind MySQL to 127.0.0.1; restrict remote access
- [ ] Configure UFW firewall: default-deny inbound, permit only required services
- [ ] Suppress Apache server version information
- [ ] Update all packages: `apt-get update && apt-get upgrade`

After hardening, the same Nmap scans will be re-run to confirm attack surface reduction.

---

### Phase 4 — Further Learning & Development

This project is one milestone in a longer security development roadmap:

```
Current Status                    →  Certifications Roadmap
─────────────────────────────────────────────────────────────
✅ Network+ (Completed)            →  CompTIA Security+ SY0-701
✅ Network Security Audit          →  CompTIA PenTest+
🔄 Security+ (Exam Pending)        →  eJPT (eLearnSecurity)
🔄 TryHackMe Jr PenTester Path     →  OSCP (OffSec Certified Professional)
```

**Practical Skills Pipeline:**
- Complete TryHackMe Jr Penetration Tester path (username: Emmy)
- Progress through PortSwigger Web Security Academy (free)
- Participate in HackerOne bug bounty program (beginner scope)
- Build a DVWA (Damn Vulnerable Web App) web application testing writeup
- Set up a full Splunk SIEM lab for defensive security practice

---

## 📖 References

| Resource | URL |
|----------|-----|
| CVE-2011-2523 (vsftpd backdoor) | https://nvd.nist.gov/vuln/detail/CVE-2011-2523 |
| CVE-2007-2447 (Samba RCE) | https://nvd.nist.gov/vuln/detail/CVE-2007-2447 |
| CVE-2010-2075 (UnrealIRCd) | https://nvd.nist.gov/vuln/detail/CVE-2010-2075 |
| Metasploitable 2 Documentation | https://docs.rapid7.com/metasploit/metasploitable-2/ |
| Nmap Official Documentation | https://nmap.org/docs.html |
| Nmap NSE Script Reference | https://nmap.org/nsedoc/ |
| Wireshark User Guide | https://www.wireshark.org/docs/wsug_html/ |
| CVSS Scoring Calculator | https://www.first.org/cvss/calculator/3.1 |
| PTES (Pentest Standard) | http://www.pentest-standard.org/ |
| MITRE ATT&CK Framework | https://attack.mitre.org/ |
| OWASP Testing Guide | https://owasp.org/www-project-web-security-testing-guide/ |
| NIST National Vulnerability Database | https://nvd.nist.gov/ |

---

---

## 📁 Repository Structure

```
network-security-audit-metasploitable2/
│
├── README.md                          ← This file
│
├── scans/
│   ├── nmap_host_discovery.txt        ← Phase 2 scan results
│   ├── nmap_port_scan.txt             ← Phase 3 scan results
│   ├── nmap_service_version.txt       ← Phase 4 scan results
│   └── nmap_vuln_scripts.txt          ← Phase 5 vulnerability scan
│
├── wireshark/
│   ├── telnet_capture.pcap            ← Raw Wireshark capture file
│   └── telnet_tcp_stream.png          ← Screenshot of credential capture
│
├── screenshots/
│   ├── nmap_full_scan.png
│   ├── port_1524_bindshell.png
│   ├── ftp_anonymous_login.png
│   └── wireshark_telnet_creds.png
│
└── report/
    └── Network_Security_Audit_Report.pdf  ← Full presentation/report
```

---

<div align="center">

**⭐ If this project was useful or interesting, please star the repository**

*This project is part of an ongoing cybersecurity portfolio. Follow for Phase 2 (Exploitation) and Phase 3 (Hardening) updates.*

---

![Made with](https://img.shields.io/badge/Made%20with-Kali%20Linux-557C94?style=flat-square&logo=kalilinux)
![Tools](https://img.shields.io/badge/Nmap-7.94-green?style=flat-square)
![Tools](https://img.shields.io/badge/Wireshark-4.x-blue?style=flat-square)
![Education](https://img.shields.io/badge/New%20Horizon-Nigeria-orange?style=flat-square)

</div>
