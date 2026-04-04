
---

# 🔵 PHASE 1 — DAY 3
# AD Hardening — Closing Every Door the Attacker Used

---

## 🧭 THE DAY 3 MINDSET

In Day 2 we were the **attacker** — we broke into SecureCorp step by step.

Today we are the **architect** — Hareesh has just read the incident report and must ensure it **never happens again.**

```
DAY 2 — What the attacker did:
─────────────────────────────────────────────
Phishing → Gani's laptop
→ Kerberoasted SQLSvc (weak password)
→ Pass-the-Hash with Hareesh's cached hash
→ DCSync → got KRBTGT
→ Golden Ticket → permanent access
→ Ransomware via GPO
─────────────────────────────────────────────

DAY 3 — What Hareesh does now:
─────────────────────────────────────────────
Close every single one of those doors.
Permanently.
Before the next attacker tries.
─────────────────────────────────────────────
```

---

## 📌 SECTION 1 of 6
## Understanding the Hardening Framework

---

### The Security Layers Model

Think of AD hardening like **securing a bank building** — not one lock, but multiple independent layers:

```
SECURECORP AD — DEFENCE IN DEPTH
══════════════════════════════════════════════════════
LAYER 6 — DETECTION & RESPONSE
         Microsoft Defender for Identity
         Microsoft Sentinel — KQL alerts
         "If they get in — we see them immediately"
──────────────────────────────────────────────────────
LAYER 5 — PRIVILEGED ACCESS CONTROL
         Tiered Admin Model
         Privileged Identity Management (PIM)
         Privileged Access Workstations (PAW)
         "Limit what attackers can reach even if inside"
──────────────────────────────────────────────────────
LAYER 4 — CREDENTIAL PROTECTION
         Protected Users Group
         Disable NTLM
         Credential Guard
         "Make stolen credentials useless"
──────────────────────────────────────────────────────
LAYER 3 — SERVICE ACCOUNT HARDENING
         gMSA for all service accounts
         Audit & remove unnecessary SPNs
         "Remove the Kerberoasting target"
──────────────────────────────────────────────────────
LAYER 2 — GROUP POLICY HARDENING
         CIS Benchmark GPOs
         AppLocker / WDAC
         PowerShell Constrained Language Mode
         "Control what runs on every machine"
──────────────────────────────────────────────────────
LAYER 1 — FOUNDATION
         Patch Management
         LAPS
         Secure DC configuration
         "Build on solid ground"
══════════════════════════════════════════════════════
```

> 🎯 **Architect's Rule:** No single layer stops everything. Every layer slows the attacker down and gives Navi (SOC) more time to detect and respond. The goal is not to make intrusion impossible — it is to make it **noisy and slow.**

---

### The CIS Benchmark — Your Hardening Bible

Before writing a single GPO, every security engineer refers to the **CIS (Center for Internet Security) Benchmark.**

```
WHAT IS CIS BENCHMARK?
─────────────────────────────────────────────────────────
• A globally accepted set of security configuration standards
• Written by security experts across government, industry, academia
• Free to download from cisecurity.org
• Covers: Windows Server, Active Directory, Azure, Linux etc.

FOR AD HARDENING — KEY DOCUMENTS:
• CIS Microsoft Windows Server 2025 Benchmark
• CIS Microsoft Windows 10 Benchmark
• CIS Active Directory Benchmark

TWO LEVELS:
Level 1 — Essential controls. Apply everywhere. Low disruption.
Level 2 — Advanced controls. High security environments.
          May impact usability. Apply carefully.

HAREESH'S RULE:
→ Start with ALL Level 1 controls
→ Evaluate Level 2 controls one by one
→ Test in lab before production always
```

> 🔗 **Your connection:** At your organization, when you configured Windows Servers — you likely followed some internal baseline. CIS is the industry-standard version of that baseline. Now you understand WHY each setting exists.

---

## 📌 SECTION 2 of 6
## Layer 1 — Foundation Hardening

---

### 2.1 — Domain Password Policy (You've Done This — Now Let's Do It RIGHT)

You've set password policies via GPO before. Let's see what **SecureCorp's policy looks like before and after hardening:**

```
SECURECORP PASSWORD POLICY — BEFORE HARDENING:
════════════════════════════════════════════════
Minimum password length:     8 characters   ❌
Password complexity:         Enabled         ✅
Maximum password age:        90 days         ⚠️
Minimum password age:        1 day           ⚠️
Password history:            5 passwords     ❌
Account lockout threshold:   0 (disabled!)   ❌
════════════════════════════════════════════════

PROBLEMS:
• 8 chars = cracked in minutes with Hashcat
• No lockout = brute force attacks work freely
• 90-day rotation = users just add "1" at end
• History of 5 = users rotate through 5 then reuse
```

