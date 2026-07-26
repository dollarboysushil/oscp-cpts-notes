# Lab 2 Forced OAuth profile linking

This lab gives you the option to attach a social media profile to your account so that you can log in via OAuth instead of using the normal username and password. Due to the insecure implementation of the OAuth flow by the client application, an attacker can manipulate this functionality to obtain access to other users' accounts.

To solve the lab, use a CSRF attack to attach your own social media profile to the admin user's account on the blog website, then access the admin panel and delete `carlos`.

The admin user will open anything you send from the exploit server and they always have an active session on the blog website.

You can log in to your own accounts using the following credentials:

* Blog website account: `wiener:peter`
* Social media profile: `peter.wiener:hotdog`

***

## Feature Being Attacked

The blog app lets a logged-in user "attach" a social media profile to their existing account, so they can log in via OAuth instead of a password. This uses a normal OAuth flow - `/auth` → login/consent → redirect with `code` → `/oauth-linking?code=...` on the blog's backend, which **links** that social profile to whichever account is currently logged in.

## Key Problem

The **linking request has no `state` parameter** (or any other CSRF protection):

```
GET /auth?client_id=...&redirect_uri=.../oauth-linking&response_type=code&...
```

Because `/oauth-linking?code=...` is just a plain `GET` request with no CSRF token tying it to the user who originally started the flow, it becomes a **CSRF-able endpoint**. The `code` itself is normally single-use and tied to _the attacker's own_ social account - but the server has no way of confirming _who_ the browser making the `/oauth-linking` request actually is, beyond whatever session cookie is attached at request time.

## The Exploit Logic

1. Attacker starts the "attach social profile" flow using **their own** social media account
2. Attacker intercepts (but doesn't let complete) the callback: `GET /oauth-linking?code=ATTACKER_CODE`
3. Attacker hosts this URL in a hidden `<iframe>` on the exploit server
4. Victim (already logged into the blog as admin) visits the page → browser loads the iframe → sends `GET /oauth-linking?code=ATTACKER_CODE` **using the victim's own session cookie**
5. Server links the **attacker's social profile** to the **victim's (admin's) account** - because it just uses whatever session cookie rides along, with zero verification that the person completing the OAuth flow is the same person who started it

## Root Cause

> Classic **CSRF**, just wrapped around an OAuth linking flow. `state` exists specifically to bind the authorization request and its callback to _the same user session_ - without it, the callback can be triggered by (or on behalf of) any victim via a forged request, since the server can't distinguish "I initiated this" from "someone else initiated this and I just clicked a link/loaded an iframe."

## The Fix

* Generate a random, unguessable `state` value when the "attach social profile" flow starts
* Store it tied to the current user's session
* Require the `/oauth-linking` callback to include the **same `state`** value and validate it matches before performing the link
* This ensures the account being linked belongs to the session that _initiated_ the request - not whichever session happens to be active when the callback fires

## One-Line Takeaway

> Any OAuth redirect/callback endpoint that changes account state (login, linking, permission grants) **must** use `state` to bind the request to the originating session - otherwise it's just a CSRF attack wearing OAuth clothing.
