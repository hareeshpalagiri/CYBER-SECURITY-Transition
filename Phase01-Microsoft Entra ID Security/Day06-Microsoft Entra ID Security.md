
---

# 🔵 PHASE 2 — DAY 6
# Microsoft Entra ID Security
## *Your Admin Knowledge Becomes Your Security Weapon*

---

## 🧭 DAY 6 MINDSET

```
PHASE 1 RECAP — What we built:
─────────────────────────────────────────────────────────
✅ AD structure, attacks, hardening, logs, labs
   You now think like both attacker AND defender
   for on-premises Active Directory

PHASE 2 — What we're adding:
─────────────────────────────────────────────────────────
Entra ID = AD's cloud cousin
Same identity concepts → completely new attack surface
Your existing admin experience = massive head start

KEY DIFFERENCE:
On-prem AD:   Attacker needs to be INSIDE the network
Entra ID:     Attacker only needs an INTERNET CONNECTION
              → Your attack surface is now GLOBAL
─────────────────────────────────────────────────────────
```

> 🔗 **Your connection:** You've managed users, groups, app registrations, sign-in logs, and fixed Entra Connect sync errors. Everything you've done — we now look at through a security lens. Same portal. Completely different questions.

---

## 📌 SECTION 1 of 5
## Entra ID Architecture — Security View

---

### SecureCorp's Entra ID Environment

```
SECURECORP ENTRA ID TENANT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tenant:  securecorp.onmicrosoft.com
Custom:  securecorp.com
Licence: Microsoft 365 E3 + Entra ID P2

CONNECTED TO:
→ On-premises AD (via Entra Connect — your sync work!)
→ Microsoft 365 (Exchange, Teams, SharePoint)
→ Azure Subscriptions (Dev + Prod)
→ SaaS Apps (Salesforce, GitHub, ServiceNow)
→ Custom apps (registered in App Registrations)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### The Identity Types — Security Risk View

```
TYPE 1 — MEMBER USERS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┌──────────┬──────────────────┬──────────────────────┐
│ User     │ Role             │ Security Risk        │
├──────────┼──────────────────┼──────────────────────┤
│ Hareesh  │ Global Admin     │ #1 target always     │
│ Priya    │ Developer        │ App reg access       │
│ Jayanth  │ End User Support │ Password reset perms │
│ Bharath  │ Network Engineer │ Azure network access │
│ Chaitu   │ HR Manager       │ Sensitive data access│
│ Gani     │ ML Engineer      │ No MFA = weakest link│
│ Navi     │ SOC Analyst      │ Log access — protect │
└──────────┴──────────────────┴──────────────────────┘

TYPE 2 — GUEST USERS (B2B)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SecureCorp has 23 active guest accounts
→ Vendors, consultants, auditors
→ Problem: 11 of them haven't logged in for 90+ days
→ Stale access = forgotten door left open
→ If vendor's tenant is breached → your data at risk

TYPE 3 — SERVICE PRINCIPALS & MANAGED IDENTITIES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
These are NON-HUMAN identities — apps, automation, scripts

SecureCorp examples:
→ App-SQLSync     → syncs data between SQL and SharePoint
→ App-BackupAgent → Azure Backup service
→ App-Monitoring  → Azure Monitor alerts
→ GitHubActions   → CI/CD pipeline

SECURITY PROBLEM WITH THESE:
→ Often created → given broad permissions → forgotten
→ Client secrets that never expire (= permanent password)
→ Nobody reviews them quarterly
→ One of the most common breach vectors in Azure today
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### The Entra ID Security Perimeter

This is the most important concept of Phase 2:

```
ON-PREMISES AD — SECURITY PERIMETER:
─────────────────────────────────────────────────────────
[INTERNET] ──firewall──> [INTERNAL NETWORK] ──> [DC01]

Attacker must:
1. Break through perimeter firewall
2. Get onto internal network
3. Then attack AD

Physical boundary protects you.

ENTRA ID — SECURITY PERIMETER:
─────────────────────────────────────────────────────────
[INTERNET] ──────────────────────────────> [ENTRA ID]

No firewall between internet and Entra ID.
Login page: login.microsoftonline.com
Accessible from ANYWHERE on earth.

Attacker needs:
1. Internet connection
2. A valid username (findable via LinkedIn)
3. That's it — they're already at the front door

THE NEW PERIMETER = IDENTITY ITSELF
Your username + password + MFA = your only wall
─────────────────────────────────────────────────────────
```

> 🎯 **This is why identity security is the #1 priority in modern security.** Not firewalls. Not network segmentation. Identity.

---

📌 SECTION 2 of 5
How Entra ID Authentication Works — Complete Picture

The Modern Authentication Flow
You fixed Entra Connect sync errors before — so you know identities flow from on-prem AD to Entra ID. Now let's see what happens when someone actually logs in through Entra ID.
SECURECORP LOGIN FLOW — GANI LOGS INTO TEAMS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

STEP 1 — Gani opens Teams, types his email:
  gani@securecorp.com

  Entra ID checks:
  → Is this a managed domain? YES (securecorp.com)
  → Is this a federated domain? NO
  → Show password page

