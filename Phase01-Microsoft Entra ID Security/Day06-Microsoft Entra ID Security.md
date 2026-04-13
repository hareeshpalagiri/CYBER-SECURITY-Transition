📘 DAY 6 — MICROSOFT ENTRA ID SECURITY
Phase 2: Microsoft Entra ID Security | SecureCorp Ltd.

🧭 DAY 6 MINDSET
PHASE 1 RECAP — What you mastered:
─────────────────────────────────────────────────────────
On-premises Active Directory:
  → AD structure, attacks, hardening, logs, labs
  → You think like both attacker AND defender for AD
  → Kerberoasting, PtH, DCSync, Golden Ticket — all clear

PHASE 2 — What changes:
─────────────────────────────────────────────────────────
Entra ID = Active Directory's cloud cousin
Same identity concepts → completely new attack surface

CRITICAL DIFFERENCE:
On-premises AD:
  Attacker needs to be INSIDE your network first
  Physical/VPN boundary provides some protection
  Attack surface = internal network perimeter

Entra ID:
  Login page: login.microsoftonline.com
  Accessible from ANYWHERE on earth
  Attacker only needs an internet connection
  No perimeter. No firewall protection.

THE NEW SECURITY PERIMETER = IDENTITY ITSELF
Your username + password + MFA = your only wall
This is why identity security is now the
#1 priority in modern enterprise security.
─────────────────────────────────────────────────────────

📌 SECTION 1 of 5
Entra ID Architecture — The Foundation

What is Microsoft Entra ID
SIMPLE DEFINITION:
Microsoft Entra ID (formerly Azure Active Directory)
is Microsoft's cloud-based identity and access
management service.

It is the identity backbone for:
→ Microsoft 365 (Teams, Outlook, SharePoint, OneDrive)
→ Azure (all cloud resources and services)
→ SaaS applications (Salesforce, ServiceNow, GitHub)
→ Custom applications your org builds
→ On-premises apps (via Application Proxy)

RELATIONSHIP WITH ON-PREMISES AD:
─────────────────────────────────────────────────────────
On-premises AD (DC01):
  Manages: Domain-joined Windows computers
           On-prem file shares, print servers
           On-prem applications
           Local network resources
  Protocol: Kerberos, NTLM, LDAP

Entra ID (Cloud):
  Manages: Cloud apps (Teams, SharePoint)
           Azure resources
           Mobile devices
           SaaS applications
           External partner access
  Protocol: OAuth 2.0, OIDC, SAML

HYBRID IDENTITY (SecureCorp setup):
  Both exist together
  Connected via Entra Connect (you manage this!)
  User created in on-prem AD →
  Synced to Entra ID via Entra Connect →
  Same identity works for both on-prem and cloud

YOUR CONNECTION:
  The Entra Connect sync errors you fixed at work?
  Those sync errors meant identities existed in one
  place but not the other — creating orphaned accounts.
  From a security perspective, orphaned accounts
  with no owner = exactly what attackers look for.
  You were doing security work without realising it.
SecureCorp's Entra ID Environment
SECURECORP TENANT:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tenant:     securecorp.onmicrosoft.com
Domain:     securecorp.com (custom verified domain)
Licence:    Microsoft 365 E3 + Entra ID P2
Users:      500 employees
Guests:     23 external (vendors, consultants)
Apps:       47 enterprise applications connected

CONNECTED TO:
→ On-premises AD (via Entra Connect sync)
→ Microsoft 365 (Exchange, Teams, SharePoint)
→ Azure Subscriptions (Dev + Production)
→ SaaS Apps (Salesforce, ServiceNow, GitHub, Zoom)
→ Custom apps (HR portal, Finance app, ML platform)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Identity Types — Security Risk View
TYPE 1 — MEMBER USERS (Internal):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┌──────────┬──────────────────┬──────────────────────────┐
│ User     │ Role             │ Security Risk            │
├──────────┼──────────────────┼──────────────────────────┤
│ Hareesh  │ Global Admin     │ #1 target always         │
│ Priya    │ Developer        │ App registration access  │
│ Jayanth  │ End User Support │ Password reset perms     │
│ Bharath  │ Network Engineer │ Azure network access     │
│ Chaitu   │ HR Manager       │ Sensitive data access    │
│ Gani     │ ML Engineer      │ No MFA = weakest link    │
│ Navi     │ SOC Analyst      │ Log access — protect     │
└──────────┴──────────────────┴──────────────────────────┘

TYPE 2 — GUEST USERS (External/B2B):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SecureCorp has 23 active guest accounts.
Vendors, consultants, auditors.

Security risks:
→ 11 of them haven't logged in for 90+ days (stale)
→ If vendor's tenant is breached → your data at risk
→ Less monitored than internal users
→ Often have access never removed after project ends
→ Stale access = forgotten door left permanently open