```
SECURECORP PASSWORD POLICY — AFTER HARDENING:
════════════════════════════════════════════════
Minimum password length:     15 characters  ✅
Password complexity:         Enabled         ✅
Maximum password age:        180 days        ✅
Minimum password age:        1 day           ✅
Password history:            24 passwords    ✅
Account lockout threshold:   5 attempts      ✅
Account lockout duration:    15 minutes      ✅
Observation window:          15 minutes      ✅
════════════════════════════════════════════════

WHY THESE NUMBERS:
• 15 chars → exponentially harder to crack
• 180 days → less pressure = stronger passwords chosen
• History 24 → 2 years of history, true prevention
• Lockout 5 → stops brute force, not too aggressive
• 15 min auto-unlock → reduces helpdesk calls
```

> 🔗 **Microsoft's latest guidance (2023):** They actually recommend **removing mandatory rotation** and instead using **longer passwords + MFA.** Forced rotation causes weaker passwords because users just increment numbers.

### GPO Location for Password Policy

```
Group Policy Management Console:
→ Forest → Domains → securecorp.local
→ Right-click → Create GPO → "SecureCorp-Password-Policy"
→ Edit:

Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Account Policies
                └── Password Policy
                    ├── Minimum password length: 15
                    ├── Maximum password age: 180
                    ├── Enforce password history: 24
                    └── Password must meet complexity: Enabled

                └── Account Lockout Policy
                    ├── Account lockout threshold: 5
                    ├── Account lockout duration: 15
                    └── Reset account lockout counter: 15
```

### Fine-Grained Password Policy (FGPP) — The Advanced Move

```
PROBLEM WITH SINGLE DOMAIN PASSWORD POLICY:
One policy applies to everyone — Gani AND Hareesh AND SQLSvc

BETTER APPROACH — Different policies for different groups:

┌─────────────────┬──────────────────┬───────────────────────┐
│ Group           │ Min Length       │ Special Rules         │
├─────────────────┼──────────────────┼───────────────────────┤
│ Domain Admins   │ 20 characters    │ No expiry + MFA       │
│ Service Accts   │ gMSA (240 chars) │ Auto-managed          │
│ Standard Users  │ 15 characters    │ 180-day rotation      │
│ Helpdesk        │ 15 characters    │ Extra lockout rules   │
└─────────────────┴──────────────────┴───────────────────────┘

HOW TO CREATE FGPP:
─────────────────────────────────────────────────────────
# PowerShell on DC01:
New-ADFineGrainedPasswordPolicy `
  -Name "DomainAdmin-PSO" `
  -Precedence 10 `
  -MinPasswordLength 20 `
  -PasswordHistoryCount 24 `
  -MaxPasswordAge "180.00:00:00" `
  -LockoutThreshold 3 `
  -LockoutDuration "00:30:00" `
  -LockoutObservationWindow "00:30:00" `
  -ComplexityEnabled $true `
  -ReversibleEncryptionEnabled $false

# Apply to Domain Admins group:
Add-ADFineGrainedPasswordPolicySubject `
  -Identity "DomainAdmin-PSO" `
  -Subjects "Domain Admins"

# Verify:
Get-ADFineGrainedPasswordPolicy -Identity "DomainAdmin-PSO"
```

> 🎯 **SOC Analyst Note — Navi monitors this:** If a Domain Admin account has a failed login — lockout threshold is 3 not 5. More sensitive. Navi gets an alert faster.

---

### 2.2 — LAPS — Local Administrator Password Solution

Remember Day 2? The attacker moved from Gani's laptop to SQLDB01 because **every machine had the same local admin password.**

LAPS fixes this permanently.

```
WITHOUT LAPS — THE PROBLEM:
─────────────────────────────────────────────────────────
Every machine in SecureCorp:
  Local Admin username: Administrator
  Local Admin password: SecureCorp@123  ← SAME ON ALL 500 MACHINES

Attacker compromises Gani's laptop:
→ Gets local admin hash: 8f14e45fceea167...
→ Pass-the-Hash to EVERY other machine
→ Same hash works everywhere
→ 500 machines compromised from 1 breach
─────────────────────────────────────────────────────────

WITH LAPS — THE FIX:
─────────────────────────────────────────────────────────
Gani's laptop:     Local Admin password: xK9#mP2$vL8@nQ5!  ← unique
SQLDB01:           Local Admin password: rT4&wJ7*bN1^cF6%  ← unique
FILESVR01:         Local Admin password: yH3!qM8@zX2#pD9$  ← unique

Attacker compromises Gani's laptop:
→ Gets local admin hash for THAT machine only
→ Hash does NOT work on any other machine
→ Lateral movement via local admin: STOPPED
─────────────────────────────────────────────────────────
```

