

# 📘 DAY 10 — PHASE 2 REVIEW + MOCK INTERVIEW
**Phase 2: Microsoft Entra ID Security — Complete Consolidation**

---

## 🧭 DAY 10 MINDSET

```
WHAT WE'VE COVERED IN PHASE 2 (Days 6-9.1):
─────────────────────────────────────────────────────────
Day 6:   Entra ID architecture + 4 attack types
Day 7:   Conditional Access — 5 policies from scratch
Day 8:   Entra ID logs + KQL + SOC monitoring
Day 9:   App registrations + Enterprise Apps
Day 9.1: Log pipeline — Entra ID → Sentinel + on-prem

TODAY:
  Part 1: Phase 2 complete consolidation
  Part 2: 20 real SOC interview questions
          Answer them as if in a real interview
  Part 3: Gaps identified → fill before Phase 3

IF YOU CAN ANSWER TODAY'S QUESTIONS CONFIDENTLY:
  You are ready for SOC Analyst interviews on
  identity security — one of the most tested areas.
─────────────────────────────────────────────────────────
```

---

## 📌 PART 1 — PHASE 2 COMPLETE CONSOLIDATION

---

### The Complete Entra ID Security Picture

```
HOW EVERYTHING CONNECTS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ATTACK OCCURS          LAYER THAT STOPS IT       LOG EVIDENCE
─────────────────────────────────────────────────────────
Password spray         CA002 (MFA required)       SigninLogs
                       CA004 (risk HIGH block)     50126 errors

MFA fatigue            Number matching             SigninLogs
                       CA003 (compliant device)    MFA approved
                       FIDO2 keys                  unusual location

AiTM token theft       CAE (real-time revocation)  SigninLogs
                       Compliant device CA03        2-location anomaly
                       FIDO2/Windows Hello          IsInteractive gap

Illicit consent        Admin consent required       AuditLogs
                       Defender Cloud Apps          New app consent
                       Regular app audits           Unverified publisher

Leaked app secret      Azure Key Vault              AuditLogs
                       Max 12-month expiry          Secret created
                       Certificate over secret      App activity

Admin account abuse    PIM (just-in-time)           AuditLogs
                       CA003 (device required)      Role assignment
                       Sentinel alerts              After-hours activity

Stale guest access     Access Reviews quarterly     AuditLogs
                       Auto-removal on non-response  Last signin date

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Phase 2 Master Reference

```
ENTRA ID KEY CONCEPTS:
─────────────────────────────────────────────────────────
Identity types:       Member, Guest, Service Principal
Auth flow:            Password → Risk → CA → MFA → Token
Token types:          Access (1h), Refresh (90d), PRT (14d)
Log sources:          Sign-in, Audit, Risk/Identity Protection
Revoke compromise:    Entra ID → Users → Revoke sessions

CONDITIONAL ACCESS — 5 POLICIES:
─────────────────────────────────────────────────────────
CA001: Block Legacy Auth    → All users → Block IMAP/POP3/SMTP
CA002: Require MFA All      → All users → MFA always
CA003: Admin Compliant Dev  → Admin roles → MFA + device
CA004: Block High Risk      → All users → Block risk=HIGH
CA005: Block Outside India  → Most users → Block non-India

Golden rules:
  Report-only first → Exclude break-glass always
  Test 1 user → Monitor CA tab → Document everything

APP REGISTRATIONS — SECURITY CHECKLIST:
─────────────────────────────────────────────────────────
Authentication:  Exact redirect URIs, disable implicit grant
Secrets:         Key Vault only, max 12m expiry, rotate early
Certificates:    Better than secrets, private key in Key Vault
Federated creds: Best — no secret, for GitHub/DevOps pipelines
API permissions: Least privilege, no broad app permissions
                 Review and justify every admin consent
App roles:       Custom RBAC in Entra ID, not in app code
Owners:          Max 2 per app, remove on team changes

ENTERPRISE APPS — KEY SETTINGS:
─────────────────────────────────────────────────────────
Properties:      Enabled=NO for instant breach containment
                 User assignment required=YES always
Users & Groups:  Explicit assignment, quarterly Access Reviews
SSO:             SAML (enterprise SaaS), OIDC (modern apps)
                 Track SAML cert expiry (3yr default)
Provisioning:    SCIM auto-creates/disables accounts in apps
                 Scope: assigned users only
App Proxy:       On-prem apps without VPN, secure tunnel
Self-service:    Business owner approves, include in reviews

LOG PIPELINE:
─────────────────────────────────────────────────────────
Cloud:
  Entra ID → Diagnostic Settings → Log Analytics Workspace
  → Microsoft Sentinel
  Tables: SigninLogs, AuditLogs, AADRiskyUsers

Defender:
  Sentinel Data Connector → Microsoft Defender XDR
  Tables: DeviceEvents, IdentityLogonEvents,
          IdentityDirectoryEvents, EmailEvents

On-Premises:
  Azure Monitor Agent on all servers
  Data Collection Rules (DCR) with specific Event IDs
  Tables: SecurityEvent, Event (Sysmon/PS logs)

ManageEngine ADAudit Plus:
  Dedicated AD auditing, pre-built reports
  No KQL needed, compliance reports
  Forwards to Sentinel: ADAuditPlus_CL

