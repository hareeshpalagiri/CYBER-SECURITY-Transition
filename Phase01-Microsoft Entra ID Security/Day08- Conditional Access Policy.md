
---

# 📘 DAY 7 — CONDITIONAL ACCESS POLICIES
**Phase 2: Microsoft Entra ID Security | SecureCorp Ltd.**

---

## 🧭 DAY 7 MINDSET

```
DAY 6 — Learned Entra ID architecture + 4 attack types
DAY 7 — Build the defences using Conditional Access

Conditional Access = the most powerful identity
security control in Microsoft's entire stack.

One well-configured CA policy stops more attacks
than any firewall rule ever written.

Your existing admin knowledge of Entra ID means
you already know WHERE these settings live.
Today you learn WHY each setting exists and
WHAT attack it stops.
```

---

## 📌 SECTION 1 of 5
## What is Conditional Access — The Complete Mental Model

---

### The Old World vs New World

```
OLD SECURITY MODEL:
─────────────────────────────────────────────────────────
Username + Password = Access ✅

That's it. If you know the password — you're in.
An attacker in Russia with a stolen password
has the same access as Hareesh in Bangalore.
No questions asked.

NEW SECURITY MODEL — CONDITIONAL ACCESS:
─────────────────────────────────────────────────────────
Username + Password
+ WHO are you? (user identity, role, group)
+ WHERE are you? (country, IP, named location)
+ WHAT device? (managed, compliant, platform)
+ WHICH app? (Teams vs Azure Portal vs Salesforce)
+ HOW risky? (AI-evaluated risk score)
= THEN decide:
  Allow / Require MFA / Require device / Block

Context determines access — not just credentials.
```

### The Security Guard Analogy

```
OLD MODEL — Dumb security guard:
─────────────────────────────────────────────────────────
Guard checks ID card.
Card matches → You're in.
No other questions.
Stolen card = full access.

CONDITIONAL ACCESS — Smart security guard:
─────────────────────────────────────────────────────────
Guard checks ID card. Then asks:

"Is this your usual entry time?" (time/behaviour)
"Are you coming from your registered vehicle?" (device)
"You're entering from a different country today — why?" (location)
"This is the server room — do you have clearance?" (app)
"Our system flagged your IP as suspicious — verify again" (risk)

Only after all checks pass → You're in.
```

### The 5 Signals Evaluated on Every Sign-in

```
SIGNAL 1 — USER IDENTITY
─────────────────────────────────────────────────────────
Who is signing in?
→ Which user? (Gani vs Hareesh)
→ Which group? (standard users vs admins)
→ Which directory role? (Global Admin vs standard)
→ Member or Guest? (internal vs external)

Example policy use:
  IF user is in "Domain Admins" group
  THEN require MFA + compliant device

SIGNAL 2 — LOCATION
─────────────────────────────────────────────────────────
Where is the sign-in coming from?
→ Named locations (trusted — office IP ranges)
→ Country (India vs Russia vs Netherlands)
→ Compliant networks (specific known ranges)
→ MFA trusted IPs

Example policy use:
  IF location is NOT India
  THEN block access

SIGNAL 3 — DEVICE
─────────────────────────────────────────────────────────
What device is being used?
→ Intune-managed (enrolled in MDM)
→ Compliant (meets security baseline)
→ Hybrid Azure AD joined (domain + cloud)
→ Platform (Windows / iOS / Android / Linux)

Example policy use:
  IF admin accessing Azure Portal
  AND device is NOT compliant
  THEN block access

SIGNAL 4 — APPLICATION
─────────────────────────────────────────────────────────
Which app or resource is being accessed?
→ All cloud apps
→ Specific apps (Microsoft Teams, Azure Portal)
→ User actions (register security info)
→ Authentication context (high sensitivity)

Example policy use:
  IF accessing Azure Portal (high sensitivity)
  THEN require stronger MFA than Teams access

SIGNAL 5 — SIGN-IN RISK (Entra ID P2)
─────────────────────────────────────────────────────────
What is the AI risk score for this sign-in?
→ Low: Normal behaviour pattern
→ Medium: Suspicious but not confirmed
→ High: Strong indicators of compromise

Risk detections that raise the score:
→ Anonymous IP (Tor, VPN)
→ Impossible travel (India at 9 AM, Russia at 9:05 AM)
→ Leaked credentials (found on dark web)
→ Password spray detected
→ Malware-linked IP address
→ Unfamiliar sign-in properties

Example policy use:
  IF sign-in risk = HIGH
  THEN block completely — no exceptions
```