### Deploying LAPS in SecureCorp

```powershell
# STEP 1 — Install LAPS on DC01 (Windows LAPS built into Server 2025):
# For Windows Server 2025 — already built in, just enable it

# STEP 2 — Extend AD Schema for LAPS:
Update-LapsADSchema

# STEP 3 — Set permissions (who can READ the passwords):
# Only Hareesh (IT Admin) and Navi (SOC - read only) should see LAPS passwords
Set-LapsADComputerSelfPermission -Identity "OU=Workstations,DC=securecorp,DC=local"

# Grant Hareesh read access to LAPS passwords:
Set-LapsADReadPasswordPermission `
  -Identity "OU=Workstations,DC=securecorp,DC=local" `
  -AllowedPrincipals "Domain Admins"

# STEP 4 — Create LAPS GPO:
# Computer Configuration → Policies → Admin Templates → LAPS
# → Enable: "Configure password backup directory" = Active Directory
# → Enable: "Password settings" → Length: 20, Complexity: 4, Age: 30 days

# STEP 5 — Retrieve a LAPS password when needed:
Get-LapsADPassword -Identity "GANI-LAPTOP" -AsPlainText

# Output:
# ComputerName  : GANI-LAPTOP
# DistinguishedName : CN=GANI-LAPTOP,OU=Workstations...
# Account       : Administrator
# Password      : xK9#mP2$vL8@nQ5!
# PasswordUpdateTime : 01/04/2026 08:00:00
# ExpirationTimestamp : 01/05/2026 08:00:00
```

> 🔗 **Your connection:** At your organization, when a user called saying "I'm locked out of my local admin account" — someone had to manually reset it. With LAPS, Hareesh just runs one PowerShell command, reads the current password, and gives it securely. No shared passwords. Full audit trail of who read which password when.

---

## 📌 SECTION 3 of 6
## Layer 2 — Group Policy Security Hardening

---

### The SecureCorp GPO Structure

```
RECOMMENDED GPO LAYOUT FOR SECURECORP:
══════════════════════════════════════════════════════
GPO NAME                        LINKED TO
──────────────────────────────────────────────────────
SecureCorp-Password-Policy      Domain root
SecureCorp-Audit-Policy         Domain root
SecureCorp-DC-Hardening         Domain Controllers OU
SecureCorp-Server-Hardening     Servers OU
SecureCorp-Workstation-Baseline Workstations OU
SecureCorp-AppLocker            Workstations OU
SecureCorp-LAPS                 All computers
SecureCorp-Defender             All computers
══════════════════════════════════════════════════════

RULE: One GPO per purpose. Never mix settings.
      Easier to troubleshoot. Easier to audit.
```

### 3.1 — Audit Policy — The Eyes of SecureCorp

Without proper auditing, Navi is blind. This is the most important GPO after password policy.

```
GPO: SecureCorp-Audit-Policy
Location: Computer Configuration → Policies →
          Windows Settings → Security Settings →
          Advanced Audit Policy Configuration

WHAT TO ENABLE:
═══════════════════════════════════════════════════════════════
CATEGORY              SETTING              WHY
───────────────────────────────────────────────────────────────
Account Logon         Success + Failure    Detect brute force
                                           Detect PtH (NTLM)

Account Management    Success + Failure    Detect rogue accounts
                                           Detect group changes

DS Access             Success + Failure    Detect DCSync
                                           Detect AD object changes

Logon/Logoff          Success + Failure    Detect lateral movement
                                           Detect impossible travel

Object Access         Failure              Detect unauthorised
                                           file/folder access

Policy Change         Success + Failure    Detect GPO tampering

Privilege Use         Success + Failure    Detect admin abuse

Process Creation      Success              Detect malware execution
                                           Detect tool usage

System               Success + Failure    Detect security changes
═══════════════════════════════════════════════════════════════
```

```powershell
# Apply audit policy via PowerShell (alternative to GPO):
# Run on DC01:

# Account logon events:
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"Kerberos Authentication Service" /success:enable /failure:enable
auditpol /set /subcategory:"Kerberos Service Ticket Operations" /success:enable /failure:enable

# Account management:
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable

# DS Access (catches DCSync):
auditpol /set /subcategory:"Directory Service Access" /success:enable /failure:enable
auditpol /set /subcategory:"Directory Service Changes" /success:enable /failure:enable

# Process creation (catches Mimikatz, BloodHound):
auditpol /set /subcategory:"Process Creation" /success:enable

# Verify settings:
auditpol /get /category:*
```

