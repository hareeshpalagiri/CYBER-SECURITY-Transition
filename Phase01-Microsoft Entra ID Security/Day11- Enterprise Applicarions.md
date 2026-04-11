
# PART 2 — ENTERPRISE APPLICATIONS
## *Every Tab Explained — Where Access is Managed*

---

```
LOCATION:
Entra ID → Enterprise Applications → [Your App or SaaS App]

REMEMBER THE DISTINCTION:
App Registration = Blueprint (what the app is)
Enterprise Application = Instance (who uses it, how they sign in)

For SaaS apps (Salesforce, ServiceNow):
  Only Enterprise App exists in your tenant
  You configure SSO, provisioning, access here

For custom apps:
  Both App Registration AND Enterprise App exist
  App Registration = developer configures
  Enterprise App = admin configures access
```

---

## 📌 TAB 1 — PROPERTIES

```
LOCATION:
Enterprise Applications → [App] → Properties

KEY FIELDS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Name:
  Display name of the application
  Shown in My Apps portal (myapps.microsoft.com)
  Users see this when they look for their apps

Application ID:
  Same as in App Registration
  The unique identifier

Object ID:
  Enterprise App's own object ID
  (Different from App Registration's Object ID)

Enabled for users to sign-in:
  YES / NO toggle
  
  MOST IMPORTANT FIELD ON THIS TAB:
  
  YES = Users can sign into the app
  NO  = All sign-ins blocked immediately
  
  SECURITY USE:
  → App compromised? Flip to NO instantly.
  → Blocks ALL access within seconds
  → No need to change secrets or delete app
  → Fast containment during incident response
  
  Navi's first action when app is compromised:
  Flip "Enabled for users to sign-in" to NO
  Then investigate. Then remediate. Then flip back to YES.

User assignment required:
  YES / NO toggle
  
  NO (default): ANY user in tenant can access the app
                even without being explicitly assigned
  
  YES:          ONLY users/groups explicitly assigned
                can access the app
                Everyone else gets "Not assigned" error
  
  SECURITY RECOMMENDATION:
  Always set to YES for sensitive applications
  Forces explicit access provisioning
  Prevents accidental access by all 500 users

Visible to users:
  YES = App appears in My Apps portal
  NO  = App hidden from user portal
        (but still accessible if they know the URL)
  
  Use for: Background service apps that users
           don't sign into directly

Logo:
  Displayed in My Apps portal and consent screens

Notes:
  Free text field — use for:
  → Application purpose
  → Owner contact
  → Last review date
  → Ticket number for initial provisioning
  SECURITY: Good documentation here helps
            during incident response

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 TAB 2 — OWNERS
### *(Enterprise Application level)*

```
LOCATION:
Enterprise Applications → [App] → Owners

WHAT ENTERPRISE APP OWNERS CAN DO:
  → Manage SSO configuration
  → Manage provisioning configuration
  → Assign users and groups to the app
  → Manage app-level CA policies
  → View sign-in logs for this app

DIFFERENCE FROM APP REGISTRATION OWNERS:
  App Registration Owner → modify the app's code/config
  Enterprise App Owner   → manage who uses the app

EXAMPLE IN SECURECORP:
  HR Portal - Enterprise App Owners:
  → Chaitu (HR Manager) — manages who gets access
  → Hareesh (IT Admin) — backup owner

  HR Portal - App Registration Owners:
  → Priya (Developer) — manages the code/secrets
  → Hareesh (IT Admin) — backup owner

SECURITY RULE:
  Don't put developers as Enterprise App owners
  unless they also manage access
  Separate concerns:
  → Developer = App Registration owner
  → Business owner = Enterprise App owner
```

---

## 📌 TAB 3 — ROLES AND ADMINISTRATORS
### *(Enterprise Application level)*

```
LOCATION:
Enterprise Applications → [App] → Roles and administrators

WHAT THIS SHOWS:
Similar to App Registration version but scoped to
the Enterprise Application object specifically.

CLOUD APPLICATION ADMINISTRATOR:
  Can manage this specific Enterprise Application
  Configure SSO, provisioning, user assignment
  Cannot manage Application Proxy

RELEVANT BUILT-IN ROLES:
─────────────────────────────────────────────────────────
Global Administrator:
  Full control over everything including this app