Navi's monthly check:
  Filter guests not logged in 90+ days
  Review with business owner
  Remove if no longer needed

TYPE 3 — SERVICE PRINCIPALS & MANAGED IDENTITIES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Non-human identities — apps, automation, scripts.

SecureCorp examples:
→ App-SQLSync     → syncs SQL data to SharePoint
→ App-BackupAgent → Azure Backup automation
→ App-Monitoring  → Azure Monitor alerting
→ GitHubActions   → CI/CD deployment pipeline

Why attackers love these:
→ Created and then forgotten
→ Broad permissions granted (nobody questions it)
→ Client secrets that never expire
→ No MFA — apps don't do MFA
→ No Conditional Access evaluation
→ Survive user account changes
→ Nobody monitors them
→ One compromised secret = all permissions available
The Entra ID Security Perimeter — Most Important Concept
ON-PREMISES AD SECURITY MODEL:
─────────────────────────────────────────────────────────
[INTERNET] ──firewall──> [CORPORATE NETWORK] ──> [DC01]

Attacker process:
  1. Break through perimeter firewall
  2. Get onto internal network
  3. Gain initial foothold
  4. THEN attack Active Directory

Physical and network boundary provides protection.
VPN + firewall = layers before reaching AD.

ENTRA ID SECURITY MODEL:
─────────────────────────────────────────────────────────
[INTERNET] ──────────────────────────────> [ENTRA ID]
                No firewall.
                No network boundary.
                No VPN required.
                login.microsoftonline.com is public.

Attacker process:
  1. Get internet connection (they already have it)
  2. Know a username (LinkedIn, Hunter.io, OSINT)
  3. Try to authenticate
  4. They're ALREADY at the front door

THE NEW PERIMETER = IDENTITY:
Username + Password + MFA + Device + Location = Access

This is why:
→ MFA is non-negotiable (Day 7)
→ Conditional Access is critical (Day 7)
→ Monitoring sign-in logs daily is essential (Day 8)
→ Every leaked credential is an immediate threat

📌 SECTION 2 of 5
How Entra ID Authentication Works — Complete Picture

The Modern Authentication Flow
GANI LOGS INTO MICROSOFT TEAMS — STEP BY STEP:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

STEP 1 — User enters email:
  Gani types: gani@securecorp.com
  Entra ID checks:
  → Is securecorp.com a managed domain? YES
  → Is it federated (ADFS)? NO
  → Show password page

STEP 2 — Password submitted:
  Password sent to Entra ID over HTTPS
  
  Entra ID validates against:
  → PHS (Password Hash Sync): validates hash in cloud
     (This is what SecureCorp uses — you know from Entra Connect!)
  → PTA (Pass-Through Auth): forwards to on-prem AD
  → Federation: redirects to ADFS server

STEP 3 — Sign-in Risk Evaluated:
  Microsoft AI checks EVERY sign-in:
  → Is this IP known malicious?
  → Is this impossible travel?
  → Is this an anonymous IP (Tor/VPN)?
  → Is this a new device?
  → Does this match Gani's normal behaviour?
  Risk score assigned: Low / Medium / High

STEP 4 — Conditional Access Policies evaluated:
  Every policy checked sequentially:
  → CA001: Is this legacy auth? → Block
  → CA002: Is MFA completed? → Require
  → CA003: Is device compliant? (not for Gani — not admin)
  → CA004: Is risk HIGH? → Block
  → CA005: Is location India? → Allow
  Result: ALLOW / BLOCK / REQUIRE MFA

STEP 5 — MFA Challenge (if required by CA):
  Gani gets push notification on Authenticator app
  Number shown on screen: "47"
  Gani types 47 in his phone → MFA satisfied

STEP 6 — Tokens Issued:
  Access Token  → Valid 1 hour → access Teams API
  Refresh Token → Valid 90 days → get new access tokens
  Primary Refresh Token (PRT) → Device-bound → SSO
  Session Cookie → Browser session

STEP 7 — Gani sees Teams ✅

EVERY STEP = ONE ENTRY in Sign-in Logs
That entry contains ALL step results
That is why sign-in logs are so powerful
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHS vs PTA — Connection to Your Entra Connect Experience
PASSWORD HASH SYNC (PHS) — Most common:
─────────────────────────────────────────────────────────
Hareesh sets up Entra Connect with PHS.
When Gani changes password in on-prem AD:
  → Password hash synced to Entra ID (not plain password)
  → Entra ID validates the hash for cloud logins
  → Authentication happens IN THE CLOUD

Benefits:
  → Works even if on-prem AD is down
  → Leaked credentials detection works (hash comparison)
  → Simpler to deploy and maintain

You already know this from managing Entra Connect.
Security perspective: PHS hash in cloud = attack surface
But: Microsoft stores it securely, one-way hash