> 🎯 **Navi's perspective:** Without process creation auditing enabled, when Mimikatz runs on Gani's laptop — there is NO log of it. With this enabled, Event ID 4688 fires showing exactly what process ran, from where, and under which user account. That's how Navi catches it.

### 3.2 — Critical Security GPO Settings

```
GPO: SecureCorp-Workstation-Baseline
═══════════════════════════════════════════════════════════════

SETTING 1 — Disable NTLM (Prevents Pass-the-Hash)
──────────────────────────────────────────────────
Path: Computer Config → Windows Settings → Security Settings
      → Local Policies → Security Options

"Network security: LAN Manager authentication level"
→ Set to: "Send NTLMv2 response only. Refuse LM & NTLM"

"Network security: Restrict NTLM: Outgoing NTLM traffic"
→ Set to: "Deny all"

⚠️  WARNING: Test this thoroughly before production!
    Some legacy apps NEED NTLM. They will break.
    Audit first: Security log → Event 4776 shows NTLM usage
    Fix those apps BEFORE disabling NTLM domain-wide.
───────────────────────────────────────────────────────────────

SETTING 2 — Restrict Remote Access to LSASS (Prevents Mimikatz)
──────────────────────────────────────────────────────────────
Path: Computer Config → Admin Templates
      → System → Local Security Authority

"Additional LSA protection" → Enabled

Registry equivalent:
HKLM\SYSTEM\CurrentControlSet\Control\Lsa
RunAsPPL = 1  (Protected Process Light)

Effect: LSASS runs as protected process
        Mimikatz cannot read it directly
        Attacker needs kernel-level exploit to bypass
───────────────────────────────────────────────────────────────

SETTING 3 — Disable PowerShell v2 (Prevents Bypass)
──────────────────────────────────────────────────────
Modern PowerShell has logging. PowerShell v2 does NOT.
Attackers specifically downgrade to v2 to avoid detection.

Path: Computer Config → Admin Templates
      → Windows Components → Windows PowerShell

"Turn on PowerShell Script Block Logging" → Enabled
"Turn on PowerShell Transcription" → Enabled
Log output path: \\FILESVR01\PSLogs\

# Also disable PS v2 via PowerShell:
Disable-WindowsOptionalFeature -Online -FeatureName MicrosoftWindowsPowerShellV2Root
───────────────────────────────────────────────────────────────

SETTING 4 — Block Unnecessary Ports (Network Hardening)
──────────────────────────────────────────────────────
Windows Firewall via GPO:

Inbound rules — BLOCK by default, allow only:
Port 445 (SMB)       → Only from management subnet
Port 3389 (RDP)      → Only from PAW machines
Port 135  (RPC)      → Only from domain controllers
Port 5985 (WinRM)    → Only from management subnet

Outbound rules — Allow by default BUT:
Block: Port 4444, 4445 → Common Metasploit/reverse shell ports
Block: Tor exit nodes → Blocked at perimeter
───────────────────────────────────────────────────────────────

SETTING 5 — Disable Unused Services
──────────────────────────────────────────────────────
Services that attackers abuse — disable if not needed:

Computer Config → Windows Settings → System Services:
→ Print Spooler     → Disabled (on DCs and servers)
                      Prevents PrintNightmare exploit
→ Remote Registry   → Disabled
                      Prevents remote registry access
→ NetBIOS           → Disabled
                      Old protocol, attackers abuse it
→ LLMNR             → Disabled via GPO
                      Prevents LLMNR poisoning attacks
→ mDNS              → Disabled
                      Prevents mDNS poisoning
```

> 🔗 **PrintNightmare connection:** In 2021, the Print Spooler service on Domain Controllers had a critical vulnerability. Every DC with Print Spooler running was compromised. Hareesh's hardening GPO disabling Print Spooler on DCs would have prevented this completely.

---

## 📌 SECTION 4 of 6
## Layer 3 — Privileged Access Hardening

---

### 4.1 — The Tiered Administration Model (Full Implementation)

This is the most important architectural change Hareesh can make in SecureCorp.

