# Race Conditions

## Race Conditions - Overview

### 1. What it is

* App processes two+ requests concurrently, both touch the same data, no safeguard against it → a "collision"
* Attacker sends carefully timed requests on purpose to trigger that collision
* Closely related to business logic flaws - same root idea (app in a state the devs didn't plan for), just triggered by timing instead of input
* **Race window** = the tiny gap where the collision is possible (often just milliseconds)

### 2. Limit overrun - the classic type

* Exceeds a limit the business logic is supposed to enforce
* Why it happens: check → apply → update, done as separate steps, not atomically
  1. Check code hasn't been used
  2. Apply discount
  3. Update DB to mark code as used
* Fire two requests before step 3 finishes on the first one → both pass the check → limit bypassed
* Common variations:
  * Redeem a gift card / discount code multiple times
  * Rate a product multiple times
  * Withdraw/transfer more than account balance
  * Reuse one CAPTCHA solution
  * Bypass anti-brute-force rate limiting
* Technical name for this subtype: TOCTOU (time-of-check to time-of-use)

### 3. How to actually fire the requests together

* Sending "at the same time" isn't good enough - network jitter throws off ordering
* **HTTP/1** → last-byte synchronization (hold back final byte of each request, release together)
* **HTTP/2** → single-packet attack - jams 20-30 requests into one TCP packet, removes jitter almost entirely
* Burp Repeater (2023.9+) does this automatically via "send group in parallel"
* Turbo Intruder (BApp store) - needed for heavier attacks: retries, staggered timing, huge request volume
  * Set `engine=Engine.BURP2`, `concurrentConnections=1`
  * Queue requests to a named `gate`, fire with `engine.openGate()`

### 4. Beyond limit overruns - hidden multi-step sequences

* A single request can internally pass through multiple hidden "sub-states" before finishing
* Example: login sets session as logged-in _before_ checking if MFA is required → race a login request against a sensitive authenticated request → land in the logged-in sub-state before MFA gets enforced
* These are app-specific - no universal payload, need the methodology below to find them

### 5. Methodology - Predict, Probe, Prove

#### Predict

* [ ] Is this endpoint security-critical? Skip the ones that aren't
* [ ] Is there collision potential - do two+ requests touch the _same_ record?
  * Password reset keyed by username in the URL → two different users, two different records, no collision
  * Password reset keyed by session → both requests hit the same session's data → collision possible

#### Probe

* Benchmark normal behavior first - send the request group in sequence (separate connections)
* Then send the same group in parallel (single-packet / last-byte sync)
* Compare - anything different counts as a clue: response change, different email content, visible behavior shift afterward

#### Prove

* Strip down to the minimum requests that reproduce it
* Confirm you can repeat it reliably
* Treat it as a structural weakness, not a one-off - the same flaw often has more than one angle of impact

### 6. Multi-endpoint race conditions

* Race windows across _different_ endpoints, not just one
* Classic pattern: pay for cart → add more items → force-browse to confirmation, all racing the payment-validate-then-confirm step
* Getting the windows to line up is harder than single-endpoint:
  * **Back-end connection delays** - usually affects all parallel requests equally, so it's often harmless. Test by "warming" the connection first (send a throwaway GET, then the real group) - if that smooths out timing, ignore the delay
  * **Endpoint-specific processing delays** - genuinely different processing times per endpoint. Fix by:
    * Turbo Intruder client-side delay (breaks single-packet attack, unreliable on jittery targets), or
    * Deliberately trip a rate/resource limit with junk requests to force a server-side delay, keeping single-packet viable

### 7. Single-endpoint race conditions

* Parallel requests to the _same_ endpoint, different values, from the same session
* Example: password reset stores `reset-user` and `reset-token` in session. Fire two resets, same session, different usernames → session can end up with victim's user ID paired with the token sent to the attacker's inbox
* Needs the right interleaving of operations - may take several attempts
* Good targets: anything email-based - emails are usually sent from a background thread after the response returns, widening the effective race window

### 8. Session-locking - a thing that can hide bugs

* Some frameworks (e.g. PHP's native session handler) process one request per session at a time, serializing everything
* If every request looks like it's running sequentially, that's the tell - try each request with a _different_ session token to break the artificial serialization

### 9. Partial construction race conditions

* Multi-step object creation leaves a temporary "half-built" middle state
* Example: user created in DB, API key set in a separate statement → brief window where API key is uninitialized (empty/null)
* Exploit: send a request where your supplied value _also_ evaluates to that same uninitialized value
* Framework array syntax tricks that produce empty/null values:
  * PHP: `param[]=foo` → `['foo']`, bare `param[]` → `[]`
  * Rails: `param[key]` (no value) → key present, value `nil`
* Example attack during the window: `GET /api/user/info?user=victim&api-key[]=`
* Same idea works on passwords too, but since passwords are hashed, you need input that hashes to the same "empty" digest - harder

### 10. Time-sensitive attacks (not strictly races, same toolkit)

* Precise request timing can expose weak token generation even without a true collision
* Example: reset token derived from a timestamp instead of a secure random string → time two reset requests (different users) to land in the same timestamp tick → same token issued to both

### 11. Prevention (good context for reports)

* Make sensitive state changes atomic - single DB transaction instead of separate check/update steps
* Don't mix data across storage layers (e.g. session data can't safeguard a database limit)
* Use datastore constraints (uniqueness, etc.) as a backup layer
* Update session variables as a batch, not individually - same care needed with ORMs, they hide the transaction boundary
* Where it fits the architecture, consider pushing state client-side (e.g. JWT) instead of server-side sub-states - though that trades in a different set of risks