PASS-THROUGH AUTHENTICATION (PTA):
─────────────────────────────────────────────────────────
Authentication validated by on-prem AD in real time.
Cloud sign-in → request forwarded to on-prem AD → validated.

Benefits:
  → Password never leaves on-prem
  → Password policy enforced from on-prem AD
  → Compliance requirement (some orgs need this)

Risk:
  → If on-prem AD is down → cloud login fails
  → PTA agent becomes critical infrastructure
Token Types — Understanding the New Attack Currency
WHY TOKENS MATTER MORE THAN PASSWORDS NOW:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OLD WORLD (password era):
  Attacker steals password → Change password → Safe

NEW WORLD (token era):
  Attacker steals token → Change password → NOT SAFE
  Token remains valid until it expires
  Token theft = access independent of password

TOKEN TYPES:
┌─────────────────┬──────────────┬──────────────────────┐
│ Token           │ Lifetime     │ Risk if Stolen       │
├─────────────────┼──────────────┼──────────────────────┤
│ Access Token    │ 1 hour       │ 1 hour of access     │
│ Refresh Token   │ 90 days      │ 90 days persistent   │
│ PRT (Primary    │ 14 days      │ Full SSO to all apps  │
│ Refresh Token)  │ renewable    │ on that device       │
│ Session Cookie  │ Browser life │ Active session       │
└─────────────────┴──────────────┴──────────────────────┘

PRIMARY REFRESH TOKEN (PRT) — Most Valuable:
─────────────────────────────────────────────────────────
Issued when user logs into Windows device.
Stored in device's TPM chip (secure hardware).
Grants SSO to ALL Microsoft services automatically.

If stolen:
→ Attacker has SSO to Teams, SharePoint, Azure Portal
→ No MFA triggered for individual app access
→ Works until device is removed from Entra ID

How attackers steal PRT:
→ Mimikatz module: sekurlsa::cloudap
→ Requires local admin on the machine
→ Why PAW matters: admin PRT on compromised machine
   = keys to entire Microsoft 365 kingdom

REVOKE ALL TOKENS IMMEDIATELY WHEN COMPROMISED:
  Entra ID → Users → [user] → Revoke sessions
  OR PowerShell:
  Revoke-MgUserSignInSession -UserId "gani@securecorp.com"
  
  This invalidates ALL active tokens immediately.
  User must sign in fresh — gets new tokens.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📌 SECTION 3 of 5
Entra ID Attack Types — The 4 Methods Attackers Use

ATTACK 1 — Password Spray
CONCEPT:
One common password tried against ALL users in tenant.
One attempt per account → lockout never triggered.
Runs for hours or days undetected.

ATTACKER RECONNAISSANCE FIRST:
─────────────────────────────────────────────────────────
Attackers find SecureCorp email addresses via:
→ LinkedIn (company page → employee list)
→ Hunter.io (finds email patterns from public data)
→ OSINT (GitHub commits, conference talks, blog posts)
→ Email format guessing: firstname.lastname@securecorp.com

Builds a list of 500 valid email addresses.
Tries password: "Welcome@2024" (most common India default)
Tries password: "SecureCorp@123" (company name pattern)
Tries password: "India@2024" (country + year pattern)

RESULT AFTER 6 HOURS:
  498 accounts → FAIL
  gani@securecorp.com → SUCCESS
  (Gani joined last month, still on temp password pattern)

WHY IT'S HARD TO DETECT WITHOUT PROPER MONITORING:
  Each account has only 1 failure
  Standard "5 failures = lockout" never triggers
  Attempts spaced 30 seconds apart = looks like humans
  Source IPs rotated through different countries/proxies

ENTRA ID SMART LOCKOUT:
  Microsoft has built-in protection
  Learns each account's unusual behaviour
  Locks out suspected spray IPs automatically
  But: sophisticated attackers use multiple IPs
  And: it doesn't catch every spray

WHAT NAVI SEES IN SIGN-IN LOGS:
─────────────────────────────────────────────────────────
Filter: Last 1 hour | Status: Failure

02:00 AM — hareesh@securecorp.com — FAIL — 185.220.101.45
02:00 AM — priya@securecorp.com   — FAIL — 185.220.101.45
02:00 AM — gani@securecorp.com    — SUCCESS — 185.220.101.45
02:01 AM — chaitu@securecorp.com  — FAIL — 185.220.101.45
02:01 AM — bharath@securecorp.com — FAIL — 185.220.101.45

Pattern: Same IP, multiple users, 1 attempt each
          One success mixed with failures = SPRAY WORKED

Navi's immediate actions:
  1. Block 185.220.101.45 via CA Named Location
  2. Force Gani password reset + MFA re-registration
  3. Check what Gani's account accessed in last 30 mins
  4. Check same IP against other accounts
  5. Alert all users: potential credential exposure

