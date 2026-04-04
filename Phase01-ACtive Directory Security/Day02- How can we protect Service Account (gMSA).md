
---

# 🔑 THE BIG REVEAL — How gMSA Actually Works

Let me start with the most important concept first.

## 💡 The Fundamental Difference in Thinking

```
REGULAR SERVICE ACCOUNT:
─────────────────────────────────────────────────
Application says:
"To run, I need a USERNAME and PASSWORD"
→ Password stored somewhere humans can read
→ Password must be manually updated everywhere
→ Password can be stolen from config files

gMSA:
─────────────────────────────────────────────────
Application says:
"To run, I need a USERNAME"
→ NO PASSWORD anywhere
→ Windows OS itself fetches the password from AD
→ The password is NEVER visible to any human
→ NEVER stored in any script or config file
→ Nothing to steal. Nothing to update.
```

> 🧠 **The Key Insight:** With gMSA, the **Windows OS becomes the password manager.** Not you. Not a script. Not a config file. Windows handles it completely invisibly.

---

## 🏢 SECURECORP STORY — Let's See This Live

Let me show you **exactly** what happens with SQLSvc right now vs gMSA.

---

### 📌 CURRENT SITUATION — Regular Service Account (The Problem)

```
SECURECORP TODAY:
─────────────────────────────────────────────────────────────

Hareesh created SQLSvc account manually:
  Username: sqlsvc@securecorp.local
  Password: SqlServer@12345678901  ← 20 chars, looks strong

This password is currently stored in:

LOCATION 1 — Windows Service on SQLDB01:
  Services.msc → SQL Server → Log On As:
  Username: securecorp\sqlsvc
  Password: SqlServer@12345678901  ← stored here

LOCATION 2 — Scheduled Task on SQLDB01:
  Task Scheduler → DB Backup Job → Run As:
  Username: securecorp\sqlsvc
  Password: SqlServer@12345678901  ← stored here

LOCATION 3 — PowerShell script:
  $username = "securecorp\sqlsvc"
  $password = "SqlServer@12345678901"  ← hardcoded 🚨

LOCATION 4 — Application config file:
  <connectionString>
    uid=sqlsvc;pwd=SqlServer@12345678901  ← plaintext 🚨
  </connectionString>

LOCATION 5 — Another scheduled task on FILESVR01:
  DB Report Generation → Run As:
  Username: securecorp\sqlsvc
  Password: SqlServer@12345678901  ← stored here too

─────────────────────────────────────────────────────────────
TOTAL: Password exists in 5 different places
       Any one of them can be compromised
       Changing password = update all 5 manually
       Nobody does this → Password is 3 years old
       → PERFECT Kerberoasting target
```

---

### 📌 THE gMSA SOLUTION — How It Actually Works

Here's the magic. Follow this carefully 👇

```
THE gMSA CORE MECHANISM:
─────────────────────────────────────────────────────────────

1. gMSA password is stored ONLY in Active Directory (DC01)
2. The password is 240 characters, randomly generated
3. Windows auto-rotates it every 30 days automatically
4. ONLY authorized computers can ASK DC01 for the password
5. The computer asks, gets password, uses it — all invisibly
6. No human ever sees the password
7. No script ever contains the password
8. No config file ever stores the password

THE INVISIBLE HANDSHAKE:
─────────────────────────────────────────────────────────────

SQLDB01 needs to run SQL Server service:

Step 1: SQLDB01 wakes up / service starts
Step 2: Windows on SQLDB01 says to DC01:
        "I am SQLDB01. I am authorized to use gMSA-SQL.
         Please give me the current password."
Step 3: DC01 checks: "Is SQLDB01 in the authorized list? YES"
        DC01 sends back the current 240-char password
        (encrypted, over secure channel)
Step 4: Windows on SQLDB01 uses that password to start service
Step 5: Password is NEVER written to disk
        NEVER shown to any user
        NEVER stored in any file

When password rotates after 30 days:
Step 1: DC01 generates new 240-char random password
Step 2: Next time SQLDB01 asks → gets new password automatically
Step 3: Service restarts with new password
Step 4: ZERO human intervention required
        ZERO config file updates
        ZERO script updates
─────────────────────────────────────────────────────────────
```