ManageEngine EventLog Analyzer:
  All server logs, Linux, network devices
  Correlation rules, MITRE mapping
  Forwards to Sentinel: EventLogAnalyzer_CL
```

---

## 📌 PART 2 — MOCK INTERVIEW: 20 SOC ANALYST QUESTIONS

---

```
FORMAT:
Question first → Think about your answer → 
Read the model answer → Compare → Note gaps

These are REAL questions asked in SOC Analyst interviews
at companies using Microsoft security stack.
```

---

### SECTION A — CONCEPTUAL QUESTIONS (5 questions)

---

**Q1: What is the difference between Active Directory and Microsoft Entra ID? When would you use each?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
Active Directory is an on-premises directory service
that manages Windows computers, domain-joined resources,
Kerberos authentication, and internal network access.
It uses LDAP and Kerberos protocols.

Microsoft Entra ID is Microsoft's cloud-based identity
platform that manages authentication to cloud services
like Microsoft 365, Azure, and SaaS applications.
It uses modern protocols — OAuth 2.0, OIDC, and SAML.

You use Active Directory for:
→ Managing domain-joined Windows workstations
→ On-premises file servers and print servers
→ Internal network resources requiring Kerberos auth
→ Group Policy to enforce settings on Windows machines

You use Entra ID for:
→ Cloud application access (Teams, SharePoint, Salesforce)
→ Azure resource access and management
→ Mobile device identity (BYOD scenarios)
→ External partner access (B2B guest users)

In hybrid environments like SecureCorp, both coexist.
Entra Connect synchronises identities from on-prem AD
to Entra ID so users have a single identity for both.

KEY SECURITY DIFFERENCE:
On-prem AD: Attacker needs to breach network perimeter first.
Entra ID: Login page is publicly accessible globally.
The security perimeter for Entra ID is identity itself —
strong credentials, MFA, and Conditional Access.
─────────────────────────────────────────────────────────
```

---

**Q2: A user calls saying they approved an MFA request they did not initiate. What do you do?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
This is a potential account compromise via MFA fatigue.
My immediate response:

STEP 1 — Immediate containment (first 2 minutes):
Revoke all active sessions for this user immediately:
  Entra ID → Users → [User] → Revoke sessions
  OR: Revoke-MgUserSignInSession -UserId "[email]"
This invalidates all current tokens instantly.

STEP 2 — Assess the damage (next 10 minutes):
Check sign-in logs for this user in the last 2 hours:
  → Did a login succeed from an unusual location?
  → What IP address? What country?
  → What application was accessed?
  → Was it interactive or token replay?

Check audit logs for this user:
  → Were any changes made while attacker had access?
  → Were any files downloaded?
  → Were any admin actions performed?

STEP 3 — Reset credentials:
  → Force password reset for the user
  → Force MFA re-registration from scratch
  → Ensure new MFA uses number matching

STEP 4 — Check for persistence:
  → Were any app registrations created?
  → Were any OAuth consents granted?
  → Were any forwarding rules added to their mailbox?
  → Was any new device registered under their account?

STEP 5 — Block the attacker:
  → Add the source IP to CA Named Location → block
  → Check if same IP targeted other accounts

STEP 6 — Document and escalate:
  → Create P1 incident ticket with full timeline
  → Escalate to security lead / CISO
  → Notify user of the breach and steps taken

PREVENTION going forward:
  Enable number matching MFA — attacker cannot
  trick the user with a simple push notification anymore.
─────────────────────────────────────────────────────────
```

---

**Q3: What is an AiTM attack and how does it bypass MFA?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
AiTM stands for Adversary in the Middle.

In a standard MFA flow:
  User → Entra ID → MFA completed → Token issued → Access

In an AiTM attack:
  User → ATTACKER'S PROXY → Entra ID → MFA completed
         Proxy captures → Token stolen → Access

The attacker sets up a reverse proxy (tools like Evilginx2)
that sits between the user and the real login page.
The proxy looks identical to Microsoft's login page
and even has a valid SSL certificate.

The user enters their password — the proxy forwards it
to the real Entra ID. The user completes MFA — the proxy
forwards that too. The real Entra ID issues a session token
and the proxy captures it before passing it to the user.

MFA is NOT bypassed — it is completed correctly by the user.
The token is stolen AFTER MFA, not instead of MFA.
This is why traditional MFA does not stop AiTM.

DETECTION:
The signature of AiTM in sign-in logs is two entries
for the same user within seconds of each other:
  Entry 1: Interactive login, user's IP (Bangalore)
  Entry 2: Non-interactive token use, different IP (Netherlands)
This is called impossible travel with token replay.

PREVENTION:
  FIDO2 security keys or Windows Hello — these create
  tokens that are cryptographically bound to the real domain.
  The proxy cannot capture a domain-bound token.
  
  Requiring a compliant Intune device also helps because
  the attacker's machine will not be compliant.
  
  Continuous Access Evaluation (CAE) revokes tokens
  in real time when risk signals change.
─────────────────────────────────────────────────────────
```

---

**Q4: What is an Illicit Consent Grant attack? How do you detect and prevent it?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
An Illicit Consent Grant attack exploits Microsoft's
legitimate OAuth consent framework.

