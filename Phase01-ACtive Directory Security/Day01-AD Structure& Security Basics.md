
---

# 🔵 PHASE 1 — DAY 1
# Active Directory Security — Structure & Security Basics

---

## 📌 1. TOPIC OVERVIEW

**What is it?**
Active Directory (AD) is Microsoft's system for managing users, computers, and resources in a company network — but from a **security perspective**, it's also the **#1 target** for attackers inside corporate networks.

**Why does it matter for security?**
If an attacker compromises AD, they own the entire company. Every user, every server, every file share — all controlled through AD. That's why attacking and defending AD is the **most critical skill** in enterprise cybersecurity.

> 🔗 **Your connection:** You've been managing AD at your organization — creating users, managing groups, handling DNS/DHCP. Today we look at the **exact same things** through an attacker's eyes.

---

## 🧠 2. AD STRUCTURE — SECURITY LENS

Think of AD like a **company building**:

```
🏢 FOREST (the entire company group)
    └── 🏬 DOMAIN (one office building e.g. tcs.com)
            └── 🚪 OU - Organizational Unit (floors/departments)
                    └── 👤 Users, 💻 Computers, 📁 Groups
```

### Key Components & Why Attackers Care

| Component | What It Is | Why Attackers Target It |
|-----------|-----------|------------------------|
| **Domain Controller (DC)** | The server that runs AD | Compromise DC = own everything |
| **Domain Admin** | Highest privilege user | Stealing this account = game over |
| **GPO (Group Policy)** | Rules pushed to all computers | Attackers abuse GPOs to run malware on all machines |
| **OU (Org Unit)** | Folder for users/computers | Misconfigurations here expose sensitive accounts |
| **Service Accounts** | Accounts used by apps/services | Often have high privileges + weak passwords |
| **Trust Relationships** | Links between domains/forests | Attackers use trusts to move between domains |

---

## ⚙️ 3. HOW AD AUTHENTICATION WORKS

This is **critical** — you must understand this before Day 2 (attacks).

### The Kerberos Flow (How Login Works)
```
👤 User types password
        ↓
🎫 DC issues a "Ticket Granting Ticket" (TGT)  ← like a wristband at an event
        ↓
👤 User presents TGT to access a resource
        ↓
🎫 DC issues a "Service Ticket"  ← like a room key
        ↓
✅ User accesses the resource
```

**Simple analogy:** Kerberos is like an airport.
- You show your passport (password) once at check-in
- You get a boarding pass (TGT)
- You show the boarding pass at each gate (service ticket) — no need to show passport again

> 🔗 **Your connection:** When you log into a domain-joined machine at your organization and it automatically accesses file shares — that's Kerberos silently working in the background.

---

## 🏢 4. CRITICAL AD SECURITY CONCEPTS

### 4.1 — Privileged Groups (The Crown Jewels)

```
👑 Domain Admins       — Full control over the entire domain
👑 Enterprise Admins   — Full control over the entire forest
👑 Schema Admins       — Can change AD structure itself
⚠️  Account Operators   — Can create/modify user accounts
⚠️  Backup Operators    — Can access all files for backup (often abused)
⚠️  Server Operators    — Can log into domain controllers
```

**Security Rule #1:** The fewer people in these groups, the better. Most breaches happen because too many people are Domain Admins.

### 4.2 — Service Accounts (Most Overlooked Risk)

Service accounts are accounts used by software (SQL Server, backup agents like your Commvault, monitoring tools).

**The problem:**
- They often have high privileges
- Their passwords are set once and never changed
- They never expire or get disabled
- They are **perfect targets** for Kerberoasting attacks (Day 2!)

> 🔗 **Your connection:** Think about your Commvault backup service account at TCS — what permissions does it have? That's exactly what an attacker looks for.

### 4.3 — Group Policy Objects (GPOs)

GPOs push settings to all computers in a domain — things like password policies, software restrictions, desktop backgrounds.

**Security uses of GPO:**
- Force screen lock after 5 minutes
- Disable USB drives
- Block command prompt for standard users
- Deploy security software

**Attacker use of GPO:**
- If attacker becomes Domain Admin → create a malicious GPO → runs malware on every machine in the domain simultaneously

### 4.4 — ACLs & Delegation (Common Misconfiguration)

ACL = Access Control List. Every AD object has one — it defines who can do what to that object.

**Dangerous delegations attackers look for:**
- User A can **reset passwords** for Domain Admins
- User B has **Write permissions** on Domain Controller object
- Service account has **DCSync rights** (can dump all password hashes)

