# SSO (Single Sign-On)

SSO is a very common HLD interview topic, especially when discussing Authentication, OAuth, JWT, and Identity Providers.

---

# What is SSO?

**Single Sign-On (SSO)** means:

> Login once and access multiple applications without logging in again.

Example:

```text
Login to Google
      |
      +--> Gmail
      +--> YouTube
      +--> Google Drive
      +--> Google Photos
```

You authenticate once.

All applications trust Google's authentication.

---

# Problem Without SSO

Suppose a company has:

```text
HR Portal
Payroll Portal
Employee Dashboard
Wiki
Jira
```

Without SSO:

```text
Login to HR
Login to Payroll
Login to Wiki
Login to Jira
```

Multiple usernames/passwords.

Poor user experience.

---

# With SSO

```text
Login Once
      |
      v
Identity Provider (IdP)
      |
      +--> HR Portal
      +--> Payroll Portal
      +--> Wiki
      +--> Jira
```

One login gives access to all trusted applications.

---

# Real-Life Analogy

Imagine a tech conference.

Without SSO:

```text
Room A -> Show ID
Room B -> Show ID
Room C -> Show ID
Room D -> Show ID
```

With SSO:

```text
Verify identity once
Get conference badge
```

Then:

```text
Show badge everywhere
```

The badge is your SSO credential.

---

# Key Components

## 1. User

```text
You
```

---

## 2. Identity Provider (IdP)

Authenticates users.

Examples:

```text
Google
Microsoft Entra ID (Azure AD)
Okta
Auth0
Keycloak
```

---

## 3. Service Provider (SP)

Applications users want to access.

Examples:

```text
Gmail
Slack
Jira
Salesforce
Confluence
```

---

# SSO Architecture

```text
                +----------------+
                | Identity Provider |
                | (Google/Okta)     |
                +----------------+
                        ^
                        |
                Authentication
                        |
                        v

+----------+     +----------+     +----------+
| App 1    |     | App 2    |     | App 3    |
| Jira     |     | Slack    |     | Confluence|
+----------+     +----------+     +----------+
```

All applications trust the Identity Provider.

---

# SSO Login Flow

Suppose user opens:

```text
https://jira.company.com
```

---

### Step 1

User requests Jira.

```text
User -> Jira
```

---

### Step 2

Jira sees:

```text
User not logged in
```

Redirects to IdP.

```text
Jira -> Okta
```

---

### Step 3

User authenticates.

```text
Username
Password
MFA
```

---

### Step 4

Okta creates authentication token.

```text
JWT / SAML Assertion
```

---

### Step 5

User redirected back.

```text
Okta -> Jira
```

---

### Step 6

Jira validates token.

```text
User authenticated
```

Access granted.

---

# What Happens When User Opens Another App?

User now opens:

```text
Slack
```

Slack redirects to Okta.

But Okta already has a session.

```text
User already authenticated
```

No login screen shown.

Slack immediately gets token.

This is the "single sign-on" experience.

---

# Why SSO Works

The secret is:

```text
Central Authentication
```

Instead of every application storing passwords:

```text
All apps trust one Identity Provider
```

---

# Protocols Used in SSO

Two protocols are commonly asked in interviews.

---

# 1. SAML

(Security Assertion Markup Language)

Older enterprise standard.

Uses:

```text
XML
```

Flow:

```text
User
  |
  v
Application
  |
Redirect
  |
  v
Identity Provider
  |
SAML Assertion
  |
  v
Application
```

Common in:

```text
Large enterprises
Banks
Government systems
```

Examples:

```text
Okta + Salesforce
ADFS + Enterprise Apps
```

---

# 2. OAuth2 + OpenID Connect (OIDC)

Modern approach.

Uses:

```text
JSON
JWT
```

instead of XML.

Common for:

```text
Google Login
Microsoft Login
GitHub Login
```

---

# SAML vs OIDC

| Feature         | SAML       | OIDC        |
| --------------- | ---------- | ----------- |
| Format          | XML        | JSON/JWT    |
| Age             | Older      | Modern      |
| Mobile Friendly | No         | Yes         |
| API Friendly    | No         | Yes         |
| Complexity      | High       | Lower       |
| Common Today    | Enterprise | Modern Apps |

---

# SSO Using OAuth/OIDC

Example:

```text
Login with Google
```

Flow:

```text
User
 |
 v
Application
 |
Redirect
 |
 v
Google
 |
Authenticate
 |
 v
ID Token (JWT)
 |
 v
Application
```

Application trusts Google.

User logs in once.

---

# Benefits of SSO

### Better User Experience

One login.

```text
No repeated passwords
```

---

### Better Security

Centralized authentication.

Can enforce:

```text
MFA
Password Policies
Device Policies
```

---

### Easier User Management

Employee joins company:

```text
Create account once
```

Employee leaves:

```text
Disable account once
```

Access removed everywhere.

---

# Drawbacks of SSO

### Single Point of Failure

If IdP fails:

```text
Nobody can log in
```

---

### Bigger Security Impact

If attacker compromises IdP:

```text
Many applications affected
```

---

### Complexity

Requires trust relationships between applications and IdP.

---

# SSO in Microservices

A common HLD architecture:

```text
User
  |
  v
Identity Provider
(Okta/Auth0/Keycloak)
  |
  v
JWT
  |
  v
API Gateway
  |
  +--> User Service
  +--> Order Service
  +--> Payment Service
```

User logs in once.

JWT is used across services.

---

# SSO vs OAuth vs JWT

### JWT

```text
Token format
```

Example:

```text
Header.Payload.Signature
```

---

### OAuth2

```text
Authorization framework
```

Defines:

```text
How tokens are issued
```

---

### SSO

```text
User experience / architecture pattern
```

Allows:

```text
One login
Multiple applications
```

---

# Interview One-Liner

> SSO (Single Sign-On) allows users to authenticate once with a centralized Identity Provider (such as Okta, Google, or Microsoft Entra ID) and then access multiple applications without logging in again. Modern SSO is typically implemented using OAuth 2.0 + OpenID Connect with JWT tokens, while older enterprise systems often use SAML.
