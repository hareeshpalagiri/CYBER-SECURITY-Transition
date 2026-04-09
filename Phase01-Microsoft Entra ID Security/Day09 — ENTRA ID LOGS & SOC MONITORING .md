# 🔵 PHASE 2 — DAY 8
# Entra ID Logs & SOC Monitoring 
### *SecureCorp Ltd. | The Art of Reading Cloud Identity Logs*

---

## 🧭 DAY 8 MINDSET

```
DAY 6 — You learned WHAT Entra ID is and HOW it's attacked
DAY 7 — You learned HOW to defend with Conditional Access
DAY 8 — You learn HOW TO SEE EVERYTHING IN LOGS

This is your daily weapon as a SOC Analyst.
Every attack leaves footprints in Entra ID logs.
Your job is to find those footprints before damage is done.

Navi's morning starts here — not with email, not with Teams.
He opens Sentinel, checks Entra ID logs, looks for anomalies.
That habit is what separates reactive security from proactive.
```

---

## 📌 SECTION 1 of 5
## The Three Log Sources — Where Everything Lives

---

### Log Source 1 — Sign-in Logs

This is the most important log for SOC Analysts. Every authentication attempt — success or failure — lands here.

```
LOCATION:
Entra ID Portal → Monitoring → Sign-in logs

WHAT IT CAPTURES:
Every login attempt to any Microsoft cloud service:
→ Microsoft 365 (Teams, Outlook, SharePoint)
→ Azure Portal
→ Custom apps registered in Entra ID
→ SaaS apps connected via SSO
→ Both interactive and non-interactive sign-ins

RETENTION:
Entra ID Free:  7 days
Entra ID P1/P2: 30 days
Microsoft Sentinel: Indefinite (you control)

VOLUME IN SECURECORP:
~500 users × ~20 sign-ins/day = ~10,000 events/day
As SOC Analyst — you don't read all 10,000
You write KQL queries that surface the 5 that matter
```

### The Two Types of Sign-ins Navi Monitors

```
TYPE 1 — INTERACTIVE SIGN-INS:
─────────────────────────────────────────────────────────
User actively typed credentials or approved MFA
Examples:
  → Gani logs into Teams at 9 AM
  → Hareesh signs into Azure Portal
  → Chaitu opens Outlook Web Access

What Navi sees:
  User, app, IP, location, MFA result, CA result, risk

TYPE 2 — NON-INTERACTIVE SIGN-INS:
─────────────────────────────────────────────────────────
Background authentications — no user action required
Examples:
  → Teams refreshing access token silently
  → Outlook fetching new email in background
  → App using refresh token to get new access token

Why Navi cares:
  Token theft shows up HERE
  Attacker using stolen refresh token = non-interactive
  sign-in from different IP/country
  This is harder to detect — requires specific KQL
```

### Reading a Sign-in Log Entry — All 5 Tabs

```
SIGN-IN LOG ENTRY — Full anatomy:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 1 — BASIC INFO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Date/Time:          08/04/2026  09:14:32 IST
Request ID:         abc123-def456 (unique per attempt)
Correlation ID:     xyz789 (groups related sign-ins)
User:               gani@securecorp.com
User type:          Member
Application:        Microsoft Teams
Application ID:     unique GUID
Client app:         Browser              ← modern auth ✅
IP address:         103.21.xx.xx
Location:           Bangalore, KA, IN
Status:             Success
Error code:         0                    ← 0 = success
Failure reason:     -

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 2 — AUTHENTICATION DETAILS        ← KEY TAB
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Authentication requirement: Multifactor auth
MFA result:               MFA completed in Entra ID
Authentication method:    Microsoft Authenticator
Auth step details:
  Step 1: Password  → Satisfied ✅
  Step 2: MFA push  → Approved  ✅
Token issuer type:        Azure AD
Sign-in identifier:       gani@securecorp.com

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 3 — CONDITIONAL ACCESS TAB        ← CRITICAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CA001-Block-Legacy-Auth:     Not applied (modern auth)
CA002-Require-MFA-All:       ✅ Applied — Satisfied
CA003-Admin-Compliant:       Not applicable (not admin)
CA004-Block-High-Risk:       ✅ Applied — Not blocked
CA005-Block-Outside-India:   Not applicable (India IP)

Navi reads: All CA policies behaving correctly ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 4 — DEVICE INFO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Device ID:         abc123-def456
Device name:       GANI-LAPTOP
OS:                Windows 11
Compliant:         Yes ✅
Managed by:        Intune ✅
Join type:         Azure AD Joined
Browser:           Chrome 123.0

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 5 — SIGN-IN RISK (Entra ID P2)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Sign-in risk level:    Low
Risk state:            None
Risk detail:           None
Risk detections:       None

Navi reads: Clean sign-in. Nothing to investigate. ✅
```

