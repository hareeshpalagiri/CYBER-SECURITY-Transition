
---

## Entra ID Attack Types — The Cloud Identity Threat Landscape

---

### ATTACK 1 — Password Spray

```
CONCEPT:
Same as on-prem spray — but now targeting cloud.
One common password tried against ALL users.
Never locks anyone out.
Runs for days undetected.

HOW IT TARGETS SECURECORP:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Attacker discovers SecureCorp emails via:
→ LinkedIn (company page → employees)
→ Email format guessing: firstname.lastname@securecorp.com
→ Hunter.io → finds email patterns
→ OSINT on GitHub, conference talks, blog posts

Attacker builds list:
  hareesh@securecorp.com
  priya@securecorp.com
  gani@securecorp.com
  chaitu@securecorp.com
  bharath@securecorp.com
  jayanth@securecorp.com
  navi@securecorp.com
  ... 500 users

Tries password: "Welcome@2024"
  (Most common corporate default in India)

Result after 6 hours:
  gani@securecorp.com → SUCCESS ✅
  (Gani joined recently → still on temp password pattern)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT NAVI SEES IN SIGN-IN LOGS:
Filter: Status=Failure + Last 1 hour
Pattern:
  02:00 AM — hareesh  — FAIL — 185.220.xx.xx
  02:00 AM — priya    — FAIL — 185.220.xx.xx
  02:00 AM — gani     — SUCCESS — 185.220.xx.xx ← breach
  02:00 AM — chaitu   — FAIL — 185.220.xx.xx
  02:00 AM — bharath  — FAIL — 185.220.xx.xx

KEY INDICATORS:
→ Same IP across multiple users
→ One attempt per user (no lockout triggered)
→ Failures spread across accounts not concentrated
→ Unusual time (2 AM IST)
→ Success mixed with failures = spray that worked

NAVI'S IMMEDIATE ACTIONS:
1. Block IP via Named Location in CA
2. Force password reset on gani immediately
3. Check what gani's account accessed post-login
4. Run query: did this IP target other tenants?
   (Microsoft Threat Intelligence shows this)
5. Alert all users — potential credential leak
```

---

### ATTACK 2 — MFA Fatigue (Prompt Bombing)

```
CONCEPT:
Attacker has the password.
MFA is the only obstacle.
Solution: overwhelm the user with push notifications
until they approve out of frustration.

THE UBER BREACH — REAL EXAMPLE (2022):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Attacker bought Uber employee's credentials
   from dark web data breach

2. Triggered MFA push notifications repeatedly
   Employee's phone: APPROVE? APPROVE? APPROVE?
   (47 notifications over 20 minutes at midnight)

3. Attacker also WhatsApped the employee:
   "Hi, I'm from Uber IT. Please approve the
    MFA — we're doing a security verification"

4. Exhausted employee approved

5. Attacker had full access to Uber's systems
   Including: AWS, Google Cloud, Slack, HackerOne

6. Total damage: significant data exposure,
   $148M in fines (previous breach settlement)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

IN SECURECORP — SAME SCENARIO:
Attacker gets Hareesh's password from spray
Bombards Hareesh's phone with MFA pushes
Hareesh approves at 11 PM to make it stop
Attacker now has Global Admin session

WHAT LOGS SHOW:
Sign-in log for hareesh:
  MFA result: MFA satisfied ← looks legitimate!
  Method: Authenticator app push
  Status: Success
  IP: 185.220.xx.xx ← different from usual
  Location: Netherlands ← different from usual
  Time: 23:47 IST ← outside work hours

The MFA PASSING is the suspicious part here.
Why did Hareesh approve MFA from Netherlands at midnight?

DEFENCES:
─────────────────────────────────────────────────────────
1. NUMBER MATCHING (most important):
   Instead of just APPROVE/DENY button —
   Screen shows a 2-digit number
   User must TYPE that number in Authenticator
   Attacker can't trick user — they see the number
   User thinks: "I didn't initiate this login"
   → DENY

   Enable: Entra ID → Security → MFA →
           Additional settings → Number matching → Enabled

2. ADDITIONAL CONTEXT:
   Push notification shows:
   → Which app is being accessed
   → Geographic location of sign-in
   User sees: "Sign in to Teams from Netherlands"
   Thinks: "I'm in Bangalore — DENY!"

3. LIMIT MFA RETRIES:
   After 3 failed MFA attempts → block for 1 hour
   Stops the bombing strategy

4. PHISHING-RESISTANT MFA (best long term):
   FIDO2 Security Keys (YubiKey)
   Windows Hello for Business
   Cannot be triggered remotely → fatigue impossible
```

