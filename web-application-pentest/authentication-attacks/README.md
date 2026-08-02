# Authentication Attacks

## Authentication Attacks - Overview

### 1. Authentication vs Authorization

* **Authentication** = proving _who_ you are (login).
* **Authorization** = proving _what you're allowed to do_ once logged in.
* Attacks here target the first check - getting into an account without legitimately proving ownership.

Three factor types (MFA = combining 2+ of these):

* Something you **know** - password, PIN
* Something you **have** - phone, hardware token, email inbox
* Something you **are** - fingerprint, behavior

### 2. Root causes of every auth bug

Basically always one of these two:

1. **Weak brute-force protection** - mechanism is fine, but nothing stops unlimited guessing
2. **Broken logic** - a flaw lets you skip/bypass a step entirely, password strength irrelevant

### 3. Why it's high severity

* Low-priv account compromised → still leaks data + opens extra attack surface
* Admin/system account compromised → full app control, possible pivot to internal infra
* A one-line logic bug here often outranks a "deeper" bug elsewhere

### 4. Password-based login attacks

#### Brute-forcing

* **Password guessing** - works with no rate limit + weak password
* **Username guessing** - useful once enumeration confirms real accounts
* **Credential stuffing** - reuse leaked username/password pairs from other breaches

#### Username enumeration - check these for valid vs invalid username differences

* [ ] Status code (200 vs 404, redirect vs error)
* [ ] Error message wording ("invalid username" vs "invalid password")
* [ ] Response length (even a few bytes off)
* [ ] Response timing (valid username = server hashes password = slower response)

**Why it matters:** confirms real accounts → focus password brute-force only on those.

#### Flaws in brute-force protection

* **Account locking bypass** - lock resets the moment _one correct_ credential pair is submitted → smuggle unlimited guesses alongside one known-good pair in a batch
* **IP-based rate limit bypass** - limit is per-IP, not per-account → rotate IPs or spoof `X-Forwarded-For`
* **Multiple credentials per request** - endpoint accepts an array of username/password pairs in one request, processes each separately server-side, but front-end limiter only counts it as one request → unlimited attempts disguised as a single request

**Rule of thumb:** if the protection can be reset or split by changing the shape/origin of a request, it's broken.

#### HTTP Basic authentication

* No built-in lockout/rate limiting - browser resends creds freely, Burp Intruder works fine on the header
* Base64 ≠ encryption - trivially reversible, only safe over HTTPS
* Sent on every request → more exposure to logs, caching, referrer leaks

### 5. Multi-factor authentication (MFA) attacks

#### Outright bypass

* Server sets "logged in" state/cookie **before** 2FA check completes
* Try navigating directly to a post-login page after step 1 - sometimes you're just already in

#### Flawed 2FA verification logic

* Verification step identifies _which account's_ code to check using a client-controlled value (cookie/hidden param), not the server-side session
* Fix the value to point at another username, submit _your own_ valid code → verify into their account

#### Brute-forcing 2FA codes

* 4-6 digit codes are brute-forceable if the verify endpoint has no rate limit
* Check: does the code invalidate after one failed try?
* Check: does the rate limit reset the same way account-lock bugs do (one correct guess in a batch of wrong ones)?

### 6. Other authentication mechanisms

#### "Remember me" / persistent login

* Cookie should be random + server-validated
* Red flag: cookie is just username, or a weakly-obfuscated version of it → brute-force or reconstruct offline
* Check client-side JS for the generation algorithm - it might just tell you how to forge one

#### Password reset

* **Emails the actual current password** → red flag, means passwords aren't hashed properly server-side
* **Reset via URL/token** - check:
  * [ ] Token long + random, or guessable?
  * [ ] Invalidated after use?
  * [ ] Bound server-side to the account it was issued for?
* **Password reset poisoning** - reset link is built from a client-controlled header (`Host`, `X-Forwarded-Host`) → poison it, victim clicks the link, their reset token lands in your logs

#### Changing passwords

* Should require current password before setting new one
* If that check is skippable, or the endpoint doesn't verify the request targets the logged-in user's own account → account takeover

### 7. OAuth (third-party auth)

* Own topic, own attack surface: implicit vs auth-code flow, `redirect_uri` validation, `state` param handling
* Not covered here - separate notes

### 8. Quick checklist before testing any login flow

* [ ] Does the response differ at all for valid vs invalid usernames?
* [ ] Is rate limiting tied to account, or just IP/request?
* [ ] Can the limit/lockout be reset by an attacker-controlled trick (batch requests, new IP)?
* [ ] Is every step of a multi-step login enforced server-side, or can a later step be reached directly?
* [ ] Is any step (2FA target, reset scope, password-change target) identified by a client-controlled value instead of the session?
* [ ] Are tokens (reset links, remember-me cookies) random enough + invalidated after use?

Most labs = pick one of these questions, find the exact spot where the answer is "no."