---

### Log Source 2 — Audit Logs

```
LOCATION:
Entra ID Portal → Monitoring → Audit logs

WHAT IT CAPTURES:
Every CHANGE made to the tenant — not logins, but changes:
→ User created, modified, deleted
→ Group membership changes
→ Role assignments added or removed
→ MFA method registered or removed
→ App registration created or modified
→ Conditional Access policy changed
→ Password reset
→ Device enrolled or deleted

WHY THIS MATTERS TO NAVI:
Attackers don't just log in — they CHANGE things:
→ Add themselves to Global Admin
→ Create backdoor accounts
→ Disable audit logging
→ Add client secrets to app registrations
→ Remove MFA from accounts before attacking

Audit logs catch ALL of these.

RETENTION: Same as sign-in logs
           30 days Entra ID P1/P2
           Indefinite in Sentinel

KEY FIELDS IN AUDIT LOG:
Activity:          What happened ("Add member to role")
Status:            Success / Failure
Actor:             Who did it (hareesh@securecorp.com)
Target:            What was changed (gani@securecorp.com)
Modified props:    What changed (old value → new value)
Date/Time:         When it happened
```

### Log Source 3 — Identity Protection (Risk Logs)

```
LOCATION:
Entra ID → Security → Identity Protection
  → Risky sign-ins
  → Risky users
  → Risk detections

WHAT IT CAPTURES (Entra ID P2 required):
AI-generated risk signals for every user and sign-in

RISK DETECTION TYPES:
─────────────────────────────────────────────────────────
Anonymous IP address:
  Sign-in from Tor, known VPN, or anonymous proxy
  Risk level: Medium/High

Atypical travel:
  Sign-in from location physically impossible
  given previous login location and time
  Example: India at 9 AM → Russia at 9:05 AM

Leaked credentials:
  Microsoft found user's credentials for sale
  on dark web or hacker forums
  Risk level: High

Password spray:
  AI detected multiple accounts targeted
  with same password in short window
  Risk level: High

Malware linked IP:
  Sign-in from IP known to be botnet-controlled
  Risk level: High

Unfamiliar sign-in properties:
  New location, new device, new browser
  Doesn't match established user pattern
  Risk level: Low/Medium

Token issuer anomaly:
  Token properties don't match expected pattern
  Indicator of AiTM attack
  Risk level: High

RISKY USERS VIEW:
Shows users with accumulated risk over time
Even if individual sign-ins were Low risk —
pattern of risky behaviour = High user risk
```

---

## 📌 SECTION 2 of 5
## Critical Sign-in Error Codes — Navi's Reference Card

```
ERROR CODE    MEANING                    ATTACK INDICATOR
──────────────────────────────────────────────────────────────
0             Success                    Check for anomalies
16000         Multiple accounts          User selecting account
50053         Account locked             Lockout triggered
50055         Password expired           Normal / attacker exploit
50057         Account disabled           Targeting old accounts
50058         Silent sign-in failed      Token refresh issue
50072         MFA required               MFA not yet done
50074         MFA required (strong)      Step-up auth required
50076         MFA required (location)    New location triggered MFA
50079         MFA registration needed    New user setup
50126         Invalid credentials        🚨 Wrong password
50128         Invalid domain             Tenant misconfiguration
50129         Device not workplace joined Compliance issue
50133         Session invalid (PW change) Post-breach remediation
50140         Keep me signed in          Normal browser session
50173         Password change required   Admin forced reset
50196         Loop detected              Auth configuration error
53000         Compliance policy required Intune compliance check
53001         Domain not compliant       Device compliance fail
53003         Blocked by CA              🚨 CA policy enforced
53004         MFA setup not complete     New user, no MFA yet
70011         Invalid scope              App permission issue
70043         Refresh token expired      Re-auth needed
70044         Session lifetime expired   Token lifetime policy
75011         Auth method mismatch       Wrong auth method
75016         SAML assertion invalid     Federation issue
80001         Auth agent unavailable     PTA connector down
90010         Tenant not found           Wrong tenant targeted
90025         Dispatcher service failed  Infrastructure issue
──────────────────────────────────────────────────────────────

MOST IMPORTANT FOR ATTACK DETECTION:
50126 → Wrong password   = Brute force / spray
53003 → Blocked by CA    = CA policy working (good!)
50053 → Account locked   = Multiple failures hit threshold
70044 → Token expired    = Could indicate token theft attempt
0     → Success but...   = Check all other fields for anomalies
```