STEP 2 — Gani types password:
  Password sent to Entra ID over HTTPS
  Entra ID checks against:
  → If PHS (Password Hash Sync): checks hash in cloud
  → If PTA (Pass-Through Auth): sends to on-prem AD
  
  You know this from your Entra Connect work! ✅

STEP 3 — Entra ID evaluates Sign-in Risk:
  AI checks:
  → Is this IP known malicious?
  → Is this an impossible travel?
  → Is this an anonymous IP (Tor/VPN)?
  → Is this a new device?
  → Does this match Gani's usual pattern?
  
  Risk Level assigned: Low / Medium / High

STEP 4 — Conditional Access Policies evaluated:
  Every policy checked in sequence:
  → CA001: Is this legacy auth? → Block
  → CA002: Is MFA done? → Require MFA
  → CA003: Is device compliant? → Check
  → CA004: Is risk HIGH? → Block
  
  Result: ALLOW / BLOCK / REQUIRE MFA

STEP 5 — MFA Challenge (if required):
  Gani gets push notification on Authenticator app
  Gani approves → MFA satisfied

STEP 6 — Tokens Issued:
  Access Token  → valid 1 hour → access Teams
  Refresh Token → valid 90 days → get new access tokens
  Session Cookie → browser session

STEP 7 — Gani is in Teams ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ALL OF THIS = ONE ENTRY in Sign-in Logs
EVERY step's result is recorded
THAT is why sign-in logs are so powerful
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Reading a Sign-in Log Entry — Full Breakdown
This is what you partially looked at before. Now let's read every field:
SAMPLE SIGN-IN LOG ENTRY — Gani's normal login:
━

BASIC INFO TAB:
─
Date:               05/04/2026  09:14:32 IST
User:               gani@securecorp.com
User type:          Member
Application:        Microsoft Teams
Client app:         Browser                ← modern auth ✅
IP address:         103.21.xx.xx           ← Bangalore IP
Location:           Bangalore, KA, IN      ← expected ✅
Status:             Success                ✅
Conditional Access: Success                ✅

──────
AUTHENTICATION DETAILS TAB:           ← YOU NEED THIS TAB
──────

Authentication requirement: Multifactor auth  ✅
MFA result:               MFA satisfied       ✅
Auth method:              Authenticator app   ✅
Token issuer type:        Azure AD
Sign-in identifier:       gani@securecorp.com
Resource:                 Microsoft Teams

─────
CONDITIONAL ACCESS TAB:               ← CRITICAL FOR SOC
─────

CA001-Block-Legacy-Auth:    Not applied (modern auth)
CA002-Require-MFA-All:      ✅ Applied — Satisfied
CA003-Admin-Compliant:      Not applicable (not admin)
CA004-Block-High-Risk:      ✅ Applied — Not blocked
                                        (risk was LOW)

───────
DEVICE INFO TAB:
───────

Device ID:         abc123-def456-...
Device name:       GANI-LAPTOP
Operating system:  Windows 11
Compliant:         Yes ✅
Managed by:        Intune ✅
Browser:           Chrome 123

───────
SIGN-IN RISK TAB:                     ← ENTRA ID P2
──────

Sign-in risk level:   Low
Risk detections:      None
Risk detail:          None
━━━━━━━

THIS IS A CLEAN LOGIN.
Now let's see what a SUSPICIOUS login looks like.

The Same Login — But Something Is Wrong
SUSPICIOUS SIGN-IN LOG ENTRY — Gani at 2 AM:
━━━━━━

BASIC INFO TAB:
───────

Date:               06/04/2026  02:17:44 IST
User:               gani@securecorp.com
Application:        Microsoft Teams
Client app:         Other clients          ← 🚨 LEGACY AUTH
IP address:         185.220.xx.xx          ← 🚨 Tor exit node
Location:           Amsterdam, NL          ← 🚨 Not India!
Status:             Failure
Error code:         53003                  ← Blocked by CA

─────
AUTHENTICATION DETAILS TAB:
─────

Authentication requirement: Single factor  ← 🚨 No MFA!
MFA result:               Not attempted   ← 🚨
Auth method:              Password only
Token issuer type:        Azure AD

──────
CONDITIONAL ACCESS TAB:
──────

CA001-Block-Legacy-Auth:  ✅ Applied → BLOCKED  ← saved us!

───────
SIGN-IN RISK TAB:
───────

Sign-in risk level:  HIGH  🚨
Risk detections:
  → Anonymous IP address (Tor)
  → Unfamiliar sign-in properties
  → Atypical travel
━━━━━━


WHAT NAVI READS FROM THIS:
→ Attacker has Gani's password
→ Using Tor to hide location
→ Tried legacy auth to bypass MFA
→ CA001 blocked it ✅
→ But: attacker KNOWS Gani's password
→ Action: Force Gani to change password NOW
          Review how attacker got his credentials
          Check for other accounts from same IP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔗 Your experience: When you opened sign-in logs before — you were probably looking at the Basic Info tab only. The Authentication Details and Conditional Access tabs are where the security story lives. That's the difference between admin troubleshooting and SOC investigation
