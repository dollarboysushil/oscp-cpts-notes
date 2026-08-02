# Access Control Vulnerabilities

## Access Control Vulnerabilities - Overview

### 1. What access control actually is

* Sits after authentication + session management in the chain:
  * **Authentication** - are you who you say you are
  * **Session management** - which requests belong to that same "you"
  * **Access control** - are you allowed to do _this specific thing_
* Broken access control = common, and usually critical when found
* Root cause is mostly human: access rules are business decisions translated into code, and humans get that translation wrong a lot

### 2. Three types of access control

* **Vertical** - different user _types_ get different functions (admin vs regular user)
* **Horizontal** - same user _type_, but restricted to their own subset of resources (your bank transactions, not someone else's)
* **Context-dependent** - restricted based on app state/sequence (can't edit your cart after payment)

### 3. Vertical privilege escalation

#### Unprotected functionality

* Sensitive URL just... isn't checked (`/admin` reachable by anyone who knows/guesses it)
* "Hiding" it with an unguessable URL isn't access control - check:
  * [ ] `/robots.txt` - often lists the exact path it's trying to hide
  * [ ] Wordlist brute-force of common admin paths
  * [ ] Page source / JS - UI-building scripts sometimes hardcode the admin URL even for non-admin users, just wrapped in an `if (isAdmin)` that doesn't stop you from reading the script

#### Parameter-based access control

* Role/privilege decided at login, then stored somewhere the client can touch: hidden field, cookie, query string
* Example: `?admin=true`, `?role=1`
* If it's client-controlled, just change it

#### Platform misconfiguration

* Access rule enforced at platform/proxy layer, tied to a specific URL + method, e.g. `DENY: POST /admin/deleteUser for managers`
* Bypass angles:
  * [ ] Override headers - `X-Original-URL`, `X-Rewrite-URL` (if the app trusts these to determine the "real" URL, front-end filtering on the original path is useless)
  * [ ] Swap HTTP method - if `POST` is blocked but the backend still processes `GET` (or others) on the same endpoint, the platform-layer rule never even sees it

#### URL-matching discrepancies

* Front-end control and back-end routing disagree on what counts as "the same" URL
* Angles to test:
  * [ ] Case changes - `/ADMIN/DELETEUSER` vs `/admin/deleteUser`
  * [ ] Extra file extension - Spring's `useSuffixPatternMatch` (default pre-5.3) maps `/admin/deleteUser.anything` to `/admin/deleteUser`
  * [ ] Trailing slash - `/admin/deleteUser/` vs `/admin/deleteUser` treated as different endpoints by one layer but not the other

### 4. Horizontal privilege escalation

* Same mechanics as vertical, different target: `?id=123` → change to another user's ID
* This _is_ IDOR (insecure direct object reference) - object/user identified directly by user-controlled input, no ownership check
* If the ID is a GUID instead of an incrementing number, it's harder to guess directly - but:
  * [ ] Check if other users' GUIDs leak elsewhere (messages, reviews, public profile data)
* Watch for partial leaks: even if the app redirects you to login when access is denied, check if the redirect response body still contains the target's sensitive data anyway

### 5. Horizontal → Vertical escalation

* Compromising _any_ other user's account can become vertical escalation if that user turns out to be privileged
* Same `?id=` tampering trick, just aimed at an admin account instead of a random user
* Once in: reset/read their password, or use their session directly for privileged actions

### 6. Multi-step process flaws

* Sensitive action split across steps (load form → submit changes → review/confirm)
* Bug: access control only checked on steps 1-2, not step 3
* Attack: skip straight to submitting step 3's request with the right parameters, bypassing whatever gate was on steps 1-2

### 7. Referer-header-based access control

* Main page (`/admin`) properly protected, sub-pages (`/admin/deleteUser`) only check that `Referer` contains `/admin`
* `Referer` is fully attacker-controlled → forge it, hit the sub-page directly, bypass entirely

### 8. Location-based access control

* Restriction based on geolocation (banking, media licensing, etc.)
* Bypassed with VPNs, proxies, or just spoofing client-side geolocation values

### 9. Testing checklist

* [ ] Try reaching admin/sensitive URLs directly, without a UI link - check `robots.txt`, JS source, common wordlists
* [ ] Look for role/privilege values sitting in cookies, hidden fields, or query params - try changing them
* [ ] Try `X-Original-URL` / `X-Rewrite-URL` headers against platform-level path restrictions
* [ ] Try swapping the HTTP method on a restricted endpoint
* [ ] Try case changes, extra extensions, and trailing slashes on protected paths
* [ ] Change any user-facing ID parameter to another user's value - check both predictable IDs and leaked GUIDs
* [ ] In multi-step flows, jump straight to the final step's request without doing the earlier ones
* [ ] Check if access-denied responses (redirects, errors) still leak the target data in the body
* [ ] If `Referer`-based, just forge the header
* [ ] If geo-restricted, test with a VPN/proxy

### 10. Prevention (useful context for writeups)

* Never rely on obscurity (hidden URLs, unlisted endpoints) as the actual control
* Deny by default - only allow what's explicitly meant to be public
* One consistent, application-wide enforcement mechanism, not scattered per-endpoint checks
* Require developers to explicitly declare access rules per resource, default-deny otherwise
* Actually test access controls - don't assume the design holds up in code
