# 🔵 PHASE 2 — DAY 9
# App Registrations & Enterprise Applications — Complete Deep Dive
### *SecureCorp Ltd. | The Identity Layer Every Attacker Targets*

---

## 🧭 DAY 9 MINDSET

```
WHAT YOU KNOW FROM ADMIN WORK:
─────────────────────────────────────────────────────────
You've created app registrations.
You've seen Enterprise Applications in the portal.
You know they're related but not exactly how.

WHAT WE'RE ADDING TODAY:
─────────────────────────────────────────────────────────
Every setting you've configured has a security story.
Every permission you granted has an attack surface.
Every secret you created has an expiry someone forgot.

Today we go through EVERY tab, EVERY setting,
with full explanation of what it does, why it matters,
and what happens when it's misconfigured.

This is one of the most overlooked attack surfaces
in enterprise Microsoft environments.
Attackers love app registrations because:
→ Nobody monitors them
→ They have broad permissions
→ Their secrets never expire
→ They survive user account changes
→ They give persistent access without human credentials
```

---

## 🏗️ FOUNDATION — App Registration vs Enterprise Application

```
THE MOST IMPORTANT CONCEPT OF TODAY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

APP REGISTRATION = The Blueprint (the recipe)
ENTERPRISE APPLICATION = The Instance (the cooked meal)

SIMPLE ANALOGY:
─────────────────────────────────────────────────────────
App Registration is like a BLUEPRINT of a building.
It defines:
→ What the building does
→ What permissions it needs
→ How people authenticate to it
→ Who owns it

Enterprise Application is like the ACTUAL BUILDING
standing in your city (your tenant).
It defines:
→ Who gets access to it
→ How they sign in (SSO)
→ Who is provisioned from your directory
→ How it integrates with your org

─────────────────────────────────────────────────────────
WHEN EACH IS CREATED:
─────────────────────────────────────────────────────────
App Registration:
  You build a CUSTOM app for SecureCorp
  (internal tool, API, automation script)
  → You create the App Registration
  → It automatically creates an Enterprise App
     in YOUR tenant

Enterprise Application (only):
  You connect a THIRD PARTY SaaS app
  (Salesforce, ServiceNow, GitHub, Zoom)
  → Microsoft creates the Enterprise App
     from the published app template
  → No App Registration in your tenant
     (it lives in the vendor's tenant)

─────────────────────────────────────────────────────────
RELATIONSHIP IN SECURECORP:
─────────────────────────────────────────────────────────
Hareesh builds internal HR portal:
  → Creates App Registration "SecureCorp-HR-Portal"
  → Entra ID auto-creates Enterprise App
     "SecureCorp-HR-Portal" in the tenant
  → App Registration = define what it is
  → Enterprise App = manage who uses it

SecureCorp subscribes to Salesforce:
  → Hareesh adds Salesforce from Gallery
  → Enterprise App "Salesforce" appears
  → No App Registration (Salesforce owns that)
  → Enterprise App = manage SSO, provisioning, access
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

# PART 1 — APP REGISTRATIONS
## *Every Tab Explained — Security First*

---

## 📌 TAB 1 — AUTHENTICATION

```
LOCATION:
Entra ID → App Registrations → [Your App] → Authentication

WHAT THIS TAB CONTROLS:
How users and services authenticate TO your application.
The most security-critical configuration in App Registrations.
```

### 1.1 — Platform Configurations

```
WHAT IT IS:
Defines HOW your app receives authentication responses
from Entra ID after a user signs in.

THREE PLATFORM TYPES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PLATFORM 1 — WEB (for server-side web apps):
─────────────────────────────────────────────────────────
Redirect URIs:
  Where Entra ID sends the auth code/token after login
  Example: https://hrportal.securecorp.com/auth/callback

  SECURITY RISK:
  If redirect URI is too broad (wildcard) or wrong:
  → Attacker registers app with your redirect URI
  → Tricks user into auth flow
  → Token sent to attacker's redirect URI
  → This is called "Open Redirect" or "Token Hijacking"

  SECURE CONFIGURATION:
  ✅ Exact URIs only — no wildcards
  ✅ HTTPS only — never HTTP in production
  ✅ Remove unused redirect URIs
  ✅ Never use localhost in production