---

## 📌 SECTION 3 of 5
## Navi's KQL Playbook — Every Query You Need

```
LOCATION: Microsoft Sentinel → Logs → KQL editor
OR:       Entra ID → Logs (limited, no correlation)

All queries below run against SigninLogs and AuditLogs
tables in Microsoft Sentinel.
```

### QUERY 1 — Password Spray Detection

```kql
// Detect password spray: multiple accounts, same IP, 1 failure each
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType == "50126"  // Wrong password
| summarize
    FailedAccounts = dcount(UserPrincipalName),
    AttemptCount   = count(),
    AccountList    = make_set(UserPrincipalName, 10)
    by IPAddress, bin(TimeGenerated, 10m)
| where FailedAccounts > 5
| sort by FailedAccounts desc
| project TimeGenerated, IPAddress,
          FailedAccounts, AttemptCount, AccountList

// Alert threshold: 5+ accounts, same IP, 10-min window
// If FailedAccounts is high but AttemptCount/FailedAccounts ≈ 1
// = classic spray (1 attempt per account)
```

### QUERY 2 — Impossible Travel Detection

```kql
// Find users logging in from two locations too far apart
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == "0"  // Successful logins only
| project TimeGenerated, UserPrincipalName,
          Location, IPAddress, AppDisplayName
| sort by UserPrincipalName, TimeGenerated asc
| serialize
| extend PrevTime     = prev(TimeGenerated),
         PrevLocation = prev(Location),
         PrevUser     = prev(UserPrincipalName)
| where UserPrincipalName == PrevUser
| extend TimeDiffMins = datetime_diff('minute', TimeGenerated, PrevTime)
| where TimeDiffMins < 60           // Less than 1 hour apart
| where Location != PrevLocation    // Different locations
| where PrevLocation != ""
| project UserPrincipalName, PrevLocation,
          Location, TimeDiffMins, IPAddress, TimeGenerated
| sort by TimeGenerated desc

// Any result = investigate immediately
// India to Netherlands in 45 mins = impossible
```

### QUERY 3 — Legacy Authentication Usage

```kql
// Who is still using legacy auth? These bypass MFA.
SigninLogs
| where TimeGenerated > ago(30d)
| where ClientAppUsed in (
    "Exchange ActiveSync",
    "IMAP4",
    "POP3",
    "SMTP Auth",
    "Other clients",
    "Authenticated SMTP"
  )
| where ResultType == "0"  // Successful legacy logins
| summarize
    Count       = count(),
    LastSeen    = max(TimeGenerated),
    AppUsed     = make_set(AppDisplayName)
    by UserPrincipalName, ClientAppUsed
| sort by Count desc

// These users bypass MFA via legacy protocol
// Fix their apps THEN enable CA001
// Before CA001: this list = your blast radius
```

### QUERY 4 — MFA Bypass Detection

```kql
// Find successful logins where MFA was NOT required
// Could indicate CA policy gap or legacy auth success
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == "0"
| where AuthenticationRequirement == "singleFactorAuthentication"
| where ClientAppUsed !in ("Exchange ActiveSync", "Other clients")
// Excluding legacy — those are handled by CA001
| project TimeGenerated, UserPrincipalName,
          AppDisplayName, IPAddress, Location,
          ClientAppUsed, ConditionalAccessStatus
| sort by TimeGenerated desc

// Any modern auth login without MFA = CA policy gap
// Should be empty if CA002 is correctly configured
```