Application Administrator:
  Manage all Enterprise Apps in tenant
  Assign users, configure SSO, manage provisioning

Cloud Application Administrator:
  Same as above minus Application Proxy management

Helpdesk Administrator:
  Can reset passwords for users assigned to apps
  Cannot modify app configuration

─────────────────────────────────────────────────────────
LEAST PRIVILEGE APPROACH IN SECURECORP:
  Jayanth (End User Support) = Helpdesk Admin
  → Can manage user access assignments
  → Cannot modify SSO configuration
  → Cannot add secrets or permissions
  
  Hareesh = Application Administrator
  → Full app management
  → Use only when needed (consider PIM here too)
```

---

## 📌 TAB 4 — USERS AND GROUPS

```
LOCATION:
Enterprise Applications → [App] → Users and groups

WHAT THIS TAB CONTROLS:
Who has access to the application.
The ACCESS CONTROL layer for the Enterprise App.

REQUIRES: "User assignment required = YES" in Properties
          Otherwise this tab has no effect.

WHAT YOU SEE HERE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
List of all users and groups assigned to this app.
For each assignment:
  → User or Group name
  → Role assigned (if app roles are defined)

ADDING ASSIGNMENTS:
+ Add user/group
Search for: Chaitu → Assign role: HR.Admin → Assign
Search for: All-Staff group → Assign role: HR.SelfService

REMOVING ASSIGNMENTS:
Select user → Remove
User immediately loses access on next sign-in attempt
(or within token lifetime if already signed in)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURECORP HR PORTAL ASSIGNMENTS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
User/Group              Role
────────────────────────────────────────────────────────
Chaitu                  HR.Admin
HR-Admins-Group         HR.Admin
All-Managers-Group      HR.ReadOnly
All-Staff-Group         HR.SelfService
Gani (new joinee)       HR.SelfService (auto via group)

When Gani joins → added to All-Staff-Group
→ Automatically gets HR.SelfService access
→ No manual assignment needed

When someone leaves SecureCorp:
→ Account disabled in AD → synced to Entra ID
→ Entra ID login blocked
→ But assignment remains (for audit purposes)
→ Navi's quarterly review removes stale assignments

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURITY REVIEW PROCESS:
─────────────────────────────────────────────────────────
Quarterly Access Review (automated with Azure feature):
  Entra ID → Identity Governance → Access Reviews
  → + New access review → Select HR Portal
  → Reviewer: Chaitu (HR Manager)
  → Duration: 2 weeks
  → If reviewer doesn't respond: Remove access

Chaitu gets email:
  "Please confirm these 47 employees still need HR Portal"
  For each: Continue / Remove
  Automated removal of denied access
  Full audit trail

This is called Access Reviews — automates what would
otherwise be a manual quarterly spreadsheet exercise.
```

---

## 📌 TAB 5 — SINGLE SIGN-ON (SSO)

```
LOCATION:
Enterprise Applications → [App] → Single sign-on

WHAT THIS TAB CONTROLS:
How users authenticate to the application.
The SSO configuration is where you connect
Entra ID identity to the application's login.

FOUR SSO MODES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MODE 1 — SAML (most common for enterprise SaaS):
─────────────────────────────────────────────────────────
Used for: Salesforce, ServiceNow, AWS Console,
          legacy enterprise apps

How it works:
  User clicks Salesforce in My Apps portal
  Entra ID generates a SAML assertion (XML document)
  Signed with Entra ID's certificate
  Sent to Salesforce
  Salesforce trusts Entra ID → user is logged in
  No separate Salesforce password needed

Key SAML configuration fields:

BASIC SAML CONFIGURATION:
  Identifier (Entity ID):
    Unique ID for your app in SAML federation
    Usually provided by the app vendor
    Example: https://securecorp.salesforce.com
    
  Reply URL (Assertion Consumer Service URL):
    Where Entra ID sends the SAML response
    Example: https://securecorp.salesforce.com/saml
    
  Sign-on URL:
    The login page of the app
    Example: https://securecorp.salesforce.com/login
    
  Relay State:
    Where to redirect user after SAML login
    Optional — vendor-specific

ATTRIBUTES AND CLAIMS:
  What user information Entra ID includes in SAML token
  Default claims:
    emailaddress → user.mail
    name         → user.userprincipalname
    givenname    → user.givenname
    surname      → user.surname
    
  Custom claims (if Salesforce needs a specific field):
    salesforceProfileId → user.extensionattribute1