---

### ATTACK 3 — Adversary in the Middle (AiTM)

```
CONCEPT:
The most sophisticated identity attack today.
Bypasses MFA completely.
Steals session AFTER MFA is completed.

SIMPLE ANALOGY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Normal login:
  You → Bank → Show ID → Get access card → Enter vault

AiTM:
  You → FAKE BANK FRONT → Real Bank → Show ID
                ↓
         Attacker standing
         in fake front copies
         your access card
                ↓
              → Enter vault AS YOU

The real bank never knows.
You authenticated successfully.
Attacker has a copy of your access card (session token).
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

HOW IT WORKS TECHNICALLY:
─────────────────────────────────────────────────────────
Step 1: Gani gets phishing email:
        "Your Teams subscription is expiring —
         click here to renew"
        Link: teams-securecorp-login.com ← FAKE

Step 2: Fake site is Evilginx2 proxy:
        Looks EXACTLY like Microsoft login
        Real SSL certificate (green padlock!)
        Proxies everything to real Entra ID

Step 3: Gani enters password → proxy forwards to Entra ID
        Gani completes MFA → proxy forwards to Entra ID
        Entra ID issues session token → proxy CAPTURES it

Step 4: Gani lands on real Teams — thinks nothing happened
        Attacker has his session token
        Uses token from their machine
        → Full access as Gani, NO MFA needed, NO password

Step 5: Token is valid for 90 days (refresh token)
        Even if Gani changes password → token still works
        Until password change forces re-auth

WHAT NAVI SEES:
─────────────────────────────────────────────────────────
Two sign-in entries very close together:

Entry 1 — Gani's real login:
  Time: 10:14:32
  IP: 103.21.xx.xx (Bangalore)
  MFA: Satisfied ✅
  Status: Success

Entry 2 — Attacker using stolen token:
  Time: 10:14:45  ← 13 seconds later!
  IP: 185.220.xx.xx (Netherlands)
  MFA: Not required ← token replay, no MFA needed
  Status: Success

ALERT: Same user, two locations, 13 seconds apart
       = Impossible travel = AiTM attack

DEFENCES:
─────────────────────────────────────────────────────────
1. Phishing-resistant MFA → token binding prevents replay
2. Conditional Access → require compliant device
   Token stolen on unknown device → CA blocks it
3. Continuous Access Evaluation (CAE):
   Token revoked in real time when risk detected
   Not waiting for 1-hour expiry
4. Microsoft Defender for Office 365:
   Catches the phishing email before Gani clicks
5. Entra ID Protection:
   "Impossible travel" alert fires → auto block
```

---

### ATTACK 4 — Illicit Consent Grant

```
CONCEPT:
Uses OAuth — the system behind "Login with Google/Microsoft"
Attacker creates fake app → tricks user into granting
permissions → app gets persistent token access
WITHOUT needing user's password or MFA.

WHY IT'S SO DANGEROUS:
→ No password stolen
→ MFA never triggered
→ Looks like legitimate app authorization
→ Survives password resets
→ Access persists for months

HOW IT TARGETS SECURECORP:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 1: Attacker registers app in THEIR OWN tenant:
        App name: "SecureCorp HR Dashboard"
        Publisher: Looks legitimate
        
Step 2: Requests these permissions:
        → Mail.Read (read all emails)
        → Mail.Send (send as user)
        → Files.ReadWrite (read/write all files)
        → Contacts.Read (read all contacts)

Step 3: Sends phishing email to Chaitu (HR):
        "New HR Dashboard launched — connect your
         account to access payroll reports"
        Link opens REAL Microsoft consent screen

Step 4: Chaitu sees:
        "SecureCorp HR Dashboard wants to:
         ✓ Read your email messages
         ✓ Send emails on your behalf
         ✓ Access your files"
        Thinks it's official → clicks ACCEPT

Step 5: Attacker's app now has OAuth token for Chaitu
        → Reads ALL her emails (payroll, HR data)
        → Searches for: salary, password, credentials
        → Finds Hareesh's email with Azure credentials
        → Sends emails AS Chaitu to other employees
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT NAVI FINDS DURING INVESTIGATION:
Entra ID → Enterprise Applications → All Applications

Suspicious app found:
  Name: SecureCorp HR Dashboard
  Publisher: Unverified ← red flag
  Users consented: 3 (Chaitu, Priya, Gani)
  Permissions: Mail.Read, Mail.Send, Files.ReadWrite
  Last used: 10 minutes ago ← actively being used!
  Created: Yesterday ← brand new app!

IMMEDIATE ACTIONS:
1. Revoke consent for all users:
   Enterprise Apps → App → Users and groups → Remove all
2. Revoke all tokens issued to this app
3. Delete the app registration if you own it
4. Check what data was accessed via audit logs
5. Notify affected users

DEFENCES:
─────────────────────────────────────────────────────────
1. Disable user consent entirely:
   Entra ID → Enterprise Apps →
   Consent and Permissions →
   "Do not allow user consent"
   → All app consent requires Admin approval

2. Admin Consent Workflow:
   User requests → notification sent to Hareesh
   Hareesh reviews permissions → approves or denies
   Nothing gets through without admin eyes

3. Microsoft Defender for Cloud Apps:
   Detects suspicious OAuth apps automatically
   Alerts on: new app, broad permissions,
              unverified publisher, rapid adoption
```