### QUERY 5 — Admin Role Changes (Critical Alert)

```kql
// Who was added to or removed from admin roles?
AuditLogs
| where TimeGenerated > ago(24h)
| where OperationName in (
    "Add member to role",
    "Remove member from role",
    "Add eligible member to role"
  )
| extend
    Actor       = tostring(InitiatedBy.user.userPrincipalName),
    TargetUser  = tostring(TargetResources[0].userPrincipalName),
    RoleName    = tostring(TargetResources[0]
                    .modifiedProperties[0].newValue)
| where RoleName has_any (
    "Global Administrator",
    "Privileged Role Administrator",
    "Security Administrator",
    "Exchange Administrator"
  )
| project TimeGenerated, Actor, TargetUser,
          RoleName, OperationName, Result
| sort by TimeGenerated desc

// Any result = immediate investigation
// Was this authorized? Was it during business hours?
// If Actor = unknown or TargetUser = new account = BREACH
```

### QUERY 6 — MFA Method Changes

```kql
// Who added or removed an MFA method?
// Removing MFA = attacker preparing account for takeover
AuditLogs
| where TimeGenerated > ago(7d)
| where OperationName in (
    "User registered security info",
    "User deleted security info",
    "User registered all required security info",
    "Admin registered security info for user"
  )
| extend
    Actor      = tostring(InitiatedBy.user.userPrincipalName),
    TargetUser = tostring(TargetResources[0].userPrincipalName),
    Method     = tostring(TargetResources[0]
                   .modifiedProperties[0].newValue)
| project TimeGenerated, OperationName,
          Actor, TargetUser, Method
| sort by TimeGenerated desc

// "User deleted security info" = RED FLAG
// Especially if Actor != TargetUser (admin removing MFA)
// Or if followed by immediate successful login
```

### QUERY 7 — New App Registrations (Backdoor Detection)

```kql
// Find new app registrations and their permissions
AuditLogs
| where TimeGenerated > ago(7d)
| where OperationName in (
    "Add application",
    "Add service principal",
    "Add OAuth2PermissionGrant",
    "Add app role assignment"
  )
| extend
    Actor   = tostring(InitiatedBy.user.userPrincipalName),
    AppName = tostring(TargetResources[0].displayName)
| project TimeGenerated, OperationName,
          Actor, AppName, Result
| sort by TimeGenerated desc

// New app registration + broad permissions
// + created by compromised account = illicit consent grant
```

### QUERY 8 — Token Replay / AiTM Detection

```kql
// Non-interactive sign-ins from different IP than interactive
// This is the fingerprint of token theft
let InteractiveLogins =
    SigninLogs
    | where TimeGenerated > ago(1h)
    | where IsInteractive == true
    | where ResultType == "0"
    | project User = UserPrincipalName,
              InteractiveIP = IPAddress,
              InteractiveTime = TimeGenerated,
              InteractiveLocation = Location;

SigninLogs
| where TimeGenerated > ago(1h)
| where IsInteractive == false  // Background token refresh
| where ResultType == "0"
| project User = UserPrincipalName,
          TokenIP = IPAddress,
          TokenTime = TimeGenerated,
          TokenLocation = Location
| join kind=inner InteractiveLogins on User
| where TokenIP != InteractiveIP
| where TokenLocation != InteractiveLocation
| project User, InteractiveIP, InteractiveLocation,
          InteractiveTime, TokenIP, TokenLocation, TokenTime
| sort by TokenTime desc

// Same user, different IPs within 1 hour
// Interactive (human) vs non-interactive (token replay)
// = Strong AiTM indicator
```

### QUERY 9 — Risky Sign-ins Summary (Morning Briefing)

```kql
// Navi's daily morning summary of overnight risky activity
SigninLogs
| where TimeGenerated > ago(12h)
| where RiskLevelDuringSignIn in ("high", "medium")
| summarize
    Count         = count(),
    HighRisk      = countif(RiskLevelDuringSignIn == "high"),
    MediumRisk    = countif(RiskLevelDuringSignIn == "medium"),
    UniqueUsers   = dcount(UserPrincipalName),
    BlockedByCA   = countif(ConditionalAccessStatus == "failure")
    by bin(TimeGenerated, 1h)
| sort by TimeGenerated desc

// Spike at 2-4 AM? Attack window.
// High BlockedByCA = CA policies working
// High HighRisk + low BlockedByCA = CA gap
```