SAML SIGNING CERTIFICATE:
  Entra ID signs SAML assertions with this cert
  The app validates the signature using the public key
  Certificate expires → SSO breaks if not renewed
  
  SECURITY CRITICAL:
  → Track expiry dates (default 3 years)
  → Renew proactively 60 days before expiry
  → Download new cert → upload to vendor portal
  → Never let SAML cert expire in production

MODE 2 — OIDC / OAuth (for modern apps):
─────────────────────────────────────────────────────────
Used for: Modern SaaS apps, custom apps,
          anything built after 2015

Based on OAuth 2.0 and OpenID Connect
More modern than SAML, simpler to configure
Uses JSON Web Tokens (JWT) instead of XML

Configuration usually auto-populated when adding
from the Azure App Gallery

MODE 3 — PASSWORD-BASED SSO (legacy support):
─────────────────────────────────────────────────────────
Used for: Apps that don't support SAML or OIDC
          Legacy apps that only have a login form

How it works:
  Entra ID stores username/password for the app
  When user accesses app → Entra ID auto-fills credentials
  Using browser extension (My Apps Secure Sign-in Extension)

SECURITY NOTE:
  This is essentially a password manager in Entra ID
  Passwords are stored encrypted but they EXIST
  Not as secure as SAML/OIDC
  Use only for truly legacy apps with no SSO support
  Modern apps should always use SAML or OIDC

MODE 4 — LINKED (SSO via URL only):
─────────────────────────────────────────────────────────
Used for: Apps that already have SSO configured elsewhere
          Custom intranet links in My Apps portal

Just creates a link in My Apps portal
No Entra ID authentication involved
User clicks link → goes to URL
Entra ID not involved in actual authentication

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURITY BENEFITS OF SSO:
─────────────────────────────────────────────────────────
Without SSO:
  500 users × 10 apps = 5000 separate passwords
  Users reuse passwords → one breach = all apps breached
  IT resets passwords 100x/day

With SSO:
  One Entra ID identity → access all 10 apps
  MFA happens once → applies to all apps
  User leaves → disable Entra ID account → all 500 apps
  Single point of control for access
  Conditional Access applies to all SSO apps
```

---

## 📌 TAB 6 — PROVISIONING

```
LOCATION:
Enterprise Applications → [App] → Provisioning

WHAT THIS TAB CONTROLS:
Automatic creation, update, and deletion of user accounts
in the target application based on Entra ID directory.

SIMPLE EXPLANATION:
  Without provisioning:
  Gani joins SecureCorp.
  HR tells IT to create his account in Entra ID.
  IT also manually creates his account in Salesforce.
  And ServiceNow. And GitHub. And Zoom. And 10 other apps.
  One by one. Takes days. Error-prone.

  With provisioning:
  Gani joins → account created in Entra ID.
  Entra ID automatically creates accounts in all 10 apps.
  Using SCIM protocol (System for Cross-domain
  Identity Management).
  Gani leaves → Entra ID disables → all 10 apps disabled.
  Automatically. Instantly.

PROVISIONING MODES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AUTOMATIC (SCIM):
  Entra ID pushes users to the app automatically
  App must support SCIM 2.0 protocol
  Most modern SaaS apps support this
  (Salesforce, ServiceNow, GitHub, Slack, Zoom etc.)

  Setup:
  → Provisioning mode: Automatic
  → Admin credentials: API endpoint + token from Salesforce
  → Test connection → Mappings → Start provisioning

MANUAL:
  Entra ID does nothing automatically
  IT admin creates accounts in target app manually
  Only use if app doesn't support SCIM

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ATTRIBUTE MAPPINGS:
What fields sync from Entra ID to the target app:

  Entra ID attribute    → Target app attribute
  userPrincipalName     → userName
  displayName           → displayName
  mail                  → emails[primary].value
  givenName             → name.givenName
  surname               → name.familyName
  jobTitle              → title
  department            → urn:ietf:params:scim:...:department
  telephoneNumber       → phoneNumbers[work].value