These misconfigurations are **invisible to most admins** — but BloodHound (a tool you'll use in Day 5) finds them in seconds.

---

## ⚠️ 5. TOP AD ATTACK TYPES (Preview — Deep Dive Day 2)

| Attack | Simple Explanation |
|--------|-------------------|
| **Pass-the-Hash** | Steal a password hash → use it to login without knowing the actual password |
| **Pass-the-Ticket** | Steal a Kerberos ticket → impersonate that user |
| **Kerberoasting** | Request a service ticket for any service account → crack the password offline |
| **DCSync** | Pretend to be a DC → ask the real DC to send all password hashes |
| **Golden Ticket** | Forge a Kerberos ticket → unlimited access, survives password changes |
| **BloodHound Attack Paths** | Map AD permissions → find hidden path to Domain Admin |

---

## 🛡️ 6. HOW TO DEFEND AD

### The Three Most Important Defenses

**1. Tiered Administration Model**
```
Tier 0 — Domain Controllers only (only special admin accounts touch these)
Tier 1 — Servers (separate admin accounts for servers)
Tier 2 — Workstations (helpdesk accounts work here only)
```
Never use the same admin account across tiers. If Tier 2 is compromised, Tier 0 stays safe.

**2. LAPS — Local Administrator Password Solution**
Every Windows computer has a local Administrator account. Most companies set the same password on all machines.

Problem: Attacker compromises one machine → same password works on ALL machines.

LAPS fixes this by giving every machine a **unique, auto-rotating local admin password** stored in AD.

> 🔗 **Your connection:** You can deploy LAPS via GPO — something you already know how to work with!

**3. Protected Users Security Group**
Add sensitive accounts (Domain Admins) to this group. It automatically:
- Prevents caching of credentials
- Forces Kerberos (no NTLM fallback)
- Reduces ticket lifetime

---

## 📋 7. CRITICAL EVENT IDs TO KNOW

These are the log entries a SOC analyst watches for AD attacks:

| Event ID | What It Means | Why It Matters |
|----------|--------------|----------------|
| **4625** | Failed login | Brute force detection |
| **4648** | Login with explicit credentials | Lateral movement indicator |
| **4672** | Admin privileges assigned | Privilege escalation |
| **4720** | User account created | Rogue account creation |
| **4726** | User account deleted | Covering tracks |
| **4768** | Kerberos TGT requested | Normal, but monitor for anomalies |
| **4769** | Kerberos service ticket requested | Kerberoasting leaves fingerprints here |
| **4771** | Kerberos pre-authentication failed | Password spraying |
| **4776** | NTLM authentication | Should be rare in modern environments |

> 🔗 **Your connection:** You've seen Event Viewer before in Windows Server. Now you know exactly which IDs to filter for during a security incident.

---

## 👨‍💻 8. WHAT SECURITY PROFESSIONALS DO HERE

**SOC Analyst (Tier 1):**
- Monitor alerts for suspicious login patterns
- Watch for bulk Event ID 4625 (brute force)
- Escalate unusual Domain Admin activity

**SOC Analyst (Tier 2):**
- Investigate Kerberoasting attempts (Event 4769 spike)
- Correlate lateral movement across systems
- Hunt for unauthorized admin group changes

**Pen Tester / Red Team:**
- Run BloodHound to find attack paths
- Attempt Kerberoasting on service accounts
- Try DCSync if sufficient privileges gained

**Security Engineer:**
- Implement tiered admin model
- Deploy LAPS, Protected Users group
- Build Sentinel detection rules for AD attacks

---

## 🎯 9. INTERVIEW QUESTIONS WITH ANSWERS

**Q1: What is the difference between a Domain Admin and an Enterprise Admin?**
> Domain Admin controls one domain. Enterprise Admin controls all domains in the entire forest. Enterprise Admin is the highest privilege in an AD environment.

**Q2: What is Kerberos and why is it used instead of NTLM?**
> Kerberos is the default authentication protocol in AD. It uses tickets instead of sending passwords over the network, making it more secure than NTLM. NTLM is older, weaker, and vulnerable to Pass-the-Hash attacks.

**Q3: An alert shows 500 failed logins (Event 4625) from one IP in 2 minutes. What do you do?**
> This is a brute force attack. First, identify the source IP. Block it at the firewall. Check if any login succeeded. If yes, immediately disable that account and investigate that machine. Report to Tier 2 for deeper investigation.

**Q4: What is Kerberoasting?**
> Kerberoasting is an attack where an attacker requests service tickets for service accounts (which any domain user can do) and then cracks those tickets offline to recover the plaintext password. The defense is using long, complex passwords for service accounts and monitoring Event ID 4769.

**Q5: Why are service accounts risky?**
> Service accounts often have high privileges, never have their passwords changed, never expire, and aren't monitored like regular user accounts. Attackers specifically target them because compromising one often gives access to critical systems.

**Q6: Scenario — You find a junior helpdesk user is a member of Domain Admins. What do you do?**
> This is a privilege violation. Remove them from Domain Admins immediately, assign only the permissions they need (least privilege). Check audit logs to see how they were added, who added them, and what actions they performed with those privileges. Escalate as a potential insider threat or misconfiguration incident.

---

## ❓ 10. THINK ABOUT THIS

1. In your organization, how many people have Domain Admin access? Is that number justified?
2. Your Commvault service account — what AD groups is it in? Could an attacker abuse it?
3. If a single helpdesk user's laptop is compromised, what's the worst an attacker could reach from there?
4. How would you detect if someone added themselves to Domain Admins at 3 AM?

---

## 📚 11. TOMORROW — DAY 2

**Topic:** AD Attack Types — Pass-the-Hash, Kerberoasting, DCSync, Golden Ticket

**Tonight's homework:**
- Open Event Viewer on any Windows machine
- Filter Security log for Event ID **4625** and **4648**

---

> 💡 **Pro Tip from a Security Engineer:** *"Most AD breaches don't happen because of zero-day exploits. They happen because of misconfigurations that have existed for years — wrong group memberships, over-privileged service accounts, weak GPOs. Your admin experience makes you dangerous as a defender because you know where those misconfigurations hide."*