### CA Policy Structure — Anatomy of Every Policy

```
EVERY CA POLICY HAS EXACTLY TWO PARTS:

PART 1 — ASSIGNMENTS (the conditions):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1a. Users and Groups
    → Who does this policy apply to?
    → Include: All users / specific groups / specific roles
    → Exclude: Break-glass accounts (ALWAYS EXCLUDE!)

1b. Target Resources
    → Which apps does this cover?
    → All cloud apps / specific apps / user actions

1c. Conditions (optional filters to narrow scope)
    → Sign-in risk: Low / Medium / High
    → Device platforms: Windows, iOS, Android, Linux
    → Locations: Any / Named locations / Countries
    → Client apps: Browser / Mobile / Legacy protocols
    → Filter for devices: Compliant / Managed / Joined

PART 2 — ACCESS CONTROLS (what to enforce):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2a. Grant Controls
    → Block access (deny completely)
    → Require multi-factor authentication
    → Require compliant device (Intune)
    → Require hybrid Azure AD joined device
    → Require approved client app
    → Multiple controls: require ALL or require ONE

2b. Session Controls (advanced)
    → Sign-in frequency: force re-auth every N hours
    → Persistent browser session: allow or prevent
    → Application enforced restrictions

THE FORMULA:
IF [Assignments are true] THEN [Apply Access Controls]
```

---

## 📌 SECTION 2 of 5
## SecureCorp's 5 CA Policies — Built From Scratch

---

### POLICY 1 — CA001: Block Legacy Authentication

**This is the single most important CA policy you will ever create.**

```
WHY LEGACY AUTH IS DANGEROUS:
─────────────────────────────────────────────────────────
Legacy protocols (IMAP, POP3, SMTP, Exchange ActiveSync)
were built before MFA existed.

They do NOT support MFA.
They only support username + password.

So even if you enable MFA for every user in Entra ID —
an attacker can use IMAP to authenticate as Gani
and MFA is COMPLETELY SKIPPED.

Your entire MFA investment = bypassed.

Microsoft data: Blocking legacy auth stops
67% of identity-based attacks.

MODERN AUTH FLOW (CA policies apply):
─────────────────────────────────────────────────────────
Browser/Teams app → Password → MFA prompt → CA evaluated
→ All policies checked → Access granted or denied

LEGACY AUTH FLOW (CA policies bypassed):
─────────────────────────────────────────────────────────
IMAP client → Password → DONE
MFA = not triggered
CA policies = not evaluated
Attacker = authenticated

CA001 intercepts the legacy auth request and BLOCKS it
before it even reaches the MFA stage.
```

**Portal Steps — Create CA001:**

```
Entra ID Portal → Security → Conditional Access → + New Policy

Name: CA001-Block-Legacy-Authentication

STEP 1 — Users:
  Include: All users
  Exclude: bkglass-emergency@securecorp.com
           (CRITICAL — never lock out your emergency account!)

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions → Client apps:
  Configure: YES (toggle on)
  ✅ Exchange ActiveSync clients
  ✅ Other clients
  (Leave Browser, Mobile apps unchecked — those support MFA)

STEP 4 — Grant:
  Select: Block access

STEP 5 — Enable policy:
  ⚠️  Set to REPORT-ONLY first
  Run for 2 weeks → check sign-in logs
  Fix any legitimate legacy apps
  THEN switch to ON
```

**Before enabling — find who uses legacy auth:**

```kql
SigninLogs
| where TimeGenerated > ago(30d)
| where ClientAppUsed in (
    "Exchange ActiveSync",
    "IMAP4",
    "POP3",
    "SMTP Auth",
    "Other clients"
  )
| where ResultType == "0"
| summarize Count=count(), LastSeen=max(TimeGenerated)
    by UserPrincipalName, ClientAppUsed
| sort by Count desc

// Fix these users' apps BEFORE enabling CA001
// Otherwise they lose access on day 1
```

**What Navi sees in logs after CA001 is enabled:**