### QUERY 10 — Guest Account Anomaly

```kql
// External/guest users accessing sensitive resources
SigninLogs
| where TimeGenerated > ago(7d)
| where HomeTenantId != ResourceTenantId  // Cross-tenant = guest
| where ResultType == "0"
| summarize
    Count       = count(),
    LastSeen    = max(TimeGenerated),
    Apps        = make_set(AppDisplayName)
    by UserPrincipalName, IPAddress, Location
| sort by Count desc

// Guest users accessing Azure Portal or admin apps = unusual
// Guest from unexpected country = investigate
```

---

## 📌 SECTION 4 of 5
## Navi's Real Morning Investigation — SecureCorp Breach Story

---

```
⏰ TUESDAY 09:00 AM — NAVI STARTS HIS SHIFT

Opens Sentinel → Incidents → Last 12 hours

THREE ALERTS WAITING:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 ALERT 1 (HIGH): Password Spray Detected
   Time:    02:14 AM - 02:47 AM
   Source:  185.220.101.45
   Details: 47 accounts targeted, 1 failure each
            1 success: gani@securecorp.com
   Error:   50126 (wrong password) for 46 accounts
            0 (success) for Gani

🟡 ALERT 2 (MEDIUM): MFA Security Info Deleted
   Time:    02:48 AM
   Account: jayanth@securecorp.com
   Actor:   hareesh@securecorp.com
   Details: Authenticator app removed from Jayanth

🔴 ALERT 3 (HIGH): New Global Admin Added
   Time:    02:51 AM
   New Admin: svc-reporting@securecorp.com
   Added by: hareesh@securecorp.com
   Details: Account created 02:49 AM, made GA 02:51 AM

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NAVI'S INVESTIGATION PROCESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

STEP 1 — Was Gani's login legitimate? (2 min)
─────────────────────────────────────────────
Query Sign-in logs for Gani at 02:14 - 02:50 AM:

Result:
  02:47 AM — Gani — SUCCESS — 185.220.101.45 (Netherlands)
  Client app: Browser (modern auth)
  MFA result: MFA NOT required ← 🚨
  CA002: NOT APPLIED ← 🚨 WHY?

Navi checks CA002 settings:
  Found: Gani excluded from CA002 (Helpdesk ticket last month
         excluded him due to MFA app issues — never re-added)
  = CA POLICY GAP

Gani's login: COMPROMISED ✅ (confirmed breach)

STEP 2 — What did attacker do after Gani's login? (3 min)
──────────────────────────────────────────────────────────
Query AuditLogs for actor = gani@securecorp.com
after 02:47 AM:

Result:
  02:47 AM — Gani — Read user list (all 500 users)
  02:47 AM — Gani — Read group memberships
  02:48 AM — Gani — No audit activity

Gani (standard user) can't do much.
Attacker pivoted — how did they get to Hareesh?

STEP 3 — Check Hareesh's login at 02:48 AM (3 min)
─────────────────────────────────────────────────────
Query Sign-in logs for Hareesh:

Result:
  02:48 AM — Hareesh — SUCCESS — 185.220.101.45 ← SAME IP!
  MFA result: MFA satisfied (Authenticator push approved)
  CA003: Applied — MFA required — satisfied

Navi calls Hareesh:
  "Did you approve an MFA request at 2:48 AM?"
  Hareesh: "Yes — I got multiple requests, approved one
            to make them stop."
  = MFA FATIGUE ATTACK

Hareesh's account: COMPROMISED ✅

STEP 4 — What did attacker do as Hareesh? (5 min)
─────────────────────────────────────────────────────
Query AuditLogs for actor = hareesh@securecorp.com
after 02:48 AM:

Result:
  02:48 AM — Created user: svc-reporting@securecorp.com
  02:49 AM — Reset password for svc-reporting
  02:51 AM — Added svc-reporting to Global Admins 🚨
  02:52 AM — Removed MFA from jayanth@securecorp.com 🚨
  02:53 AM — Disabled CA004 (High risk block policy) 🚨
  02:54 AM — Created app registration: "ReportingApp"
  02:55 AM — Added client secret to ReportingApp
             Expiry: Never 🚨

FULL ATTACK CHAIN CONFIRMED:
  Spray → Gani (CA gap, no MFA)
  → Enumerated users
  → MFA fatigue on Hareesh
  → Created backdoor admin
  → Removed Jayanth's MFA (next target)
  → Disabled CA policy
  → Created persistent app with secret

STEP 5 — CONTAINMENT (next 10 minutes)
─────────────────────────────────────────────────────
09:15 AM: Revoke sessions — Gani and Hareesh
          Entra ID → Users → Revoke sessions

09:16 AM: Disable svc-reporting@securecorp.com
          Remove from Global Admins

09:17 AM: Re-enable CA004 (high risk policy)

09:18 AM: Delete malicious app registration "ReportingApp"
          Revoke all OAuth tokens issued to it

09:19 AM: Re-add MFA to Jayanth's account
          Block 185.220.101.45 via Named Location CA policy

09:20 AM: Force password reset — Gani and Hareesh
          Force MFA re-registration

09:21 AM: Add Gani back to CA002 (fix the gap!)

09:25 AM: Engage Microsoft Security Response Center
          Evidence preservation begins

09:30 AM: Brief CISO — full incident timeline presented

LESSONS FROM THIS BREACH:
─────────────────────────────────────────────────────────
✗ CA002 exclusion for Gani never removed = CA gap
  FIX: Quarterly CA policy review — audit all exclusions

✗ MFA was push notification = fatigue attack worked
  FIX: Number matching MFA — should have been enabled

✗ CA003 required MFA but not compliant device for Hareesh
  FIX: Add compliant device requirement to CA003

✗ No alert on Hareesh doing admin actions at 2 AM
  FIX: Sentinel alert — admin operations outside hours

✗ App registration with non-expiring secret created
  FIX: App governance policy — max 90-day secret lifetime
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 SECTION 5 of 5
## Navi's Daily SOC Routine — Entra ID Edition

---

### Morning Checklist (15 minutes every day)

```
09:00 AM — OPEN SENTINEL DASHBOARD:
─────────────────────────────────────────────────────────
□ Check overnight incidents → sort by severity
□ Run Query 9 (risky sign-ins summary) → any spikes?
□ Run Query 5 (admin role changes) → any unauthorized?
□ Check Identity Protection → new risky users?
□ Review audit logs → any CA policy changes overnight?

