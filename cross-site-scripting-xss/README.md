# Cross-Site Scripting (XSS)

## XSS (Cross-Site Scripting) - Overview

### 1. What it is

* App returns attacker-controlled data in a way the browser executes as JavaScript
* Breaks same-origin policy protections - script runs as if it's the app's own code, in the victim's session
* Attacker can act as the victim: read data, perform actions, capture credentials

### 2. Three types

* **Reflected** - payload comes from the current request (URL/param), echoed straight back in the response
* **Stored** (persistent) - payload saved server-side (comment, username, message), served to other users later
* **DOM-based** - vulnerability is purely client-side JS mishandling data, server never even sees the payload in a dangerous form

### 3. Reflected XSS

* Needs external delivery (malicious link) - so a user has to click something
* Example:

```
  https://site.com/status?message=<script>alert(1)</script>
```

* Impact capped by delivery requirement - if timed wrong (user not logged in when they click), attack fails

### 4. Stored XSS

* Self-contained - attacker plants it once, no separate delivery step needed
* Guarantees victim is logged in when they hit it (since it fires during normal browsing)
* Generally higher impact than reflected for this reason
* Test tip: use a different unique string per input field (`test123comment`, `test123username`, etc.) so you can tell which field's output is firing

### 5. DOM-based XSS

* Source (attacker-controlled input, e.g. `location.hash`, `document.URL`) flows into a sink (dangerous JS function, e.g. `innerHTML`, `eval()`) with no safe handling in between
* Example vulnerable pattern:

js

```js
  var search = document.getElementById('search').value;
  results.innerHTML = 'You searched for: ' + search;
```

* **Reflected DOM XSS** - server echoes request data into the response, client-side script then mishandles it
* **Stored DOM XSS** - server stores data, later response's client-side script mishandles it
* Common in third-party libs too - jQuery selector sinks, AngularJS templates
* Test with browser dev tools - place unique input, search the DOM for it, check every place it lands
* Non-URL sources (`document.cookie`) and non-HTML sinks (`setTimeout`) are much harder to spot manually - needs actual code review or Burp DOM Invader

### 6. Injection contexts - what payload fits where

#### Between HTML tags

html

```html
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```

#### Inside an HTML attribute value

* If angle brackets aren't filtered - break out and add a new tag:

html

```html
  "><script>alert(document.domain)</script>
```

* If angle brackets ARE filtered - stay inside the tag, add a new attribute that creates an event handler:

html

```html
  " autofocus onfocus=alert(document.domain) x="
```

* If the attribute itself is scriptable (e.g. `href`) - no need to break out at all:

html

```html
  <a href="javascript:alert(document.domain)">
```

#### Inside existing JavaScript

* Close the script block entirely, start a fresh one:

html

```html
  </script><img src=1 onerror=alert(document.domain)>
```

* Inside a quoted string literal - break out, run code, patch the rest so it doesn't error:

js

```js
  '-alert(document.domain)-'
  ';alert(document.domain)//
```

* If app backslash-escapes quotes but not the backslash itself - use your own backslash to cancel theirs out
* If parentheses/certain chars are filtered - use `throw` to call a function without parens:

js

```js
  onerror=alert;throw 1
```

* Inside a JS template literal (backticks) - no need to terminate, just use the expression syntax:

js

```js
  ${alert(document.domain)}
```

#### HTML-encoding trick for attribute-embedded JS

* If quotes are blocked/escaped in an event-handler attribute, HTML-encode them instead - browser decodes attribute values before JS parsing kicks in:

html

```html
  &apos;-alert(document.domain)-&apos;
```

#### Client-side template injection

* Frameworks like AngularJS render templates client-side - unsafely embedding user input lets you inject template expressions instead of raw HTML/JS
* AngularJS specifically has its own sandbox with known escape techniques

### 7. What XSS is used for once you've got it

* Steal session cookies (if not `HttpOnly`)
* Capture credentials (fake login form injected into the page)
* Perform actions as the victim, including bypassing CSRF protection (since the script runs same-origin, it can read CSRF tokens too)
* Deface the page, or plant further malicious functionality

### 8. Dangling markup injection

* Fallback technique when full XSS isn't possible (filters block script execution) but you can still inject some raw markup
* Used to leak data cross-domain by injecting an unterminated tag (e.g. an `<img src=`) that captures subsequent page content up to the next quote/tag as part of its "URL," sent to an attacker-controlled server
* Can leak sensitive data visible to the victim, including CSRF tokens

### 9. Content Security Policy (CSP) - the usual mitigation

* Browser-enforced allowlist for script/style/resource sources
* If present, XSS may be blocked or limited - but CSP is frequently bypassable:
  * Overly permissive source list (allows a domain that itself hosts an exploitable JSONP endpoint or open redirect)
  * `unsafe-inline` or `unsafe-eval` left enabled defeats most of the point
  * Policy injection (if you can inject into the CSP header itself)
* Always check the actual CSP header before assuming XSS is dead - permissive real-world CSPs are extremely common

### 10. Testing checklist

* [ ] Submit a unique string into every input, then search all responses for where it lands
* [ ] Identify the exact context for each hit (tag body, attribute, JS string, template literal)
* [ ] Pick the payload matching that context from section 6
* [ ] For DOM XSS - use dev tools / DOM Invader, trace source → sink
* [ ] Check for CSP - if present, check the policy for gaps before assuming you're blocked
* [ ] Confirm impact with `alert(document.domain)` (or `print()` on Chrome 92+ for cross-origin iframe cases)

### 11. Prevention (context for writeups)

* Filter input on arrival - validate against what's actually expected
* Encode on output - context-appropriate (HTML/JS/URL/CSS encoding, not just one blanket filter)
* Set correct `Content-Type` and `X-Content-Type-Options: nosniff` so browsers don't reinterpret non-HTML responses as HTML
* CSP as a last line of defense, not the only one
* If allowing some HTML (rich text fields), use a real sanitization library, not a regex blacklist