```
Sign-in log entry — attacker attempt blocked:
  User:       gani@securecorp.com
  Client app: IMAP4              ← legacy protocol
  Status:     Failure
  Error:      53003              ← Blocked by CA policy
  CA001:      Applied → BLOCKED  ← policy working ✅
  IP:         185.220.101.45     ← attacker's IP
  Location:   Netherlands
  Time:       2:17 AM

Navi reads:
  → Attacker had Gani's password
  → Tried to bypass MFA via IMAP
  → CA001 blocked it before MFA stage
  → Action: Force Gani to change password
            Investigate how attacker got his credentials
```

---

### POLICY 2 — CA002: Require MFA for All Users

```
WHY THIS EXISTS:
─────────────────────────────────────────────────────────
Without this policy, MFA is optional or per-user.
Per-user MFA = someone always slips through the gaps.

With CA002 = MFA is mandatory for EVERY user
on EVERY app on EVERY sign-in. No exceptions.

An attacker who successfully sprays Gani's password
still cannot log in without his phone.
The password becomes worthless.
```

**Portal Steps — Create CA002:**

```
Name: CA002-Require-MFA-All-Users

STEP 1 — Users:
  Include: All users
  Exclude: bkglass-emergency@securecorp.com

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions:
  None (applies always, to everything)

STEP 4 — Grant:
  ✅ Require multi-factor authentication

STEP 5 — Enable policy:
  Report-only first
  BEFORE enabling — make sure all users have MFA registered
  Otherwise they get locked out immediately

  Check who has no MFA registered:
  Entra ID → Security → MFA → User registration details
  Filter: Authentication methods registered = 0
  Register MFA for these users FIRST
```

**Enable Number Matching BEFORE CA002 goes live:**

```
WHY NUMBER MATCHING MATTERS:
─────────────────────────────────────────────────────────
Without number matching:
  MFA push shows: [APPROVE] [DENY]
  Attacker triggers 47 push notifications
  Frustrated user hits APPROVE at midnight
  = MFA fatigue attack succeeds (Uber 2022 breach method)

With number matching:
  MFA push shows: "Enter this number: 47"
  User must type 47 in Authenticator app
  Attacker's sign-in page shows a DIFFERENT number
  User sees mismatch → DENIES
  = MFA fatigue attack impossible

ENABLE IT:
  Entra ID → Security → MFA → Additional settings
  → Number matching: Enabled
  → Additional context (show app + location): Enabled
  → Limit authentication methods (disable SMS if possible)
```

**MFA Method Strength — What Hareesh should enforce:**

```
STRONGEST — Phishing-resistant (recommended for admins):
  FIDO2 Security Keys (YubiKey)
  → Physical key required — impossible to steal remotely
  → Cannot be phished (bound to domain)
  → Use for: Hareesh, all Global Admins

  Windows Hello for Business
  → Biometric (face/fingerprint) tied to device
  → Phishing-resistant by design
  → Use for: All Intune-managed devices

STRONG — Acceptable (with number matching):
  Microsoft Authenticator app (push with number match)
  → Good protection once number matching is enabled
  → Vulnerable to fatigue without number matching
  → Use for: All standard users — Priya, Gani, Chaitu etc.

WEAK — Avoid if possible:
  SMS OTP (text message)
  → Vulnerable to SIM swap attacks
  → Carrier can be social engineered
  → Use only as fallback for users without smartphones
  → Never for admin accounts

NEVER USE:
  Voice call OTP
  → Same risks as SMS, slower
  → Disable this method in MFA settings
```

---

### POLICY 3 — CA003: Admins Need MFA + Compliant Device

```
WHY TWO CONTROLS INSTEAD OF JUST MFA:
─────────────────────────────────────────────────────────
CA002 requires MFA for everyone including admins.
But admins have much higher risk.

Scenario: Attacker performs AiTM attack on Hareesh.
  → Steals session token AFTER MFA completed
  → Has valid token → accesses Azure Portal
  → CA002 is satisfied (MFA was done — by Hareesh)
  → But attacker is accessing from their laptop
  → That laptop is NOT enrolled in Intune
  → NOT compliant with SecureCorp policies

CA003 adds: Device must ALSO be compliant.
  Stolen token on unmanaged device = BLOCKED.
  Attacker needs both Hareesh's MFA AND his exact device.
  Much harder attack surface.
```