If any HIGH severity alert → start incident response
If all clear → document "No incidents" in shift log

09:15 AM — WEEKLY CHECKS (every Monday):
─────────────────────────────────────────────────────────
□ Run Query 3 (legacy auth) → still zero CA001-blocked users?
□ Run Query 6 (MFA changes) → any MFA removed from accounts?
□ Review CA policy effectiveness report:
  Entra ID → Security → CA → Insights and reporting
□ Check guest accounts → any stale (90+ day inactive)?
□ Review app registrations → any new with broad permissions?

MONTHLY CHECKS:
─────────────────────────────────────────────────────────
□ Audit all CA policy exclusions — are they still needed?
□ Review Global Admin list → should have max 3-4 people
□ Check break-glass account → last used? (should be never)
□ Export risky users report → review with security team
□ App governance → any secrets expiring? Renew proactively
```

### How to Investigate Any Alert — Navi's Framework

```
THE 5W FRAMEWORK — Apply to every Entra ID alert:
─────────────────────────────────────────────────────────
WHO:    Which account? Member or Guest? Admin or standard?
        hareesh@securecorp.com vs unknown@gmail.com

WHAT:   What did they do? Sign-in? Configuration change?
        Login attempt vs Admin role assignment

WHERE:  Which IP? Which country? Known/unknown location?
        Bangalore office IP vs Tor exit node

WHEN:   Business hours? After hours? Pattern?
        9 AM weekday vs 2 AM Sunday

HOW:    Which protocol? MFA used? Device managed?
        Modern auth + MFA satisfied vs legacy auth + no MFA

THEN:   What happened AFTER? Timeline of activity?
        Innocent login vs login → enumeration → admin add