An attacker registers an application in their own
Entra ID tenant and configures it to request broad
permissions — such as Mail.Read, Files.ReadWrite,
or Contacts.Read.

They then send a phishing link to a target user.
When the user clicks the link, they see a real
Microsoft OAuth consent screen asking them to grant
the app permissions. If the user clicks Accept, the
attacker's app receives OAuth tokens for that user's
account and can access their data persistently.

This bypasses both password AND MFA because OAuth
tokens are issued after the user's successful
authentication. The attack does not need the password
at all — it gets tokens through consent.

The access persists even after password changes because
OAuth tokens are independent of passwords.

DETECTION:
  Entra ID → Enterprise Applications → All Applications
  Look for: Recently created apps, unverified publishers,
  broad permissions (Mail.Read, Files.ReadWrite application
  level), rapid adoption across multiple users.
  
  Sentinel alert:
  AuditLogs
  | where OperationName == "Add OAuth2PermissionGrant"
  | where Result == "success"
  | extend App = tostring(TargetResources[0].displayName)

PREVENTION:
  Disable user consent entirely:
  Entra ID → Enterprise Apps → Consent and Permissions →
  "Do not allow user consent"
  
  Implement Admin Consent Workflow:
  Users request app access → notification to admin →
  admin reviews permissions → approves or denies.
  Nothing gets through without admin review.
  
  Microsoft Defender for Cloud Apps monitors for
  suspicious OAuth apps automatically.
─────────────────────────────────────────────────────────
```

---

**Q5: What is Conditional Access and what are the 5 signals it evaluates?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
Conditional Access is Microsoft Entra ID's policy engine
that evaluates every sign-in attempt and decides whether
to allow, challenge with MFA, or block access based on
contextual signals rather than just credentials alone.

The formula is: IF conditions THEN access controls.
This replaces the old model of username + password = access
with a context-aware decision.

The 5 signals evaluated on every sign-in:

1. USER IDENTITY:
   Who is signing in? What role do they have?
   A Global Admin signing in triggers different policies
   than a standard user. Guest users trigger different
   policies than members.

2. LOCATION:
   Where is the sign-in coming from? Is it a trusted
   named location (office IP range)? Which country?
   Signing in from India vs signing in from Russia
   triggers different responses.

3. DEVICE:
   What device is being used? Is it enrolled in Intune?
   Is it marked as compliant with security policies?
   A managed, compliant device gets more trust than
   an unknown personal device.

4. APPLICATION:
   Which resource is being accessed? Accessing Teams
   might allow MFA-only while accessing the Azure Portal
   might require MFA plus a compliant device.

5. SIGN-IN RISK (Entra ID P2):
   Microsoft's AI evaluates each sign-in for risk signals:
   anonymous IP addresses, impossible travel, leaked
   credentials, malware-linked IPs, password spray
   patterns. The result is a risk score: Low, Medium, or High.

Based on these 5 signals together, CA decides:
Allow access / Require MFA / Require compliant device /
Block access completely.
─────────────────────────────────────────────────────────
```

---

### SECTION B — SCENARIO QUESTIONS (10 questions)

---

**Q6: You see 847 failed login attempts against one account from a single IP between 2-3 AM. What do you do?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
This is a brute force attack pattern.
847 attempts against one account from one IP in one hour.

IMMEDIATE ACTIONS:

1. Check if any attempt succeeded:
   SigninLogs
   | where UserPrincipalName == "[target account]"
   | where TimeGenerated between (start .. end)
   | where ResultType == "0"  // Success
   If any success → account is compromised → escalate to P0

2. Check if account is locked (Error 50053):
   If locked → it protected itself via lockout policy
   But: if lockout threshold was not configured → problem

3. Block the attacking IP via Conditional Access:
   Create Named Location → add the IP → create CA policy
   blocking that location immediately

4. Check if same IP attacked other accounts:
   SigninLogs
   | where IPAddress == "[attacker IP]"
   | where TimeGenerated > ago(24h)
   | summarize count() by UserPrincipalName
   Multiple accounts = spray + brute force combination

5. If account was compromised:
   Revoke sessions → reset password → force MFA re-register
   Check for post-compromise activity in audit logs

6. If account not compromised:
   Monitor for continued attempts
   Notify user to change password proactively
   Check if their password was in any breach database

7. Escalate and document:
   P1 incident if account was accessed
   P2 incident if only attempts, no success
   Timeline of all events in incident report

ROOT CAUSE:
   Understand why this account was targeted:
   Was it an admin account? Were credentials leaked?
   Was it a former employee account (should be disabled)?