**Portal Steps — Create CA003:**

```
Name: CA003-Admins-Compliant-Device-MFA

STEP 1 — Users:
  Include: Directory roles:
    ✅ Global Administrator
    ✅ Privileged Role Administrator
    ✅ Security Administrator
    ✅ Exchange Administrator
    ✅ SharePoint Administrator
  Exclude: bkglass-emergency@securecorp.com

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions:
  None

STEP 4 — Grant:
  ✅ Require multi-factor authentication
  ✅ Require device to be marked as compliant
  Select: Require ALL the selected controls
  (BOTH must be satisfied — not just one)

STEP 5 — Enable policy:
  Test with Hareesh's account first
  Verify his laptop is Intune-enrolled and compliant
  Entra ID → Devices → [Hareesh's device] → Compliant = Yes
  Then enable for all admin roles
```

**What compliance means in Intune:**

```
INTUNE COMPLIANCE POLICY — SECURECORP BASELINE:
─────────────────────────────────────────────────────────
For a device to be "compliant" it must have:
  ✅ BitLocker encryption enabled
  ✅ Secure Boot enabled
  ✅ Windows Defender Antivirus running
  ✅ Latest critical patches installed (within 30 days)
  ✅ Minimum OS version (Windows 11 22H2+)
  ✅ No jailbreak/root detected
  ✅ PIN/password screen lock configured
  ✅ Firewall enabled

Non-compliant device = CA003 blocks admin access
Even with valid credentials and completed MFA.
```

---

### POLICY 4 — CA004: Block High-Risk Sign-ins

```
WHY LET AI DO THE HEAVY LIFTING:
─────────────────────────────────────────────────────────
Microsoft processes billions of sign-ins daily
across all Azure AD tenants worldwide.

Their AI models see patterns that no human SOC
analyst could detect manually:
→ IP addresses associated with attack campaigns
→ Credential databases leaked on dark web
→ Timing patterns matching known attack tools
→ Browser fingerprints matching malware C2

When Microsoft's AI says risk = HIGH —
the false positive rate is extremely low.
Block it. No questions asked.

This requires Entra ID P2 licence.
```

**Risk Levels and What Triggers Them:**

```
LOW RISK — Monitor but allow:
  Unfamiliar sign-in properties (new browser/location)
  First time signing in from this country
  IP not seen before for this user
  → CA action: Allow (or require MFA as step-up)

MEDIUM RISK — Step up authentication:
  Anonymous IP (commercial VPN detected)
  Atypical travel (unusual but possible)
  Malware-linked IP (IP in threat intel feeds)
  → CA action: Require MFA + password change

HIGH RISK — Block immediately:
  Tor exit node (anonymization = attacker pattern)
  Leaked credentials (found in breach database)
  Password spray detected (AI confirmed attack)
  Token issuer anomaly (AiTM fingerprint)
  → CA action: Block access completely
```

**Portal Steps — Create CA004:**

```
Name: CA004-Block-High-Risk-SignIn

STEP 1 — Users:
  Include: All users
  Exclude: bkglass-emergency@securecorp.com

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions → Sign-in risk:
  Configure: YES
  ✅ High

STEP 4 — Grant:
  Select: Block access

STEP 5 — Enable policy:
  Can go ON directly (low false positive rate for HIGH risk)
  But start Report-only for 1 week to validate

BONUS — User Risk Policy:
  CA policy for USER risk (not just sign-in risk):
  IF user risk = HIGH (leaked credentials confirmed)
  THEN require password change + MFA
  This forces remediation when Gani's creds found on dark web
```

---

### POLICY 5 — CA005: Block Sign-ins Outside India

```
WHY LOCATION-BASED BLOCKING MATTERS:
─────────────────────────────────────────────────────────
SecureCorp is an India-based organization.
99% of legitimate sign-ins come from India.

By blocking all non-India sign-ins:
→ Eliminates the entire global attacker pool
   except those routing through Indian IPs
→ Significantly reduces attack surface
→ Easy to implement, high impact

Exceptions needed:
→ Hareesh (travels internationally for work)
→ Navi (SOC may work remotely from other locations)
→ Any user who genuinely needs international access
```

**Portal Steps — Create Named Location First:**