DEFENCE:
  CA002: Require MFA → stolen password = useless
  Entra ID Password Protection: Block common passwords
  Smart Lockout: Tune thresholds
  Sentinel Alert: Multiple users same IP same window

ATTACK 2 — MFA Fatigue (Prompt Bombing)
CONCEPT:
Attacker has the correct password already (from spray).
MFA is the only remaining obstacle.
Solution: Send push notifications REPEATEDLY
until the exhausted user approves one.

THE UBER BREACH — REAL WORLD EXAMPLE (2022):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Attacker bought Uber contractor's credentials
   from a dark web data breach dump

2. Triggered MFA push notifications repeatedly:
   Employee's phone: [APPROVE?] [APPROVE?] [APPROVE?]
   47 notifications over 20 minutes at midnight

3. Attacker also WhatsApped the employee:
   "Hi, I'm from Uber IT Security.
    We're doing a security verification.
    Please approve the MFA prompt."

4. Exhausted employee approved one notification

5. Attacker had full Uber access:
   AWS, Google Cloud, Slack, HackerOne portal
   Multiple systems compromised simultaneously

6. Damage: Significant data exposure
           $148M in regulatory fines (previous breach)
           Massive reputational damage

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IN SECURECORP:
Attacker gets Hareesh's password (from spray or phishing)
Bombards Hareesh's Authenticator app with push requests
Sends WhatsApp: "Hi Hareesh, IT dept — please approve MFA"
Hareesh approves at 11 PM (tired, watching TV)
Attacker has Global Admin session

WHAT LOGS SHOW:
  Sign-in log for hareesh:
  MFA result:    MFA satisfied ← looks legitimate!
  MFA method:    Authenticator push
  Status:        Success
  IP:            185.220.xx.xx ← Netherlands
  Location:      Amsterdam ← not India
  Time:          23:47 IST ← outside work hours

  The MFA COMPLETING is the suspicious indicator.
  "Why did Hareesh approve MFA from Netherlands at midnight?"

DEFENCE:
─────────────────────────────────────────────────────────
NUMBER MATCHING (most impactful defence):
  Before: Push shows [APPROVE] [DENY] buttons
  Attacker triggers notification → user panics → APPROVE
  
  After number matching:
  Sign-in page shows number: "47"
  Authenticator push shows: "Enter code: ___"
  User must TYPE 47 to approve
  Attacker's sign-in shows different number
  User realizes mismatch → DENY
  MFA fatigue attack = impossible
  
  Enable: Entra ID → Security → MFA → Additional settings
          Number matching: Enabled
          Additional context (show app + location): Enabled

ADDITIONAL CONTEXT:
  Push notification shows:
  → "Microsoft Teams"
  → "Sign-in from Amsterdam, Netherlands"
  Hareesh sees: "I'm in Bangalore — DENY!"

LIMIT MFA RETRIES:
  Block account for 1 hour after 3 failed MFA attempts
  Stops the rapid-fire bombing strategy

PHISHING-RESISTANT MFA (best long-term):
  FIDO2 Security Keys or Windows Hello
  Cannot be triggered remotely
  Fatigue attack = mathematically impossible

ATTACK 3 — Adversary in the Middle (AiTM)
CONCEPT:
The most sophisticated and increasingly common attack.
Bypasses MFA completely.
Steals session token AFTER the user completes MFA.
The real MFA is completed — by the legitimate user.
Attacker captures the resulting token.

SIMPLE ANALOGY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Normal bank visit:
  You → Show ID to bank teller → Get access to vault

