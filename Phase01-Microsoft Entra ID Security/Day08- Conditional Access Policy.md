Perfect! Blue/Navy SecureCorp brand + summary image + inline diagrams. Let me build this properly. 🎯

First let me read the design skill to make sure diagrams are top quality 👇Now let me build the complete Day 7 — Conditional Access Policies with full diagrams. This will take a moment to craft properly. 🎨Now I have everything I need. Let me build this beautifully — summary image + inline concept diagrams throughout. 🎨There's your **full Day 7 summary image** — copy this to GitHub as the header for Day 7. Now let's go section by section with inline diagrams. 👇

---

# 🔵 PHASE 2 — DAY 7
# Conditional Access Policies — From Scratch
### *SecureCorp Ltd. — Building Real Policies Step by Step*

---

## 📌 SECTION 1 of 5
## What is Conditional Access — The Complete Mental Model

---

The simplest way to understand Conditional Access:

> Old security: **Username + Password = Access**
> Modern security: **Who + Where + What + How + Risk = Decision**

Think of it like a **smart security guard** at SecureCorp's entrance who doesn't just check your ID card — he asks five questions before letting you in.The top flow shows Gani's clean login — all signals green, access allowed. The bottom shows the attacker using Gani's stolen credentials via Tor at 2 AM — CA001 catches it and blocks before any damage is done.

---

## 📌 SECTION 2 of 5
## The 5 Building Blocks of Every CA Policy

---

Every CA policy has the same structure — **Assignments** (who/what/when) + **Access controls** (what to do). Let's build this understanding before writing any policy.

```
POLICY ANATOMY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PART 1 — ASSIGNMENTS (conditions that must be true)
─────────────────────────────────────────────────────
1a. USERS & GROUPS
    WHO does this policy apply to?
    Options:
    → All users
    → Specific groups (e.g. "Domain Admins")
    → Specific users (e.g. only Hareesh)
    → Exclude: Break-glass accounts (ALWAYS!)

1b. TARGET RESOURCES
    WHICH apps does this cover?
    Options:
    → All cloud apps
    → Specific apps (Teams, SharePoint, Azure Portal)
    → User actions (register security info, join device)

1c. CONDITIONS (optional filters)
    WHEN does this policy apply?
    Options:
    → Sign-in risk: Low / Medium / High
    → Device platforms: Windows / iOS / Android
    → Locations: Trusted / Untrusted / Specific countries
    → Client apps: Browser / Mobile / Legacy protocols
    → Filter for devices: Compliant / Managed

PART 2 — ACCESS CONTROLS (what to enforce)
─────────────────────────────────────────────────────
2a. GRANT CONTROLS
    What must the user satisfy?
    Options:
    → Block access (deny completely)
    → Require MFA
    → Require compliant device
    → Require hybrid Azure AD joined device
    → Require approved client app
    → Require app protection policy
    → Multiple controls: require ALL or require ONE

2b. SESSION CONTROLS (advanced)
    How long is the session valid?
    → Sign-in frequency: re-authenticate every N hours
    → Persistent browser session: allow or prevent
    → Application enforced restrictions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 SECTION 3 of 5
## SecureCorp's 5 CA Policies — Built From Scratch

---

Now let's build each policy with full reasoning. I'll show you what Hareesh configures and why — then what Navi sees in logs.

### POLICY 1 — CA001: Block Legacy Authentication

This is the **single highest-impact CA policy** you can create. Microsoft reports it stops 67% of identity attacks.**Step by step — how Hareesh creates CA001 in the portal:**

```
Entra ID Portal → Security → Conditional Access → + New Policy

Name: CA001-Block-Legacy-Authentication

STEP 1 — Users:
  Include: All users
  Exclude: BreakGlass-Emergency@securecorp.com
           (CRITICAL — never lock out your emergency account)

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions → Client apps:
  Configure: YES (toggle on)
  Check ONLY these two boxes:
  ✅ Exchange ActiveSync clients
  ✅ Other clients
  (Leave Browser / Mobile apps unchecked — those support MFA)

STEP 4 — Grant:
  Select: Block access

STEP 5 — Enable policy:
  ⚠️  Set to REPORT-ONLY first (not ON)
  → Run for 2 weeks
  → Check sign-in logs for impact
  → Fix any legitimate legacy apps
  → THEN switch to ON
```

---

### POLICY 2 — CA002: Require MFA for All Users```
PORTAL STEPS — CA002:

Name: CA002-Require-MFA-All-Users

STEP 1 — Users:
  Include: All users
  Exclude: BreakGlass-Emergency@securecorp.com

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions:
  None (applies always, to everything)

STEP 4 — Grant:
  ✅ Require multi-factor authentication

STEP 5 — Enable policy:
  Report-only first → monitor 2 weeks → then ON