```
STEP 0 — Create Named Location "India":
  Entra ID → Security → Conditional Access → Named locations
  → + New location → Countries location
  Name: "SecureCorp-Trusted-India"
  Countries: ✅ India
  Include unknown countries/regions: NO
  → Create

THEN CREATE CA005:
  Name: CA005-Block-Outside-India

STEP 1 — Users:
  Include: All users
  Exclude:
    → bkglass-emergency@securecorp.com
    → hareesh@securecorp.com (international travel)
    → navi@securecorp.com (remote SOC work)
    → Any other legitimate international users

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions → Locations:
  Configure: YES
  Include: Any location
  Exclude: SecureCorp-Trusted-India

STEP 4 — Grant:
  Select: Block access

STEP 5 — Enable policy:
  Report-only first — 2 weeks
  Review: any legitimate non-India logins?
  Add those users to exclusion list
  Then enable ON
```

---

## 📌 SECTION 3 of 5
## The Golden Rules — What Every Security Engineer Must Know

---

### Rule 1 — Always Use Report-only Mode First

```
WHAT IS REPORT-ONLY MODE:
─────────────────────────────────────────────────────────
The policy evaluates every sign-in.
Logs what WOULD have happened.
But does NOT actually enforce anything.
Zero user impact.

WHY THIS MATTERS:
  You create CA001 to block legacy auth.
  You go straight to ON.
  Priya's email client uses SMTP authentication.
  Priya can no longer send email.
  She calls Jayanth (helpdesk).
  Jayanth calls Hareesh.
  Hareesh disables CA001 "temporarily".
  "Temporarily" becomes permanent.
  Attack surface stays open.

CORRECT PROCESS:
  Enable Report-only → run 2 weeks
  Check sign-in logs → CA tab → "Would have blocked"
  Find every user/app that would be affected
  Fix those apps and users
  THEN switch to ON
  Zero disruption. Full confidence.
```

### Rule 2 — Always Exclude Break-Glass Accounts

```
WHAT IS A BREAK-GLASS ACCOUNT:
─────────────────────────────────────────────────────────
An emergency Global Admin account with NO MFA.
Used ONLY when all other admin access is broken.
Password is 60+ characters random.
Stored in a physical sealed envelope in the CISO's safe.
Never used for daily work. Ever.

WHY IT MUST BE EXCLUDED FROM ALL CA POLICIES:
  Scenario: Hareesh enables a new CA policy.
  The policy has a bug — blocks ALL logins.
  Hareesh is now locked out.
  His regular admin account = also locked out.
  His MFA device = not helping (policy blocks before MFA).
  Break-glass = the only way to log in and fix the policy.

  Without break-glass exclusion:
  One bad policy = no admin access = call Microsoft support
  = hours or days to recover = massive business impact.

SECURECORP BREAK-GLASS SETUP:
  Account:  bkglass-emergency@securecorp.com
  MFA:      NONE (intentionally — excluded from all CA)
  Password: 60-character random string
  Location: Printed, sealed, in CISO physical safe
  Monitor:  ANY login from this account = P0 alert
  Review:   Password rotated annually by CISO
  Use:      ONLY when all other admin access is unavailable
```

### Rule 3 — Test One User Before All Users

```
PROCESS:
  Create a dedicated test account: ca-test@securecorp.com
  Apply new policy ONLY to this account first.
  Sign in as ca-test from various scenarios:
    → Normal browser login
    → Mobile app login
    → Legacy client (if relevant)
    → Different location (use VPN to test)
  Check sign-in logs → all 5 tabs.
  Confirm policy behaves exactly as expected.
  Then change scope to All users.
```

### Rule 4 — Monitor CA Tab in Sign-in Logs After Every Change

```
WHERE: Entra ID → Sign-in logs → Any entry → CA tab

WHAT TO LOOK FOR AFTER ENABLING A NEW POLICY:

Healthy state:
  CA001: Not applied (modern auth — correct)
  CA002: Applied → Satisfied ✅
  CA003: Applied → Satisfied ✅ (for admin accounts)
  CA004: Applied → Not blocked (risk was low) ✅

Warning signs:
  CA002: Not applied  ← MFA not required → INVESTIGATE
  Why? User excluded? Guest account? Policy scope wrong?

  CA001: Applied → Blocked  ← Legacy auth stopped ✅
  But also check: Is this a legitimate user being blocked?
  If yes → fix their app, don't disable the policy

  CA003: Failure  ← Admin's device not compliant
  Action: Fix device compliance in Intune
```