Front-channel logout URL:
  Where Entra ID notifies your app when user signs out
  from another app in the same session (Single Sign-Out)

PLATFORM 2 — SINGLE PAGE APPLICATION (SPA):
─────────────────────────────────────────────────────────
Used for JavaScript apps running in the browser
(React, Angular, Vue apps)

Key difference from Web platform:
  SPA uses Authorization Code flow with PKCE
  (Proof Key for Code Exchange)
  → More secure for browser-based apps
  → No client secret needed (browser can't keep secrets)
  → PKCE prevents authorization code interception

Redirect URIs: Same security rules as Web platform

PLATFORM 3 — MOBILE AND DESKTOP:
─────────────────────────────────────────────────────────
For native apps on Windows, iOS, Android, macOS

Uses custom URI schemes:
  Example: myapp://auth/callback
  Example: msal{appid}://auth

Recommended for native apps:
  msal{clientId}://auth
  (Microsoft Authentication Library standard)

PUBLIC CLIENT FLOWS:
  Found under "Advanced settings" on this tab
  "Allow public client flows" = YES/NO

  Public client = app that CANNOT keep a secret
  (mobile apps, desktop apps, scripts)
  They authenticate without a client secret

  SECURITY RISK IF ENABLED UNNECESSARILY:
  → Enables device code flow, ROPC flow
  → ROPC (Resource Owner Password Credential):
    App gets username + password directly
    No MFA triggered
    No CA policies evaluated
    = Complete authentication bypass if misused
  → Should be DISABLED unless specifically needed
```

### 1.2 — Implicit Grant and Hybrid Flows

```
WHAT IT IS:
Older OAuth flows that are now considered LESS SECURE.

IMPLICIT FLOW:
  Access tokens or ID tokens returned DIRECTLY
  in the redirect URI (in the URL fragment #token=...)
  Not recommended — tokens visible in browser history,
  server logs, referrer headers

HYBRID FLOW:
  Returns authorization code AND tokens together
  More flexible but more complex

CURRENT RECOMMENDATION (Microsoft):
  ✅ Use Authorization Code Flow with PKCE instead
  ❌ Do NOT enable implicit grant for new apps

SECURECORP RULE:
  Both "Access tokens" and "ID tokens" checkboxes
  under Implicit Grant = UNCHECKED for all new apps
  Only enable if supporting legacy apps that require it

SECURITY IMPACT IF LEFT ENABLED:
  Tokens can be stolen from:
  → Browser history
  → Referrer headers sent to third-party CDNs
  → Browser extensions reading URL bar
  → Shoulder surfing in logs
```

### 1.3 — Supported Account Types

```
WHAT IT IS:
Defines WHO can authenticate to your application.

FOUR OPTIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OPTION 1 — Single tenant (most common, most secure):
"Accounts in this organizational directory only
 (SecureCorp only — Single tenant)"

  Only users from securecorp.com tenant can sign in
  Perfect for: internal HR portal, internal tools
  Security: Strongest isolation

OPTION 2 — Multi-tenant:
"Accounts in any organizational directory
 (Any Azure AD tenant — Multitenant)"

  ANY user from ANY Azure AD tenant can sign in
  Perfect for: SaaS apps you sell to other companies
  Security risk: Anyone with an Entra ID account
               from any company can attempt sign-in
  USE ONLY if your app is intentionally multi-tenant

OPTION 3 — Multi-tenant + personal accounts:
"Accounts in any organizational directory
 and personal Microsoft accounts"

  Adds xbox.com, outlook.com, hotmail.com accounts
  Perfect for: Consumer apps (Xbox games, OneDrive)
  Security risk: Massive user pool can authenticate
  NEVER use for enterprise internal applications

OPTION 4 — Personal accounts only:
"Personal Microsoft accounts only"

  Only personal Microsoft accounts (Xbox, Outlook.com)
  Rarely used in enterprise

SECURECORP RULE:
  ALL internal apps = Single tenant ONLY
  Multi-tenant only when explicitly building a SaaS product
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 TAB 2 — CERTIFICATES & SECRETS

```
LOCATION:
Entra ID → App Registrations → [Your App]
→ Certificates & secrets

WHAT THIS TAB CONTROLS:
The CREDENTIALS that your application uses to
prove its identity to Entra ID.

Think of it as the APPLICATION'S password.
Except it's used by code, not humans.
```

### 2.1 — Client Secrets

```
WHAT IS A CLIENT SECRET:
A password string that your application sends
along with its App ID to authenticate to Entra ID.

Format: A long random string
Example: ABC8Q~xyzAbcDeFgHiJkLmNoPqRsTuVwXyZ12345

HOW IT WORKS:
─────────────────────────────────────────────────────────
Your app (e.g. SecureCorp-HR-Portal) needs to call
Microsoft Graph API to read user profiles.

Step 1: App sends to Entra ID:
        "I am SecureCorp-HR-Portal
         Here is my App ID: abc123
         Here is my secret: XYZ789..."

Step 2: Entra ID verifies App ID + Secret match
Step 3: Entra ID issues access token
Step 4: App uses token to call Graph API

─────────────────────────────────────────────────────────
CREATING A CLIENT SECRET:
─────────────────────────────────────────────────────────
+ New client secret
Description: "Production-HR-Portal-2026"
             (always descriptive — never "secret1")
Expires: 6 months / 12 months / 18 months / 24 months
         or Custom date

⚠️  CRITICAL: Copy the secret VALUE immediately.
              It is shown ONLY ONCE.
              After you leave the page = gone forever.
              You must create a new one.

─────────────────────────────────────────────────────────
THE #1 SECURITY PROBLEM WITH CLIENT SECRETS:
─────────────────────────────────────────────────────────
They get hardcoded into source code.

Common violations Navi finds:
  In Python scripts: password = "ABC8Q~xyz..."
  In config files:   clientSecret: "ABC8Q~xyz..."
  In GitHub repos:   AZURE_CLIENT_SECRET=ABC8Q~xyz...
  In Teams messages: "here's the secret: ABC8Q~..."
  In emails:         "I'm sending you the app secret"

Once in GitHub = compromised. Always. Forever.
GitHub's secret scanning catches some. Not all.
Dark web databases have thousands of leaked secrets.

SECURECORP RULES FOR SECRETS:
  ✅ Store ONLY in Azure Key Vault
  ✅ App reads secret from Key Vault at runtime
  ✅ Maximum expiry 12 months (never "never expires")
  ✅ Descriptive names with date: "Prod-HRPortal-Apr2026"
  ✅ Delete old/expired secrets immediately
  ✅ Rotate secrets proactively before expiry
  ✅ One secret per environment (dev/staging/prod)
  ❌ Never in source code
  ❌ Never in config files committed to Git
  ❌ Never in emails or Teams messages
  ❌ Never set to "never expire"

─────────────────────────────────────────────────────────
HOW ATTACKER ABUSES A LEAKED SECRET:
─────────────────────────────────────────────────────────
Step 1: Find secret on GitHub/Pastebin/dark web
Step 2: Authenticate as the application:
        POST https://login.microsoftonline.com/
             {tenantId}/oauth2/v2.0/token
        Body:
          client_id={appId}
          client_secret={stolenSecret}
          scope=https://graph.microsoft.com/.default
          grant_type=client_credentials

Step 3: Get access token (no MFA — apps don't do MFA)
Step 4: Use token to access whatever the app can access
        If app has Mail.Read → read all emails
        If app has User.ReadWrite.All → modify all users
        If app has Directory.ReadWrite → change AD

No username. No password. No MFA. No CA policies.
Just the App ID and the stolen secret.
This is why leaked secrets are so catastrophic.
```

### 2.2 — Certificates (Better Than Secrets)

```
WHAT IS A CERTIFICATE:
A public/private key pair used instead of a secret.
Much more secure than client secrets.

HOW IT WORKS:
─────────────────────────────────────────────────────────
Your app has a CERTIFICATE (public + private key)
Private key stays on your server/Key Vault
Public key uploaded to App Registration

Authentication flow:
  App creates a JWT (JSON Web Token)
  Signs it with the PRIVATE KEY
  Sends the signed JWT to Entra ID
  Entra ID verifies signature using the PUBLIC KEY
  Issues access token

─────────────────────────────────────────────────────────
WHY CERTIFICATES ARE BETTER THAN SECRETS:
─────────────────────────────────────────────────────────
Secret:      A string → can be copied, emailed, committed
Certificate: A mathematical key pair → private key
             NEVER leaves the secure store

Even if attacker sees the public key (it's in Azure) —
they cannot authenticate without the private key.
Private key stays in Azure Key Vault or HSM.

─────────────────────────────────────────────────────────
HOW TO UPLOAD A CERTIFICATE:
─────────────────────────────────────────────────────────
Step 1: Generate a certificate
  # PowerShell:
  $cert = New-SelfSignedCertificate `
    -Subject "CN=SecureCorp-HR-Portal" `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyExportPolicy Exportable `
    -KeySpec Signature `
    -KeyLength 2048 `
    -HashAlgorithm SHA256 `
    -NotAfter (Get-Date).AddYears(1)

Step 2: Export public key as .cer file
  Export-Certificate `
    -Cert $cert `
    -FilePath "C:\certs\hrportal-public.cer"

Step 3: Upload .cer to App Registration
  Certificates & secrets → Certificates → Upload certificate
  Upload hrportal-public.cer

Step 4: Store private key securely
  Import to Azure Key Vault
  App references Key Vault — never handles key directly

SECURECORP RULE:
  All production app registrations = certificates
  Client secrets only for development/testing
```

### 2.3 — Federated Identity Credentials

```
WHAT IS IT:
The newest and most secure authentication method.
No secret. No certificate. No credential to steal.

Used for: GitHub Actions, Azure DevOps, Kubernetes,
          other identity providers deploying to Azure

HOW IT WORKS (GitHub Actions example):
─────────────────────────────────────────────────────────
Traditional (secrets in GitHub):
  AZURE_CLIENT_SECRET = "ABC8Q~..."  ← stored in GitHub
  Any repo admin can see this
  If GitHub is breached = secret exposed

Federated Identity (no secrets):
  GitHub's OIDC provider issues a token during workflow run
  That token presented to Entra ID
  Entra ID trusts GitHub's OIDC → issues Azure token
  No secret ever stored anywhere

Attacker has nothing to steal.
Even full GitHub repo access = no credentials found.

SETUP:
  App Registration → Certificates & secrets
  → Federated credentials → + Add credential
  Select: GitHub Actions deploying Azure resources
  Organization: securecorp-github
  Repository: infrastructure-repo
  Entity: Branch → main
  → Add

Now GitHub Actions authenticates to Azure without secrets.
```

---

## 📌 TAB 3 — API PERMISSIONS

```
LOCATION:
Entra ID → App Registrations → [Your App] → API permissions

WHAT THIS TAB CONTROLS:
What resources and data your application can ACCESS.
This is the SCOPE of the application's power.
The most abused tab in App Registrations.
```

### 3.1 — Permission Types

```
TWO FUNDAMENTAL TYPES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TYPE 1 — DELEGATED PERMISSIONS:
─────────────────────────────────────────────────────────
App acts ON BEHALF OF a signed-in user.
The app can only do what the USER can do.

Example:
  Priya signs into SecureCorp HR Portal.
  Portal has delegated permission: Mail.Read
  Portal can read Priya's emails.
  But NOT Bharath's emails (Priya can't read those).

Flow:
  User signs in → User's permissions + App's permissions
  = intersection is what the app can do

Use case: Any app where a human user is involved
          Web apps, mobile apps, desktop apps

SECURITY NOTE:
  Delegated permissions are SAFER
  Because they're limited to what the user can access
  If user is a standard user → app is also limited

TYPE 2 — APPLICATION PERMISSIONS:
─────────────────────────────────────────────────────────
App acts AS ITSELF with no user involved.
Background services, daemons, automation.

Example:
  SecureCorp-BackupAgent runs at 2 AM.
  No user signed in.
  Has Application permission: Mail.Read
  Can read ALL users' emails across the entire tenant.

Flow:
  App authenticates with its own credentials
  (secret or certificate)
  No user context — app has full access to what's granted

Use case: Background jobs, automation, API-to-API calls

SECURITY NOTE:
  Application permissions are MUCH MORE POWERFUL
  They apply tenant-wide
  A compromised app with User.ReadWrite.All
  can modify EVERY user in the org
  These require admin consent — cannot be user-consented

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3.2 — Microsoft Graph Permissions (Most Important)

```
MICROSOFT GRAPH = the API for everything Microsoft 365.
Every permission here = access to some Microsoft service.

HIGH-RISK PERMISSIONS NAVI WATCHES FOR:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CRITICAL — NEVER grant unless absolutely required:
─────────────────────────────────────────────────────────
Directory.ReadWrite.All
  → Read and write entire Azure AD directory
  → Create/delete users, groups, modify any object
  → Application permission = modify all of Entra ID

User.ReadWrite.All
  → Read and update all user profiles
  → Change passwords, disable accounts
  → Application permission = control all 500 users

Mail.ReadWrite (Application)
  → Read and write ALL mailboxes in the org
  → 500 users' emails, full read/write
  → Used in data exfiltration attacks

Files.ReadWrite.All (Application)
  → Read/write ALL SharePoint and OneDrive files
  → All company documents accessible
  → Ransomware vector if app is compromised

RoleManagement.ReadWrite.Directory
  → Assign or remove Entra ID roles
  → Can make accounts Global Admin
  → Extremely dangerous

AppRoleAssignment.ReadWrite.All
  → Modify app role assignments
  → Escalate app permissions
  → Privilege escalation vector

─────────────────────────────────────────────────────────
MEDIUM RISK — Use with care:
─────────────────────────────────────────────────────────
Mail.Read (Application)
  → Read ALL mailboxes read-only
  → Data exfiltration risk

User.Read.All (Application)
  → Read all user profiles
  → Reconnaissance tool for attackers

Group.ReadWrite.All
  → Modify all group memberships
  → Add anyone to any group

─────────────────────────────────────────────────────────
LOW RISK — Generally acceptable:
─────────────────────────────────────────────────────────
User.Read (Delegated)
  → Read signed-in user's own profile
  → Standard for any app with login

Mail.Read (Delegated)
  → Read signed-in user's own emails
  → Limited to that one user

Calendars.Read (Delegated)
  → Read signed-in user's calendar

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3.3 — Admin Consent

```
WHAT IS ADMIN CONSENT:
Some permissions are so powerful that a regular user
cannot grant them — an admin must explicitly approve.

TWO CONSENT TYPES:
─────────────────────────────────────────────────────────
USER CONSENT:
  User can grant the permission themselves
  Only for delegated permissions that affect only them
  Example: User.Read (read my own profile)
  User sees consent screen → clicks Accept → done

ADMIN CONSENT:
  Only a Global Admin or Privileged Role Admin can grant
  Required for:
  → All Application permissions (no user context)
  → High-risk delegated permissions (Mail.Read all)
  → Any permission that affects other users or the org

  Admin consent screen shows:
  "Grant admin consent for SecureCorp"
  Once clicked → ALL users in tenant implicitly consented
  They don't see individual consent screens anymore

─────────────────────────────────────────────────────────
THE SECURITY PROBLEM — OVER-CONSENTED APPS:
─────────────────────────────────────────────────────────
Scenario in SecureCorp:
  Developer Priya creates an HR Portal app.
  Needs User.Read.All to show employee list.
  Also adds Mail.ReadWrite "just in case".
  Asks Hareesh to click "Grant admin consent".
  Hareesh trusts Priya → clicks without reviewing.

  Now:
  HR Portal has Mail.ReadWrite.All application permission
  With admin consent
  If HR Portal is compromised → attacker reads/writes
  every email in the entire company

SECURECORP RULE:
  Hareesh reviews EVERY permission before admin consent
  Principle of least privilege — minimum permissions needed
  Mail.ReadWrite.All when you only need User.Read = REJECTED
  Review apps quarterly — remove permissions no longer needed

─────────────────────────────────────────────────────────
CHECKING CONSENTED PERMISSIONS — NAVI'S AUDIT QUERY:
─────────────────────────────────────────────────────────
# PowerShell — find all apps with high-risk permissions:
Connect-MgGraph -Scopes "Application.Read.All"

Get-MgServicePrincipal -All |
  ForEach-Object {
    $sp = $_
    Get-MgServicePrincipalAppRoleAssignment `
      -ServicePrincipalId $sp.Id |
    Where-Object {
      $_.PrincipalType -eq "ServicePrincipal"
    } | ForEach-Object {
      [PSCustomObject]@{
        AppName    = $sp.DisplayName
        Permission = $_.AppRoleId
        GrantedTo  = $_.PrincipalId
      }
    }
  }

# GUI alternative:
# Entra ID → Enterprise Apps → [App] → Permissions
# Shows all granted permissions clearly
```

---

## 📌 TAB 4 — EXPOSE AN API

```
LOCATION:
Entra ID → App Registrations → [Your App] → Expose an API

WHAT THIS TAB CONTROLS:
When YOUR app is the RESOURCE that other apps call.
You are defining what YOUR API offers and
who is allowed to call it.

SECURECORP SCENARIO:
SecureCorp builds a custom API: SecureCorp-Data-API
Other internal apps (HR Portal, Finance App) call this API.
"Expose an API" defines what those calling apps can do.
```

### 4.1 — Application ID URI

```
WHAT IT IS:
A unique identifier for your API.
Used as the "scope" prefix when calling your API.

DEFAULT: api://{applicationId}
CUSTOM:  https://api.securecorp.com
         api://securecorp-data-api

HOW IT'S USED:
When HR Portal requests access to SecureCorp-Data-API:
  scope = api://securecorp-data-api/Employee.Read

The scope tells Entra ID:
  "I want access to SecureCorp-Data-API
   specifically the Employee.Read permission"

SECURITY NOTE:
  Custom URIs must be verified domains
  api://{appId} is always safe (globally unique)
  Never use the same URI as another app
```

### 4.2 — Scopes (What your API offers)

```
SCOPES DEFINE WHAT YOUR API ALLOWS CALLERS TO DO:

Example for SecureCorp-Data-API:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SCOPE 1:
  Scope name:    Employee.Read
  Who can consent: Admins and users
  Admin display name: Read employee data
  Admin description:  Allows the app to read
                      employee records on behalf of user
  User display name:  Read your employee data
  State:         Enabled

  Used by: HR Portal to show employee list
  Call: GET /api/employees → returns employee data

SCOPE 2:
  Scope name:    Employee.Write
  Who can consent: Admins only
  Admin display name: Modify employee data
  Admin description:  Allows modification of employee records
  State:         Enabled

  Used by: HR Admin Portal only
  Restricted to admin consent = standard users can't grant

SCOPE 3:
  Scope name:    Salary.Read
  Who can consent: Admins only
  Admin display name: Read salary information
  Admin description:  Access to confidential salary data
  State:         Enabled

  Only finance apps should have this scope
  Admin consent required = Hareesh must explicitly approve

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURITY PRINCIPLE:
  Design scopes around least privilege
  Fine-grained scopes = better control
  Employee.Read and Employee.Write are better than
  Employee.All (too broad)
```

### 4.3 — Authorized Client Applications

```
WHAT IT IS:
Pre-authorised apps that can call your API
WITHOUT users seeing a consent prompt.

SECURECORP EXAMPLE:
  SecureCorp-Data-API pre-authorises HR-Portal:
  → HR Portal calls the API silently
  → Users don't see "HR Portal wants to access Data API"
  → Seamless experience

HOW TO SET UP:
  Add a client application → enter HR Portal's App ID
  Select which scopes it's pre-authorised for
  → Employee.Read ✅
  → Employee.Write ❌ (requires explicit consent)

SECURITY NOTE:
  Only pre-authorise apps YOU own and trust
  Pre-authorisation skips consent screen
  If malicious app gets pre-authorised = silent access
  Audit this list regularly
```

---

## 📌 TAB 5 — APP ROLES

```
LOCATION:
Entra ID → App Registrations → [Your App] → App roles

WHAT THIS TAB CONTROLS:
Custom roles that YOUR application defines.
Different from Entra ID directory roles.
These roles live inside your app.

SECURECORP HR PORTAL EXAMPLE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

APP ROLE 1:
  Display name:  HR.Admin
  Value:         HR.Admin
                 (what your app code checks for)
  Description:   Full admin access to HR portal
  Allowed member types: Users/Groups + Applications
  State: Enabled

  Assigned to: Chaitu (HR Manager), HR-Admins group

APP ROLE 2:
  Display name:  HR.ReadOnly
  Value:         HR.ReadOnly
  Description:   Read-only view of HR data
  Allowed member types: Users/Groups
  State: Enabled

  Assigned to: All managers group (read their team's data)

APP ROLE 3:
  Display name:  HR.SelfService
  Value:         HR.SelfService
  Description:   Employees access own records only
  Allowed member types: Users/Groups
  State: Enabled

  Assigned to: All employees (default role)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
HOW YOUR APP USES ROLES:
─────────────────────────────────────────────────────────
When user signs in → their token contains:
  "roles": ["HR.Admin"]

Your app code checks:
  if (user.roles.includes("HR.Admin")) {
    showAdminPanel();
  } else {
    showReadOnlyView();
  }

If user has no role assigned:
  They cannot access the app at all (if app requires role)
  This is "require app role assignment" feature

─────────────────────────────────────────────────────────
SECURITY BENEFIT:
  Access control lives in Entra ID — not in app code
  Hareesh manages who gets HR.Admin via Enterprise App
  No code deployment needed to change access
  Full audit trail of role assignments
  Roles appear in sign-in logs and audit logs
```

---

## 📌 TAB 6 — OWNERS

```
LOCATION:
Entra ID → App Registrations → [Your App] → Owners

WHAT THIS TAB CONTROLS:
Who can modify this app registration.

WHAT OWNERS CAN DO:
  → Add/remove client secrets
  → Add/remove certificates
  → Modify redirect URIs
  → Change API permissions
  → Add/remove other owners
  → Delete the app registration

WHAT OWNERS CANNOT DO:
  → Grant admin consent (needs Global Admin)
  → Assign users to the Enterprise App
  → Manage SSO configuration

SECURITY RISKS:
─────────────────────────────────────────────────────────
RISK 1 — No owners set:
  App Registration created → developer leaves company
  Account disabled → nobody owns the app
  App continues to work with its existing secrets
  Nobody notices when secrets expire or need rotating
  → Orphaned app = unmanaged attack surface

RISK 2 — Too many owners:
  Every developer added as owner
  Any one of them can add a new secret
  Malicious insider adds secret → exfiltrates data
  → Over-privileged ownership

SECURECORP RULE:
  Maximum 2 owners per app registration
  → Primary: App developer/team lead
  → Secondary: Hareesh (IT Admin as backup)
  Review owners quarterly
  Remove owners when they leave the team
  Service accounts should NOT be owners
```

---

## 📌 TAB 7 — ROLES AND ADMINISTRATORS
### *(at App Registration level)*

```
LOCATION:
Entra ID → App Registrations → [Your App]
→ Roles and administrators

WHAT THIS TAB SHOWS:
Entra ID directory roles that have permissions
over THIS specific app registration.

ROLES VISIBLE HERE:
─────────────────────────────────────────────────────────
Application Administrator:
  Can manage ALL app registrations in the tenant
  Can add/remove secrets, certificates, permissions
  Can grant admin consent

Cloud Application Administrator:
  Same as above but CANNOT manage Application Proxy

App Registration Owner:
  Scoped to this specific app only
  (set in Owners tab — shows here for visibility)

─────────────────────────────────────────────────────────
SECURITY CONSIDERATION:
  Application Administrator is a powerful role
  Often granted to developers "so they can manage apps"
  But Application Administrator can:
  → Add secrets to ANY app registration
  → Read credentials of ANY app
  → Effectively impersonate ANY application

  Principle of least privilege:
  → Use "Owner" role scoped to specific app
  → Not "Application Administrator" for the whole tenant
  → Unless the person truly needs tenant-wide app management
```

---

## 📌 TAB 8 — BRANDING AND PROPERTIES

```
LOCATION:
Entra ID → App Registrations → [Your App]
→ Branding and properties

WHAT THIS TAB CONTROLS:
Visual identity and metadata of your app.
Less security-critical but important for compliance
and user trust.

FIELDS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Name:
  Display name shown on consent screens
  Shown in sign-in logs
  "SecureCorp HR Portal" not "App1" or "TestApp"
  SECURITY: Clear names help users identify
            legitimate apps vs phishing apps

Logo:
  Your company logo shown on consent screen
  Helps users verify it's a legitimate app
  Attackers sometimes fake logos for phishing apps

Homepage URL:
  The main URL of your app
  Used in enterprise app listings

Terms of service URL:
  Legal requirement for multi-tenant apps
  Users see this link on consent screen

Privacy statement URL:
  Legal requirement for apps processing personal data
  GDPR compliance requirement

Publisher domain:
  Verified domain shown on consent screen
  ✅ Verified: securecorp.com (green checkmark)
  ❌ Unverified: shows warning on consent screen
  SECURITY: Users should be suspicious of
            apps showing "Unverified" on consent screen

Application ID (read-only):
  The unique GUID for your app
  Used in all API calls and authentication flows
  This is NOT a secret — it's public

Object ID (read-only):
  Internal Entra ID object identifier
  Used in PowerShell/Graph API calls

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PUBLISHER VERIFICATION — CRITICAL FOR TRUST:
─────────────────────────────────────────────────────────
Unverified app consent screen shows:
  "⚠️  Unverified
   This app hasn't been verified by Microsoft.
   [AppName] wants permission to..."
  Users should DENY unknown unverified apps

Verified app consent screen shows:
  "✅ securecorp.com
   [AppName] wants permission to..."
  Clear indication of publisher

HOW TO VERIFY:
  App Registration → Branding and properties
  → Publisher domain → Verify domain
  Requires: TXT DNS record on securecorp.com
  Microsoft verifies you own the domain
  Once verified → shown on all consent screens
```

---