─────────────────────────────────────────────────────────
```

---

**Q7: A SOC alert fires: "New Global Admin added outside business hours." How do you investigate?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
This is a critical alert — potentially an active breach
in progress. Treat as P0 until proven otherwise.

STEP 1 — Identify who was added and who added them:
AuditLogs
| where OperationName == "Add member to role"
| where TargetResources has "Global Administrator"
| extend
    NewAdmin = tostring(TargetResources[0].userPrincipalName),
    AddedBy  = tostring(InitiatedBy.user.userPrincipalName),
    Time     = TimeGenerated
| project Time, NewAdmin, AddedBy

STEP 2 — Is the new admin account legitimate?
  → Is it a known employee?
  → When was this account created?
    AuditLogs | where OperationName == "Add user"
    | where TargetResources has "[new admin email]"
  → If created recently (same night) = backdoor account

STEP 3 — Was the actor (AddedBy) compromised?
  → Check their sign-in logs that night
  → Unusual IP? Country? Time?
  → Did they show impossible travel pattern?
  → Did they approve unexpected MFA push?

STEP 4 — What did the new admin do after being elevated?
  AuditLogs
  | where InitiatedBy has "[new admin email]"
  | where TimeGenerated > [elevation time]
  | project TimeGenerated, OperationName, TargetResources

STEP 5 — Containment if confirmed compromise:
  → Disable new admin account immediately
  → Remove from Global Admins
  → Revoke all sessions for both accounts
  → Reset passwords + MFA for the actor account
  → Check for any other changes made during the session

STEP 6 — Check for additional persistence:
  → Any other accounts elevated?
  → Any app registrations created?
  → Any CA policies disabled?
  → Any conditional access exclusions added?

ESCALATION:
  Confirmed breach → CISO notification within 30 minutes
  All actions documented with timestamps
  Preserve evidence (do not delete logs)
─────────────────────────────────────────────────────────
```

---

**Q8: How would you detect a password spray attack in Microsoft Sentinel using KQL?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
Password spray has a distinctive signature:
Multiple accounts, one failure each, same IP, same window.
This is different from brute force (one account, many failures).

KQL query:

SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType == "50126"
| summarize
    FailedAccounts   = dcount(UserPrincipalName),
    TotalAttempts    = count(),
    TargetedAccounts = make_set(UserPrincipalName, 10)
    by IPAddress, bin(TimeGenerated, 10m)
| where FailedAccounts > 5
| extend
    AttemptsPerAccount = round(
        toreal(TotalAttempts) / FailedAccounts, 2)
| where AttemptsPerAccount < 2
| sort by FailedAccounts desc

WHY THIS WORKS:
  FailedAccounts > 5 = multiple distinct targets
  AttemptsPerAccount < 2 = roughly 1 attempt each
  These two conditions together = spray pattern

WHAT I LOOK AT IN RESULTS:
  High FailedAccounts with low AttemptsPerAccount
  = Classic spray pattern

  I also check for SUCCESS mixed in:
  SigninLogs
  | where IPAddress == "[spray IP]"
  | where ResultType == "0"
  = Did any spray attempt succeed?

  If yes — that's the account to investigate immediately.

I would create this as a Scheduled Analytics Rule in Sentinel
with a 1-hour evaluation window, alerting when
FailedAccounts > 5 from the same IP within 10 minutes.
─────────────────────────────────────────────────────────
```

---

**Q9: A developer says their app registration secret expired and their application stopped working. How do you handle this?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
This is both an incident response and a process failure.
The secret should never have expired unexpectedly.

IMMEDIATE RESOLUTION:

1. Verify the expiry in App Registrations:
   Entra ID → App Registrations → [App]
   → Certificates & Secrets → Check expiry dates

2. Create new client secret:
   + New client secret
   Description: "[AppName]-Prod-[Month][Year]"
   Expiry: 12 months maximum
   Copy the VALUE immediately (shown only once)

3. Update the application to use new secret:
   The developer updates wherever the secret is used.
   SHOULD BE: Azure Key Vault only
   Should NOT be: Config file or source code
   
   If it's in a config file:
   → Address this during the fix: move to Key Vault
   
   If it's in source code:
   → STOP: This is a security incident
   → Treat the secret as compromised immediately
   → Check if code is in a Git repository
   → If in Git: rotate secret, search all repo history
     for the exposed value, notify security team

4. Delete the expired secret (clean up):
   Old secret → Delete
   Reduce attack surface from old credentials

ROOT CAUSE AND PERMANENT FIX:

This happened because there was no expiry monitoring.
The fix:

Monitoring via PowerShell (add to weekly automation):
  Get-MgApplication -All |
  ForEach-Object {
    $_.PasswordCredentials |
    Where-Object { $_.EndDateTime -lt (Get-Date).AddDays(60) }
  }
  Alert when any secret within 60 days of expiry.

Process fix:
  → All secrets stored in Azure Key Vault
  → Key Vault has built-in expiry notifications
  → App reads secret from Key Vault at runtime
  → Key Vault rotation policy: auto-rotate every 90 days
  → Developers never handle the secret value directly
─────────────────────────────────────────────────────────
```

---

**Q10: Explain how you would set up logging from Entra ID to Microsoft Sentinel.**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
The pipeline is:
Entra ID → Diagnostic Settings → Log Analytics Workspace
→ Microsoft Sentinel

STEP 1 — Create Log Analytics Workspace:
  Azure Portal → Log Analytics workspaces → + Create
  Name: law-securecorp-siem
  Region: Same as your resources (South India)
  Retention: Set to 90 days minimum
              (default is 30, increase for compliance)

STEP 2 — Deploy Microsoft Sentinel:
  Microsoft Sentinel → + Create → Select the workspace
  Sentinel activates on top of Log Analytics.
  All logs ingested to the workspace are available in Sentinel.