---

## 📌 SECTION 4 of 5
## Token Security — The New Attack Currency

---

```
WHY TOKENS MATTER MORE THAN PASSWORDS NOW:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OLD WORLD (passwords):
  Attacker steals password → change password → safe

NEW WORLD (tokens):
  Attacker steals token → change password → STILL NOT SAFE
  Token is still valid until it expires
  Password change alone doesn't revoke active tokens

TOKEN TYPES IN SECURECORP:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┌─────────────────┬──────────────┬──────────────────────┐
│ Token Type      │ Lifetime     │ Risk if Stolen       │
├─────────────────┼──────────────┼──────────────────────┤
│ Access Token    │ 1 hour       │ 1 hour of access     │
│ Refresh Token   │ 90 days      │ 90 days of access    │
│ PRT (Primary    │ 14 days      │ Full device session  │
│ Refresh Token)  │ (renewable)  │ SSO to everything    │
│ Session Cookie  │ Until closed │ Browser session      │
└─────────────────┴──────────────┴──────────────────────┘

THE PRIMARY REFRESH TOKEN (PRT) — Most Valuable:
─────────────────────────────────────────────────────────
→ Issued when you log into a Windows machine
→ Stored in device's TPM chip (secure hardware)
→ Gives SSO to ALL Microsoft services
→ If stolen → attacker has SSO to everything
  No MFA needed for any service

How attackers steal PRT:
→ Mimikatz module: sekurlsa::cloudap
→ Requires local admin on the machine
→ Why PAW matters: admin's PRT on a compromised machine
  = keys to the entire Microsoft 365 kingdom

HOW TO REVOKE ALL TOKENS IMMEDIATELY:
─────────────────────────────────────────────────────────
# If Gani's account is compromised:

# Option 1 — Entra ID Portal:
Entra ID → Users → Gani → Revoke sessions
→ Kills all active tokens immediately

# Option 2 — PowerShell:
Connect-MgGraph -Scopes "User.ReadWrite.All"
Revoke-MgUserSignInSession -UserId "gani@securecorp.com"

# Option 3 — Force password reset + revoke:
# Entra ID → Users → Gani
# → Reset password (forces token revocation)
# → Revoke sessions
# → Require MFA registration again
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 SECTION 5 of 5
## SecureCorp Breach — Full Entra ID Story

---

```
⏰ MONDAY 9:47 PM — SECURECORP IS BREACHED

THE ATTACK UNFOLDS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
09:47 PM: Password spray runs against securecorp.com
          1 attempt per user, spaced 30 seconds apart
          Password tried: "Welcome@2024"

10:02 PM: gani@securecorp.com → SUCCESS
          Gani joined last month, still on temp password
          No MFA registered ← Biggest failure point

10:03 PM: Attacker logs into Outlook Web Access as Gani
          Searches emails for: password, credential, admin
          Finds: Email from Hareesh to Gani last week:
          "Welcome to SecureCorp! Your initial Azure
           portal password is SecureCorp@Azure2024"
          
          🚨 Hareesh sent credentials via email — bad practice

10:08 PM: Attacker tries Azure Portal with found credentials
          hareesh@securecorp.com / SecureCorp@Azure2024
          MFA triggered → 47 push notifications sent
          
10:31 PM: Hareesh approves MFA (frustrated, watching TV)
          Attacker has Global Admin session token

10:32 PM: Attacker immediately:
          → Creates new Global Admin: svc-azure@securecorp.com
          → Disables MFA for this account
          → Creates App Registration with full permissions
          → Client secret: never expires
          → Exports list of all users, groups, apps