### Rule 5 — Document Every Policy

```
FOR EACH CA POLICY IN SECURECORP, DOCUMENT:
─────────────────────────────────────────────────────────
Policy name:     CA001-Block-Legacy-Authentication
Purpose:         Block legacy authentication protocols
                 that cannot support MFA
Attacks stopped: MFA bypass via IMAP/POP3/SMTP
Owner:           Hareesh (IT Admin)
Created:         01/04/2026
Last reviewed:   01/04/2026
Last changed:    01/04/2026
Exclusions:      bkglass-emergency@securecorp.com
                 (emergency access — no legitimate
                  use of legacy auth needed)
Known exceptions: None currently
Review date:     01/07/2026 (quarterly)
─────────────────────────────────────────────────────────

WHY DOCUMENTATION SAVES LIVES:
  6 months later — new IT admin joins SecureCorp.
  They see CA001 blocking something.
  No documentation = they disable it "temporarily".
  "Temporarily" = permanently.
  Attack surface reopens.

  With documentation:
  They understand why it exists.
  They find the right fix.
  Policy stays intact.
```

---

## 📌 SECTION 4 of 5
## CA Policies in Action — Navi Investigates

---

### Reading the CA Tab — Three Scenarios

```
SCENARIO 1 — Gani's clean login (Tuesday 9 AM):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Sign-in log → CA tab:
Policy                     Result         Reason
─────────────────────────────────────────────────────
CA001-Block-Legacy         Not applied    Modern auth used
CA002-Require-MFA          Satisfied ✅   MFA completed
CA003-Admin-Compliant      Not applied    Gani not an admin
CA004-Block-High-Risk      Satisfied ✅   Risk = Low
CA005-Block-Outside-India  Not applied    Bangalore IP

Navi reads: All policies behaving correctly.
            MFA was required and completed.
            Clean login. No action needed. ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SCENARIO 2 — Attacker using Gani's stolen password
             via IMAP from Netherlands at 2 AM:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Sign-in log → CA tab:
Policy                     Result         Reason
─────────────────────────────────────────────────────
CA001-Block-Legacy         FAILURE 🚨     BLOCKED
CA002-Require-MFA          Not reached    CA001 blocked first
CA004-Block-High-Risk      Not reached    CA001 blocked first

Sign-in status: Failure
Error code:     53003 (blocked by CA)

Navi reads:
  → Attacker had Gani's correct password
  → Tried IMAP to bypass MFA
  → CA001 blocked before MFA was reached
  → Policy working perfectly ✅

  BUT: Attacker has Gani's password.
  Immediate action:
  → Force Gani password reset
  → Check how password was obtained
  → Review last 30 days of Gani's activity

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SCENARIO 3 — CA policy GAP discovered:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Sign-in log → CA tab:
Policy                     Result         Reason
─────────────────────────────────────────────────────
CA001-Block-Legacy         Not applied    Modern auth used
CA002-Require-MFA          Not applied ← 🚨 WHY?
CA003-Admin-Compliant      Not applied
CA004-Block-High-Risk      Not applied

Authentication requirement: singleFactorAuthentication 🚨
MFA result:                Not required   🚨
Status:                    Success

Navi reads:
  → Successful modern auth login with NO MFA
  → CA002 should have required MFA but did NOT apply
  → Something excluded this user from CA002

Navi investigates CA002 settings:
  Found: Gani was excluded from CA002 last month
         (helpdesk ticket — MFA app issue)
         Issue resolved but exclusion never removed

  Action:
  → Add Gani back to CA002 immediately
  → Check if this sign-in was legitimate or attacker
  → Review other accounts excluded from CA002
  → Quarterly review of all CA exclusions → added to process
```

---

## 📌 SECTION 5 of 5
## Complete CA Policy Reference for SecureCorp

---