STEP 3 — Connect Entra ID via Data Connector:
  Sentinel → Data Connectors → Search "Microsoft Entra ID"
  Open connector page → Connect each log type:
  → Sign-in logs (SigninLogs table)
  → Audit logs (AuditLogs table)
  → Non-interactive sign-ins
  → Service principal sign-ins
  → Risk events (requires P2 — AADRiskyUsers)
  → Provisioning logs

  Behind the scenes this creates a Diagnostic Setting
  in Entra ID that streams logs to the workspace.

STEP 4 — Connect Microsoft Defender XDR:
  Sentinel → Data Connectors → Microsoft Defender XDR
  This brings in endpoint, identity, and email data:
  Tables: DeviceEvents, IdentityLogonEvents,
          IdentityDirectoryEvents, EmailEvents

STEP 5 — Enable Analytics Rules:
  Sentinel → Analytics → Rule templates
  Enable pre-built rules for:
  Brute force, MFA spam, admin role changes,
  impossible travel, Kerberoasting detection.

VERIFICATION:
  After 30 minutes run:
  SigninLogs | take 10
  If you see rows → pipeline working correctly.

For on-premises servers:
  Install Azure Monitor Agent on each server.
  Create Data Collection Rule with specific Event IDs.
  This populates the SecurityEvent table in Sentinel.
  Both cloud and on-prem logs then available together
  for cross-platform KQL queries.
─────────────────────────────────────────────────────────
```

---

**Q11 — Q15: Rapid Fire Scenarios**

```
Q11: Guest user from a vendor hasn't logged in for 6 months
     but still has access to your SharePoint. What do you do?

ANSWER:
This is stale access — a common compliance and security issue.

Immediate: Review what SharePoint content they can access.
           Is it sensitive? If yes → remove access NOW.

Process: Run quarterly Access Reviews:
         Entra ID → Identity Governance → Access Reviews
         → Create review for guest users
         Reviewer: Business owner of the SharePoint site
         Duration: 2 weeks to respond
         If no response: Auto-remove access
         
         After review: If vendor relationship ended →
         disable guest account completely.
         If still active vendor → confirm with business owner
         they still need the specific access granted.

Prevention: Set guest user inactivity policy:
            Entra ID → External Identities → Settings
            Guest user access expires after: 90 days inactive

─────────────────────────────────────────────────────────
Q12: Someone added 50 users to a security group that has
     access to sensitive financial data. How do you investigate?

ANSWER:
AuditLogs
| where OperationName == "Add member to group"
| where TargetResources has "[Finance-Group-Name]"
| project TimeGenerated,
          Actor  = tostring(InitiatedBy.user.userPrincipalName),
          Member = tostring(TargetResources[0].userPrincipalName)
| sort by TimeGenerated asc

Check:
  → Who added them? Was it an authorised manager?
  → Were all 50 added at the same time? (bulk operation)
  → Is the actor account compromised?
  → Did any of the 50 access the data after being added?
  → Was there a change management ticket for this?

If unauthorised:
  → Remove all 50 from the group immediately
  → Revoke sessions for the actor if compromised
  → Check what data was accessed during the window
  → Escalate to data protection officer if sensitive data accessed

─────────────────────────────────────────────────────────
Q13: A Conditional Access policy was disabled at 3 AM.
     How do you respond?

ANSWER:
This is CRITICAL. Disabling CA policies is a common
attacker tactic to remove security controls.

First: AuditLogs → filter OperationName = "Update policy"
       or "Delete conditional access policy"
       Who disabled it? When? From which IP?

If actor account is compromised:
  → Re-enable the CA policy immediately
  → Revoke actor's sessions
  → Reset actor's password + MFA
  → Check what access occurred WHILE policy was disabled
  → Any sign-ins that would have been blocked by the policy?

If it was a legitimate admin who disabled it:
  → Verify they had a change management ticket
  → Understand why it was disabled
  → Re-enable it
  → Create an alert: CA policy changes always notify SOC

Prevention:
  Create Sentinel alert:
  AuditLogs
  | where OperationName has "conditional access"
  | where Result == "success"
  Fires immediately on any CA policy change.

─────────────────────────────────────────────────────────
Q14: Navi sees this in sign-in logs:
     User: gani@securecorp.com
     Status: Success
     Authentication: Single factor only
     Client app: Browser (modern auth)
     CA002: Not applied
     What does this mean and what do you do?

ANSWER:
This means Gani successfully logged in WITHOUT MFA.
CA002 should require MFA for all users but it did NOT apply.
This is a Conditional Access policy gap.

Why CA002 might not apply:
  → Gani was excluded from CA002 (most common cause)
  → Gani's account is a guest (CA scope missed it)
  → CA002 was scoped to a specific group Gani isn't in
  → CA002 is in report-only mode (not enforcing)

Immediate actions:
  → Check CA002 configuration → is Gani excluded?
  → If excluded → add him back immediately
  → Check sign-in timestamp: was this a legitimate login?
    If from unusual location/time → treat as compromised

Fix:
  → Remove unnecessary exclusions from CA002
  → CA002 should be: All users, no conditions
    with ONLY break-glass excluded
  → Quarterly audit of all CA exclusions

─────────────────────────────────────────────────────────
Q15: How do you use Privileged Identity Management (PIM)
     with Entra ID to reduce admin risk?

