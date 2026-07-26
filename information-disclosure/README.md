# Information Disclosure

## Information Disclosure - Overview

### 1. What it is

* App leaks info it shouldn't, to users or attackers - aka "information leakage"
* What gets leaked:
  * Other users' data (usernames, emails, financial info)
  * Business/commercial data
  * Technical infra details (server, framework, versions)
* Often surfaces while testing something else entirely - keep an eye out even off-task

### 2. Why it matters even when it "looks minor"

* Direct leak (credit card numbers, PII) → obviously severe on its own
* Technical leak (framework version, directory structure) → looks harmless, but:
  * Confirms attack surface for something else
  * Old/vulnerable framework version + public exploit = easy win
  * Can be the missing piece for a bigger, chained attack
* **Severity test:** does the leaked info let you _do_ something harmful? If not exploitable, don't over-flag it - common sense over box-ticking

### 3. Root causes (pick one, this is basically always why)

* **Leftover internal content** - dev comments, debug notes not stripped before prod
* **Insecure configuration** - debug/diagnostic features left on, verbose default errors, unneeded features enabled
* **Flawed app logic** - different responses for different internal states → lets you enumerate data (ties directly into username enumeration)

### 4. How to test - techniques

* **Fuzz parameters** - throw unexpected types/fuzz strings, watch for:
  * Status code differences
  * Response length differences
  * Timing differences
  * Use Burp Intruder + grep-match/extract for `error`, `SQL`, `invalid`, etc.
* **Burp Scanner** - auto-flags leaked keys, emails, card numbers, backup files, directory listings
* **Burp engagement tools** (right-click → Engagement tools):
  * Search - find/negative-search any keyword
  * Find comments - pulls dev comments out automatically
  * Discover content - finds unlinked dirs/files
* **Force informative errors** - feed invalid input on purpose, see what the stack trace/debug response gives up

### 5. Common places to check

* [ ] `/robots.txt` and `/sitemap.xml` - lists paths crawlers are told to skip (often the interesting ones)
* [ ] Directory listings - enabled dirs with no index page expose every file inside
* [ ] HTML source / dev comments - view-source, look for leftover notes hinting at hidden paths or logic
* [ ] Error messages - verbose errors reveal expected input types, DB/template engine names, version numbers
* [ ] Debug endpoints/data - session vars, backend hostnames, credentials, encryption keys, file paths
* [ ] User account/profile pages - check if a `user` parameter can be swapped to pull another user's data without full account access
* [ ] Backup files - `.bak`, `~`, `.old`, `.swp` on known source file names → can return raw source instead of executed output
* [ ] Insecure config - HTTP `TRACE` method enabled can echo back internal headers (e.g. reverse-proxy auth headers)
* [ ] `.git` directory exposed - download it, read commit history/diffs for hardcoded secrets

### 6. Prevention (dev-side, good to know for reports/writeups)

* Everyone on the team should know what counts as "sensitive" - non-obvious stuff leaks too
* Strip dev comments before deploy (automate it in CI/QA)
* Use generic error messages - don't hand attackers free clues
* Disable debug/diagnostic features in production, every time
* Understand the config + security implications of every third-party tool in use, turn off what you don't need