IMPORTANT — Enable Number Matching BEFORE enabling CA002:
  Entra ID → Security → MFA → Additional settings
  → Number matching: Enabled
  → Additional context: Enabled
  → This prevents MFA fatigue attacks
```

---

### POLICY 3 — CA003: Admins Need MFA + Compliant Device

This one protects Hareesh specifically. Even if his password AND MFA are compromised, attacker still can't access the Azure Portal without his managed device.

```
PORTAL STEPS — CA003:

Name: CA003-Admins-Compliant-Device-MFA

STEP 1 — Users:
  Include: Directory roles:
    ✅ Global Administrator
    ✅ Privileged Role Administrator
    ✅ Security Administrator
    ✅ Exchange Administrator
  Exclude: BreakGlass-Emergency@securecorp.com

STEP 2 — Target resources:
  Include: All cloud apps

STEP 3 — Conditions:
  None

STEP 4 — Grant:
  ✅ Require MFA
  ✅ Require compliant device
  Require: ALL selected controls
  (Both must be satisfied — not just one)

STEP 5 — Enable policy:
  Test with one admin first
  Verify Intune compliance before enabling
```

---

### POLICY 4 — CA004: Block High-Risk Sign-Ins

This is Entra ID's AI doing the heavy lifting for Navi. Requires P2 licence.```
PORTAL STEPS — CA004:

Name: CA004-Block-High-Risk-SignIn

STEP 1 — Users: All users
STEP 2 — Apps: All cloud apps
STEP 3 — Conditions → Sign-in risk:
  Configure: YES
  Select: High
STEP 4 — Grant: Block access

BONUS — User risk policy (separate from sign-in risk):
  When USER risk = High (leaked credentials detected):
  Grant: Require password change + MFA
  This forces remediation when dark web leak found
```

---

### POLICY 5 — CA005: Block Outside India

```
PORTAL STEPS — CA005:

Name: CA005-Block-Outside-India

STEP 1 — Users:
  Include: All users
  Exclude:
    → BreakGlass-Emergency@securecorp.com
    → Hareesh (travels internationally)
    → Navi (SOC — may work remotely)

STEP 2 — Apps: All cloud apps

STEP 3 — Conditions → Locations:
  Configure: YES
  Include: Any location
  Exclude: Named location "India"
     (Create Named Location first:
      Entra ID → Security → CA → Named locations
      → + New location → Countries → India)

STEP 4 — Grant: Block access

RESULT: Anyone not in India = blocked
        Attackers from Russia, Netherlands, China = blocked
        Reduces attack surface to Indian IPs only
```

---

## 📌 SECTION 4 of 5
## The Critical Rules — What Every SOC Analyst Must Know---

## 📌 SECTION 5 of 5
## What Navi Sees — CA in Sign-in Logs

---

When Navi investigates any sign-in, the **Conditional Access tab** tells the complete story. Here's what he looks for:

```
SIGN-IN LOG → CONDITIONAL ACCESS TAB:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SCENARIO 1 — Clean login (Gani, normal day):
Policy Name              Result     Controls Applied
─────────────────────────────────────────────────────
CA001-Block-Legacy       Not applied  (modern auth used)
CA002-Require-MFA        Success      MFA satisfied
CA003-Admin-Device       Not applied  (Gani not admin)
CA004-Block-High-Risk    Success      Risk = Low, allowed
CA005-Block-Outside-IN   Not applied  (Bangalore IP)

Navi reads: All green → legitimate login ✅

SCENARIO 2 — Suspicious login (Gani, Netherlands, IMAP):
Policy Name              Result     Controls Applied
─────────────────────────────────────────────────────
CA001-Block-Legacy       Failure    BLOCKED ← stops here
CA002-Require-MFA        Not reached (CA001 already blocked)
CA004-Block-High-Risk    Not reached (CA001 already blocked)

Navi reads: CA001 blocked → legacy auth = attacker tool
            Force Gani password reset immediately

SCENARIO 3 — Policy GAP found:
Policy Name              Result
─────────────────────────────────────────────────────
CA001-Block-Legacy       Not applied (modern auth)
CA002-Require-MFA        Not applied  ← 🚨 WHY?
CA003-Admin-Device       Not applied
CA004-Block-High-Risk    Not applied

Status: Success — single factor only
Navi reads: MFA policy didn't apply → find why
            → Was this account excluded? Was it a guest?
            → CA gap = risk = fix it NOW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 🧪 DAY 7 LABS

