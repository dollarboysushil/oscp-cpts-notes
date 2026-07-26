# JWT header parameter injections (jku injection)

The idea

Instead of embedding the public key directly in the header (`jwk`), the `jku` parameter just gives the server a **URL**. The server is expected to fetch a **JWK Set** (a JSON object containing an array of keys) from that URL, find the right key (usually matched by `kid`), and use it to verify the signature.

json

```json
{
  "keys": [
    { "kty": "RSA", "e": "AQAB", "kid": "75d0ef47-...", "n": "o-yy1wpY..." },
    { "kty": "RSA", "e": "AQAB", "kid": "d8fDFo-...",   "n": "fc3f-yy1w..." }
  ]
}
```

This is the same trust problem as `jwk`, just one layer removed: instead of _handing_ the server your public key, you're telling the server _where to go get it._

**Why this is dangerous**

If the server fetches the JWK Set from **whatever URL the token specifies**, without restricting it to a fixed, trusted domain - you can:

1. Host your own JWK Set on a server you control (containing the public key that matches a private key you hold).
2. Sign a tampered token (e.g. `sub: administrator`) with your **private key**.
3. Set the `jku` header to point at **your** hosted JWK Set URL.
4. Server fetches your JWK Set, finds the key matching the token's `kid`, and uses it to verify.
5. Verification succeeds - same reason as the `jwk` attack: you control both the signature and the key being verified against.

**Comparison to `jwk`**

| x                         | `jwk`                          | `jku`                                    |
| ------------------------- | ------------------------------ | ---------------------------------------- |
| Key location              | Embedded directly in the token | Fetched from an external URL             |
| What attacker needs       | Just the key pair              | Key pair + somewhere to host the JWK Set |
| Server behavior exploited | Trusts an embedded key         | Trusts a URL it fetches from             |

Same underlying bug either way: **the server lets the token decide which key to trust, rather than pinning that decision server-side.**

**When servers try to defend against this**

Better-designed servers restrict `jku` fetches to an **allowlist of trusted domains** (e.g., only their own `/.well-known/jwks.json`). But this defense can sometimes be bypassed using the same kind of **URL parsing discrepancies** used in SSRF whitelist bypasses - e.g., inconsistencies between how the filter parses a URL versus how the actual HTTP client that fetches it parses the same URL, letting you sneak a malicious host past a naive domain check.

**Root cause (same as `jwk`)**

The verification key should be a fixed, server-side decision - never something the incoming token gets to point to, whether by embedding it directly or by URL.

**Fix**

* Don't honor `jku` at all if avoidable, or
* Strictly allowlist trusted domains for key fetching, validated properly (not just a substring/prefix check), and
* Treat any server-side fetch triggered by user input as an SSRF risk in general - apply the same rigor as any other user-controlled URL fetch.

FLOW is

```
JWT header says:  "jku": "https://attacker.com/jwks.json"
                              │
                              ▼
Server makes an HTTP GET request to that URL
                              │
                              ▼
Response body = the JWK Set JSON (the array of keys)
                              │
                              ▼
Server picks the key from that array matching the token's `kid`
                              │
                              ▼
Uses that key to verify the JWT's signature
```

{% content-ref url="jwt-attack-labs/lab-6-jwt-authentication-bypass-via-kid-header-path-traversal.md" %}
[lab-6-jwt-authentication-bypass-via-kid-header-path-traversal.md](jwt-attack-labs/lab-6-jwt-authentication-bypass-via-kid-header-path-traversal.md)
{% endcontent-ref %}