─────────────────────────────────────────────────────────
DECISION MATRIX:
─────────────────────────────────────────────────────────
Familiar IP + Normal time + MFA + Compliant = Monitor only
New IP + Normal time + MFA + Known user    = Low priority
Foreign IP + Off hours + No MFA            = Investigate NOW
Foreign IP + Off hours + Admin action      = P0 Incident
Multiple accounts + Same IP + 1 attempt   = Spray — contain
```

---

## 🧪 DAY 8 LABS

```
LAB 1 — Run KQL Queries in Sentinel (30 min)
Platform: Microsoft Sentinel → Logs
─────────────────────────────────────────────────────────
Run each of the 10 queries above against your tenant data.
For each query note:
→ Did it return any results?
→ Were any results suspicious?
→ What would you do if this fired as an alert?

Start with Query 3 (legacy auth) and Query 4 (MFA bypass)
These will show real gaps in your current tenant.

LAB 2 — Investigate Your Own Sign-in Logs (20 min)
Platform: Entra ID → Monitoring → Sign-in logs
─────────────────────────────────────────────────────────
Find YOUR last 5 sign-ins.
For each one — open all 5 tabs and answer:
→ Which CA policies applied?
→ Was MFA required? Which method?
→ Was device compliant?
→ What was the risk level?
→ Would this login raise an alert?

LAB 3 — Microsoft Sentinel Analytics Rules (30 min)
Platform: Microsoft Sentinel → Analytics
─────────────────────────────────────────────────────────
Go to: Analytics → Rule templates
Filter by: MITRE tactic = Initial Access
Look for built-in rules covering:
→ "Sign-ins from IPs that attempt sign-ins to disabled accounts"
→ "Successful logon from IP and failure from a different IP"
→ "Brute force attack against Azure Portal"
Enable any that match your environment.
These are pre-built versions of our KQL queries above.

