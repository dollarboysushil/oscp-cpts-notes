# SSRF - Server Side Request Forgery

## SSRF (Server-Side Request Forgery) - Overview

### 1. What it is

* Attacker makes the **server** send a request to a place it shouldn't
* Attack surface = any param/field that holds a URL, hostname, or path the server fetches
* Two flavors: full SSRF (you see the response) and blind SSRF (you don't)

### 2. Why it's dangerous

* Bypasses network-level access control - internal systems trust "local" requests
* Can leak internal data, credentials, cloud metadata
* Sometimes escalates to full RCE
* Can also be used to attack third parties, making it look like it came from the victim org

### 3. SSRF against the server itself

* Target `127.0.0.1` / `localhost` / the server's own loopback
* Why this works - admin panels often trust local requests because:
  * Access control lives in a front-end layer that gets skipped on loopback calls
  * "Break glass" recovery access is intentionally open to local requests
  * Admin interface just listens on a different port, assumed unreachable
* Classic payload swap:

```
  stockApi=http://localhost/admin
```

### 4. SSRF against other back-end systems

* Internal IPs (e.g. `192.168.x.x`) often unauthenticated - protected only by "you can't reach it," not real auth
* Same payload trick, just point at the internal IP instead:

```
  stockApi=http://192.168.0.68/admin
```

* Use Burp Intruder to sweep internal IP ranges once you find the vector

### 5. Bypassing blacklist filters

* Filter blocks `127.0.0.1` / `localhost` literally → try:
  * [ ] Alt IP formats: `2130706433`, `017700000001`, `127.1`
  * [ ] Your own domain that resolves to `127.0.0.1`
  * [ ] URL-encoding or case variation on blocked strings
  * [ ] A redirect you control → target URL (try different redirect codes, try switching `http:` → `https:` mid-redirect)

### 6. Bypassing whitelist filters

* Filter checks if input matches/starts-with an allowed value → abuse URL-parsing quirks:
  * [ ] Embed creds before hostname: `https://expected-host:fakepass@evil-host`
  * [ ] Use `#` fragment trick: `https://evil-host#expected-host`
  * [ ] Subdomain trick: `https://expected-host.evil-host`
  * [ ] URL-encode (or double-encode) characters to desync filter vs actual request parsing
  * [ ] Combine two or more of the above

### 7. Bypassing filters via open redirect

* If the allowed domain has its own open-redirect bug, chain it:

```
  stockApi=http://allowed-domain.com/redirect?path=http://192.168.0.68/admin
```

* Filter only checks the first (allowed) domain; server follows the redirect to the real target

### 8. Blind SSRF

* Request fires server-side, but you never see the response
* Detect with OAST (out-of-band) - Burp Collaborator is the standard tool:
  * Send a unique Collaborator domain as the payload
  * Poll for DNS/HTTP interaction
* DNS hit but no HTTP hit = common - outbound DNS often allowed even when outbound HTTP is blocked
* Exploitation even when blind:
  * Blind-sweep internal IP ranges with OAST payloads to find unpatched internal vulns
  * Get the server to connect to a host you control, return a malicious response, target a bug in its HTTP client → possible RCE

### 9. Hidden SSRF attack surface

* [ ] Partial URLs - only hostname or path segment is user-controlled, rest built server-side (limited but still worth testing)
* [ ] Data formats that embed URLs - XML is the big one (→ SSRF via XXE)
* [ ] `Referer` header - server-side analytics tools sometimes fetch URLs found in `Referer` to log incoming links

### 10. Quick test checklist

* [ ] Any param holding a URL/hostname/path? Try `localhost`, `127.0.0.1`, internal IP
* [ ] Filter present? Try blacklist bypasses, then whitelist parsing tricks
* [ ] No visible response? Switch to Collaborator/OAST, assume blind
* [ ] App has an open redirect anywhere? Chain it through the filter
* [ ] Check XML inputs and the `Referer` header as extra vectors
