
Let me build you a **structured, realistic daily plan** that's tailored specifically to your background and goal of becoming a **SOC Analyst / Security Engineer**.

---

# 🗓️ HAREESH'S CYBERSECURITY DAILY PLAN
## *From IT Pro → Cyber Security Professional*

---

## 🧭 STRATEGY FIRST — How This Plan Is Built

Your background already covers **infrastructure** deeply. So this plan does two things:
1. **Converts** your existing knowledge into security knowledge (AD → AD Attacks, Azure → Cloud Security)
2. **Adds new layers** — SIEM, threat detection, vulnerability management, offensive basics

**Target Role:** SOC Analyst Tier 1 → Tier 2 → Security Engineer
**Target Certs:** SC-200 (Microsoft Sentinel) → AZ-500 → CompTIA Security+ / CEH

---

## 📅 THE PLAN — 12 PHASES

---

### 🔵 PHASE 1 — Days 1–5
## Active Directory Security

> You already know AD administration. Now learn to **attack and defend** it.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 1 | AD Structure & Security Basics | Users, Groups, OUs, GPOs from a **security lens** |
| 2 | AD Attack Types | Pass-the-Hash, Kerberoasting, DCSync, Golden Ticket |
| 3 | AD Hardening | Tiered Admin model, LAPS, Protected Users group |
| 4 | AD Logs & Event IDs | Event ID 4625, 4648, 4768, 4769 — what to watch |
| 5 | Hands-On Lab | BloodHound + AD attack simulation on TryHackMe |

**Tools:** BloodHound, Mimikatz (lab only), Event Viewer, PowerShell
**Lab:** TryHackMe — "Active Directory Basics" + "Attacktive Directory"

---

### 🔵 PHASE 2 — Days 6–10
## Microsoft Entra ID (Azure AD) Security

> You use Entra ID at TCS. Now learn its **attack surface and defense**.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 6 | Entra ID Architecture | Tenants, users, roles, app registrations |
| 7 | Entra ID Attack Types | Token theft, MFA bypass, Illicit consent grants |
| 8 | Conditional Access Policies | How to block risky sign-ins |
| 9 | Entra ID Logs & Monitoring | Sign-in logs, audit logs, risky users |
| 10 | Hands-On Lab | SC-300 labs + Microsoft Learn sandbox |

**Tools:** Microsoft Entra Portal, Defender for Identity, Azure Monitor
**Cert Connection:** SC-300 Identity Administrator

---

### 🔵 PHASE 3 — Days 11–15
## Linux Security

> You use Linux servers. Now harden them and detect attacks on them.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 11 | Linux File Permissions & Users | chmod, chown, sudo abuse risks |
| 12 | Linux Hardening | SSH hardening, firewall (ufw/iptables), fail2ban |
| 13 | Linux Logs | /var/log/auth.log, syslog, journalctl |
| 14 | Linux Attack Techniques | Privilege escalation, cron job abuse, SUID exploits |
| 15 | Hands-On Lab | TryHackMe "Linux PrivEsc" + "Blue" room |

**Tools:** Lynis (hardening audit), Auditd, fail2ban, Nmap

---

### 🔵 PHASE 4 — Days 16–20
## Windows Security & Hardening

> Your Windows Server admin experience becomes your security weapon here.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 16 | Windows Security Architecture | SAM, LSA, NTLM, Kerberos internals |
| 17 | Windows Hardening | CIS Benchmarks, AppLocker, Windows Firewall rules |
| 18 | Windows Event Logs | Event IDs for login, process creation, lateral movement |
| 19 | Windows Attack Techniques | Pass-the-Hash, token impersonation, WMI abuse |
| 20 | Hands-On Lab | TryHackMe "Blue", "Investigating Windows" |

**Tools:** Sysmon, Windows Event Viewer, Autoruns, Process Monitor

---

### 🔵 PHASE 5 — Days 21–25
## Networking & Protocol Security

> You know DNS & DHCP. Now learn what attackers do to them.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 21 | TCP/IP & Packet Analysis (Wireshark) | Normal vs suspicious traffic patterns |
| 22 | DNS Security | DNS poisoning, DNS tunneling, DNSSEC |
| 23 | Firewalls & Network Segmentation | NSG rules, DMZ, Zero Trust network concepts |
| 24 | VPNs, Proxies & Tunneling | Split tunneling risks, C2 over HTTPS/DNS |
| 25 | Hands-On Lab | Wireshark PCAP analysis on malware-traffic-analysis.net |

**Tools:** Wireshark, Nmap, Zeek, Suricata

---

### 🔵 PHASE 6 — Days 26–30
## Cryptography

> From encoding to full encryption — this powers HTTPS, VPNs, and malware.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 26 | Hashing (MD5, SHA) | What it is, password hashing, hash cracking |
| 27 | Symmetric Encryption (AES) | How ransomware encrypts files |
| 28 | Asymmetric Encryption (RSA) | How HTTPS & digital signatures work |
| 29 | PKI & Certificates | How SSL/TLS works, cert pinning, MITM attacks |
| 30 | Hands-On Lab | CyberChef challenges + crack hashes on CrackStation |

**Tools:** CyberChef, Hashcat, OpenSSL

---

### 🔵 PHASE 7 — Days 31–37
## SIEM — Microsoft Sentinel