```
THE PROBLEM TODAY IN SECURECORP:
─────────────────────────────────────────────────────────
Hareesh uses ONE account for everything:
  hareesh@securecorp.local

He uses it to:
→ Check his email (Outlook)          ← Tier 2 activity
→ Browse internal SharePoint          ← Tier 2 activity
→ RDP into SQLDB01                    ← Tier 1 activity
→ Manage DC01                         ← Tier 0 activity
→ Reset user passwords                ← Tier 2 activity

RISK:
→ Hareesh reads a phishing email → malware on his laptop
→ His Tier 0 (Domain Admin) credentials are cached
→ Attacker has Domain Admin from a phishing email
─────────────────────────────────────────────────────────

THE TIERED MODEL — THREE SEPARATE IDENTITIES:
─────────────────────────────────────────────────────────
TIER 0 — Domain Controller / AD management ONLY
Account: hareesh-t0@securecorp.local
Used on: ONLY the PAW (Privileged Access Workstation)
Never: touches email, internet, regular work
Access: DC01, AD management, PKI, ADFS

TIER 1 — Server management ONLY
Account: hareesh-t1@securecorp.local
Used on: Dedicated jump server
Access: SQLDB01, FILESVR01, WEB01, PRINT01
Never: touches workstations or DC directly

TIER 2 — Daily work (email, SharePoint, helpdesk)
Account: hareesh@securecorp.local (regular account)
Used on: His daily laptop
Access: Regular user resources only
Never: touches servers or DCs
─────────────────────────────────────────────────────────
```

### Setting Up Tiers in SecureCorp

```powershell
# Create Tier 0 admin account for Hareesh:
New-ADUser `
  -Name "Hareesh-T0" `
  -SamAccountName "hareesh-t0" `
  -UserPrincipalName "hareesh-t0@securecorp.local" `
  -AccountPassword (Read-Host -AsSecureString "Enter T0 password") `
  -Enabled $true `
  -Description "Tier 0 - DC/AD Admin account - Hareesh"

# Add ONLY to Domain Admins:
Add-ADGroupMember -Identity "Domain Admins" -Members "hareesh-t0"

# Create Tier 1 admin account:
New-ADUser `
  -Name "Hareesh-T1" `
  -SamAccountName "hareesh-t1" `
  -UserPrincipalName "hareesh-t1@securecorp.local" `
  -AccountPassword (Read-Host -AsSecureString "Enter T1 password") `
  -Enabled $true `
  -Description "Tier 1 - Server Admin account - Hareesh"

# Add to Server Operators (not Domain Admins!):
Add-ADGroupMember -Identity "SecureCorp-Server-Admins" -Members "hareesh-t1"

# IMPORTANT: Remove Hareesh's regular account from Domain Admins:
Remove-ADGroupMember -Identity "Domain Admins" -Members "hareesh" -Confirm:$false
```

### 4.2 — Protected Users Security Group

This is the **single highest-impact, lowest-effort** hardening step for privileged accounts.

```
WHAT PROTECTED USERS GROUP DOES AUTOMATICALLY:
════════════════════════════════════════════════════════
✅ Kerberos ONLY — NTLM authentication completely blocked
✅ No credential caching — hash never stored on remote machines
✅ No Kerberos delegation — cannot impersonate to other services
✅ Kerberos ticket lifetime: 4 hours max (not 10 hours)
✅ No DES or RC4 encryption — AES only
✅ Cannot use CredSSP — prevents credential forwarding
════════════════════════════════════════════════════════

WHAT THIS KILLS:
→ Pass-the-Hash          ← NTLM blocked = PtH impossible
→ Pass-the-Ticket        ← No caching = nothing to steal
→ Kerberoasting          ← RC4 blocked = uncrackable tickets
→ Overpass-the-Hash      ← All hash-based attacks broken
→ Credential theft        ← Nothing cached anywhere
════════════════════════════════════════════════════════
```

```powershell
# Add all privileged accounts to Protected Users:
$privilegedAccounts = @(
  "hareesh-t0",    # Domain Admin
  "hareesh-t1",    # Server Admin
  "navi"           # SOC Analyst (read-only but sensitive)
)

foreach ($account in $privilegedAccounts) {
  Add-ADGroupMember `
    -Identity "Protected Users" `
    -Members $account
  Write-Host "Added $account to Protected Users" -ForegroundColor Green
}

# Verify:
Get-ADGroupMember -Identity "Protected Users" |
  Select Name, SamAccountName

# ⚠️  DO NOT ADD SERVICE ACCOUNTS TO PROTECTED USERS
#     gMSA handles service accounts separately
#     Protected Users breaks service functionality
```

> ⚠️ **Critical Warning from Hareesh the Architect:** Before adding anyone to Protected Users — test in a lab first. If the account uses any legacy authentication (old apps, VPNs using NTLM) — those will break immediately. Always check Event ID 4776 first to see if the account uses NTLM anywhere.

### 4.3 — Privileged Access Workstation (PAW)