ANSWER:
PIM implements just-in-time privileged access.

Without PIM:
  Hareesh is a permanent Global Admin.
  His account has Global Admin 24/7/365.
  If compromised at any moment → attacker is Global Admin.
  Attack window = infinite.

With PIM:
  Hareesh is ELIGIBLE for Global Admin but not active.
  Day-to-day he has no admin privileges — just a normal user.
  When he needs admin access, he activates PIM:
  → Goes to PIM portal
  → Requests Global Admin for 1 hour
  → Provides business justification
  → May require MFA re-authentication
  → Approval may be required (second admin approves)
  → Role activates for 1 hour then auto-expires

Benefits:
  → Admin privileges exist for minutes/hours not years
  → If account is compromised at idle time = no admin access
  → Every activation has justification + audit trail
  → Approvals add human verification for sensitive roles

Setup:
  Entra ID → Identity Governance → Privileged Identity Management
  → Azure AD Roles → Settings → Global Administrator:
    Maximum activation duration: 1 hour
    Require MFA on activation: Yes
    Require approval: Yes (second Global Admin)
    Require justification: Yes

Navi monitors PIM activations as normal SOC activity:
  AuditLogs
  | where OperationName has "PIM"
  | project TimeGenerated, Actor, OperationName, Result
```

---

### SECTION C — TECHNICAL DEEP DIVE (5 questions)

---

**Q16: What is the difference between delegated and application permissions in Microsoft Graph?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
DELEGATED PERMISSIONS:
  The app acts ON BEHALF OF a signed-in user.
  The app can only do what the USER can do.
  
  Example: HR Portal has Mail.Read delegated.
  When Chaitu signs in, the portal can read Chaitu's emails.
  It CANNOT read Priya's emails (Chaitu can't do that).
  
  Authentication flow: User signs in first,
  their permissions + app's permissions = intersection
  
  Use case: Any app where a human uses the app interactively.
  Safer because limited to individual user scope.

APPLICATION PERMISSIONS:
  The app acts AS ITSELF with no user context.
  Runs as a background service, daemon, or automation.
  
  Example: Backup agent has Mail.Read application.
  It can read ALL mailboxes in the entire organisation.
  No user signs in — the app uses its own credentials.
  
  Authentication: Client credentials flow
  (App ID + Secret or Certificate)
  
  Use case: Background jobs, automation, API-to-API calls.
  MUCH more powerful — these are tenant-wide.
  Require admin consent (users cannot grant them).
  
SECURITY IMPLICATION:
  Application permissions are the high-risk ones.
  Mail.Read application permission = read 500 mailboxes.
  Mail.Read delegated permission = read 1 user's mailbox.
  
  Always use delegated when a user is involved.
  Application permissions only for truly headless services.
  Audit application permissions quarterly.
─────────────────────────────────────────────────────────
```

---

**Q17: How does SAML SSO work? What can go wrong from a security perspective?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
SAML SSO (Security Assertion Markup Language):

FLOW:
1. User accesses Salesforce → not logged in
2. Salesforce redirects to Entra ID
   (Salesforce = Service Provider, Entra ID = Identity Provider)
3. Entra ID authenticates user (password + MFA if required)
4. Entra ID generates SAML Assertion (XML document):
   → Contains user identity
   → Contains user attributes (email, groups, roles)
   → Signed with Entra ID's signing certificate
   → Time-stamped with validity window
5. Entra ID sends SAML assertion to Salesforce
   via the browser (POST to Assertion Consumer Service URL)
6. Salesforce validates the signature using Entra ID's
   public certificate → trusts the assertion
7. User is logged into Salesforce — no separate password

SECURITY RISKS:

1. SAML SIGNING CERTIFICATE EXPIRY:
   Entra ID signs assertions with a certificate.
   Default validity: 3 years.
   If certificate expires → SSO breaks for all users.
   Worse: if expired cert is replaced without updating
   the Assertion Consumer Service in Salesforce →
   Salesforce rejects all assertions.
   FIX: Track cert expiry, renew 60 days before.

2. ATTRIBUTE MANIPULATION:
   If SAML assertion is intercepted and modified →
   attacker could change role/group attributes
   to elevate their access.
   FIX: Strong signing + message encryption + HTTPS only

3. MISCONFIGURED ASSERTION CONSUMER SERVICE URL:
   If the Reply URL is too broad or wrong →
   SAML assertion sent to wrong endpoint →
   Potential token hijacking.
   FIX: Exact URL, HTTPS only, validated domain

4. XML SIGNATURE WRAPPING ATTACKS:
   Crafted XML can confuse parsers into validating
   attacker-injected content alongside legitimate content.
   FIX: Use modern SAML libraries, keep them updated.

5. SESSION DURATION TOO LONG:
   SAML sessions can outlive the intended window.
   FIX: Configure session lifetime, use CAE for
        real-time revocation when user is revoked in Entra ID