```
SECURECORP CA POLICIES — MASTER REFERENCE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

POLICY: CA001-Block-Legacy-Authentication
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Users:       All users
Exclude:     Break-glass
Apps:        All cloud apps
Conditions:  Client apps = EAS + Other clients
Control:     Block access
Stops:       MFA bypass via IMAP/POP3/SMTP/EAS
Priority:    Highest — implement first

POLICY: CA002-Require-MFA-All-Users
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Users:       All users
Exclude:     Break-glass
Apps:        All cloud apps
Conditions:  None
Control:     Require MFA
Pre-req:     Enable number matching first
             Ensure all users have MFA registered
Stops:       Password spray succeeding without MFA

POLICY: CA003-Admins-Compliant-Device-MFA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Users:       Global Admin + Privileged Role Admin
             + Security Admin + Exchange Admin
Exclude:     Break-glass
Apps:        All cloud apps
Conditions:  None
Control:     Require MFA AND compliant device (both)
Pre-req:     Admin devices enrolled in Intune
             Compliance policy configured in Intune
Stops:       AiTM token theft + stolen creds on unmanaged device

POLICY: CA004-Block-High-Risk-SignIn
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Users:       All users
Exclude:     Break-glass
Apps:        All cloud apps
Conditions:  Sign-in risk = High
Control:     Block access
Pre-req:     Entra ID P2 licence
Stops:       Tor logins, leaked credential use,
             spray attacks, AiTM sign-ins

POLICY: CA005-Block-Outside-India
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Users:       All users
Exclude:     Break-glass, Hareesh, Navi
             (legitimate international users)
Apps:        All cloud apps
Conditions:  Location NOT = Named location "India"
Control:     Block access
Pre-req:     Create Named Location for India first
Stops:       All global attackers using non-Indian IPs

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IMPLEMENTATION ORDER (deploy in this sequence):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Week 1:  CA001 Report-only → find legacy auth users
         Fix legacy apps → CA001 ON

Week 2:  CA002 Report-only → ensure all users have MFA
         Enable number matching → CA002 ON

Week 3:  CA003 Report-only → ensure admin devices compliant
         Fix compliance gaps → CA003 ON

Week 4:  CA004 Report-only → review HIGH risk detections
         Validate false positive rate → CA004 ON

Week 5:  CA005 Report-only → identify international users
         Add to exclusion list → CA005 ON

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GOLDEN RULES SUMMARY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
① Report-only first — always, no exceptions
② Exclude break-glass from every single policy
③ Test one user before all users
④ Monitor CA tab in sign-in logs after every change
⑤ Document every policy with owner and review date
```

---

## 🧪 DAY 7 LABS

```
LAB 1 — Create CA001 in Report-only (20 min)
Platform: Entra ID Portal → Security → Conditional Access
──────────────────────────────────────────────────────────
1. Create CA001-Block-Legacy-Auth (steps in Section 2)
2. Set to: Report-only mode only
3. Sign in with your test account
4. Check sign-in logs → CA tab
   → Does it show "Would have blocked legacy auth"?
5. Note which applications would have been affected

LAB 2 — Audit Legacy Auth Users Before CA001 (15 min)
Platform: Entra ID → Sign-in logs → KQL
──────────────────────────────────────────────────────────
Run the KQL query from Section 2 (CA001 section)
→ Find all users with successful legacy auth in last 30 days
→ This is your CA001 blast radius
→ List the apps they are using
→ Research how to configure those apps for modern auth

LAB 3 — Check CA tab on your sign-ins (15 min)
Platform: Entra ID → Monitoring → Sign-in logs
──────────────────────────────────────────────────────────
Find your own last 5 sign-ins
For each one open ALL tabs and answer:
→ CA tab: Which policies applied?
→ CA tab: Was any policy NOT applied that should be?
→ Auth Details: Was MFA required? What method?
→ Device: Is device compliant?

LAB 4 — Microsoft Learn Free CA Lab (45 min)
Platform: learn.microsoft.com (FREE, sandbox included)
──────────────────────────────────────────────────────────
Search: "SC-300 implement conditional access policies"
→ Free guided lab with real Entra ID sandbox
→ Create CA policies in a test tenant
→ Test sign-ins and observe CA enforcement
→ No Azure subscription needed
```

---

## 📋 DAY 7 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║        DAY 7 — CONDITIONAL ACCESS POLICIES               ║
║                 SecureCorp Ltd.                          ║
╚══════════════════════════════════════════════════════════╝

