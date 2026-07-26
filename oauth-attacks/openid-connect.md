# OpenID Connect

## The Problem OIDC Solves

You already know OAuth 2.0 is an **authorization** framework - "can this app access this data on my behalf?" It was never designed to answer the question **"who is this user?"**

But in practice, everyone started using OAuth for **login** ("Sign in with Google/Facebook") anyway - by requesting a `profile`/`email` scope and treating whatever `/userinfo` returned as proof of identity. You saw this exact pattern in every lab so far: get a token, call `/userinfo`, trust the email/username back.

**The problem with that approach:** OAuth access tokens were never designed to prove identity to the client. They're designed to prove "you're allowed to call this API." Using them for authentication is a **repurposing**, not their intended use - and this repurposing is exactly why so many of the vulnerabilities you've seen exist (missing audience checks, trusting client-supplied identity, tokens leaking and being replayable as "login proof," etc.).

## What OIDC Actually Is

**OpenID Connect is a thin identity layer built on top of OAuth 2.0**, standardizing the _authentication_ use case that everyone was already improvising.

> OAuth 2.0 = authorization protocol\
> OIDC = authentication protocol, built using OAuth 2.0 as its transport mechanism

It adds a few concrete things on top of vanilla OAuth:

### 1. A new token: the **ID Token**

This is the key addition. Alongside the (optional) `access_token`, the authorization server also issues an **`id_token`** - a **JWT** (JSON Web Token), specifically designed to assert identity.

Unlike calling `/userinfo` (which requires a follow-up API call, and which you've seen apps mishandle), the `id_token` is:

* **Signed** by the authorization server (usually with RS256/JWT signature)
* **Self-contained** - the client can verify it locally without an extra network round-trip
* Contains standardized **claims** about the user directly inside it

### 2. Standardized claims (structured fields inside the JWT)

Example decoded `id_token` payload:

json

```json
{
  "iss": "https://oauth-server.com",
  "sub": "10769150350006150715113082367",
  "aud": "client_id_12345",
  "exp": 1700000000,
  "iat": 1699996400,
  "nonce": "abc123",
  "email": "wiener@example.com",
  "name": "Peter Wiener"
}
```

| Claim                 | Meaning                                                                  |
| --------------------- | ------------------------------------------------------------------------ |
| `iss`                 | Issuer - who created/signed this token                                   |
| `sub`                 | Subject - unique, stable identifier for the user                         |
| `aud`                 | Audience - which client this token was issued for                        |
| `exp` / `iat`         | Expiry / issued-at timestamps                                            |
| `nonce`               | Ties this token to a specific authentication request (replay protection) |
| `email`, `name`, etc. | Actual profile data, depending on requested scopes                       |

### 3. The `openid` scope

This is the trigger - you've seen it in every lab already:

```
scope=openid profile email
```

Including `openid` in the scope is what tells the authorization server: _"this is an OIDC request, please issue an `id_token` too, not just an access token."_ Without it, you're just doing plain OAuth.

### 4. A standardized `/userinfo` endpoint

OIDC also formalizes what plain OAuth left ambiguous - a consistent endpoint and response shape for fetching profile claims using the access token, so different providers behave predictably.

### 5. Discovery document

OIDC providers publish a well-known metadata endpoint:

```
GET /.well-known/openid-configuration
```

This returns the provider's endpoints (`authorization_endpoint`, `token_endpoint`, `jwks_uri`, etc.) and supported capabilities - so clients can auto-configure themselves instead of hardcoding everything.

## Why the `id_token` Matters (and why it's more "correct" than what you saw in the labs)

Recall Lab 1: the client trusted `email`/`username` sent raw from the browser, alongside a token - with zero binding between them. That was the core flaw.

**OIDC's `id_token` structurally prevents that exact mistake** - _if implemented correctly_ - because:

1. The `id_token` is **signed** by the authorization server (using its private key)
2. The client verifies that signature using the authorization server's **public key** (fetched from `jwks_uri`)
3. If the signature is valid, the client can trust the claims **inside** the token completely - because only the authorization server could have produced a validly-signed token
4. Identity is no longer "asserted by the browser" - it's **cryptographically proven by the issuer**

This is the theoretically correct way to solve the exact problem you exploited in Lab 1.

## But - New Attack Surface

Because OIDC introduces JWTs, signature verification, and new claims (`nonce`, `aud`, `iss`), it also introduces **new ways implementations can screw it up**:

* Not verifying the signature at all
* Accepting `alg: none`
* Not checking `aud` (a token meant for App A gets accepted by App B)
* Not checking `nonce` (replay attacks)
* Algorithm confusion attacks (RS256 → HS256 swap)

This is almost certainly where the next set of PortSwigger labs will head - same underlying story (trust boundary violations), new mechanism (JWTs) to break.

## Quick Comparison

| x                     | OAuth 2.0                  | OpenID Connect                            |
| --------------------- | -------------------------- | ----------------------------------------- |
| Purpose               | Authorization              | Authentication                            |
| Core token            | `access_token` (opaque)    | `id_token` (JWT, signed)                  |
| Answers               | "Can this app do X?"       | "Who is this user?"                       |
| Verifiable by client? | No (opaque, must call API) | Yes (signature check, no API call needed) |
| Scope trigger         | N/A                        | `openid`                                  |

## One-Line Mental Model

> OAuth hands out a **key card** (access token) that opens doors. OIDC additionally hands out a **signed ID badge** (`id_token`) that cryptographically proves who you are - instead of the door attendant just asking "so, who are you?" and taking your word for it.