SCOPING FILTER:
  Which users to provision?
  Option 1: Sync all users
  Option 2: Sync only assigned users
             (those in Users and Groups tab)

  SECURITY RECOMMENDATION:
  Always use "Sync only assigned users and groups"
  Don't provision all 500 users to every app
  Only provision those who actually need access

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROVISIONING LOGS:
  Shows every provisioning action with result
  Gani created in Salesforce: ✅ Success
  Priya updated in ServiceNow: ✅ Success
  BackupSvc account failed: ❌ Error — investigate

  SECURITY VALUE:
  Navi checks provisioning logs when:
  → User can't access an app (provisioning failed?)
  → Investigating access after an incident
  → Verifying leaver's accounts were deprovisioned

DEPROVISIONING (CRITICAL):
  When user is disabled in Entra ID:
  → Provisioning engine detects the change
  → Pushes disable/delete to all connected apps
  → Usually within 40 minutes

  This is why proper offboarding matters:
  Disable Entra ID account →
  Salesforce disabled ✅
  ServiceNow disabled ✅
  GitHub disabled ✅
  Zoom disabled ✅
  All 10 apps → automatically
```

---

## 📌 TAB 7 — APPLICATION PROXY

```
LOCATION:
Enterprise Applications → [App] → Application Proxy

WHAT THIS TAB CONTROLS:
Secure remote access to on-premises applications
WITHOUT a VPN.

SECURECORP SCENARIO:
SecureCorp has an on-premises SharePoint server.
Before Application Proxy:
  Remote users needed VPN to access it.
  VPN = complex, slow, requires client software.
  VPN credentials = separate attack surface.

After Application Proxy:
  Users access: https://sharepoint.securecorp.com
  Looks like a normal HTTPS website.
  No VPN needed. Works on any device. Any browser.
  Entra ID authenticates the user first.
  THEN the request is forwarded to on-prem SharePoint.
  On-prem server never directly exposed to internet.

HOW IT WORKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Components:
  Application Proxy Service (in Azure cloud)
  Application Proxy Connector (agent on-premises)

Flow:
  Step 1: User accesses external URL
          https://intranet.securecorp.com
  
  Step 2: Entra ID authenticates user
          MFA required (CA policy applies here too)
  
  Step 3: Application Proxy Service receives request
          Forwards to on-prem Connector
          Via outbound HTTPS tunnel (no inbound ports!)
  
  Step 4: Connector forwards to internal app
          http://intranet.securecorp.local
  
  Step 5: Response travels back through tunnel
          User sees the internal app seamlessly

SECURITY BENEFITS:
  ✅ No inbound firewall ports needed
     Connector makes outbound connections only
     Your internal network never directly exposed
  
  ✅ Entra ID authentication BEFORE reaching the app
     Attacker can't reach the app without valid credentials
     + MFA + Conditional Access
  
  ✅ Works with Pre-authentication
     Entra ID checks access before forwarding request
     Unauthorized requests never reach internal servers
  
  ✅ No VPN client required
     HTTPS only — works everywhere