─────────────────────────────────────────────────────────
```

---

**Q18: What is SCIM provisioning and why is it important for security?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
SCIM stands for System for Cross-domain Identity Management.
It is a standardised protocol for automatic user account
provisioning and deprovisioning across systems.

HOW IT WORKS:
  When a user is created in Entra ID:
  → SCIM pushes the user to connected apps automatically
  → Salesforce account created → GitHub account created
  → ServiceNow account created → Zoom account created
  All happen within minutes, no manual IT action needed.

  When a user is disabled in Entra ID (offboarding):
  → SCIM pushes the disable to all connected apps
  → Salesforce disabled → GitHub suspended
  → ServiceNow disabled → Zoom disabled
  All happen within 40 minutes automatically.

SECURITY IMPORTANCE:

JOINERS (new employees):
  Manual provisioning = IT creates accounts one by one
  Takes days = user has no access = productivity impact
  Or: accounts created too early with wrong permissions
  SCIM = automatic, immediate, consistent, correct

MOVERS (role changes):
  Employee changes department → group membership changes
  → SCIM updates roles in all apps automatically
  → Previous department's access removed
  → New department's access granted
  Without SCIM: old access never removed = overprivileged

LEAVERS (offboarding — most critical):
  Employee leaves → Entra ID account disabled
  → SCIM propagates to all 47 apps automatically
  Without SCIM: IT must manually disable in each app
  Manual process = mistakes = former employee
  retains access to Salesforce for months after leaving
  This is how insider threats and data theft happen.

AUDIT VALUE:
  Provisioning logs show every account created/updated
  Errors show which apps didn't sync → manual follow-up needed
  Full audit trail of who had access to what and when
─────────────────────────────────────────────────────────
```

---

**Q19: How does Application Proxy work and what security benefits does it provide?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
Application Proxy provides secure remote access to
on-premises web applications without requiring a VPN.

ARCHITECTURE:
  On-premises server runs: Application Proxy Connector
  Azure hosts: Application Proxy Service (cloud)

FLOW:
  1. User accesses external URL:
     https://intranet.securecorp.com (public HTTPS)
  2. Application Proxy Service receives the request
  3. User must authenticate to Entra ID first
     (MFA + Conditional Access applies here)
  4. Authenticated request forwarded to on-prem Connector
     via outbound HTTPS tunnel
  5. Connector forwards to internal app:
     http://intranet.securecorp.local
  6. Response travels back through the tunnel
  7. User sees the internal app through their browser

SECURITY BENEFITS:

NO INBOUND FIREWALL PORTS:
  The Connector makes only OUTBOUND connections.
  Your internal network never directly exposed to internet.
  Traditional VPN requires opening inbound ports → attack surface.
  Application Proxy has zero inbound ports needed.

AUTHENTICATION BEFORE REACHING THE APP:
  The user must authenticate to Entra ID BEFORE
  the request even reaches your internal server.
  Unauthenticated requests never reach internal network.
  MFA and Conditional Access apply → full identity controls.

NO VPN CLIENT REQUIRED:
  HTTPS only → works on any device, any browser.
  No VPN software installation → no VPN credential management.
  Users on personal devices (BYOD) can access without VPN.

PRE-AUTHENTICATION:
  Set to Azure Active Directory (recommended):
  → Entra ID authenticates before proxy forwards
  → Attacker can't reach the app at all without valid creds
  
  Set to Passthrough (not recommended):
  → Request forwarded without authentication
  → App handles its own auth → weaker security

SINGLE SIGN-ON TO LEGACY APPS:
  Kerberos Constrained Delegation (KCD) allows
  Application Proxy to authenticate to Windows
  Integrated Auth apps on behalf of the user.
  Old intranet apps with NTLM/Kerberos get SSO via cloud.
─────────────────────────────────────────────────────────
```

---

**Q20: A company says "we don't need Conditional Access because all our users have MFA." How do you respond?**

```
MODEL ANSWER:
─────────────────────────────────────────────────────────
This is a common misconception. MFA is one layer
but it does not address all attack vectors.
Conditional Access adds critical layers that MFA alone cannot.

WHAT MFA ALONE DOESN'T STOP:

1. MFA FATIGUE:
   Attacker triggers push notifications until user approves.
   MFA was "completed" — but by a tricked user.
   CA + Number Matching prevents this.

2. LEGACY AUTHENTICATION BYPASS:
   If legacy protocols (IMAP, POP3, SMTP) are allowed,
   attackers can authenticate without MFA at all.
   Legacy auth doesn't support MFA.
   CA001 blocks legacy auth completely — MFA can't do this.

3. AITMTOKEN THEFT:
   Attacker steals session token AFTER MFA.
   MFA was completed correctly — token still stolen.
   CA requiring compliant device means attacker's unmanaged
   device cannot use the stolen token.

4. COMPROMISED DEVICE:
   User completes MFA on a malware-infected device.
   Malware steals the token post-MFA.
   CA requiring Intune-compliant device (verified clean)
   prevents access from infected devices.

5. HIGH-RISK SIGN-INS:
   User completes MFA from a Tor node or leaked credential.
   MFA doesn't evaluate whether the sign-in itself is risky.
   CA004 blocks when risk score = HIGH regardless of MFA.

6. LOCATION ANOMALIES:
   User MFA from India. Token replayed from Netherlands.
   MFA doesn't know that's impossible travel.
   CA with risk evaluation detects this automatically.