AiTM bank:
  You → Show ID to FAKE TELLER (attacker's proxy)
             ↓
     Fake teller passes everything to real bank
             ↓
  Real bank gives you access (MFA completed correctly)
             ↓
  Fake teller COPIES your vault access card
             ↓
  Attacker uses copy of your vault card → enters vault

You authenticated correctly. Bank verified you.
But attacker copied your access card in between.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

HOW IT WORKS TECHNICALLY:
─────────────────────────────────────────────────────────
STEP 1: Attacker sets up Evilginx2 proxy server
        Creates fake domain: secure-teams-securecorp.com
        Gets valid SSL certificate (green padlock!)
        Looks EXACTLY like Microsoft's login page

STEP 2: Sends phishing email to Gani:
        "Your Teams subscription expiring — renew here"
        Link: https://secure-teams-securecorp.com/login

STEP 3: Gani clicks → sees "Microsoft" login page
        (It's actually Evilginx2 proxy)
        Gani types his password → proxy forwards to real Entra ID
        Gani completes MFA → proxy forwards to real Entra ID
        Real Entra ID issues session token
        PROXY CAPTURES the session token before Gani gets it

STEP 4: Gani lands on real Teams (seems normal to him)
        Attacker now has Gani's valid session token
        Uses token from their machine in Netherlands

STEP 5: Attacker authenticates to Teams, SharePoint,
        Graph API using captured token
        No password needed. No MFA needed.
        Token valid for 90 days.

WHAT NAVI SEES (the two sign-in fingerprint):
─────────────────────────────────────────────────────────
ENTRY 1 — Gani's real login:
  Time:     10:14:32 IST
  IP:       103.21.xx.xx (Bangalore — Gani's real IP)
  MFA:      Satisfied ✅
  Status:   Success

ENTRY 2 — Attacker using stolen token:
  Time:     10:14:45 IST  ← 13 SECONDS LATER
  IP:       185.220.xx.xx (Amsterdam — attacker)
  MFA:      Not required ← token replay, no MFA needed
  Status:   Success
  IsInteractive: False ← background token use

ALERT: Same user, two countries, 13 seconds apart
       = Physically impossible = AiTM confirmed

DEFENCE:
─────────────────────────────────────────────────────────
Phishing-resistant MFA (FIDO2, Windows Hello):
  Token is cryptographically bound to the real domain
  Evilginx2 cannot capture what's domain-bound
  AiTM attack fails completely

Conditional Access: Require compliant device:
  Even if token is stolen → attacker's device is unmanaged
  CA003 blocks access: device not compliant → BLOCKED

Continuous Access Evaluation (CAE):
  Token revoked in REAL TIME when risk detected
  Not waiting for 1-hour token expiry
  "Impossible travel detected" → token immediately invalid

Defender for Office 365:
  Detects the phishing email before Gani clicks
  Safe Links rewrites the URL and blocks at click time

Entra ID Protection:
  "Token issuer anomaly" detection
  "Impossible travel" alert fires automatically
  High-risk sign-in → CA004 blocks or requires step-up

ATTACK 4 — Illicit Consent Grant
CONCEPT:
Uses OAuth — the legitimate system behind
"Sign in with Google/Microsoft" buttons.
Attacker creates a fake app → tricks user into
granting permissions → app gets persistent OAuth
tokens WITHOUT needing user's password or MFA.

WHY IT'S THE MOST OVERLOOKED ATTACK:
→ No password stolen → password reset doesn't help
→ MFA never triggered → MFA doesn't help
→ Uses Microsoft's own legitimate OAuth system
→ Looks like a normal app authorization
→ Access persists for months
→ Survives all password changes

HOW IT TARGETS SECURECORP:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 1: Attacker registers app in THEIR tenant:
  App name: "SecureCorp HR Document Viewer"
  Publisher: Displays as "Unverified" (red flag users miss)
  
  Requests permissions:
  → Mail.Read (read all emails)
  → Mail.Send (send as the user)
  → Files.ReadWrite (read/write all OneDrive files)
  → Contacts.Read (access all contacts)

STEP 2: Sends phishing email to Chaitu (HR):
  "New HR document viewer launched.
   Connect your account to access payroll documents."
  Link opens REAL Microsoft consent screen

STEP 3: Chaitu sees REAL Microsoft consent page:
  "SecureCorp HR Document Viewer wants to:
   ✓ Read your email messages
   ✓ Send emails on your behalf
   ✓ Read and write your files
   ✓ Read your contacts"
  
  Chaitu thinks it's official → clicks ACCEPT
  (It IS a real Microsoft OAuth consent page
   The attacker just registered the app themselves)

STEP 4: Attacker's app now has OAuth token for Chaitu:
  → Reads ALL her emails (payroll data, HR decisions)
  → Searches for keywords: password, salary, invoice
  → Finds email from Hareesh with admin credentials
  → Sends emails AS Chaitu to other employees
  → Downloads all her OneDrive files
  → Access persists for 90 days minimum
  → Even if Chaitu changes her password → STILL WORKS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WHAT NAVI FINDS:
  Entra ID → Enterprise Applications → All Applications
  
  Suspicious entry:
  Name:           SecureCorp HR Document Viewer
  Publisher:      ⚠️ Unverified
  Users consented: Chaitu, Priya, Gani (3 victims)
  Permissions:    Mail.Read, Mail.Send, Files.ReadWrite
  Last active:    3 minutes ago ← being used right now!
  Created:        Yesterday ← brand new app!

IMMEDIATE RESPONSE:
  1. Revoke consent:
     Enterprise Apps → App → Users and groups → Remove all
  2. Revoke all tokens issued to this app:
     Enterprise Apps → App → Permissions → Revoke grants
  3. Check audit logs for what was accessed
  4. Notify affected users (Chaitu, Priya, Gani)
  5. Check for any emails sent AS these users

DEFENCE:
  Disable user consent entirely (most secure):
  Entra ID → Enterprise Apps → Consent and Permissions
  "Do not allow user consent"
  → ALL app consent requires admin approval
  
  Admin Consent Workflow:
  User requests app access → notification to Hareesh
  Hareesh reviews permissions → approves or denies
  Nothing gets through without admin eyes on permissions
  
  Microsoft Defender for Cloud Apps:
  Detects OAuth apps with suspicious permissions
  Alerts on: new app, broad permissions, rapid adoption

📌 SECTION 4 of 5
Reading Sign-in Logs — The SOC Analyst View

The 5 Tabs Every SOC Analyst Must Know
EVERY SIGN-IN LOG ENTRY HAS 5 TABS:
WHO ONLY LOOKS AT TAB 1 = ADMIN (troubleshooting)
WHO LOOKS AT ALL 5 = SOC ANALYST (security)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 1 — BASIC INFO:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Date/Time:        08/04/2026  09:14:32 IST
User:             gani@securecorp.com
Application:      Microsoft Teams
Client app:       Browser ← modern auth (good)
IP address:       103.21.xx.xx
Location:         Bangalore, KA, IN
Status:           Success
Error code:       0 (zero = success)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 2 — AUTHENTICATION DETAILS (Key for SOC):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Authentication requirement: Multifactor authentication
MFA result:               MFA completed
Authentication method:    Microsoft Authenticator
Steps satisfied:          Password ✅ + MFA push ✅
Token issuer:             Azure AD

What Navi looks for here:
  → "singleFactorAuthentication" = MFA not required = GAP
  → "MFA failed" = attacker couldn't get past MFA
  → "MFA skipped" = legacy auth bypassed MFA
  → SMS OTP = weak method, watch for SIM swap

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 3 — CONDITIONAL ACCESS (Critical for security):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CA001-Block-Legacy:     Not applied (modern auth — correct)
CA002-Require-MFA:      ✅ Applied — Satisfied
CA003-Admin-Compliant:  Not applicable (Gani not admin)
CA004-Block-High-Risk:  ✅ Applied — Not blocked
CA005-Block-Outside-IN: Not applied (Bangalore IP — correct)

What Navi looks for here:
  → Policy "Not applied" when it SHOULD apply = gap
  → Policy "Failed" = user couldn't satisfy requirement
  → All policies "Not applied" = something wrong with scope
  → "Error 53003" = blocked by CA = policy working ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 4 — DEVICE INFO:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Device name:     GANI-LAPTOP
OS:              Windows 11
Compliant:       Yes ✅
Managed by:      Intune ✅
Join type:       Azure AD Joined
Browser:         Chrome 123

What Navi looks for:
  → Compliant: No = device failed security baseline
  → Managed: No = personal/unmanaged device
  → New device ID = first time sign-in from this device
  → Unknown device + admin account = suspicious

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TAB 5 — SIGN-IN RISK (Entra ID P2):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Sign-in risk level:    Low
Risk state:            None
Risk detail:           None
Risk detections:       None

What Navi looks for:
  → Risk level: High = investigate immediately
  → Detection: "Anonymous IP" = Tor or VPN
  → Detection: "Impossible travel" = multiple locations
  → Detection: "Leaked credentials" = dark web exposure
  → Detection: "Password spray" = confirmed attack
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Key Error Codes for Attack Detection
ERROR CODE    MEANING                     SECURITY ACTION
──────────────────────────────────────────────────────────
0             Success                     Check all 5 tabs
50126         Wrong password              Brute force/spray
50053         Account locked              Lockout triggered
50057         Account disabled            Old account targeted
50072         MFA required (not done)     Normal step-up
50074         Strong MFA required         CA enforcing step-up
53003         Blocked by CA policy        CA working ✅
70044         Refresh token expired       Re-auth needed
70043         Session lifetime expired    Token lifetime policy
16000         Multiple accounts listed    User choosing account
──────────────────────────────────────────────────────────

50126 PATTERNS:
  Single account, many attempts = brute force
  Many accounts, 1 attempt each = password spray
  Same IP for all failures = single attacker
  Many IPs, same account = distributed attack

53003 (Blocked by CA) IS GOOD:
  This means your CA policies are WORKING
  Attacker was stopped
  But: find WHY they attempted and WHAT they knew

📌 SECTION 5 of 5
SecureCorp Breach Story — Entra ID Edition

THE COMPLETE BREACH TIMELINE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MONDAY 09:47 PM — ATTACK BEGINS:
─────────────────────────────────────────────────────────
Attacker found SecureCorp employees on LinkedIn.
Built list of 500 email addresses.
Started password spray: "Welcome@2024"
One attempt per account, 30-second intervals.

10:02 PM: gani@securecorp.com → SUCCESS
  Reason: Gani's temp password was "Welcome@2024"
  Gani joined 3 weeks ago, never changed temp password
  Gani excluded from CA002 (helpdesk ticket — MFA issue)
  Exclusion never removed → no MFA required for Gani

10:03 PM: Attacker logged into Outlook as Gani
  Searched emails: password, credential, admin, azure
  Found email from Hareesh to Gani 3 weeks ago:
  "Welcome! Your initial Azure portal temp password:
   SecureCorp@Azure2024"
  ← Hareesh sent credentials via email (bad practice!)

10:08 PM: Tried Azure Portal as Hareesh
  hareesh@securecorp.com / SecureCorp@Azure2024
  MFA triggered → attacker initiated MFA fatigue:
  47 push notifications sent to Hareesh's phone

10:31 PM: Hareesh approved MFA (watching TV, frustrated)
  Attacker has Hareesh's session token
  Hareesh is Global Admin

MONDAY 10:32 PM — ATTACKER ACTS FAST:
─────────────────────────────────────────────────────────
10:32 PM: Created backdoor account:
          svc-reporting@securecorp.com
          Made it Global Admin immediately

10:33 PM: Created app registration "ReportingApp"
          Added client secret: Never expires 🚨
          Granted: Mail.ReadWrite.All (application)
          Admin consented himself → persistent access

10:34 PM: Downloaded all SharePoint files via Graph API
          Exported all Exchange emails for 3 key users

10:35 PM: Removed MFA from Jayanth's account
          (preparing next target for easier access)

10:36 PM: Disabled CA004 (high risk block policy)
          Tried to delete sign-in logs (Entra ID prevents this)

TUESDAY 09:00 AM — NAVI STARTS SHIFT:
─────────────────────────────────────────────────────────
Opens Sentinel → 4 HIGH severity alerts:

🔴 ALERT 1: Password spray detected
   47 accounts, 1 failure each, same IP, same time window
   1 success: gani@securecorp.com
   Time: 09:47 PM - 10:02 PM

🔴 ALERT 2: New Global Admin account created
   svc-reporting@securecorp.com added to Global Admins
   Created and elevated within 2 minutes
   Created by: hareesh@securecorp.com at 10:32 PM

🔴 ALERT 3: Impossible travel — Hareesh
   Login from Bangalore at 6:30 PM (office)
   Login from Netherlands at 10:31 PM
   Physically impossible: 8000km in 4 hours

🔴 ALERT 4: Bulk data access via Graph API
   47,000 API calls in 8 minutes
   Accessing: /messages, /drive/root/children
   Account: hareesh@securecorp.com

09:05 AM: Navi calls Hareesh:
  "Did you create svc-reporting account at 10:32 PM?"
  Hareesh: "No — but I did approve an MFA..."
  Navi: "That was the attacker. We're breached."

CONTAINMENT — 15 MINUTES:
─────────────────────────────────────────────────────────
09:06 AM: Revoke all sessions — Hareesh + Gani
09:07 AM: Disable svc-reporting@securecorp.com
09:08 AM: Remove svc-reporting from Global Admins
09:09 AM: Delete ReportingApp registration + revoke tokens
09:10 AM: Reset Hareesh + Gani passwords
09:11 AM: Re-add MFA to Jayanth
09:12 AM: Block 185.220.101.45 via Named Location CA
09:13 AM: Re-enable CA004
09:14 AM: Engage Microsoft Security Response Center

WHAT PREVENTED FULL RECOVERY:
  Data was already exfiltrated before detection
  47,000 API calls downloaded files and emails
  34 minutes from initial breach to data exfiltration
  Detection happened 11 hours later

LESSONS:
  ✗ Gani's CA exclusion never removed = CA gap
    FIX: Quarterly CA exclusion audit
  ✗ Hareesh emailed credentials to Gani
    FIX: Never email credentials — use Key Vault
  ✗ MFA was push without number matching = fatigue worked
    FIX: Enable number matching before everything else
  ✗ No real-time alert on Impossible Travel
    FIX: Sentinel alert on risk level high + success
  ✗ No alert on admin actions outside business hours
    FIX: Sentinel alert on Global Admin activity post-8PM
  ✗ App registration with non-expiring secret created
    FIX: App governance policy — max 90-day secrets
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 DAY 6 — COMPLETE NOTES
╔══════════════════════════════════════════════════════════╗
║        DAY 6 — MICROSOFT ENTRA ID SECURITY              ║
║         Architecture + Attacks + Fundamentals           ║
║                   SecureCorp Ltd.                        ║
╚══════════════════════════════════════════════════════════╝

WHAT IS ENTRA ID:
─────────────────────────────────────────────────────────
Cloud identity and access management service.
Identity backbone for M365, Azure, SaaS, custom apps.
Formerly: Azure Active Directory (Azure AD).

RELATIONSHIP WITH ON-PREM AD:
  On-prem AD: Windows domain, Kerberos, internal resources
  Entra ID: Cloud apps, Azure, mobile, SaaS apps
  Hybrid: Both connected via Entra Connect (you manage this)
  PHS: Password hash synced to cloud — you know this!
  PTA: Auth forwarded to on-prem AD in real time

SECURECORP TENANT:
  securecorp.com | M365 E3 + Entra ID P2 | 500 users
  23 guests | 47 enterprise apps | Hybrid identity

THE CRITICAL DIFFERENCE FROM ON-PREM AD:
─────────────────────────────────────────────────────────
On-prem AD: Attacker needs network access first
Entra ID: login.microsoftonline.com = globally exposed
NEW PERIMETER = IDENTITY (not firewall)
Username + Password + MFA + Device + Location = Access

IDENTITY TYPES — SECURITY VIEW:
─────────────────────────────────────────────────────────
Members:
  Internal users — credential theft / MFA bypass targets
  Hareesh (GA) = highest value target

Guests:
  External — stale access risk, vendor breach impact
  11 of 23 haven't logged in 90+ days = review monthly

Service Principals:
  Non-human — often over-privileged, secrets never expire
  Most neglected attack surface in enterprise tenants

MODERN AUTH FLOW:
─────────────────────────────────────────────────────────
Email → Password → Risk AI evaluation →
CA Policies check → MFA Challenge → Tokens issued
EVERY step recorded in sign-in logs

TOKEN TYPES:
  Access Token:  1 hour   → resource access
  Refresh Token: 90 days  → gets new access tokens
  PRT:           14 days  → SSO to everything on device
  Session Cookie: browser → active session lifetime

Revoke compromised tokens:
  Entra ID → Users → Revoke sessions
  OR: Revoke-MgUserSignInSession -UserId "[email]"

4 ATTACK TYPES:
─────────────────────────────────────────────────────────
1. PASSWORD SPRAY:
   What: 1 password tried against all 500 users
   Why hard to detect: 1 attempt per account = no lockout
   Detect: Multiple users + same IP + 1 failure each
   Prevent: MFA for all (CA002) + smart lockout

2. MFA FATIGUE (Prompt Bombing):
   What: Flood user with push notifications → user approves
   Real example: Uber breach 2022 — exact method used
   Detect: MFA approved from new country/IP at odd hours
   Prevent: Number matching MFA (types a code, not just approve)
            Additional context (show app + location in push)

3. ADVERSARY IN THE MIDDLE (AiTM):
   What: Proxy captures session token AFTER MFA completes
   Tools: Evilginx2, Modlishka
   Detect: Same user, 2 IPs, seconds apart, token replay
   Prevent: Phishing-resistant MFA (FIDO2, Windows Hello)
            Require compliant device (CA03)
            Continuous Access Evaluation (CAE)

4. ILLICIT CONSENT GRANT:
   What: Fake OAuth app → user grants access → persistent token
   Why powerful: No password needed, survives password change
   Detect: New unverified app, broad permissions, rapid adoption
   Prevent: Disable user consent → require admin approval
            Microsoft Defender for Cloud Apps

SIGN-IN LOG — 5 TABS FOR SOC:
─────────────────────────────────────────────────────────
Tab 1 — Basic Info:     Who, app, IP, location, success?
Tab 2 — Auth Details:   MFA method, single vs multi factor
Tab 3 — CA Tab:         Which policies applied, pass/fail
Tab 4 — Device Info:    Managed? Compliant? Platform?
Tab 5 — Risk Tab:       AI score, specific detections

KEY ERROR CODES:
  50126 → Wrong password      = Spray or brute force
  53003 → Blocked by CA       = Policy working ✅
  50053 → Account locked      = Lockout triggered
  0     → Success             = Check all other tabs!

SECURECORP BREACH CHAIN:
─────────────────────────────────────────────────────────
Password spray → Gani (no MFA, CA exclusion gap)
→ Found Hareesh credentials in Gani's email
→ MFA fatigue attack on Hareesh (approved push)
→ Created backdoor Global Admin account
→ Created app with non-expiring secret + Mail.ReadWrite
→ Bulk data exfiltration via Graph API
→ Removed MFA from Jayanth (next target)
→ Disabled CA004 (high risk block policy)

Detection: 11 hours later (should have been 3 minutes)
By: Impossible travel alert + Admin role change alert

BREACH PREVENTION LESSONS:
  1. Remove CA exclusions as soon as issue resolved
  2. Never send credentials via email
  3. Enable number matching MFA immediately
  4. Sentinel alert: Impossible travel → P0 incident
  5. Sentinel alert: Global Admin created outside hours
  6. App governance: Max 90-day secret lifetime
  7. CA003: Require compliant device for admins

QUICK REFERENCE — WHAT EACH ATTACK BYPASSES:
─────────────────────────────────────────────────────────
Password spray:    Bypasses nothing — just guesses
                   Stopped by: MFA
MFA fatigue:       Bypasses MFA (user approves)
                   Stopped by: Number matching
AiTM:              Bypasses MFA (token stolen after)
                   Stopped by: Phishing-resistant MFA
                               Compliant device required
Consent grant:     Bypasses both password AND MFA
                   Stopped by: Admin consent required only
