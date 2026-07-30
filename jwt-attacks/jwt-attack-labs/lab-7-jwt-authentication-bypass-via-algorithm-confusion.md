# Lab 7 JWT authentication bypass via algorithm confusion

This lab uses a JWT-based mechanism for handling sessions. It uses a robust RSA key pair to sign and verify tokens. However, due to implementation flaws, this mechanism is vulnerable to algorithm confusion attacks.

To solve the lab, first obtain the server's public key. This is exposed via a standard endpoint. Use this key to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

***

Visit `Algorithm Confusion Attack, STEPS` section for detailed steps of this attack.

{% content-ref url="../jwt-algorithm-confusion-vulnerabilities.md" %}
[jwt-algorithm-confusion-vulnerabilities.md](../jwt-algorithm-confusion-vulnerabilities.md)
{% endcontent-ref %}

**Vuln:** Server accepts both RS256 and HS256, using the same key variable for both - allowing its own public key to be reused as an HMAC secret.

**Steps:**

1. Log in as `wiener`, confirm `/admin` requires `sub: administrator`.
2. Fetch the server's public key from `/jwks.json`. Copy the single JWK object from the `keys` array.
3. In **JWT Editor Keys** → **New RSA Key** → paste the JWK → save.
4. Right-click the key → **Copy Public Key as PEM**.
5. In **Decoder**, Base64-encode the PEM string.
6. **New Symmetric Key** → Generate → replace `k` with the Base64-encoded PEM → save.
7. On the JWT: change `alg` → `HS256`, change `sub` → `administrator`.
8. **Sign** using the symmetric key created in step 6 (header unchanged otherwise).
9. Send to `/admin` → access granted, since the server's public key (as PEM bytes) matches the HMAC secret used to forge the signature.
10. Delete `carlos` via `/admin/delete?username=carlos`.

**Root cause:** Same key object reachable via two algorithm paths (RS256 verify vs HS256/HMAC), with the attacker-controlled `alg` header deciding which path runs.

**Fix:** Pin one algorithm per key/endpoint server-side (`algorithms=["RS256"]` only); never let the token's `alg` claim select the verification method.
