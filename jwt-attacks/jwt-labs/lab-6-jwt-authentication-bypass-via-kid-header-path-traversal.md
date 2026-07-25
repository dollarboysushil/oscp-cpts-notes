# Lab 6 JWT authentication bypass via kid header path traversal

This lab uses a JWT-based mechanism for handling sessions. In order to verify the signature, the server uses the `kid` parameter in JWT header to fetch the relevant key from its filesystem.

To solve the lab, forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

***

**Vuln:** Server uses the `kid` header value as a file path to fetch the verification key from its filesystem, without sanitizing it.

**Steps:**

1. Log in as `wiener`, capture session JWT, send to Repeater. Confirm `/admin` requires `sub: administrator`.
2. In JWT Editor Keys tab, create a new **symmetric key**, set `k` to an **empty string**.
3. In the JWT header, set `kid` to a path traversal sequence pointing at `/dev/null`:

```
   ../../../../../../../dev/null
```

4. In the payload, change `sub` → `administrator`.
5. Sign the token using the empty-string symmetric key (header unchanged otherwise).
6. Send to `/admin` → access granted, since server reads `/dev/null` (empty) as the key and the empty-string HMAC signature matches.
7. Delete `carlos` via `/admin/delete?username=carlos`.

**Root cause:** `kid` is treated as a trusted file path without sanitization - a symmetric key derived from a predictable/empty file lets an attacker sign tokens with a known secret.

**Fix:** Never resolve `kid` directly to a filesystem path or DB lookup without strict validation; whitelist expected `kid` values.

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>