```
LAB 1 — Create CA001 in Report-only (20 min)
Platform: Entra ID Portal → Security → Conditional Access
────────────────────────────────────────────────────────
1. Create CA001-Block-Legacy-Auth (steps above)
2. Set to: Report-only mode
3. Sign in with your test account
4. Check sign-in logs → CA tab
   → Does it show "Would have blocked"?
5. Screenshot the CA tab result — paste here!

LAB 2 — Find Your Legacy Auth Users (15 min)
Platform: Entra ID → Sign-in logs
────────────────────────────────────────────────────────
Filter: Client app = "Exchange ActiveSync" OR "Other clients"
→ Last 30 days
→ Status = Success (these bypassed MFA!)
→ Who are they? What apps are they using?
→ This is your CA001 blast radius before enabling

LAB 3 — Check CA tab on real sign-ins (15 min)
Platform: Entra ID → Sign-in logs
────────────────────────────────────────────────────────
Click any of your recent sign-ins
→ Open Conditional Access tab
→ Which policies applied?
→ Which were skipped (and why)?
→ Was MFA required? What method?

LAB 4 — Microsoft Learn: Implement CA (45 min)
Platform: learn.microsoft.com (FREE — sandbox included)
────────────────────────────────────────────────────────
Search: "SC-300 implement conditional access"
→ Free guided lab with Entra ID sandbox
→ Create real CA policies in a test tenant
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
Smart access policy engine:
IF (conditions about user/location/device/app/risk)
THEN (allow / require MFA+device / block)

Replaces simple password-only access decisions.
Evaluates 5 signals on EVERY sign-in.

5 SIGNALS:
─────────────────────────────────────────────────────────
1. User identity    → Who is signing in? What role?
2. Location         → Trusted/untrusted? Which country?
3. Device           → Managed/compliant? Platform?
4. Application      → Which resource? Sensitivity level?
5. Sign-in risk     → AI score: Low/Medium/High

SECURECORP'S 5 POLICIES:
─────────────────────────────────────────────────────────
CA001 — Block Legacy Authentication (HIGHEST IMPACT)
  Users: All | Apps: All | Condition: IMAP/POP3/SMTP/EAS
  Control: Block | Stops: MFA bypass via legacy protocols

CA002 — Require MFA All Users
  Users: All | Apps: All | Conditions: None
  Control: Require MFA | Stops: Password spray success

CA003 — Admins Need MFA + Compliant Device
  Users: Global Admin + Priv Role Admin roles
  Apps: All | Control: Require MFA AND compliant device
  Stops: Stolen admin credentials on unmanaged devices

CA004 — Block High Risk Sign-ins (P2 required)
  Users: All | Apps: All | Condition: Risk = High
  Control: Block | Stops: Tor, leaked creds, spray auto

CA005 — Block Outside India
  Users: All (excl. Hareesh, Navi) | Apps: All
  Condition: Location NOT = India named location
  Control: Block | Stops: Most external attackers

GOLDEN RULES:
─────────────────────────────────────────────────────────
1. ALWAYS start Report-only → run 2 weeks → then ON
2. ALWAYS exclude Break-glass from EVERY policy
3. Test one user before all users
4. Monitor CA tab in sign-in logs after every change
5. Document every policy — name, purpose, owner, date

BREAK-GLASS ACCOUNT:
─────────────────────────────────────────────────────────
Account: bkglass-emergency@securecorp.com
No MFA — excluded from ALL CA policies
60-char random password — in physical safe
Monitor ANY use of this account = automatic P0 alert
Used ONLY when locked out of all admin accounts

MFA METHOD STRENGTH:
─────────────────────────────────────────────────────────
Strongest: FIDO2 keys, Windows Hello (phishing-resistant)
Strong:    Authenticator app + number matching enabled
Weak:      SMS OTP (SIM swap vulnerable)
None:      Password only — never acceptable

CA TAB IN SIGN-IN LOGS (Navi's tool):
─────────────────────────────────────────────────────────
Shows: Which policies applied, pass/fail, why
Look for: MFA not applied → CA gap → investigate
Look for: Blocked by CA001 → legacy auth attempt = attacker
Look for: All policies not applied → guest or excluded user

PORTAL PATH:
Entra ID → Security → Conditional Access → Policies

LABS COMPLETED:
─────────────────────────────────────────────────────────
□ Created CA001 in Report-only mode
□ Found legacy auth users in sign-in logs
□ Checked CA tab on real sign-ins
□ Microsoft Learn SC-300 CA lab
```

---

> 💡 **Architect's Final Word:** *"In every Entra ID security review I've done, the same three gaps appear: no CA policy blocking legacy auth, no MFA for all users, and no alert on admin role changes. CA001 and CA002 together take 30 minutes to set up and stop the majority of cloud identity attacks. If SecureCorp had these two policies on the day of the breach — Gani's stolen password would have been completely useless. That's the power of Conditional Access done right."*

---


**Day 8 tomorrow — Entra ID Logs & SOC Monitoring** — where Navi becomes the master of the sign-in log. 🔐

Ready? Or want to go deeper on any CA policy? 💪
