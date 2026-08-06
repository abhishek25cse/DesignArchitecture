# OAuth 2.0 — A Comprehensive Tutorial

## Table of Contents

1. [Introduction](#1-introduction)
2. [Why OAuth 2.0?](#2-why-oauth-20)
3. [Core Concepts](#3-core-concepts)
4. [OAuth 2.0 Roles](#4-oauth-20-roles)
5. [Tokens](#5-tokens)
6. [Grant Types](#6-grant-types)
   - [Authorization Code Grant](#61-authorization-code-grant)
   - [Authorization Code with PKCE](#62-authorization-code-with-pkce)
   - [Client Credentials Grant](#63-client-credentials-grant)
   - [Device Authorization Grant](#64-device-authorization-grant)
   - [Refresh Token Grant](#65-refresh-token-grant)
7. [Scopes](#7-scopes)
8. [Real-World Example: "Login with Google"](#8-real-world-example-login-with-google)
9. [Real-World Example: Machine-to-Machine API Access](#9-real-world-example-machine-to-machine-api-access)
10. [Security Best Practices](#10-security-best-practices)
11. [Common Pitfalls](#11-common-pitfalls)
12. [OAuth 2.0 vs OAuth 1.0 vs OpenID Connect](#12-oauth-20-vs-oauth-10-vs-openid-connect)
13. [References](#13-references)

---

## 1. Introduction

**OAuth 2.0** (Open Authorization 2.0) is an industry-standard **authorization framework** defined in [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749). It allows a third-party application to obtain **limited access** to an HTTP service on behalf of a resource owner, without exposing the resource owner's credentials.

> **Key Insight:** OAuth 2.0 is about **authorization** (what you can do), not **authentication** (who you are). OpenID Connect (OIDC) is the authentication layer built on top of OAuth 2.0.

### Real-World Analogy

Think of OAuth 2.0 like a **hotel key card**:
- You (the guest) check in at the front desk (authorization server).
- The front desk verifies your identity and gives you a key card (access token).
- The key card opens only your room and the gym (scoped access), not every room in the hotel.
- The key card expires after your stay (token expiration).
- You never gave the hotel your house keys (no password sharing).

---

## 2. Why OAuth 2.0?

### The Problem (Before OAuth)

```
┌──────────┐    username + password     ┌──────────────┐
│  Third    │ ──────────────────────────>│   Google     │
│  Party    │                            │   (Resource) │
│  App      │ <─────────────────────────│              │
└──────────┘    full access to account   └──────────────┘
```

**Problems with direct credential sharing:**
- Third-party app gets **full access** to your account
- You **cannot revoke** access without changing your password
- If the third-party app is compromised, your password is **leaked**
- No **granular permissions** — it's all or nothing

### The Solution (With OAuth 2.0)

```
┌──────────┐   limited scoped token     ┌──────────────┐
│  Third    │ ──────────────────────────>│   Google     │
│  Party    │                            │   (Resource) │
│  App      │ <─────────────────────────│              │
└──────────┘   only requested data       └──────────────┘
```

**Benefits:**
- **No password sharing** — credentials stay with the identity provider
- **Scoped access** — apps only get permissions they need
- **Revocable** — users can revoke access anytime without changing passwords
- **Time-limited** — tokens expire automatically

---

## 3. Core Concepts

| Concept              | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| **Resource Owner**   | The user who owns the data and grants access                                |
| **Client**           | The application requesting access to the user's data                        |
| **Authorization Server** | Issues tokens after authenticating the resource owner (e.g., Google, Auth0) |
| **Resource Server**  | The API that hosts the protected resources (e.g., Google Drive API)         |
| **Access Token**     | A credential used to access protected resources (short-lived)               |
| **Refresh Token**    | A credential used to obtain new access tokens (long-lived)                  |
| **Scope**            | Defines the extent of access granted (e.g., `read:email`, `write:files`)    |
| **Redirect URI**     | The URL where the authorization server sends the user after granting access |
| **Authorization Code** | A short-lived code exchanged for an access token                          |

---

## 4. OAuth 2.0 Roles

### Component Architecture Diagram

```mermaid
graph TB
    subgraph "OAuth 2.0 Ecosystem"
        RO["Resource Owner (End User)"]
        CLIENT["Client Application (Web App / Mobile App)"]
        AS["Authorization Server (Google, Auth0, Okta)"]
        RS["Resource Server (API / Protected Data)"]
    end

    RO -->|"1. Initiates login"| CLIENT
    CLIENT -->|"2. Redirects to authorize"| AS
    RO -->|"3. Authenticates & consents"| AS
    AS -->|"4. Returns auth code"| CLIENT
    CLIENT -->|"5. Exchanges code for token"| AS
    AS -->|"6. Returns access token"| CLIENT
    CLIENT -->|"7. Requests data with token"| RS
    RS -->|"8. Returns protected data"| CLIENT

    style RO fill:#e1f5fe,stroke:#0288d1
    style CLIENT fill:#f3e5f5,stroke:#7b1fa2
    style AS fill:#e8f5e9,stroke:#388e3c
    style RS fill:#fff3e0,stroke:#f57c00
```

### Role Descriptions

| Role | Example | Responsibility |
|------|---------|---------------|
| **Resource Owner** | A Gmail user | Owns the data; grants/denies consent |
| **Client** | A task management app | Requests access; uses tokens to call APIs |
| **Authorization Server** | `accounts.google.com` | Authenticates user; issues/validates tokens |
| **Resource Server** | `www.googleapis.com` | Hosts protected resources; validates tokens on each request |

---

## 5. Tokens

### Access Token

- **Purpose:** Grants access to protected resources
- **Lifetime:** Short (typically 15 min – 1 hour)
- **Format:** Often a JWT (JSON Web Token) or opaque string

#### JWT Access Token Structure

```
Header.Payload.Signature
```

```json
// Header
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "abc123"
}

// Payload
{
  "iss": "https://auth.example.com",
  "sub": "user-12345",
  "aud": "https://api.example.com",
  "exp": 1719756000,
  "iat": 1719752400,
  "scope": "read:profile write:posts",
  "client_id": "my-web-app"
}

// Signature
RSASHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), privateKey)
```

### Refresh Token

- **Purpose:** Obtain a new access token without re-authenticating the user
- **Lifetime:** Long (days to months)
- **Storage:** Must be stored securely (never in browser `localStorage`)

### Token Lifecycle Diagram

```mermaid
stateDiagram-v2
    [*] --> Issued: Authorization Server issues token
    Issued --> Active: Token is valid
    Active --> Active: API calls succeed
    Active --> Expired: TTL exceeded
    Expired --> Refreshed: Refresh token exchange
    Refreshed --> Active: New access token
    Expired --> Revoked: User revokes access
    Active --> Revoked: User or Admin revokes
    Revoked --> [*]
```

```mermaid
sequenceDiagram
    participant Client
    participant AuthServer as Authorization Server
    participant API as Resource Server

    Note over Client,API: Token Lifecycle

    Client->>AuthServer: Request access token
    AuthServer-->>Client: Access Token (expires in 1h) + Refresh Token

    loop While access token is valid
        Client->>API: API request + Access Token
        API-->>Client: Protected resource
    end

    Client->>API: API request + Expired Access Token
    API-->>Client: 401 Unauthorized

    Client->>AuthServer: Refresh Token -- Request new Access Token
    AuthServer-->>Client: New Access Token (+ optionally new Refresh Token)

    Client->>API: API request + New Access Token
    API-->>Client: Protected resource
```

---

## 6. Grant Types

### 6.1 Authorization Code Grant

**Best for:** Server-side web applications with a backend.

This is the **most common and most secure** grant type for user-facing applications.

#### Sequence Diagram

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant WebApp as Web App<br/>(Backend)
    participant AuthServer as Authorization Server
    participant API as Resource Server

    User->>Browser: 1. Click "Login with Google"
    Browser->>WebApp: 2. GET /login
    WebApp->>Browser: 3. Redirect to AuthServer /authorize
    Note right of WebApp: response_type=code<br/>client_id=abc<br/>redirect_uri=https://app.com/callback<br/>scope=openid email profile<br/>state=xyz123

    Browser->>AuthServer: 4. GET /authorize?...
    AuthServer->>User: 5. Show login & consent screen
    User->>AuthServer: 6. Enter credentials & approve
    AuthServer->>Browser: 7. Redirect to callback with code
    Note right of AuthServer: https://app.com/callback<br/>?code=AUTH_CODE_HERE<br/>&state=xyz123

    Browser->>WebApp: 8. GET /callback?code=AUTH_CODE&state=xyz123
    WebApp->>WebApp: 9. Verify state matches

    WebApp->>AuthServer: 10. POST /token (server-to-server)
    Note right of WebApp: grant_type=authorization_code<br/>code=AUTH_CODE_HERE<br/>client_id=abc<br/>client_secret=SECRET<br/>redirect_uri=https://app.com/callback

    AuthServer->>AuthServer: 11. Validate code, client_id, secret, redirect_uri
    AuthServer-->>WebApp: 12. Access Token + Refresh Token + ID Token

    WebApp->>API: 13. GET /userinfo with Bearer token
    API-->>WebApp: 14. User profile data
    WebApp-->>Browser: 15. Logged in! Show dashboard
```

#### Code Example (Python / Flask)

```python
# app.py — OAuth 2.0 Authorization Code Flow with Google
import os
import requests
from flask import Flask, redirect, request, session, url_for

app = Flask(__name__)
app.secret_key = os.environ["FLASK_SECRET_KEY"]

# OAuth 2.0 Configuration
GOOGLE_CLIENT_ID     = os.environ["GOOGLE_CLIENT_ID"]
GOOGLE_CLIENT_SECRET = os.environ["GOOGLE_CLIENT_SECRET"]
REDIRECT_URI         = "http://localhost:5000/callback"
AUTH_URL              = "https://accounts.google.com/o/oauth2/v2/auth"
TOKEN_URL            = "https://oauth2.googleapis.com/token"
USERINFO_URL         = "https://www.googleapis.com/oauth2/v3/userinfo"


@app.route("/login")
def login():
    """Step 1: Redirect user to Google's authorization endpoint."""
    import secrets
    state = secrets.token_urlsafe(32)
    session["oauth_state"] = state

    params = {
        "client_id":     GOOGLE_CLIENT_ID,
        "redirect_uri":  REDIRECT_URI,
        "response_type": "code",
        "scope":         "openid email profile",
        "state":         state,
        "access_type":   "offline",       # request a refresh token
        "prompt":        "consent",       # force consent screen
    }
    auth_redirect = f"{AUTH_URL}?{'&'.join(f'{k}={v}' for k, v in params.items())}"
    return redirect(auth_redirect)


@app.route("/callback")
def callback():
    """Step 2: Handle the callback, exchange code for tokens."""
    # Verify state to prevent CSRF
    if request.args.get("state") != session.pop("oauth_state", None):
        return "State mismatch — possible CSRF attack", 403

    # Check for errors
    if "error" in request.args:
        return f"Authorization error: {request.args['error']}", 400

    # Exchange authorization code for tokens
    code = request.args["code"]
    token_response = requests.post(TOKEN_URL, data={
        "grant_type":    "authorization_code",
        "code":          code,
        "redirect_uri":  REDIRECT_URI,
        "client_id":     GOOGLE_CLIENT_ID,
        "client_secret": GOOGLE_CLIENT_SECRET,
    })
    tokens = token_response.json()

    if "error" in tokens:
        return f"Token error: {tokens['error_description']}", 400

    # Store tokens in session (use a secure store in production)
    session["access_token"]  = tokens["access_token"]
    session["refresh_token"] = tokens.get("refresh_token")

    # Fetch user profile
    headers = {"Authorization": f"Bearer {tokens['access_token']}"}
    user_info = requests.get(USERINFO_URL, headers=headers).json()

    return f"""
    <h1>Welcome, {user_info['name']}!</h1>
    <p>Email: {user_info['email']}</p>
    <img src="{user_info['picture']}" alt="Profile picture" />
    <br/><a href="/logout">Logout</a>
    """


@app.route("/logout")
def logout():
    session.clear()
    return redirect("/")


if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

---

### 6.2 Authorization Code with PKCE

**Best for:** Single-page apps (SPAs), mobile apps, and native apps that **cannot securely store a client secret**.

PKCE (Proof Key for Code Exchange, pronounced "pixie") adds an extra layer of security by using a dynamically generated `code_verifier` and `code_challenge`.

#### Sequence Diagram

```mermaid
sequenceDiagram
    actor User
    participant SPA as Single-Page App<br/>(Browser)
    participant AuthServer as Authorization Server
    participant API as Resource Server

    Note over SPA: Generate PKCE pair:<br/>code_verifier = random(43-128 chars)<br/>code_challenge = BASE64URL(SHA256(code_verifier))

    User->>SPA: 1. Click "Login"
    SPA->>AuthServer: 2. GET /authorize
    Note right of SPA: response_type=code<br/>client_id=spa-app<br/>redirect_uri=http://localhost:3000/callback<br/>scope=openid profile<br/>state=random_state<br/>code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1...<br/>code_challenge_method=S256

    AuthServer->>User: 3. Show login & consent
    User->>AuthServer: 4. Authenticate & approve
    AuthServer->>SPA: 5. Redirect with auth code
    Note right of AuthServer: /callback?code=AUTH_CODE&state=random_state

    SPA->>SPA: 6. Verify state

    SPA->>AuthServer: 7. POST /token
    Note right of SPA: grant_type=authorization_code<br/>code=AUTH_CODE<br/>client_id=spa-app<br/>redirect_uri=http://localhost:3000/callback<br/>code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
    Note over AuthServer: Server computes:<br/>BASE64URL(SHA256(code_verifier))<br/>and compares to stored code_challenge

    AuthServer-->>SPA: 8. Access Token (+ ID Token)

    SPA->>API: 9. API call with Access Token
    API-->>SPA: 10. Protected data
```

#### Code Example (JavaScript — SPA)

```javascript
// oauth-pkce.js — Authorization Code + PKCE for Single-Page Apps

// --- PKCE Helper Functions ---
function generateRandomString(length) {
  const charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~';
  const values = crypto.getRandomValues(new Uint8Array(length));
  return Array.from(values, v => charset[v % charset.length]).join('');
}

async function generateCodeChallenge(codeVerifier) {
  const encoder = new TextEncoder();
  const data = encoder.encode(codeVerifier);
  const digest = await crypto.subtle.digest('SHA-256', data);
  return btoa(String.fromCharCode(...new Uint8Array(digest)))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}

// --- Configuration ---
const config = {
  clientId:     'your-spa-client-id',
  authUrl:      'https://auth.example.com/authorize',
  tokenUrl:     'https://auth.example.com/oauth/token',
  redirectUri:  'http://localhost:3000/callback',
  scope:        'openid profile email',
};

// --- Step 1: Initiate Login ---
async function login() {
  const codeVerifier  = generateRandomString(64);
  const codeChallenge = await generateCodeChallenge(codeVerifier);
  const state         = generateRandomString(32);

  // Store verifier and state for later verification
  sessionStorage.setItem('pkce_code_verifier', codeVerifier);
  sessionStorage.setItem('oauth_state', state);

  const params = new URLSearchParams({
    response_type:         'code',
    client_id:             config.clientId,
    redirect_uri:          config.redirectUri,
    scope:                 config.scope,
    state:                 state,
    code_challenge:        codeChallenge,
    code_challenge_method: 'S256',
  });

  window.location.href = `${config.authUrl}?${params}`;
}

// --- Step 2: Handle Callback ---
async function handleCallback() {
  const params       = new URLSearchParams(window.location.search);
  const code         = params.get('code');
  const returnedState = params.get('state');

  // Verify state
  if (returnedState !== sessionStorage.getItem('oauth_state')) {
    throw new Error('State mismatch — possible CSRF attack');
  }

  // Exchange code for token using the code_verifier
  const codeVerifier = sessionStorage.getItem('pkce_code_verifier');

  const response = await fetch(config.tokenUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type:    'authorization_code',
      code:          code,
      redirect_uri:  config.redirectUri,
      client_id:     config.clientId,
      code_verifier: codeVerifier,
    }),
  });

  const tokens = await response.json();
  console.log('Access Token:', tokens.access_token);

  // Cleanup
  sessionStorage.removeItem('pkce_code_verifier');
  sessionStorage.removeItem('oauth_state');

  return tokens;
}
```

---

### 6.3 Client Credentials Grant

**Best for:** Machine-to-machine (M2M) communication where no user is involved (e.g., microservice-to-microservice, batch jobs, CI/CD pipelines).

#### Sequence Diagram

```mermaid
sequenceDiagram
    participant ServiceA as Service A<br/>(Client)
    participant AuthServer as Authorization Server
    participant ServiceB as Service B<br/>(Resource Server)

    Note over ServiceA: No user involved.<br/>Service authenticates as itself.

    ServiceA->>AuthServer: 1. POST /token
    Note right of ServiceA: grant_type=client_credentials<br/>client_id=service-a-id<br/>client_secret=service-a-secret<br/>scope=read:orders write:inventory

    AuthServer->>AuthServer: 2. Validate client credentials
    AuthServer-->>ServiceA: 3. Access Token (no refresh token)

    ServiceA->>ServiceB: 4. GET /api/orders
    Note right of ServiceA: Authorization: Bearer access_token
    ServiceB->>ServiceB: 5. Validate token & scopes
    ServiceB-->>ServiceA: 6. Order data
```

#### Code Example (Python)

```python
# m2m_client.py — Client Credentials Grant
import requests

TOKEN_URL = "https://auth.example.com/oauth/token"
API_URL   = "https://api.example.com"

CLIENT_ID     = "service-inventory-id"
CLIENT_SECRET = "service-inventory-secret"


def get_m2m_token():
    """Obtain an access token using client credentials."""
    response = requests.post(TOKEN_URL, data={
        "grant_type":    "client_credentials",
        "client_id":     CLIENT_ID,
        "client_secret": CLIENT_SECRET,
        "scope":         "read:orders write:inventory",
    })
    response.raise_for_status()
    return response.json()["access_token"]


def fetch_orders():
    """Call the Orders API with the M2M token."""
    token = get_m2m_token()
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(f"{API_URL}/api/orders", headers=headers)
    response.raise_for_status()
    return response.json()


if __name__ == "__main__":
    orders = fetch_orders()
    print(f"Fetched {len(orders)} orders")
```

---

### 6.4 Device Authorization Grant

**Best for:** Input-constrained devices (Smart TVs, IoT devices, CLI tools) that cannot easily handle browser redirects.

#### Sequence Diagram

```mermaid
sequenceDiagram
    actor User
    participant Device as Smart TV /<br/>CLI Tool
    participant AuthServer as Authorization Server
    participant Browser as User's Phone<br/>or Laptop Browser

    Device->>AuthServer: 1. POST /device/code
    Note right of Device: client_id=tv-app<br/>scope=read:profile streaming

    AuthServer-->>Device: 2. Device Code + User Code + Verification URL
    Note left of AuthServer: device_code=GmRh...MkQ<br/>user_code=WDJB-MJHT<br/>verification_uri=https://auth.example.com/device<br/>interval=5<br/>expires_in=900

    Device->>User: 3. Display: "Go to https://auth.example.com/device<br/>and enter code: WDJB-MJHT"

    User->>Browser: 4. Open verification URL
    Browser->>AuthServer: 5. Navigate to /device
    User->>AuthServer: 6. Enter user code: WDJB-MJHT
    AuthServer->>User: 7. Show consent screen
    User->>AuthServer: 8. Approve access

    loop Poll every 5 seconds
        Device->>AuthServer: 9. POST /token
        Note right of Device: grant_type=urn:ietf:params:oauth:grant-type:device_code<br/>device_code=GmRh...MkQ<br/>client_id=tv-app
        AuthServer-->>Device: 10. "authorization_pending" (keep polling)
    end

    Note over User,AuthServer: User completes authorization

    Device->>AuthServer: 11. POST /token (final poll)
    AuthServer-->>Device: 12. Access Token + Refresh Token

    Device->>Device: 13. Streaming begins!
```

---

### 6.5 Refresh Token Grant

**Best for:** Maintaining long-lived sessions without re-prompting the user.

#### Sequence Diagram

```mermaid
sequenceDiagram
    participant Client
    participant AuthServer as Authorization Server
    participant API as Resource Server

    Note over Client: Access token has expired

    Client->>API: 1. GET /api/data (expired token)
    API-->>Client: 2. 401 Unauthorized

    Client->>AuthServer: 3. POST /token
    Note right of Client: grant_type=refresh_token<br/>refresh_token=REFRESH_TOKEN<br/>client_id=my-app<br/>client_secret=my-secret

    AuthServer->>AuthServer: 4. Validate refresh token
    AuthServer-->>Client: 5. New Access Token (+optional new Refresh Token)
    Note left of AuthServer: Refresh Token Rotation:<br/>Old refresh token is invalidated,<br/>new one is issued

    Client->>API: 6. GET /api/data (new access token)
    API-->>Client: 7. Protected data
```

---

## 7. Scopes

Scopes define the **granularity of access** the client is requesting. They are space-delimited strings sent in the authorization request.

### Examples from Popular APIs

| Provider | Scope | Permission |
|----------|-------|------------|
| **Google** | `openid` | Authenticate the user (OIDC) |
| **Google** | `email` | View the user's email address |
| **Google** | `https://www.googleapis.com/auth/drive.readonly` | Read-only access to Google Drive |
| **GitHub** | `repo` | Full access to repositories |
| **GitHub** | `read:user` | Read user profile data |
| **GitHub** | `repo:status` | Read/write commit statuses |
| **Microsoft** | `User.Read` | Read user's basic profile |
| **Microsoft** | `Mail.Send` | Send mail on behalf of user |
| **Slack** | `channels:read` | View channels in a workspace |
| **Slack** | `chat:write` | Send messages to channels |

### Scope Hierarchy Diagram

```mermaid
graph TD
    subgraph "GitHub Scope Hierarchy Example"
        REPO["repo<br/>(full control)"]
        REPO_STATUS["repo:status"]
        REPO_DEPLOY["repo_deployment"]
        PUBLIC["public_repo"]

        USER["user<br/>(full profile)"]
        USER_READ["read:user"]
        USER_EMAIL["user:email"]
        USER_FOLLOW["user:follow"]
    end

    REPO --> REPO_STATUS
    REPO --> REPO_DEPLOY
    REPO --> PUBLIC

    USER --> USER_READ
    USER --> USER_EMAIL
    USER --> USER_FOLLOW

    style REPO fill:#ffcdd2,stroke:#c62828
    style USER fill:#c8e6c9,stroke:#2e7d32
```

---

## 8. Real-World Example: "Login with Google"

This end-to-end example shows how a task management app ("TaskFlow") implements Google login.

### Architecture Overview

```mermaid
graph LR
    subgraph "User's Browser"
        UI["TaskFlow<br/>Frontend<br/>(React)"]
    end

    subgraph "TaskFlow Backend"
        API_GW["API Gateway"]
        AUTH["Auth Service"]
        TASKS["Task Service"]
        DB[(PostgreSQL)]
    end

    subgraph "Google"
        GOOGLE_AUTH["accounts.google.com<br/>(Authorization Server)"]
        GOOGLE_API["googleapis.com<br/>(Resource Server)"]
    end

    UI -->|"1. /login"| AUTH
    AUTH -->|"2. redirect"| GOOGLE_AUTH
    GOOGLE_AUTH -->|"3. callback + code"| AUTH
    AUTH -->|"4. exchange code"| GOOGLE_AUTH
    AUTH -->|"5. fetch profile"| GOOGLE_API
    AUTH -->|"6. create/update user"| DB
    AUTH -->|"7. issue session JWT"| UI
    UI -->|"8. API calls"| API_GW
    API_GW -->|"9. validate JWT"| AUTH
    API_GW -->|"10. fetch tasks"| TASKS
    TASKS -->|"11. query"| DB
```

### Complete Flow — Step by Step

```mermaid
sequenceDiagram
    actor User
    participant React as TaskFlow Frontend<br/>(React SPA)
    participant Backend as TaskFlow Backend<br/>(Node.js)
    participant Google as Google OAuth 2.0
    participant GoogleAPI as Google People API
    participant DB as PostgreSQL

    User->>React: 1. Click "Sign in with Google"
    React->>Backend: 2. GET /auth/google
    Backend->>Backend: 3. Generate state + PKCE
    Backend-->>React: 4. Redirect → Google /authorize

    React->>Google: 5. Redirect to consent screen
    Google->>User: 6. "TaskFlow wants access to your email and profile"
    User->>Google: 7. Click "Allow"
    Google-->>React: 8. Redirect → /auth/google/callback?code=XYZ&state=ABC

    React->>Backend: 9. GET /auth/google/callback?code=XYZ&state=ABC
    Backend->>Backend: 10. Verify state matches
    Backend->>Google: 11. POST /token (exchange code)
    Google-->>Backend: 12. { access_token, id_token, refresh_token }

    Backend->>GoogleAPI: 13. GET /userinfo (Bearer access_token)
    GoogleAPI-->>Backend: 14. { name, email, picture, sub }

    Backend->>DB: 15. UPSERT user (google_id, email, name, picture)
    DB-->>Backend: 16. User record

    Backend->>Backend: 17. Generate TaskFlow session JWT
    Backend-->>React: 18. Set-Cookie session JWT, HttpOnly, Secure, SameSite=Strict

    React->>Backend: 19. GET /api/tasks with session cookie
    Backend->>Backend: 20. Validate JWT
    Backend->>DB: 21. SELECT * FROM tasks WHERE user_id = ...
    DB-->>Backend: 22. Tasks data
    Backend-->>React: 23. [{ id: 1, title: "Buy groceries", ... }]
    React->>User: 24. Show task dashboard
```

---

## 9. Real-World Example: Machine-to-Machine API Access

### Scenario: Nightly Inventory Sync Between Microservices

```mermaid
sequenceDiagram
    participant Cron as Cron Job<br/>(Inventory Sync)
    participant Auth as Auth0<br/>(Authorization Server)
    participant Orders as Orders API<br/>(Service A)
    participant Inventory as Inventory API<br/>(Service B)

    Note over Cron: Nightly job at 02:00 UTC

    Cron->>Auth: 1. POST /oauth/token
    Note right of Cron: grant_type=client_credentials<br/>client_id=inventory-sync<br/>client_secret=••••••<br/>audience=https://api.orders.example.com<br/>scope=read:orders

    Auth-->>Cron: 2. { access_token: "eyJhbG..." }

    Cron->>Orders: 3. GET /api/orders?status=fulfilled&since=yesterday
    Note right of Cron: Authorization: Bearer eyJhbG...
    Orders->>Orders: 4. Validate token + check scope "read:orders"
    Orders-->>Cron: 5. [{ order_id: 1001, items: [...] }, ...]

    Cron->>Auth: 6. POST /oauth/token
    Note right of Cron: audience=https://api.inventory.example.com<br/>scope=write:stock

    Auth-->>Cron: 7. { access_token: "eyJhbG..." }

    loop For each fulfilled order
        Cron->>Inventory: 8. PATCH /api/stock/{sku}
        Note right of Cron: { "quantity_delta": -2 }
        Inventory-->>Cron: 9. 200 OK
    end

    Note over Cron: Sync complete. 47 items updated.
```

---

## 10. Security Best Practices

### Token Storage Decision Matrix

```mermaid
graph TD
    START["Where is your client?"] --> WEB{"Web Application?"}
    START --> MOBILE{"Mobile App?"}
    START --> SERVER{"Server / Backend?"}

    WEB --> SPA{"Single-Page App?"}
    WEB --> SSR{"Server-Rendered App?"}

    SPA --> |"In-memory + refresh via HttpOnly cookie"| SPA_STORE["In-Memory + Secure Cookie"]
    SSR --> |"Server session or encrypted cookie"| SSR_STORE["Server-Side Session"]

    MOBILE --> |"OS Keychain or Keystore"| MOBILE_STORE["Platform Secure Storage"]

    SERVER --> |"Env vars or secret manager like Vault"| SERVER_STORE["Secret Manager"]

    style SPA_STORE fill:#c8e6c9,stroke:#2e7d32
    style SSR_STORE fill:#c8e6c9,stroke:#2e7d32
    style MOBILE_STORE fill:#c8e6c9,stroke:#2e7d32
    style SERVER_STORE fill:#c8e6c9,stroke:#2e7d32
```

### Security Checklist

| Practice | Description |
|----------|-------------|
| **Always use HTTPS** | Never transmit tokens over plain HTTP |
| **Use PKCE** | Required for public clients (SPAs, mobile); recommended for all |
| **Validate `state` parameter** | Prevents CSRF attacks on the redirect URI |
| **Short token lifetimes** | Access tokens: 15–60 minutes; refresh tokens: days with rotation |
| **Refresh token rotation** | Issue a new refresh token on each use; invalidate the old one |
| **Validate redirect URIs** | Exact match only — no wildcards in production |
| **Use `HttpOnly` + `Secure` + `SameSite` cookies** | For storing tokens in web applications |
| **Never store tokens in `localStorage`** | Vulnerable to XSS attacks |
| **Validate JWT claims** | Always check `iss`, `aud`, `exp`, `nbf` |
| **Use `aud` claim** | Prevents token confusion between different APIs |
| **Implement token revocation** | Support RFC 7009 for immediate access removal |
| **Principle of least privilege** | Request only the scopes you actually need |

### Threat Model — Common Attack Vectors

```mermaid
graph TB
    subgraph "OAuth 2.0 Threat Model"
        AUTH_CODE_INTERCEPT["Authorization Code Interception"]
        TOKEN_THEFT["Token Theft via XSS or Leakage"]
        CSRF["CSRF on Redirect URI"]
        REDIRECT_MANIPULATION["Open Redirect / URI Manipulation"]
        TOKEN_REPLAY["Token Replay Attack"]
        PHISHING["Consent Screen Phishing"]
    end

    subgraph "Mitigations"
        PKCE["PKCE"]
        STATE["State Parameter"]
        STRICT_REDIRECT["Exact Redirect URI Matching"]
        HTTP_ONLY["HttpOnly Cookies / In-Memory Storage"]
        SHORT_TTL["Short Token TTL / Audience Validation"]
        BRAND_CHECK["App Verification / Branding Review"]
    end

    AUTH_CODE_INTERCEPT -.->|"mitigated by"| PKCE
    CSRF -.->|"mitigated by"| STATE
    REDIRECT_MANIPULATION -.->|"mitigated by"| STRICT_REDIRECT
    TOKEN_THEFT -.->|"mitigated by"| HTTP_ONLY
    TOKEN_REPLAY -.->|"mitigated by"| SHORT_TTL
    PHISHING -.->|"mitigated by"| BRAND_CHECK

    style AUTH_CODE_INTERCEPT fill:#ffcdd2,stroke:#c62828
    style TOKEN_THEFT fill:#ffcdd2,stroke:#c62828
    style CSRF fill:#ffcdd2,stroke:#c62828
    style REDIRECT_MANIPULATION fill:#ffcdd2,stroke:#c62828
    style TOKEN_REPLAY fill:#ffcdd2,stroke:#c62828
    style PHISHING fill:#ffcdd2,stroke:#c62828
    style PKCE fill:#c8e6c9,stroke:#2e7d32
    style STATE fill:#c8e6c9,stroke:#2e7d32
    style STRICT_REDIRECT fill:#c8e6c9,stroke:#2e7d32
    style HTTP_ONLY fill:#c8e6c9,stroke:#2e7d32
    style SHORT_TTL fill:#c8e6c9,stroke:#2e7d32
    style BRAND_CHECK fill:#c8e6c9,stroke:#2e7d32
```

---

## 11. Common Pitfalls

| Pitfall | Why It's Bad | What to Do Instead |
|---------|-------------|-------------------|
| Using the **Implicit Grant** | Tokens exposed in URL fragment; deprecated in OAuth 2.1 | Use Authorization Code + PKCE |
| Storing tokens in **`localStorage`** | Vulnerable to XSS attacks | Use `HttpOnly` cookies or in-memory |
| **Not validating `state`** | Allows CSRF attacks | Always generate, store, and verify `state` |
| **Long-lived access tokens** | Larger blast radius if compromised | Use short TTLs (15–60 min) with refresh tokens |
| **Overly broad scopes** | Violates least privilege | Request only what you need |
| **Hardcoding `client_secret`** in frontend code | Secret is extractable from JS bundles | Use PKCE for public clients; secrets only on backends |
| **Not using HTTPS** | Tokens intercepted via MITM | HTTPS is mandatory for all OAuth endpoints |
| **Ignoring token expiration** | 401 errors frustrate users | Implement silent refresh with refresh tokens |
| **No token revocation** | Users can't revoke access | Implement RFC 7009 revocation endpoint |

---

## 12. OAuth 2.0 vs OAuth 1.0 vs OpenID Connect

### Comparison Diagram

```mermaid
graph TB
    subgraph "Authentication & Authorization Stack"
        direction TB
        OIDC["OpenID Connect (OIDC)<br/>─────────────────<br/>Authentication Layer<br/>• ID Tokens (JWT)<br/>• UserInfo Endpoint<br/>• Standard Claims<br/>• Discovery & Registration"]
        OAUTH2["OAuth 2.0<br/>─────────────────<br/>Authorization Framework<br/>• Access Tokens<br/>• Scopes<br/>• Grant Types<br/>• Refresh Tokens"]
        HTTP["HTTP / TLS<br/>─────────────────<br/>Transport Security"]
    end

    OIDC -->|"built on top of"| OAUTH2
    OAUTH2 -->|"secured by"| HTTP

    style OIDC fill:#e3f2fd,stroke:#1565c0
    style OAUTH2 fill:#e8f5e9,stroke:#2e7d32
    style HTTP fill:#fce4ec,stroke:#c62828
```

### Feature Comparison

| Feature | OAuth 1.0 | OAuth 2.0 | OpenID Connect |
|---------|-----------|-----------|----------------|
| **Purpose** | Authorization | Authorization | Authentication + Authorization |
| **Token format** | Custom signed tokens | Bearer tokens (JWT/opaque) | JWT (ID Token) |
| **Transport security** | Signature-based | TLS/HTTPS required | TLS/HTTPS required |
| **Complexity** | High (crypto signatures) | Low (bearer tokens) | Medium (JWT validation) |
| **Mobile support** | Poor | Excellent | Excellent |
| **Token types** | Request + Access | Access + Refresh | Access + Refresh + ID |
| **User identity** | Not standardized | Not standardized | Standardized (claims) |
| **Discovery** | No | No | Yes (`/.well-known/openid-configuration`) |
| **Status** | Legacy | Current (RFC 6749) | Current (built on OAuth 2.0) |

---

## 13. References

| Resource | URL |
|----------|-----|
| **RFC 6749** — OAuth 2.0 Framework | https://datatracker.ietf.org/doc/html/rfc6749 |
| **RFC 7636** — PKCE | https://datatracker.ietf.org/doc/html/rfc7636 |
| **RFC 7009** — Token Revocation | https://datatracker.ietf.org/doc/html/rfc7009 |
| **RFC 8628** — Device Authorization Grant | https://datatracker.ietf.org/doc/html/rfc8628 |
| **OAuth 2.1 Draft** | https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-07 |
| **OpenID Connect Core** | https://openid.net/specs/openid-connect-core-1_0.html |
| **OAuth.net** — Community Resources | https://oauth.net/2/ |
| **Auth0 Docs** — OAuth 2.0 | https://auth0.com/docs/authenticate/protocols/oauth |
| **Google OAuth 2.0 Playground** | https://developers.google.com/oauthplayground/ |

---

> **Last Updated:** June 2026
> **License:** This tutorial is provided for educational purposes.