```
WHAT IS A PAW?
─────────────────────────────────────────────────────────
A dedicated, hardened machine used ONLY for admin work.

Hareesh has:
→ Regular laptop   → email, Teams, SharePoint, browsing
→ PAW machine      → ONLY for DC management, AD tasks

PAW HARDENING CHECKLIST:
─────────────────────────────────────────────────────────
✅ No email client installed
✅ No web browser (or heavily restricted)
✅ No Office applications
✅ Bitlocker encrypted
✅ Only connects to management network segment
✅ All internet traffic blocked at firewall level
✅ Only hareesh-t0 account can log in
✅ Enrolled in Intune with strictest policies
✅ Microsoft Defender for Endpoint with highest protection
✅ All USB ports disabled via GPO
✅ Network access: DC01 and management subnet ONLY

WHAT AN ATTACKER GETS IF PAW IS COMPROMISED:
→ Access to DC management tools
→ But: no email to pivot from, no internet, no lateral paths
→ Attack is completely contained to that one machine

WHAT AN ATTACKER GETS IF REGULAR LAPTOP IS COMPROMISED:
→ hareesh@securecorp.local (Tier 2 only now — no Domain Admin)
→ Cannot reach DCs
→ Cannot reach servers
→ Attack is completely contained
─────────────────────────────────────────────────────────
```

---

## 📌 SECTION 5 of 6
## Layer 4 — Microsoft Defender for Identity (MDI)

---

This is Navi's most powerful weapon. MDI sits on the Domain Controller and watches **every single authentication event** in real time.

```
HOW MDI WORKS IN SECURECORP:
─────────────────────────────────────────────────────────
MDI Sensor installed on DC01
         ↓
Monitors ALL AD traffic in real time:
  → Every Kerberos request
  → Every NTLM authentication
  → Every LDAP query
  → Every replication request
  → Every DNS lookup
         ↓
Sends to Microsoft Defender XDR cloud portal
         ↓
ML models compare against baseline behaviour
         ↓
Alerts fire on anomalies → Navi's dashboard
─────────────────────────────────────────────────────────
```

### What MDI Detects Automatically

```
┌────────────────────────────────┬──────────────────────────────┐
│ Attack                         │ MDI Alert                    │
├────────────────────────────────┼──────────────────────────────┤
│ Pass-the-Hash                  │ "Identity theft using PtH"   │
│ Pass-the-Ticket                │ "Identity theft using PtT"   │
│ Kerberoasting                  │ "Suspicious Kerberos request"│
│ DCSync                         │ "Suspicious replication"     │
│ Golden Ticket                  │ "Forged PAC detection"       │
│ Password Spray                 │ "Suspicious auth failures"   │
│ Brute Force                    │ "Multiple auth failures"     │
│ BloodHound Enumeration         │ "Suspicious LDAP queries"    │
│ Mimikatz LSASS access          │ "Credential access alert"    │
│ Lateral Movement               │ "Lateral movement path"      │
└────────────────────────────────┴──────────────────────────────┘
```

### MDI Deployment in SecureCorp

```powershell
# Step 1 — Download MDI sensor from Defender portal
# portal.azure.com → Microsoft Defender → Settings → Identities

# Step 2 — Install on DC01:
# Run Azure ATP Sensor Setup.exe on DC01
# Enter workspace key from Defender portal
# Installation takes ~10 minutes

# Step 3 — Verify sensor is reporting:
# Defender portal → Settings → Identities → Sensors
# DC01 should show: Running ✅

# Step 4 — Configure notification emails:
# Defender portal → Settings → Notifications
# Add Navi's email: navi@securecorp.local
# Alert on: High severity immediately
#           Medium severity: daily digest

# Step 5 — Test it works (safe test):
# From any domain machine, run:
nslookup -type=SRV _ldap._tcp.securecorp.local
# MDI sees this LDAP query and baselines it as normal
```

---

## 📌 SECTION 6 of 6
## The Hardened SecureCorp — Before vs After

---