> 🎯 **This is why nothing breaks when password rotates.** There's nothing to update — because the password was never stored anywhere in the first place!

---

## ⚙️ WHERE gMSA WORKS AND WHERE IT DOESN'T

This is the critical part of your question. Not everything can use gMSA.

```
✅ gMSA WORKS PERFECTLY HERE:
──────────────────────────────────────────────────────────
  Windows Services
  → SQL Server, IIS, Exchange, custom Windows services
  → Service just runs as gMSA — Windows handles password

  Scheduled Tasks (Task Scheduler)
  → Works natively in Windows 2012 R2 and later
  → Select gMSA as the run-as account
  → No password field needed — Windows fills it in

  IIS Application Pools
  → Application pool identity = gMSA account
  → IIS asks DC01 for password automatically

  SQL Server Agent Jobs
  → Job runs as gMSA — fully supported

──────────────────────────────────────────────────────────
❌ gMSA DOES NOT WORK HERE:
──────────────────────────────────────────────────────────
  Interactive Login
  → Cannot log into a machine as gMSA
  → No console login, no RDP as gMSA
  → gMSA is for services only, not humans

  Scripts with hardcoded credentials
  → If your script does:
    $cred = New-Object PSCredential("sqlsvc", $password)
  → This pattern CANNOT use gMSA directly
  → Script needs to be rewritten (we'll cover the fix)

  Third-party applications that need password in config
  → Some old apps need password in web.config
  → These CANNOT use gMSA
  → Alternative: Use Windows Credential Manager or
    Azure Key Vault for these cases

  Non-Windows systems
  → Linux scripts, Python apps, Java apps
  → Cannot talk to AD to get gMSA password
  → Alternative: Managed identities / Vault solutions
──────────────────────────────────────────────────────────
```

---

## 🔧 HANDS-ON — Creating and Using gMSA in SecureCorp

Let's replace SQLSvc with a proper gMSA. Step by step.

### STEP 1 — One-Time Setup: KDS Root Key

```powershell
# Run this ONCE per domain — on DC01
# This creates the key that generates gMSA passwords
# Hareesh runs this on DC01:

# For production (wait 10 hours before gMSA works):
Add-KdsRootKey -EffectiveImmediately

# Check it was created:
Get-KdsRootKey

# Output:
# KeyId        : 12345678-abcd-efgh-ijkl-123456789012
# CreationTime : 01/04/2026 08:00:00
# EffectiveTime: 01/04/2026 18:00:00  ← wait till this time

# NOTE: In a lab environment only — immediate use:
Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))
```

---

### STEP 2 — Create the gMSA Account

```powershell
# Hareesh runs this on DC01
# Creating gMSA for SQL Server on SQLDB01:

New-ADServiceAccount `
  -Name "gMSA-SQLSvc" `
  -DNSHostName "sqldb01.securecorp.local" `
  -PrincipalsAllowedToRetrieveManagedPassword "SQLDB01$" `
  -Description "gMSA for SQL Server service on SQLDB01" `
  -Enabled $true

# EXPLANATION OF EACH PARAMETER:
# -Name                  → Account name in AD (gMSA-SQLSvc)
# -DNSHostName           → Which server this gMSA is for
# -PrincipalsAllowed...  → SQLDB01$ (the computer account)
#                          ONLY this computer can get the password
#                          The $ means it's a computer account
# -Description           → Good practice for documentation

# Verify it was created:
Get-ADServiceAccount -Identity "gMSA-SQLSvc" -Properties *
```

---

### STEP 3 — Allow Multiple Servers (If Needed)

```powershell
# What if SQLSvc runs on SQLDB01 AND a second server SQLDB02?
# Create a security group first:

New-ADGroup `
  -Name "gMSA-SQLSvc-Servers" `
  -GroupScope Global `
  -Description "Servers authorized to use gMSA-SQLSvc"

# Add both servers to the group:
Add-ADGroupMember -Identity "gMSA-SQLSvc-Servers" `
  -Members "SQLDB01$", "SQLDB02$"

# Create gMSA with group instead of single server:
New-ADServiceAccount `
  -Name "gMSA-SQLSvc" `
  -DNSHostName "sqldb01.securecorp.local" `
  -PrincipalsAllowedToRetrieveManagedPassword "gMSA-SQLSvc-Servers" `
  -Description "gMSA for SQL Server on DB servers"

