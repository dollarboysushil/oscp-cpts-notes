# JWT Signature Verification Bypass

**What it is:**\
The server accepts a JWT and reads its claims _without checking the signature_. So the token is technically "signed," but nobody ever verifies that signature before trusting the data inside it.

**Why it works:**\
JWTs aren't encrypted - the payload is just Base64, readable and editable by anyone. The signature is the _only_ thing stopping tampering. If the server skips verification, the signature becomes decorative. You can edit any claim (`sub`, `role`, `admin`, etc.) and the server will happily believe it.

**How to spot it:**

1. Decode the JWT (jwt.io or Burp).
2. Find a claim that looks like it controls identity/privilege (`sub`, `username`, `role`, `isAdmin`).
3. Change it to something more privileged.
4. Re-encode, keep the old signature (or anything, really), send it.
5. If it still works → signature isn't being checked.

{% content-ref url="jwt-labs/lab-1-jwt-authentication-bypass-via-unverified-signature.md" %}
[lab-1-jwt-authentication-bypass-via-unverified-signature.md](jwt-labs/lab-1-jwt-authentication-bypass-via-unverified-signature.md)
{% endcontent-ref %}