LAB 4 — Microsoft Learn Free Lab (45 min)
Platform: learn.microsoft.com
─────────────────────────────────────────────────────────
Search: "SC-200 investigate alerts Microsoft Entra"
Module: "Investigate threats by using Microsoft Sentinel"
→ Free sandbox with real Sentinel environment
→ Investigate pre-built alerts
→ Run KQL queries against sample data
→ No subscription needed
```

---

## 📋 DAY 8 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║        DAY 8 — ENTRA ID LOGS & SOC MONITORING           ║
║                  SecureCorp Ltd.                         ║
╚══════════════════════════════════════════════════════════╝

THREE LOG SOURCES:
─────────────────────────────────────────────────────────
1. Sign-in Logs    → Every authentication attempt
                     Interactive + Non-interactive
                     30 days Entra P1/P2 · Indefinite Sentinel

2. Audit Logs      → Every configuration change
                     User/group/role/policy/app changes
                     Attacker changes show here

3. Identity Protection → AI-generated risk signals
                         Risky users + Risky sign-ins
                         Requires Entra ID P2

SIGN-IN LOG — 5 TABS (ALL MATTER):
─────────────────────────────────────────────────────────
Tab 1 - Basic Info:    Who, what app, where, when, success?
Tab 2 - Auth Details:  MFA method, single or multi factor
Tab 3 - CA Tab:        Which policies applied, pass or fail
Tab 4 - Device Info:   Managed? Compliant? Platform?
Tab 5 - Risk Tab:      AI risk level, specific detections

KEY ERROR CODES:
─────────────────────────────────────────────────────────
50126 → Wrong password      = Spray/brute force
53003 → Blocked by CA       = CA policy working ✅
50053 → Account locked       = Lockout triggered
50074 → MFA required         = Step-up auth needed
0     → Success              = Check all other fields

ATTACK DETECTION — SIGN-IN PATTERNS:
─────────────────────────────────────────────────────────
Password Spray:
  Many accounts + same IP + 1 failure each + same time
  Error: 50126 across multiple UserPrincipalNames

Impossible Travel:
  Same user + different countries + time gap impossible
  Success in both locations

Legacy Auth MFA Bypass:
  ClientAppUsed = IMAP/POP3/SMTP + Success + no MFA
  Should be zero if CA001 is enabled

MFA Fatigue Evidence:
  MFA satisfied + New IP + New country + Off hours
  Look at what happened AFTER the suspicious MFA approval

Token Theft (AiTM):
  Interactive login: India IP, 9 AM
  Non-interactive token use: Netherlands IP, 9:02 AM
  Same user, different IPs, minutes apart

ATTACK DETECTION — AUDIT LOG PATTERNS:
─────────────────────────────────────────────────────────
Backdoor Account:
  New user created + immediately added to Global Admins
  Especially outside business hours

MFA Removal Pre-attack:
  "User deleted security info"
  Followed by successful login minutes later

CA Policy Tampering:
  "Update conditional access policy"
  "Delete conditional access policy"
  = Attacker covering tracks

Illicit App Creation:
  "Add application" + "Add OAuth2PermissionGrant"
  + broad permissions + non-expiring secret

NAVI'S 10 KQL QUERIES:
─────────────────────────────────────────────────────────
Q1:  Password spray            (50126 + dcount users > 5)
Q2:  Impossible travel         (success + location change < 1hr)
Q3:  Legacy auth users         (ClientAppUsed = IMAP/POP3 etc)
Q4:  MFA bypass                (success + singleFactor + modern)
Q5:  Admin role changes        (AuditLogs + Global Admin adds)
Q6:  MFA method changes        (security info deleted/added)
Q7:  New app registrations     (Add application + permissions)
Q8:  Token replay/AiTM         (interactive vs non-interactive IP)
Q9:  Risky sign-ins summary    (morning briefing — 12hr window)
Q10: Guest anomalies           (cross-tenant + unusual apps)

SECURECORP BREACH — ATTACK CHAIN:
─────────────────────────────────────────────────────────
02:14 AM: Password spray → 47 accounts, 1 success (Gani)
02:47 AM: Gani login — no MFA (CA202 exclusion gap)
02:48 AM: Enumerated users/groups as Gani
02:48 AM: MFA fatigue attack on Hareesh — approved push
02:49 AM: Created backdoor: svc-reporting@securecorp.com
02:51 AM: Added svc-reporting to Global Admins
02:52 AM: Removed MFA from Jayanth
02:53 AM: Disabled CA004 (high risk block policy)
02:55 AM: Created app registration with non-expiring secret

CONTAINMENT ACTIONS:
→ Revoke sessions: Gani + Hareesh
→ Disable svc-reporting + remove from Global Admins
→ Re-enable CA004
→ Delete malicious app registration + revoke tokens
→ Re-add MFA to Jayanth
→ Block attacker IP via Named Location
→ Force password reset + MFA re-registration for all affected
→ Add Gani back to CA002 (fix the gap)

NAVI'S DAILY ROUTINE:
─────────────────────────────────────────────────────────
Morning (15 min):
  Check overnight incidents → risky sign-ins → admin changes
  
Weekly (Monday):
  Legacy auth audit → MFA changes audit → CA effectiveness
  Guest account review → app registration review

Monthly:
  CA exclusion audit → Global Admin count review
  Break-glass account check → app secret expiry review

INVESTIGATION FRAMEWORK — 5W:
─────────────────────────────────────────────────────────
WHO:   Which account? Role? Member or guest?
WHAT:  Sign-in or config change? What action?
WHERE: Which IP? Country? Known location?
WHEN:  Business hours? Pattern?
HOW:   Protocol? MFA? Device compliant?
THEN:  What happened after? Full timeline.

LABS COMPLETED:
─────────────────────────────────────────────────────────
□ Ran all 10 KQL queries in Sentinel
□ Investigated own sign-in logs — all 5 tabs
□ Enabled Sentinel Analytics Rule templates
□ Microsoft Learn SC-200 Sentinel investigation lab
```

---

> 💡 **Architect's Truth for Day 8:** *"In every breach I have investigated, the logs had the full story. Every step the attacker took — it was all there in Entra ID. The spray, the MFA approval, the admin creation, the CA policy change. Every single action timestamped and recorded. The question was never 'are the logs there?' — it was always 'was anyone watching?' That is your job as a SOC Analyst. Be the person watching."*

---

**Day 9 tomorrow — Entra ID Hands-On Labs.** We will run real KQL queries, investigate real alerts, and complete the Microsoft Learn SC-300 identity investigation module. 🔍