10:34 PM: Downloads all SharePoint files via Graph API
          Exports all Exchange emails via Graph API
          Both done silently via API → no user impact

10:47 PM: Disables audit logging on Azure subscription
          Attempts to delete sign-in logs ← Entra ID
          prevents complete deletion (kept 30 days minimum)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏰ TUESDAY 9:00 AM — NAVI STARTS SHIFT

NAVI'S DETECTION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
09:00 AM: Sentinel dashboard — 4 HIGH alerts overnight

🔴 ALERT 1: Password Spray Detected
   47 accounts, 1 failure each, IP: 185.220.xx.xx
   + 1 success: gani@securecorp.com
   Time: 09:47 PM - 10:02 PM

🔴 ALERT 2: New Global Admin Created
   svc-azure@securecorp.com added to Global Admins
   Created by: hareesh@securecorp.com
   Time: 10:32 PM
   "Was Hareesh working at 10:32 PM?"

🔴 ALERT 3: Impossible Travel — Hareesh
   Login from Bangalore 6:30 PM
   Login from Netherlands 10:31 PM
   = 4 hours, different continents
   = IMPOSSIBLE without teleportation

🔴 ALERT 4: Mass Data Export via Graph API
   Unusual volume: 47,000 API calls in 8 minutes
   Accessing: /messages, /drive/root/children
   Account: hareesh@securecorp.com

09:05 AM: Navi calls Hareesh
          "Were you working at 10:31 PM from Netherlands?"
          Hareesh: "No — but I did approve an MFA..."
          Navi: "That was the attacker. We're breached."

CONTAINMENT — NEXT 15 MINUTES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
09:06 AM: Revoke ALL sessions — Hareesh + Gani
09:07 AM: Disable svc-azure@securecorp.com
09:08 AM: Remove Global Admin from svc-azure
09:09 AM: Delete malicious App Registration
09:10 AM: Reset Hareesh + Gani passwords
09:11 AM: Block 185.220.xx.xx via Named Location CA policy
09:12 AM: Re-enable audit logging
09:14 AM: Engage Microsoft Security Response Center
09:20 AM: Brief CISO — full incident report begins

WHAT COULD HAVE PREVENTED THIS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ Gani had no MFA → password spray succeeded
   FIX: Block sign-in until MFA registered (CA policy)

❌ Hareesh sent credentials via email
   FIX: Use Azure Key Vault, never email credentials

❌ MFA was push notification → fatigue attack worked
   FIX: Number matching MFA enabled

❌ No alert on MFA approved from new country
   FIX: Sentinel alert on impossible travel

❌ Hareesh's account had standing Global Admin
   FIX: PIM — just-in-time admin access only
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 🧪 DAY 7 LABS

```
LAB 1 — Investigate YOUR Sign-in Logs (20 min)
Platform: Entra ID Portal → Monitoring → Sign-in logs
────────────────────────────────────────────────────────
1. Filter: Last 7 days → Status: Failure
   → What error codes appear most?
   → Any unfamiliar locations?

2. Click any SUCCESSFUL login → open all tabs:
   → Authentication Details tab
     What MFA method was used?
     Was single factor or multi factor?
   → Conditional Access tab
     Which policies applied?
     Any failures?
   → Device Info tab
     Is device compliant?

3. Filter: Client App = "Other clients"
   → Any legacy auth in your tenant?
   → Who is using it?

4. Export 7 days to CSV → open in Excel
   → Sort by IP Address
   → Any IP appearing on many accounts?
   → That could be password spray

LAB 2 — Check Enterprise Apps for Risky Consent (15 min)
Platform: Entra ID → Enterprise Applications
────────────────────────────────────────────────────────
1. All Applications → sort by Created date
2. Look for apps created recently
3. Click each app → Permissions tab
   → What permissions does it have?
   → Are they justified?
4. Check: User consent tab
   → Which users consented?
   → Did they know what they were consenting to?

LAB 3 — Microsoft Learn Free Lab (45 min)
Platform: learn.microsoft.com (FREE — no subscription)
────────────────────────────────────────────────────────
Search: "Investigate sign-in logs Microsoft Entra"
OR direct: aka.ms/sc200learn

Module: "Investigate threats using Microsoft Entra ID"
→ Fully interactive sandbox included
→ No Azure subscription needed
→ Covers exactly what we discussed today

LAB 4 — Review Guest Accounts (15 min)
Platform: Entra ID → Users → Filter: Guest
────────────────────────────────────────────────────────
# PowerShell audit of stale guests:
Connect-MgGraph -Scopes "User.Read.All"

Get-MgUser -Filter "userType eq 'Guest'" -All |
  Select DisplayName, UserPrincipalName,
         CreatedDateTime, SignInActivity |
  Where {
    $_.SignInActivity.LastSignInDateTime -lt
    (Get-Date).AddDays(-90)
  } |
  Sort-Object SignInActivity.LastSignInDateTime |
  Format-Table -AutoSize

# Any guest not logged in 90+ days = review access
# Do they still need it? If not → remove
```