CONFIGURATION:
─────────────────────────────────────────────────────────
Internal URL:     http://sharepoint.securecorp.local
External URL:     https://sharepoint.securecorp.com
                  (auto-generated: 
                   https://sharepoint-securecorp.msappproxy.net)

Pre-authentication:
  Azure Active Directory (recommended — most secure)
  Passthrough (less secure — app handles its own auth)

Connector Group:
  Select which on-prem connector handles this app
  Can have multiple connectors for redundancy

Translate URLs in:
  Headers: YES (translates internal URLs to external)
  Application body: YES (fixes internal links)
```

---

## 📌 TAB 8 — SELF-SERVICE

```
LOCATION:
Enterprise Applications → [App] → Self-service

WHAT THIS TAB CONTROLS:
Allows users to request access to the app themselves.
Without IT involvement for every request.

WITHOUT SELF-SERVICE:
  Gani needs access to HR Portal.
  Gani emails Jayanth (helpdesk).
  Jayanth creates ticket.
  Jayanth assigns to Hareesh.
  Hareesh adds Gani to Users and Groups tab.
  2-3 business days later: Gani has access.

WITH SELF-SERVICE:
  Gani opens My Apps portal (myapps.microsoft.com)
  Clicks: "Request access to HR Portal"
  Chaitu (HR Manager) gets approval email
  Chaitu clicks Approve
  Gani gets access immediately
  No IT ticket needed.

CONFIGURATION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Allow users to request access to this application:
  YES / NO

Who is allowed to approve access to this application:
  → Specific users or groups as approvers
  → Example: Chaitu (HR Manager) for HR Portal
  → Example: Hareesh (IT Admin) for Azure Portal

To which group should assigned users be added:
  → Create a group → "HR-Portal-SelfService-Users"
  → Self-service approved users go into this group
  → Group is assigned to app in Users and Groups tab

Allow self-service group assignment:
  Can users manage their own group memberships?
  Usually NO for security-sensitive apps

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURITY CONSIDERATIONS:
─────────────────────────────────────────────────────────
RISK 1 — Wrong approver:
  If approver is too permissive (approves everything)
  anyone can get access to sensitive apps
  → Choose approvers who understand business need

RISK 2 — No approval required:
  Can configure self-service WITHOUT approval
  User requests → immediately granted
  Good for: Low-risk apps (company canteen menu)
  Bad for:  Any app with sensitive data

RISK 3 — No review of self-service assignments:
  Once access granted via self-service
  It stays until revoked
  → Include self-service groups in quarterly Access Reviews
  → Same review process as manually assigned users

SECURECORP SELF-SERVICE POLICY:
  Low sensitivity apps (Zoom, Teams backgrounds):
  → Self-service enabled, no approval required
  
  Medium sensitivity (most internal apps):
  → Self-service enabled, manager approval required
  
  High sensitivity (Finance, HR, Azure Portal):
  → No self-service, Hareesh manually assigns
  → Access Review every quarter
```

---

## 🔒 SECURITY SUMMARY — APP REGISTRATIONS & ENTERPRISE APPS

```
THE COMPLETE ATTACK SURFACE MAP:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ATTACK 1 — Leaked Client Secret:
  Where: Certificates & Secrets tab
  Risk: App authenticates without MFA or CA
  Defence: Azure Key Vault, rotate secrets, use certs

ATTACK 2 — Over-permissioned App:
  Where: API Permissions tab
  Risk: Compromised app = access to all data
  Defence: Least privilege, quarterly permission audit

ATTACK 3 — Illicit Consent Grant:
  Where: API Permissions + Enterprise App consent
  Risk: Malicious app gets user's data silently
  Defence: Disable user consent, admin consent only

ATTACK 4 — Orphaned App Registration:
  Where: Owners tab
  Risk: Unmanaged app, stale secrets, no monitoring
  Defence: Every app has named owners, quarterly audit

ATTACK 5 — Broad User Assignment:
  Where: Users and Groups tab
  Risk: More users than necessary have access
  Defence: Require assignment, Access Reviews quarterly

ATTACK 6 — Never-Expiring Secrets:
  Where: Certificates & Secrets tab
  Risk: Compromised secret usable forever
  Defence: Maximum 12-month expiry, Key Vault, rotation

ATTACK 7 — Misconfigured Redirect URIs:
  Where: Authentication tab
  Risk: Token hijacking via open redirect
  Defence: Exact URIs only, HTTPS only, no wildcards

ATTACK 8 — Implicit Grant Enabled:
  Where: Authentication tab
  Risk: Tokens exposed in browser history/logs
  Defence: Disable implicit grant, use PKCE flow

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NAVI'S QUARTERLY APP AUDIT CHECKLIST:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ App Registrations with no owners → assign or delete
□ Client secrets expiring in < 60 days → rotate now
□ Client secrets with "never expires" → set expiry
□ Apps with Application permissions (Mail.Read etc.) → review
□ Apps with admin-consented permissions → still needed?
□ Implicit grant enabled → disable if not needed
□ Redirect URIs with wildcards → replace with exact URIs
□ Apps with "allow public client flows" → still needed?
□ Enterprise Apps with no user assignment required → review
□ Self-service approvers → still correct people?
□ SAML signing certificates expiring < 60 days → renew
□ Provisioning errors in last 30 days → investigate
□ App Proxy connectors → health check, up to date?
□ Guest users in enterprise apps → still valid access?
```

---

## 🧪 DAY 9 LABS

```
LAB 1 — Explore App Registrations (20 min)
Platform: Entra ID Portal → App Registrations
─────────────────────────────────────────────────────────
1. Open App Registrations → All applications
2. Click any existing app → go through EVERY tab
3. Note:
   → When do client secrets expire?
   → What permissions are granted?
   → Who are the owners?
   → What redirect URIs are configured?
4. Find any secrets expiring soon → flag for rotation

LAB 2 — Audit Enterprise Apps for Risky Consents (20 min)
Platform: Entra ID → Enterprise Applications
─────────────────────────────────────────────────────────
1. Enterprise Applications → All Applications
2. Filter: Application type = Enterprise Applications
3. For each app → Permissions tab
   → What permissions are granted?
   → Are any Application permissions present?
   → Who consented?
4. Find any with Mail.Read / Files.ReadWrite at app level
   → Document as high risk

LAB 3 — Create a Test App Registration (30 min)
Platform: Entra ID → App Registrations
─────────────────────────────────────────────────────────
Create: SecureCorp-Test-App

1. App Registrations → + New registration
   Name: SecureCorp-Test-App
   Account type: Single tenant
   Redirect URI: Web → https://localhost/auth

2. Authentication tab:
   Verify implicit grant = unchecked
   Verify public client flows = No

3. Certificates & Secrets:
   Create new secret → 6 months expiry
   Copy value immediately
   Note: you'll never see it again

4. API Permissions:
   Add → Microsoft Graph → Delegated → User.Read
   Note: no admin consent needed for User.Read

5. Expose an API:
   Set App ID URI: api://securecorp-test-app
   Add scope: TestData.Read

6. App Roles:
   Create role: TestApp.Admin

7. Enterprise Application (auto-created):
   Find it in Enterprise Applications
   Properties → User assignment required → Yes
   Users and Groups → Add yourself

LAB 4 — Run App Security Audit via PowerShell (20 min)
Platform: Kali Linux / PowerShell with Graph module
─────────────────────────────────────────────────────────
# Install Microsoft Graph PowerShell if needed:
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect:
Connect-MgGraph -Scopes "Application.Read.All"

# Find apps with expiring secrets:
Get-MgApplication -All |
  ForEach-Object {
    $app = $_
    $app.PasswordCredentials |
    Where-Object {
      $_.EndDateTime -lt (Get-Date).AddDays(60)
    } | ForEach-Object {
      [PSCustomObject]@{
        AppName    = $app.DisplayName
        SecretName = $_.DisplayName
        Expiry     = $_.EndDateTime
        DaysLeft   = ($_.EndDateTime - (Get-Date)).Days
      }
    }
  } |
  Sort-Object DaysLeft |
  Format-Table -AutoSize

# Any results = secrets expiring soon = rotate now
```

---

## 📋 DAY 9 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║    DAY 9 — APP REGISTRATIONS & ENTERPRISE APPLICATIONS   ║
║                    SecureCorp Ltd.                       ║
╚══════════════════════════════════════════════════════════╝

CORE CONCEPT:
─────────────────────────────────────────────────────────
App Registration  = Blueprint (what the app IS)
                    Developer configures
                    Authentication, permissions, secrets

Enterprise App    = Instance (who USES the app)
                    Admin configures
                    SSO, provisioning, user assignment

Custom apps:      Both exist (App Reg → auto creates Ent App)
SaaS apps:        Enterprise App only (vendor owns App Reg)

APP REGISTRATION TABS:
─────────────────────────────────────────────────────────
Authentication:
  Platform configs (Web/SPA/Mobile)
  Redirect URIs — exact only, HTTPS only, no wildcards
  Implicit grant — disable for new apps
  Supported account types — single tenant for internal
  Public client flows — disable unless needed

Certificates & Secrets:
  Client secrets — app's password
    → Azure Key Vault only, never in code
    → Max 12 months expiry, never "never expires"
    → Rotate proactively
  Certificates — better than secrets
    → Private key never leaves Key Vault
    → Mathematical proof of identity
  Federated credentials — best option
    → No secrets at all
    → For GitHub Actions, DevOps pipelines

API Permissions:
  Delegated — acts as signed-in user (safer)
  Application — acts as itself, tenant-wide (powerful)
  HIGH RISK: Directory.ReadWrite, Mail.ReadWrite,
             User.ReadWrite.All, RoleManagement.ReadWrite
  Admin consent required for application permissions
  Least privilege — minimum permissions needed

Expose an API:
  App ID URI — unique identifier for your API
  Scopes — what your API allows callers to do
  Authorized client apps — pre-consent for trusted apps

App Roles:
  Custom roles your app defines
  Assigned to users/groups in Enterprise App
  Appear in JWT token → app code uses for authz
  Access control in Entra ID not in app code

Owners:
  Who can modify the app registration
  Maximum 2 owners, review quarterly
  Remove when people leave the team

ENTERPRISE APPLICATION TABS:
─────────────────────────────────────────────────────────
Properties:
  Enabled for sign-in → NO = instant access block
  User assignment required → YES = explicit access control
  Most important emergency control

Users and Groups:
  Who has access → requires "User assignment required = YES"
  Assign by group for scalability
  Quarterly Access Reviews

Single Sign-On:
  SAML — enterprise SaaS (Salesforce, ServiceNow)
    Configure: Entity ID, Reply URL, Sign-on URL
    Certificates expire → track and renew
    Attributes/claims map user data to app
  OIDC — modern apps (recommended)
  Password-based — legacy apps only
  Linked — URL bookmark only

Provisioning (SCIM):
  Automatic user account creation in target apps
  Attribute mappings (Entra ID → target app fields)
  Scope: assigned users only (not all users)
  Deprovisioning: user disabled → apps disabled automatically
  Check provisioning logs for errors

Application Proxy:
  On-prem app access without VPN
  Connector on-prem makes outbound HTTPS tunnel
  Entra ID auth BEFORE reaching internal app
  No inbound firewall ports needed

Self-service:
  Users request their own app access
  Configured approvers (business owners) approve
  Access Reviews should include self-service groups
  No approval required = only for low-sensitivity apps

ATTACK SURFACE:
─────────────────────────────────────────────────────────
Leaked secret:          Key Vault + rotation + certs
Over-permissioned app:  Least privilege + quarterly audit
Illicit consent:        Admin consent only, disable user consent
Orphaned app:           Named owners, quarterly audit
Broad user access:      Assignment required + Access Reviews
Never-expiring secrets: Max 12 months + proactive rotation
Misconfigured redirects: Exact URIs, HTTPS only
Implicit grant enabled:  Disable, use PKCE flow

QUARTERLY AUDIT CHECKLIST:
─────────────────────────────────────────────────────────
App Registrations:
□ No owners → assign or delete
□ Secrets expiring < 60 days → rotate
□ Secrets "never expire" → set expiry
□ Application permissions → review
□ Implicit grant enabled → disable
□ Wildcard redirect URIs → replace with exact
□ Public client flows → disable if not needed

Enterprise Applications:
□ SAML certs expiring < 60 days → renew
□ Provisioning errors → investigate
□ App Proxy connector health → check
□ Guest user assignments → still valid?
□ Self-service approvers → still correct?
□ Access Reviews → run quarterly

POWERSHELL — FIND EXPIRING SECRETS:
─────────────────────────────────────────────────────────
Get-MgApplication -All |
  ForEach-Object {
    $app = $_
    $app.PasswordCredentials |
    Where-Object { $_.EndDateTime -lt (Get-Date).AddDays(60) } |
    ForEach-Object {
      [PSCustomObject]@{
        AppName  = $app.DisplayName
        Secret   = $_.DisplayName
        Expiry   = $_.EndDateTime
        DaysLeft = ($_.EndDateTime - (Get-Date)).Days
      }
    }
  } | Sort-Object DaysLeft | Format-Table -AutoSize

LABS COMPLETED:
─────────────────────────────────────────────────────────
□ Explored existing App Registrations — all tabs
□ Audited Enterprise Apps for risky permissions
□ Created SecureCorp-Test-App from scratch
□ Ran PowerShell secret expiry audit
```

---

> 💡 **Architect's Truth:** *"In every cloud security audit I've run, app registrations and enterprise applications are the most neglected attack surface in the entire tenant. Passwords changed. MFA enabled. Conditional Access configured. But app registrations — secrets from 2019, never rotated, with Mail.ReadWrite.All application permissions, granted by an admin who left two years ago. That is where the breach happens. That is what attackers find when they get initial access to a developer account. Build the habit now: quarterly app audit, Key Vault for every secret, least privilege for every permission. It takes 2 hours a quarter and prevents the breach that takes 2 months to recover from."*

---