MY RESPONSE SUMMARY:
   "MFA is essential and should absolutely be deployed.
    But MFA is one control that addresses one attack vector.
    Conditional Access wraps around MFA to ensure it applies
    to all authentication flows (not just modern auth),
    adds device compliance requirements, evaluates
    risk signals from Microsoft's AI, blocks from
    untrusted locations, and prevents legacy auth bypass.
    MFA without Conditional Access is like a lock on your
    front door but an open window at the back."
─────────────────────────────────────────────────────────
```

---

## 📌 PART 3 — PHASE 2 GAPS ASSESSMENT

```
SELF-ASSESSMENT — Rate yourself honestly:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                                    Confident  Needs Work
─────────────────────────────────────────────────────────
Entra ID architecture vs on-prem AD    □           □
4 attack types with examples           □           □
Sign-in log — all 5 tabs               □           □
CA policy — what each stops            □           □
Create CA policy in portal             □           □
App registration security              □           □
Enterprise app SSO (SAML/OIDC)         □           □
Log pipeline to Sentinel               □           □
KQL queries for attack detection       □           □
Incident response for account breach   □           □
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Any "Needs Work" items → revisit that day's content
before moving to Phase 3.
```

---

## 📋 DAY 10 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║     DAY 10 — PHASE 2 REVIEW + MOCK INTERVIEW            ║
║              Microsoft Entra ID Security                 ║
╚══════════════════════════════════════════════════════════╝

PHASE 2 COMPLETE SUMMARY:
─────────────────────────────────────────────────────────
Day 6:  Entra ID basics + 4 attacks (spray, fatigue, AiTM, consent)
Day 7:  5 CA policies (CA001-005) + golden rules + break-glass
Day 8:  3 log sources + 10 KQL queries + SOC investigation
Day 9:  App registrations (8 tabs) + Enterprise apps (8 tabs)
Day 9.1:Log pipeline — cloud + on-prem → Sentinel

PHASE 2 KEY PRINCIPLES:
─────────────────────────────────────────────────────────
1. Identity is the new perimeter
   No network boundary protects Entra ID
   Username + strong auth + context = the only wall

2. Every attack leaves log evidence
   SigninLogs: spray (50126), AiTM (2-location), fatigue (approved)
   AuditLogs: consent grant, role change, policy disable

3. MFA alone is not enough
   Legacy auth bypasses it
   AiTM steals the token after it
   Fatigue tricks the user into approving it
   CA + number matching + phishing-resistant MFA = complete

4. Principle of least privilege everywhere
   App permissions, user assignments, admin roles
   Only what is needed, reviewed quarterly

5. Monitor everything in Sentinel
   Cloud + on-prem + all tools forwarding to one place
   One KQL query = full attack chain visibility

20 INTERVIEW QUESTIONS COVERED:
─────────────────────────────────────────────────────────
Conceptual (5):
  Q1: AD vs Entra ID differences
  Q2: User approved unexpected MFA — response
  Q3: AiTM attack mechanism
  Q4: Illicit consent grant — detect and prevent
  Q5: Conditional Access and 5 signals

Scenario (10):
  Q6: 847 failed logins from one IP at 2 AM
  Q7: New Global Admin added outside business hours
  Q8: KQL to detect password spray
  Q9: App registration secret expired
  Q10: Set up Entra ID logging to Sentinel
  Q11: Stale guest user with 6-month inactive access
  Q12: 50 users bulk-added to sensitive group
  Q13: CA policy disabled at 3 AM
  Q14: CA002 not applied on successful login
  Q15: PIM and just-in-time admin access

Technical deep dive (5):
  Q16: Delegated vs application permissions
  Q17: SAML SSO security risks
  Q18: SCIM provisioning security importance
  Q19: Application Proxy architecture and benefits
  Q20: "We have MFA so we don't need CA" — response

PHASE 2 LAB CHECKLIST:
─────────────────────────────────────────────────────────
□ Investigated sign-in logs — all 5 tabs
□ Created CA001 in Report-only mode
□ Found legacy auth users via KQL
□ Created test App Registration — all tabs configured
□ Ran secret expiry audit via PowerShell
□ Connected Entra ID to Sentinel (or completed Learn lab)
□ Ran all 10 KQL queries from Day 8
□ Microsoft Learn: SC-300 modules completed

PHASE 3 PREVIEW — LINUX SECURITY:
─────────────────────────────────────────────────────────
Day 11: Linux security basics (permissions, users, sudo)
Day 12: Linux hardening (SSH, firewall, fail2ban)
Day 13: Linux logs and attack detection
Day 14: Linux attack techniques (privilege escalation)
Day 15: Linux hands-on labs

Your Kali Linux experience = head start for Phase 3!
```

---

> 💡 **Architect's Truth for Day 10:** *"The candidates who get hired for SOC Analyst roles are not the ones who memorised the most definitions. They are the ones who can tell a story. When an interviewer asks about AiTM, the candidate who says 'a proxy captures the token after MFA' is okay. The candidate who says 'Gani receives a phishing email, logs into what looks like Microsoft's login page, completes his MFA, and thinks everything is normal — but the proxy already sent his session token to the attacker in Netherlands, which shows up as impossible travel in sign-in logs 13 seconds later' — that candidate gets the job. Learn the concepts. Then tell the story. That is the difference."*

---