# NOW: Both SQLDB01 and SQLDB02 can retrieve the password
# Add future servers to the group — no gMSA changes needed
```

---

### STEP 4 — Install gMSA on the Target Server

```powershell
# Hareesh RDPs to SQLDB01 and runs:

# Install the gMSA on this server:
Install-ADServiceAccount -Identity "gMSA-SQLSvc"

# Test it works (MUST pass before using):
Test-ADServiceAccount -Identity "gMSA-SQLSvc"

# Output should be: True
# If False → SQLDB01 is not in the authorized list
#           → Check PrincipalsAllowedToRetrieveManagedPassword
```

---

### STEP 5 — Configure Windows Service to Use gMSA

```powershell
# Option A — Via PowerShell:
$svcName = "MSSQLSERVER"  # SQL Server service name

Set-Service -Name $svcName `
  -Credential (
    New-Object System.Management.Automation.PSCredential(
      "securecorp\gMSA-SQLSvc$",  # ← Note the $ at the end
      (New-Object System.Security.SecureString)  # ← Empty password!
    )
  )

Restart-Service -Name $svcName

# Option B — Via Services.msc GUI:
# Open Services → SQL Server → Properties → Log On
# Select: This account
# Enter: securecorp\gMSA-SQLSvc$   ← with $ at end
# Password: [leave completely blank]  ← NO PASSWORD NEEDED
# Confirm: [leave completely blank]
# Click OK → Restart service
```

---

### STEP 6 — Configure Scheduled Task to Use gMSA

```powershell
# For existing scheduled task:
$action = New-ScheduledTaskAction `
  -Execute "PowerShell.exe" `
  -Argument "-File C:\Scripts\DBBackup.ps1"

$trigger = New-ScheduledTaskTrigger `
  -Daily -At "2:00AM"

# KEY PART — gMSA as principal, NO password:
$principal = New-ScheduledTaskPrincipal `
  -UserId "securecorp\gMSA-SQLSvc$" `
  -LogonType Password  # Windows fetches password from AD

Register-ScheduledTask `
  -TaskName "DB-Backup-Job" `
  -Action $action `
  -Trigger $trigger `
  -Principal $principal

# NO PASSWORD SPECIFIED ANYWHERE IN THIS SCRIPT
# Windows handles it completely
```

---

## 🚨 NOW — THE HARDEST PART: Scripts with Hardcoded Passwords

You asked the most important question:

> *"What about scripts that have passwords hardcoded in them?"*

```
THE PROBLEM:
─────────────────────────────────────────────────────────
# Current script on SQLDB01 (BAD - Password hardcoded):
$username = "securecorp\sqlsvc"
$password = ConvertTo-SecureString "SqlServer@12345678901" `
            -AsPlainText -Force
$credential = New-Object PSCredential($username, $password)

# Connect to another server using these credentials:
Invoke-Command -ComputerName FILESVR01 `
               -Credential $credential `
               -ScriptBlock { Get-ChildItem C:\Reports }

# THIS CANNOT USE gMSA DIRECTLY
# Because gMSA doesn't support interactive credential objects
─────────────────────────────────────────────────────────
```

### The Fix — 3 Solutions Depending on Your Situation

---

**SOLUTION 1 — Rewrite Script to Run AS the gMSA (Best)**

```powershell
# Instead of passing credentials IN the script,
# make the SCHEDULED TASK run AS the gMSA
# Then the script runs with gMSA identity automatically

# The script itself becomes simple — NO credentials:
# DBBackup.ps1:
Invoke-Command -ComputerName FILESVR01 `
               -ScriptBlock { Get-ChildItem C:\Reports }
# No credential parameter needed!
# Why? Because the script IS ALREADY running as gMSA-SQLSvc$
# Windows uses gMSA identity for the network connection

# The task scheduler entry runs as gMSA-SQLSvc$ (Step 6 above)
# Script inherits that identity automatically

# THIS IS THE CORRECT ARCHITECTURE:
# Task runs as gMSA → Script runs as gMSA → 
# Network connections use gMSA identity → 
# No credentials anywhere in the script
```

---

**SOLUTION 2 — Azure Key Vault (For Apps That Need Passwords)**

```powershell
# For third-party apps or scripts that MUST have a password:
# Store the password in Azure Key Vault
# Script fetches it at runtime — never hardcoded