WHAT IS CONDITIONAL ACCESS:
─────────────────────────────────────────────────────────
Policy engine: IF conditions THEN controls
Evaluates 5 signals on every single sign-in
Replaces password-only access model
The new perimeter = identity + context

5 SIGNALS EVALUATED:
─────────────────────────────────────────────────────────
1. User identity   → Who? Role? Group? Member/Guest?
2. Location        → Named location? Country? Trusted IP?
3. Device          → Managed? Compliant? Platform?
4. Application     → Teams vs Azure Portal vs all apps
5. Sign-in risk    → AI score: Low/Medium/High (P2)

POLICY ANATOMY:
─────────────────────────────────────────────────────────
Part 1 — Assignments (conditions):
  Users & Groups → who it applies to
  Target Resources → which apps
  Conditions → risk, location, device, client app

Part 2 — Access Controls (enforcement):
  Grant → Block / MFA / Compliant device / combination
  Session → frequency, persistent session

SECURECORP'S 5 POLICIES:
─────────────────────────────────────────────────────────
CA001 — Block Legacy Authentication
  Stops: MFA bypass via IMAP/POP3/SMTP/EAS
  Config: All users → All apps →
          Client app = EAS + Other → Block
  Impact: Stops 67% of identity attacks

CA002 — Require MFA All Users
  Stops: Password spray succeeding
  Config: All users → All apps → No conditions → Require MFA
  Pre-req: Number matching enabled first

CA003 — Admins MFA + Compliant Device
  Stops: AiTM token theft on unmanaged devices
  Config: Admin roles → All apps →
          Require MFA AND compliant device (both)

CA004 — Block High Risk (P2 required)
  Stops: Tor, leaked creds, spray, AiTM fingerprints
  Config: All users → All apps →
          Risk = High → Block

CA005 — Block Outside India
  Stops: Global attackers using non-Indian IPs
  Config: Most users → All apps →
          Location NOT India → Block

IMPLEMENTATION ORDER:
─────────────────────────────────────────────────────────
Week 1: CA001 → Week 2: CA002 → Week 3: CA003
Week 4: CA004 → Week 5: CA005
Always Report-only first for 2 weeks each

GOLDEN RULES:
─────────────────────────────────────────────────────────
① Report-only first — always, no exceptions
② Exclude break-glass from EVERY policy
③ Test one user before all users
④ Monitor CA tab in sign-in logs after changes
⑤ Document: name, purpose, owner, review date

BREAK-GLASS ACCOUNT:
─────────────────────────────────────────────────────────
Account: bkglass-emergency@securecorp.com
MFA: None — excluded from ALL policies
Password: 60-char random — physical safe — CISO only
Monitor: Any use = immediate P0 alert
Use: ONLY when all admin access is broken

MFA METHOD STRENGTH:
─────────────────────────────────────────────────────────
FIDO2 key         → Phishing-resistant — best for admins
Windows Hello     → Phishing-resistant — Intune devices
Authenticator app → Good — MUST enable number matching
SMS OTP           → Weak — SIM swap risk — avoid for admins

CA TAB READING:
─────────────────────────────────────────────────────────
Policy not applied when it should be → CA gap → fix now
Error 53003 → Blocked by CA → policy working ✅
MFA not required on modern auth → scope/exclusion issue
All policies satisfied → clean login ✅

WHAT EACH POLICY STOPS:
─────────────────────────────────────────────────────────
CA001 → Attacker using IMAP with stolen password
CA002 → Attacker with correct password but no phone
CA003 → AiTM stolen token replayed on attacker's laptop
CA004 → Attacker routing through Tor or using leaked creds
CA005 → All global attackers outside India

LABS COMPLETED:
─────────────────────────────────────────────────────────
□ Created CA001 in Report-only mode
□ Audited legacy auth users via KQL
□ Checked CA tab on own sign-in logs
□ Microsoft Learn SC-300 CA lab completed
```

---

> 💡 **Architect's Truth:** *"If I had to pick just two CA policies for any organization — CA001 and CA002. Block legacy auth and require MFA. Those two together, properly configured with number matching, stop the overwhelming majority of cloud identity attacks. Everything else is defence in depth. But these two are the foundation. Get these right first."*

---

**Day 8 is next — Entra ID Logs & SOC Monitoring.** 