```
ATTACK CHAIN REVISITED — CAN IT STILL HAPPEN?
═══════════════════════════════════════════════════════════════

STEP 1: Phishing → Gani's laptop
─────────────────────────────────
BEFORE: Malware runs freely, no detection
AFTER:
  → PowerShell Script Block Logging enabled
    → Malicious script logged → Event 4104 → Navi alerted
  → Process Creation auditing enabled
    → Suspicious process logged → Event 4688 → MDI alerted
  → Defender for Endpoint blocks known malware
VERDICT: Detected immediately at Step 1 ✅

STEP 2: Kerberoasting SQLSvc
─────────────────────────────
BEFORE: SQLSvc cracked in 10 minutes
AFTER:
  → SQLSvc replaced with gMSA-SQLSvc (240-char password)
  → Cannot be cracked — mathematically impossible
  → MDI detects Kerberoasting attempt → Alert fires
VERDICT: Attack fails completely ✅

STEP 3: Pass-the-Hash with Hareesh's credentials
──────────────────────────────────────────────────
BEFORE: Hareesh's Domain Admin hash cached on SQLDB01
AFTER:
  → Hareesh uses hareesh-t0 ONLY on PAW
  → hareesh-t0 in Protected Users → no caching anywhere
  → Hareesh's regular account (Tier 2) has no server access
  → No admin hash available to steal
VERDICT: No hash to steal — attack impossible ✅

STEP 4: DCSync
───────────────
BEFORE: Attacker had Domain Admin → DCSync freely
AFTER:
  → Attacker cannot get Domain Admin (Steps 1-3 blocked)
  → Even if somehow reached — MDI detects DCSync instantly
  → Navi gets alert within 30 seconds
  → Account blocked before damage done
VERDICT: Blocked by previous layers + detected ✅

STEP 5: Golden Ticket
──────────────────────
BEFORE: KRBTGT stolen → permanent access
AFTER:
  → DCSync blocked → KRBTGT never stolen
  → Even if compromised → MDI detects PAC anomalies
  → KRBTGT rotation procedure documented and ready
VERDICT: Prevented by defence-in-depth ✅

STEP 6: Ransomware via GPO
────────────────────────────
BEFORE: Domain Admin deploys ransomware GPO at 3 AM
AFTER:
  → AppLocker blocks unauthorised executables
  → GPO changes logged → Sentinel alert fires
  → Hareesh-t0 required to make GPO changes (PAW only)
  → No Domain Admin available to attacker
VERDICT: Impossible without bypassing all previous layers ✅

═══════════════════════════════════════════════════════════════
RESULT: The attacker's 10-day campaign is now stopped at DAY 1
═══════════════════════════════════════════════════════════════
```

---

## 🧪 DAY 3 LABS

```
LAB 1 — Configure Password Policy via GPO (20 min)
Platform: Your Azure AD Lab / Local AD
──────────────────────────────────────────────────────
1. Open Group Policy Management on DC01
2. Create new GPO: "SecureCorp-Password-Policy"
3. Set: Min length 15, History 24, Lockout 5/15min
4. Link to domain root
5. Run: gpupdate /force on a client
6. Verify: net accounts (shows current policy)

LAB 2 — Enable Audit Policy (20 min)
Platform: DC01 PowerShell
──────────────────────────────────────────────────────
# Run these and verify with auditpol /get /category:*
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"Kerberos Service Ticket Operations" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /set /subcategory:"Directory Service Access" /success:enable /failure:enable

# Then check Security event log for new events:
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4688} -MaxEvents 5 |
  Select TimeCreated, Message

LAB 3 — Add Account to Protected Users (15 min)
Platform: DC01 PowerShell
──────────────────────────────────────────────────────
# Create a test admin account:
New-ADUser -Name "TestAdmin" -SamAccountName "testadmin" `
  -AccountPassword (ConvertTo-SecureString "TestAdmin@2024!" -AsPlainText -Force) `
  -Enabled $true

# Add to Protected Users:
Add-ADGroupMember -Identity "Protected Users" -Members "testadmin"

# Login as testadmin on a machine
# Check: klist — should show AES tickets only, no RC4
# Try: net use \\DC01\sysvol — should work (Kerberos)

LAB 4 — TryHackMe (45 min)
Room: "Attacking Active Directory — Post-Compromise"
Or:   "Breaching Active Directory"
Focus: See what hardening stops which attacks
──────────────────────────────────────────────────────

LAB 5 — WEEKEND COMBINED LAB
Do Day 2 + Day 3 labs together:
1. Run Kerberoasting attack on test lab
2. See it succeed with regular service account
3. Convert to gMSA
4. Run Kerberoasting again — watch it fail
5. That contrast is what makes both days stick
```

---

## 📋 DAY 3 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║         DAY 3 — AD HARDENING & SECURITY CONTROLS        ║
║                    SecureCorp Ltd.                       ║
╚══════════════════════════════════════════════════════════╝

HARDENING FRAMEWORK — 6 LAYERS:
─────────────────────────────────────────────────────────
Layer 1: Foundation        → Passwords, LAPS, Patching
Layer 2: Group Policy      → Audit, NTLM, Services, Logging
Layer 3: Service Accounts  → gMSA, SPN cleanup
Layer 4: Privileged Access → Tiered model, Protected Users, PAW
Layer 5: Detection         → MDI, Sentinel, KQL alerts
Layer 6: Response          → Incident playbooks (Day 10)

FOUNDATION HARDENING:
─────────────────────────────────────────────────────────
Password Policy (GPO):
  Min length: 15 | History: 24 | Max age: 180 days
  Lockout: 5 attempts | Duration: 15 min | Window: 15 min

