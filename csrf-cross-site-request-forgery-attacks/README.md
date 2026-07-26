# CSRF - Cross Site Request Forgery Attacks

## CSRF (Cross-Site Request Forgery) - Overview

### 1. What it is

* Attacker tricks the victim's browser into sending a request the victim never intended
* Victim is already logged in → browser auto-attaches their session cookie → server treats it as legit
* Partly breaks the same-origin policy's whole point (sites shouldn't be able to act on each other's behalf)

### 2. Three conditions needed for CSRF to work

* [ ] **A relevant action** - something worth forging (change email, change password, transfer funds, change permissions)
* [ ] **Cookie-only session handling** - no other check ties the request to "really came from this user"
* [ ] **No unpredictable params** - attacker can guess/construct every value needed (if current password is required, CSRF alone won't work)

### 3. How the attack actually looks

* Auto-submitting hidden form on attacker's page:

html

```html
  <form action="https://site.com/email/change" method="POST">
    <input type="hidden" name="email" value="pwned@evil.net" />
  </form>
  <script>document.forms[0].submit();</script>
```

* If the action supports GET, even simpler - just an image tag:

html

```html
  <img src="https://site.com/email/change?email=pwned@evil.net">
```

* Delivery = same as reflected XSS: link via email/social media, or planted in a comment on a popular site
* Not cookie-only either - HTTP Basic auth and client cert auth can be CSRF'd too, since browser auto-attaches those too

### 4. Common defenses (and how each fails)

#### CSRF tokens

* Random unpredictable value tied to session, required on every sensitive request
* Bypass angles:
  * [ ] Token only checked on POST, not GET → switch method
  * [ ] Token only checked if present → strip the whole parameter, not just the value
  * [ ] Token valid globally, not tied to session → attacker gets their own valid token, feeds it to victim
  * [ ] Token tied to a _different_ cookie than the session cookie → if anything anywhere lets you set a cookie in victim's browser (even a sibling subdomain), plant your own token+cookie pair, feed the token
  * [ ] "Double submit" - token just duplicated in a cookie, no server-side record → invent a token, set matching cookie via any cookie-setting bug, feed token to victim

#### SameSite cookies

* Browser withholds cookie on cross-site requests, based on restriction level:
  * **Strict** - never sent cross-site
  * **Lax** - sent cross-site only on GET + top-level navigation (default in Chrome since 2021 if unset)
  * **None** - sent everywhere, needs `Secure` flag
* Bypass angles:
  * [ ] `Lax` + server tolerates GET on a state-changing endpoint → just navigate the victim there (`document.location = ...`)
  * [ ] Method-override params some frameworks support (e.g. Symfony's `_method`) → send GET, get treated as POST
  * [ ] `Strict` + an on-site open-redirect/gadget exists → client-side redirects count as same-site, so trigger a same-site "secondary" request through the gadget (server-side redirects don't work for this - browser still treats those as cross-site)
  * [ ] Sibling domain (same site, different subdomain) has XSS or similar → use it to fire the request from within the site itself
  * [ ] Chrome's 120-second grace window on newly-issued Lax cookies before enforcement kicks in - force a cookie refresh (e.g. via an SSO flow), then fire the attack inside that window

#### Referer-based validation

* Checks that `Referer` header matches the app's own domain
* Bypass angles:
  * [ ] Header only checked if present → strip the `Referer` entirely (some requests can be made without it)
  * [ ] Loose matching (checks if domain appears _anywhere_ in the header, not that it matches exactly) → craft a URL where your domain contains the expected string

### 5. Testing checklist

* [ ] Find a sensitive action that changes state (email, password, money, permissions)
* [ ] Check if it relies purely on cookies for identifying the user
* [ ] Check if any required param is something the attacker can't predict (if not, no CSRF)
* [ ] Look for a CSRF token - test each bypass angle above
* [ ] Check `SameSite` attribute on the session cookie - test the matching bypass
* [ ] Check for Referer validation - test presence/looseness bypass
* [ ] Use Burp's CSRF PoC generator to build and test the actual exploit HTML

### 6. Prevention (context for writeups)

* Token generated with a real CSRF-random source, tied to the session, validated on every state-changing request - not just when present
* Prefer `SameSite=Strict` (or `Lax` at minimum) on session cookies
* Don't assume same-site = safe - a same-site XSS/gadget bug anywhere on the site can undo SameSite protection entirely