---

## 📋 DAY 7 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║       DAY 6 — MICROSOFT ENTRA ID SECURITY               ║
║       Architecture + Attacks + Sign-in Logs             ║
║                   SecureCorp Ltd.                        ║
╚══════════════════════════════════════════════════════════╝

KEY ARCHITECTURE DIFFERENCE:
─────────────────────────────────────────────────────────
On-prem AD:  Attacker needs network access first
Entra ID:    login.microsoftonline.com = globally exposed
             New perimeter = IDENTITY (not firewall)

IDENTITY TYPES — SECURITY RISK:
─────────────────────────────────────────────────────────
Members:           Credential theft, MFA bypass targets
Guests:            Stale access, vendor breach risk
Service Principals: Over-privileged, secrets never expire
Managed Identities: Scope too broadly, lateral movement

MODERN AUTH FLOW:
─────────────────────────────────────────────────────────
User → Password → Risk Evaluation → CA Policies
→ MFA Challenge → Tokens Issued → Access Granted
Every step recorded in Sign-in Logs

SIGN-IN LOG KEY TABS:
─────────────────────────────────────────────────────────
Basic Info:    Who, where, when, success/fail
Auth Details:  MFA method, single/multi factor result
CA Tab:        Which policies applied, pass/fail
Device Info:   Compliant? Managed? OS/browser
Risk Tab:      AI risk level, detections found

ATTACK TYPES:
─────────────────────────────────────────────────────────
1. PASSWORD SPRAY
   One password → all users → no lockout
   Detect: same IP + multiple users + 1 failure each
   Fix: MFA all users + smart lockout tuning

2. MFA FATIGUE (Prompt Bombing)
   Flood user with push notifications → user approves
   Real example: Uber 2022 breach — exact method
   Detect: MFA approved from new country/IP at odd hours
   Fix: Number matching MFA + limit retry attempts

3. ADVERSARY IN THE MIDDLE (AiTM)
   Proxy captures token AFTER MFA completes
   Detect: Same user, two locations, seconds apart
   Fix: Phishing-resistant MFA + compliant device CA

4. ILLICIT CONSENT GRANT
   Fake OAuth app → user grants access → persistent token
   No password/MFA needed → survives password reset
   Detect: New app, broad permissions, unverified publisher
   Fix: Disable user consent, require admin approval

TOKEN TYPES & RISK:
─────────────────────────────────────────────────────────
Access Token:  1 hour → resource access
Refresh Token: 90 days → gets new access tokens
PRT:           14 days renewable → SSO to everything
Session Cookie: browser session lifetime

Revoke immediately when compromised:
Entra ID → Users → [user] → Revoke sessions
OR: Revoke-MgUserSignInSession -UserId "[email]"

SECURECORP BREACH LESSONS:
─────────────────────────────────────────────────────────
✗ No MFA for Gani (new joinee) → spray succeeded
✗ Credentials sent via email → found by attacker
✗ Push MFA → fatigue attack worked on Hareesh
✗ Standing Global Admin → immediate escalation
✗ No impossible travel alert configured

FIXES:
✓ Block sign-in until MFA registered (CA policy)
✓ Never send credentials via email → Key Vault
✓ Number matching MFA → fatigue impossible
✓ PIM → just-in-time admin access
✓ Sentinel alert → impossible travel

LABS:
─────────────────────────────────────────────────────────
□ Investigated sign-in logs — all 4 tabs
□ Reviewed Enterprise Apps for risky consent
□ Microsoft Learn: SC-200 Entra ID investigation
□ Audited stale guest accounts via PowerShell
```

---

> 💡 **Architect's Reality:** *"Every week I see breach reports where the root cause is one of four things: no MFA, push MFA fatigue, legacy authentication not blocked, or stale guest access. These are not sophisticated zero-day attacks. They are basic hygiene failures. In your SOC Analyst interviews, when they ask about identity security — you now have four real attack stories with real detection methods. That's what gets you hired."*

---

**Day 8 tomorrow — Conditional Access Policies deep dive.**
We build SecureCorp's complete CA policy set from scratch — policy by policy, with real decision making behind each one. 🔐