# Script fetches secret from Key Vault at runtime:
$secret = Get-AzKeyVaultSecret `
  -VaultName "SecureCorp-KeyVault" `
  -Name "SQLSvc-Password" `
  -AsPlainText

# NOW:
# → Password not in script
# → Password not in config file
# → Rotated in Key Vault → script always gets latest
# → Full audit log of who accessed the secret
# → Hareesh controls who/what can read the secret
```

---

**SOLUTION 3 — Windows Credential Manager (For Legacy Scripts)**

```powershell
# For scripts that cannot be easily rewritten:
# Store credential in Windows Credential Manager on the server
# Script reads from there — not hardcoded

# Store once (Hareesh does this on SQLDB01):
cmdkey /add:FILESVR01 `
       /user:securecorp\sqlsvc `
       /pass:SqlServer@12345678901

# Script reads it automatically — no password in script:
$credential = Get-StoredCredential -Target "FILESVR01"

# Limitation: Still need to update Credential Manager
#             when password changes
# Better than hardcoding but not as good as gMSA
```

---

## 📊 THE 200 SERVICE ACCOUNTS PROBLEM — Migration Strategy

Now your biggest question — how to manage 200 accounts properly.

```
SECURECORP HAS 200 SERVICE ACCOUNTS.
CANNOT migrate all overnight. Here's the real strategy:

PHASE 1 — AUDIT (Week 1):
────────────────────────────────────────────────────────
Find all service accounts and categorize them:

# PowerShell to find all service accounts:
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} `
  -Properties ServicePrincipalName, PasswordLastSet, `
              PasswordNeverExpires, LastLogonDate |
Select Name, PasswordLastSet, PasswordNeverExpires, `
       LastLogonDate, ServicePrincipalName |
Export-Csv "C:\Audit\ServiceAccounts.csv" -NoTypeInformation

Result will show you:
→ Accounts with passwords never changed (3+ years old)
→ Accounts never used (can be disabled/deleted)
→ Accounts with PasswordNeverExpires = True (all of them)
────────────────────────────────────────────────────────

PHASE 2 — CATEGORIZE (Week 1):
────────────────────────────────────────────────────────
CATEGORY A — Windows Services only (gMSA ready):
  → sqlsvc, backupsvc, monitorsvc etc.
  → Estimate: ~60% of your 200 accounts (120 accounts)
  → These can be migrated to gMSA directly

CATEGORY B — Scheduled Tasks only (gMSA ready):
  → Reportsvc, cleanupsvc, syncssvc etc.
  → Estimate: ~20% (40 accounts)
  → These can be migrated to gMSA

CATEGORY C — Scripts with hardcoded passwords:
  → Estimate: ~15% (30 accounts)
  → Need script rewrite OR Key Vault solution

CATEGORY D — Third-party apps needing passwords:
  → Estimate: ~5% (10 accounts)
  → Vendor must support gMSA OR use Key Vault
────────────────────────────────────────────────────────

PHASE 3 — PRIORITIZE (Week 2):
────────────────────────────────────────────────────────
Not all 200 accounts are equal risk.
Migrate highest risk FIRST:

PRIORITY 1 — HIGH PRIVILEGE ACCOUNTS:
  → Any service account in Domain Admins (should NOT exist)
  → Any account with DCSync rights
  → Any account with local admin on multiple servers
  → Migrate these FIRST — highest Kerberoasting risk

PRIORITY 2 — ACCOUNTS WITH OLD PASSWORDS:
  → PasswordLastSet > 1 year ago
  → These are the easiest to crack
  → Migrate or force rotate immediately

PRIORITY 3 — ACCOUNTS WITH SPNs REGISTERED:
  → These are directly Kerberoastable
  → Any account with SPN = potential attack target
  → All of these should become gMSA

PRIORITY 4 — REMAINING ACCOUNTS:
  → Migrate progressively over 3 months
────────────────────────────────────────────────────────