> This is your **core SOC tool**. This phase is critical for your career.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 31 | What is a SIEM? | Log collection, correlation, alerting — the big picture |
| 32 | Microsoft Sentinel Architecture | Workspaces, connectors, data sources |
| 33 | KQL (Query Language) | Writing queries to hunt for threats |
| 34 | Analytics Rules & Alerts | How to create detection rules |
| 35 | Incidents & Investigation | Triage, assign, investigate, close incidents |
| 36 | Playbooks & SOAR | Automating responses with Logic Apps |
| 37 | Hands-On Lab | Microsoft Sentinel training environment (free on Learn) |

**Tools:** Microsoft Sentinel, KQL, Logic Apps
**Cert Connection:** SC-200 Microsoft Security Operations Analyst

---

### 🔵 PHASE 8 — Days 38–42
## Microsoft Defender Suite

> Defender for Endpoint, Identity, Office 365, Cloud Apps — the full stack.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 38 | Defender for Endpoint | EDR, alerts, device isolation, threat hunting |
| 39 | Defender for Identity | Detecting AD attacks in real-time |
| 40 | Defender for Office 365 | Phishing detection, safe links, email analysis |
| 41 | Defender for Cloud Apps | CASB, shadow IT discovery, DLP |
| 42 | Hands-On Lab | Microsoft 365 Defender portal simulation labs |

**Tools:** Microsoft 365 Defender portal, Defender XDR

---

### 🔵 PHASE 9 — Days 43–47
## Vulnerability Management

> Finding weaknesses before attackers do.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 43 | CVE, CVSS & NVD | How vulnerabilities are scored and tracked |
| 44 | Nessus / Qualys Basics | Running a vulnerability scan |
| 45 | Patch Management | WSUS, Intune patching, remediation workflow |
| 46 | Defender Vulnerability Management | Built-in vuln tracking in Defender |
| 47 | Hands-On Lab | Nessus Essentials (free) — scan your home lab |

**Tools:** Nessus Essentials, Microsoft Defender Vulnerability Management, OpenVAS

---

### 🔵 PHASE 10 — Days 48–52
## Incident Response & Threat Hunting

> What you do when an alert fires at 2 AM.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 48 | IR Lifecycle | Preparation → Detection → Containment → Recovery |
| 49 | Threat Intelligence | IOCs, TTPs, MITRE ATT&CK framework |
| 50 | Threat Hunting with KQL | Proactive hunting in Sentinel |
| 51 | Forensics Basics | Memory dumps, disk forensics, browser artifacts |
| 52 | Hands-On Lab | Blue Team Labs Online — incident response scenarios |

**Tools:** MITRE ATT&CK Navigator, Velociraptor, Autopsy, CyberDefenders

---

### 🔵 PHASE 11 — Days 53–57
## Web Security Basics

> Understanding the attacks SOC analysts see most often.

| Day | Topic | What You'll Learn |
|-----|-------|-------------------|
| 53 | OWASP Top 10 | SQL Injection, XSS, IDOR — explained simply |
| 54 | Burp Suite Basics | Intercepting and modifying web requests |
| 55 | Web Logs Analysis | Apache/Nginx logs — spotting attacks |
| 56 | WAF & Web Security | How Web Application Firewalls work |
| 57 | Hands-On Lab | OWASP WebGoat + PortSwigger Web Security Academy |

**Tools:** Burp Suite Community, OWASP ZAP, PortSwigger Academy

---

### 🔵 PHASE 12 — Days 58–60
## Certification Prep & Career Launch

| Day | Topic | Action |
|-----|-------|--------|
| 58 | SC-200 Exam Prep | Review Sentinel, Defender, KQL — practice exam |
| 59 | Resume & LinkedIn | Build your security resume, update LinkedIn with labs |
| 60 | Mock Interview Prep | Practice top 20 SOC interview questions |

---

## 🏆 CERTIFICATION ROADMAP

```
START HERE (leverages your existing knowledge)
        ↓
   SC-300 (Identity Admin)   ← Your AD/Entra background = 70% done
        ↓
   SC-200 (Security Ops)     ← Sentinel, Defender, KQL = Core SOC cert
        ↓
   AZ-500 (Azure Security)   ← Cloud security architect level
        ↓
   CompTIA Security+          ← Globally recognized baseline cert
        ↓
   CEH or OSCP               ← Offensive / Pen Testing path (optional)
```

---

## 🛠️ YOUR HOME LAB SETUP (Do This This Week)

| Tool | Purpose | Cost |
|------|---------|------|
| TryHackMe | Guided cyber labs | Free / £10/month |
| Microsoft Learn | Sentinel, Defender, SC-200 labs | **Free** |
| Blue Team Labs Online | Incident response practice | Free |
| CyberDefenders | PCAP & forensics challenges | Free |
| Nessus Essentials | Vulnerability scanning | **Free** |
| VirtualBox + Windows Server ISO | Local AD lab | **Free** |

---

## 📊 YOUR DAILY ROUTINE (1.5–2 hours/day)

```
30 min  — Read / watch the day's topic
30 min  — Take structured notes (your strength!)  
30 min  — Hands-on lab or practice
15 min  — Review yesterday's topic (spaced repetition)
```

