
## 🎯 WHAT A SOC ANALYST ACTUALLY DOES DAILY

Before Day 2, you need to know **what job you're training for** — so every topic feels relevant.

```
8:00 AM  — Check overnight alerts in Microsoft Sentinel
8:30 AM  — Triage: Real attack or false positive?
9:00 AM  — Investigate suspicious login from unusual country
9:30 AM  — Check Defender for Endpoint — malware alert on a laptop
10:00 AM — Write incident report, escalate to Tier 2
10:30 AM — Review AD logs — someone added themselves to a group
11:00 AM — Threat hunting — search for IOCs from threat intel feed
           ... and repeat
```

Now let's hit **Day 2** — and I'm teaching this like we're sitting together at a whiteboard. 👇

---

# 🔴 PHASE 1 — DAY 2
# AD Attack Types — The Attacker's Playbook

---

## 🧠 First — The Attacker's Mindset

Every attacker who gets into a corporate network follows the **same journey:**

```
🌐 Get IN          — Phishing email, VPN exploit, weak password
        ↓
🔍 Look Around     — "Where am I? What can I reach?" (Reconnaissance)
        ↓
📈 Get Higher      — Steal credentials, escalate privileges
        ↓
👑 Own the Domain  — Become Domain Admin
        ↓
💰 Do the damage   — Ransomware, data theft, persistence
```

**AD attacks happen in steps 2, 3, and 4.**

Your job as a SOC Analyst — **catch them at step 2 or 3, before they reach step 4.**

---

## ⚔️ ATTACK 1 — Pass-the-Hash (PtH)

### What is it?
In Windows, when you type your password, it gets converted into a **hash** (a scrambled version). Windows actually uses this hash — not your plain password — to authenticate you.

**Pass-the-Hash = steal the hash → use it directly → no need to know the actual password.**

### Simple Analogy
> Imagine your office door opens with a fingerprint. Someone makes a **copy of your fingerprint** using silicone. They don't know your name, don't know anything about you — but that fake fingerprint opens your door.
>
> That's Pass-the-Hash. The hash IS the key.

### How It Happens Step by Step
```
1. Attacker gets into ONE low-privilege machine (via phishing)
2. Dumps password hashes from that machine's memory using Mimikatz
3. Finds a hash belonging to an admin who logged into that machine
4. Uses that hash to authenticate to OTHER machines — no password needed
5. Moves laterally across the network
```

### What It Looks Like in Logs
```
Event ID 4624  — Successful login
Logon Type: 3  — Network logon (suspicious if unexpected)
Auth Package: NTLM  — 🚨 Red flag in modern environments
Source IP: Internal machine the admin never uses
```

> 🔗 **Your connection:** At your organization, when you RDP into servers — that creates cached credentials. If that machine is compromised, your admin hash is sitting in memory. That's the risk.

### Defense
- Enable **Protected Users** group for all admins — prevents credential caching
- Enforce **Kerberos only** — disable NTLM where possible
- Use **Privileged Access Workstations (PAW)** — dedicated machines for admin work only
- **Microsoft Defender for Identity** detects PtH automatically

---

## ⚔️ ATTACK 2 — Kerberoasting

### What is it?
Remember Kerberos from Day 1? Any domain user can request a **service ticket** for any service account. That ticket is encrypted with the service account's password hash.

**Kerberoasting = request that ticket → take it offline → crack the password.**

### Simple Analogy
> Imagine a library where anyone can borrow a special locked box. Inside the box is a secret. The box is locked with the librarian's key (password).
>
> An attacker borrows the box, takes it home, and tries millions of keys until one opens it. The library never notices — borrowing is normal!
>
> That's Kerberoasting. The "borrowing" part is completely legitimate — that's what makes it so dangerous.

### Why Service Accounts Are Vulnerable
```
❌ Password set 5 years ago — never changed
❌ Password is "ServiceAcc2019!" — weak and guessable  
❌ Account has Domain Admin privileges (over-privileged)
❌ Nobody monitors it — it's "just a service account"
```

### How It Happens Step by Step
```
1. Attacker gets ANY domain user account (even lowest privilege)
2. Runs: Rubeus.exe kerberoast  (on Kali: GetUserSPNs.py)
3. Gets a list of ALL service accounts with SPNs registered
4. Requests service tickets for all of them — COMPLETELY NORMAL ACTIVITY
5. Takes those tickets offline to Hashcat
6. Cracks weak passwords in hours or minutes
7. Now has plaintext password for a service account
```

### What It Looks Like in Logs
```
Event ID 4769 — Kerberos service ticket requested
Ticket Encryption: 0x17 (RC4) 🚨 — weak encryption, easy to crack
Multiple requests in short time from same user 🚨
Requests at 3 AM 🚨
```

> 🔗 **Your connection:** Think about every service account you've seen at your organization — SQL service accounts, backup agents, monitoring tools. Each one is a potential Kerberoasting target. The ones with weak passwords and high privileges are the jackpot.

### Defense
- Service account passwords must be **25+ characters, complex, rotated regularly**
- Use **Managed Service Accounts (MSA) or Group Managed Service Accounts (gMSA)** — Windows manages the password automatically, rotates it, impossible to Kerberoast
- Monitor **Event 4769** — alert on RC4 encryption type
- **Defender for Identity** has built-in Kerberoasting detection

---

## ⚔️ ATTACK 3 — DCSync