PHASE 4 — MIGRATE IN BATCHES (Weeks 3-12):
────────────────────────────────────────────────────────
Week 3-4:   Migrate top 20 high-priority accounts
Week 5-6:   Migrate next 40 Windows Service accounts
Week 7-8:   Migrate 40 Scheduled Task accounts
Week 9-10:  Rewrite scripts for Category C accounts
Week 11-12: Handle Category D (third-party apps)

RULE: Never migrate more than 10 accounts per week
      Test each one before moving to next
      Keep rollback plan ready
────────────────────────────────────────────────────────
```

---

## 📋 COMPLETE COMPARISON TABLE

```
┌─────────────────────┬──────────────────────┬─────────────────────┐
│ Feature             │ Regular Svc Account  │ gMSA                │
├─────────────────────┼──────────────────────┼─────────────────────┤
│ Password management │ Manual — by admin    │ Automatic — by AD   │
│ Password length     │ 20 chars (your case) │ 240 chars           │
│ Password rotation   │ Manual — never done  │ Every 30 days auto  │
│ Password in scripts │ Yes — hardcoded 🚨   │ No — never          │
│ Password in configs │ Yes — plaintext 🚨   │ No — never          │
│ Kerberoastable      │ YES 🚨               │ NO ✅               │
│ Pass-the-Hash risk  │ HIGH                 │ LOW                 │
│ Works in services   │ Yes                  │ Yes                 │
│ Works in tasks      │ Yes                  │ Yes                 │
│ Works in scripts    │ Yes (hardcoded)      │ Only if rewritten   │
│ Interactive login   │ Yes                  │ NO                  │
│ Multi-server        │ Yes                  │ Yes (via group)     │
│ Audit trail         │ Limited              │ Full AD audit       │
│ Admin overhead      │ HIGH                 │ Near ZERO           │
└─────────────────────┴──────────────────────┴─────────────────────┘
```

---

## 🎯 THE ARCHITECT'S SUMMARY

```
HAREESH'S DECISION FRAMEWORK FOR SECURECORP:

Question 1: Does the service run on Windows?
  YES → Can use gMSA
  NO  → Use Azure Managed Identity or Key Vault

Question 2: Is it a Windows Service or Scheduled Task?
  YES → Migrate to gMSA directly (easiest)
  NO  → Evaluate script rewrite

Question 3: Is there a hardcoded password in a script?
  YES → Rewrite script to run AS gMSA via task scheduler
      → OR move secret to Azure Key Vault

Question 4: Is it a third-party app needing a password?
  YES → Check if vendor supports gMSA
      → If not → Azure Key Vault
      → If not possible → Windows Credential Manager

Question 5: Is it a Linux/non-Windows service?
  YES → gMSA won't work
      → Use Azure Managed Identity
      → OR HashiCorp Vault
      → OR Azure Key Vault with managed identity
```

---

## 🧪 LAB FOR THIS TOPIC

```
LAB — Create and Test gMSA in Your Azure AD Lab
Platform: Azure VM with Windows Server (your existing setup)
Time: 30-40 minutes

STEP 1: On DC01 — Setup KDS key
  Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))

STEP 2: Create gMSA
  New-ADServiceAccount -Name "gMSA-Test" `
    -DNSHostName "server01.securecorp.local" `
    -PrincipalsAllowedToRetrieveManagedPassword "SERVER01$"

STEP 3: On SERVER01 — Install and test
  Install-ADServiceAccount -Identity "gMSA-Test"
  Test-ADServiceAccount -Identity "gMSA-Test"
  # Must return: True

STEP 4: Create a scheduled task using gMSA
  Use Task Scheduler GUI
  Set "Run As": securecorp\gMSA-Test$
  Password: leave blank
  Run the task — verify it works

STEP 5: Verify no password anywhere
  Search entire server for "gMSA-Test" in:
  → All script files
  → All config files
  → Task Scheduler
  → Services
  Password should appear: NOWHERE

---

> 💡 **Architect's Real Talk:** *"In 15 years, the #1 reason organizations get Kerberoasted is not lack of knowledge — it's 'we have 200 service accounts and no time to fix them.' The answer is not to fix all 200 overnight. It's to fix the 20 highest-risk ones this week. A Domain Admin service account with a 3-year-old password is a ticking bomb. Find it, convert it to gMSA, sleep better. Then do the next 20."*

---