Fine-Grained Password Policy (FGPP):
  Domain Admins: 20 chars, 3 attempt lockout, 30 min
  Standard Users: 15 chars, 5 attempt lockout, 15 min
  Service Accounts: gMSA (240 chars, auto-rotated)

LAPS:
  Unique local admin password per machine
  Stored in AD, readable only by authorized admins
  Auto-rotates every 30 days
  Kills lateral movement via shared local admin hash

GROUP POLICY HARDENING:
─────────────────────────────────────────────────────────
Audit Policy — Enable ALL of these:
  Credential Validation:       Success + Failure
  Kerberos Ticket Operations:  Success + Failure
  Account Management:          Success + Failure
  Directory Service Access:    Success + Failure (DCSync)
  Logon/Logoff:                Success + Failure
  Process Creation:            Success (catches Mimikatz)
  Policy Change:               Success + Failure

Security Settings:
  NTLM → Disable (NTLMv2 only → then full disable)
  LSASS → RunAsPPL = 1 (blocks Mimikatz)
  PowerShell v2 → Disabled
  Script Block Logging → Enabled
  PS Transcription → Enabled → \\FILESVR01\PSLogs\
  Print Spooler → Disabled on DCs and servers
  Remote Registry → Disabled
  LLMNR → Disabled (GPO)
  NetBIOS → Disabled

PRIVILEGED ACCESS HARDENING:
─────────────────────────────────────────────────────────
Tiered Admin Model:
  Tier 0: hareesh-t0 → DC/AD management only → PAW only
  Tier 1: hareesh-t1 → Server management only → Jump server
  Tier 2: hareesh   → Daily work → Regular laptop

Protected Users Group:
  Add: All Tier 0 accounts (Domain Admins)
  Add: All Tier 1 accounts (Server Admins)
  Effect: NTLM blocked, no caching, AES only, 4hr tickets
  DO NOT ADD: Service accounts, legacy app accounts

PAW (Privileged Access Workstation):
  Dedicated machine for Tier 0 work only
  No email, no browser, no USB, no internet
  Only connects to management network
  Hareesh-t0 is the ONLY account that can log in

MICROSOFT DEFENDER FOR IDENTITY:
─────────────────────────────────────────────────────────
Install sensor on DC01
Monitors: All Kerberos, NTLM, LDAP, DNS, replication

Auto-detects:
  Pass-the-Hash          Kerberoasting
  Pass-the-Ticket        DCSync
  Golden Ticket          Password Spray
  BloodHound enum        Mimikatz LSASS access
  Lateral Movement       Forged PAC

Navi gets real-time alerts in Microsoft Defender XDR portal

HARDENING vs ATTACK CHAIN:
─────────────────────────────────────────────────────────
Phishing → Gani:      PS Logging + Defender = Detected Day 1
Kerberoasting:        gMSA = Impossible to crack
Pass-the-Hash:        Protected Users = No hash cached
DCSync:               MDI = Detected in 30 seconds
Golden Ticket:        DCSync blocked = KRBTGT never stolen
Ransomware GPO:       AppLocker + No Domain Admin = Impossible

KEY GPO LOCATIONS:
─────────────────────────────────────────────────────────
Password Policy:
Computer Config → Windows Settings → Security Settings
→ Account Policies → Password Policy / Account Lockout

Audit Policy:
Computer Config → Windows Settings → Security Settings
→ Advanced Audit Policy Configuration → Audit Policies

NTLM Restriction:
Computer Config → Windows Settings → Security Settings
→ Local Policies → Security Options
→ "Network security: LAN Manager authentication level"

LSASS Protection:
Computer Config → Admin Templates
→ System → Local Security Authority
→ "Additional LSA protection" → Enabled

PowerShell Logging:
Computer Config → Admin Templates
→ Windows Components → Windows PowerShell
→ "Turn on Script Block Logging" → Enabled

LABS:
─────────────────────────────────────────────────────────
□ Configure password policy GPO
□ Enable audit policy via PowerShell
□ Add test account to Protected Users → verify AES tickets
□ TryHackMe: Attacking Active Directory Post-Compromise
□ Weekend: Day 2 + Day 3 combined attack/defend lab
```

---

> 💡 **Architect's Final Word for Day 3:** *"Hardening is not a one-time project. It is a continuous process. Every new vulnerability, every new attack technique, every new Microsoft update — it all changes the hardening baseline. The best security engineers I know have a calendar reminder every quarter that says: review hardening. They treat it like patching. Non-negotiable, recurring, documented. Build that habit now and you will always be ahead of attackers."*

---

**Ready for Day 4?** 💪🔐