### What is it?
Domain Controllers sync with each other to replicate AD data. This is called **DRS (Directory Replication Service)**. Normally only DCs talk to each other this way.

**DCSync = pretend to be a DC → ask real DC to send all password hashes.**

### Simple Analogy
> Two bank managers regularly transfer account records between branches — that's normal procedure.
>
> An attacker puts on a bank manager uniform, walks up to the main branch, and says "Hi, I'm from the west branch, please send me all customer account details and PINs."
>
> The main branch believes them — because they look like a manager — and hands over everything.
>
> That's DCSync. The attacker pretends to be a Domain Controller.

### What's Needed to Do DCSync
The attacker needs an account with one of these permissions:
- `Replicating Directory Changes`
- `Replicating Directory Changes All`

These are normally only on DCs — but misconfigurations sometimes give these to regular accounts. Also, once an attacker IS a Domain Admin, they can do DCSync freely.

### What They Get
```
All NTLM hashes for every user in the domain
Including: Domain Admin hashes
Including: KRBTGT hash  ← this is the nuclear option (Golden Ticket)
```

### What It Looks Like in Logs
```
Event ID 4662 — Operation performed on AD object
Access Mask: 0x100 (Replication rights used) 🚨
Source: Not a Domain Controller IP 🚨  ← biggest red flag
```

### Defense
- Audit who has **replication rights** on the domain object — should be ONLY DCs
- **Defender for Identity** has excellent DCSync detection — alerts immediately
- Monitor Event 4662 from non-DC machines

---

## ⚔️ ATTACK 4 — Golden Ticket

### What is it?
This is the **endgame attack**. The most powerful attack in AD.

There's a special account called **KRBTGT** — it's the account that signs ALL Kerberos tickets in the domain. If you steal its hash, you can **forge any ticket, for any user, with any privilege, that never expires.**

**Golden Ticket = forge a Domain Admin ticket that lasts 10 years.**

### Simple Analogy
> Every event ticket in a city is stamped by ONE official stamp — the Mayor's seal.
>
> If someone steals the Mayor's stamp, they can create tickets to ANY event, for ANY person, for FREE, FOREVER.
>
> Even if the Mayor changes — the old stamp still works on old tickets.
>
> That's the Golden Ticket. The KRBTGT hash is the Mayor's stamp.

### Why It's So Dangerous
```
✅ Works even after the real Domain Admin password is changed
✅ Works even after the compromised user is deleted
✅ Can be used to access ANY resource in the domain
✅ Tickets can be set to last 10 years
✅ Very hard to detect — looks like normal Kerberos traffic
```

### Defense (This Is Hard — Requires Specific Steps)
```
1. Rotate KRBTGT password TWICE — because old tickets still work after first rotation
2. Wait 10 hours between rotations (ticket lifetime)
3. Monitor for tickets with unusually long lifetime
4. Monitor for tickets with mismatched PAC (Privilege Attribute Certificate)
```

> ⚠️ **This is why DCSync is so catastrophic** — it gives the KRBTGT hash, which enables Golden Ticket. One attack enables the other.

---

## 🔗 HOW THESE ATTACKS CHAIN TOGETHER

Here's a **real attack scenario** — follow the chain:

```
Day 1:  Phishing email → low-level user account compromised
            ↓
Day 1:  Kerberoasting → crack SQL service account password
            ↓
Day 2:  SQL service account has local admin on DB servers
        Pass-the-Hash → move to DB server
            ↓
Day 2:  Find Domain Admin logged into DB server
        Dump Domain Admin hash from memory
            ↓
Day 3:  Use Domain Admin hash → DCSync → get KRBTGT hash
            ↓
Day 3:  Create Golden Ticket → invisible, permanent access
            ↓
Day 10: Deploy ransomware to ALL machines via GPO
```

**Your job as SOC Analyst: Catch this at Day 1 or Day 2.**
That's why monitoring Kerberoasting (Event 4769) and NTLM usage (Event 4776) matters so much.

---

## 🛡️ QUICK DEFENSE SUMMARY TABLE

| Attack | Detect With | Prevent With |
|--------|------------|-------------|
| Pass-the-Hash | Event 4624 + NTLM auth | Protected Users group, disable NTLM |
| Kerberoasting | Event 4769 + RC4 encryption | gMSA accounts, strong passwords |
| DCSync | Event 4662 from non-DC | Audit replication rights |
| Golden Ticket | Ticket lifetime anomalies | Rotate KRBTGT twice, Defender for Identity |

---

## 🎯 TODAY'S LAB TASK (30 minutes on TryHackMe)

Go to TryHackMe and search for this room:

**"Attacking Kerberos"** — it's free and covers exactly what we discussed today.

Work through the first 3 tasks. Tomorrow, tell me:
1. What surprised you most?
2. Which attack felt most dangerous to you and why?

---

## ❓ BEFORE YOU GO — QUICK QUIZ

I'll ask, you answer in your own words:

1. **Why can't you just change the Domain Admin password to stop a Golden Ticket attack?**
2. **What makes Kerberoasting so hard to detect?**
3. **At your orginization, which of these 4 attacks do you think is most likely to happen and why?**



---

> 💡 **Senior Engineer's Advice:** *"In every SOC interview I've ever conducted, I ask one question — 'What is Kerberoasting and how would you detect it?' If the candidate answers confidently with Event IDs and detection logic, they get the job. You now know the answer. Own it."*

See you on **Day 3 — AD Hardening & Logs**! 🔐
