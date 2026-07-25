# JWT Accepting Unsigned Tokens (alg: none)

What it is:\*\*\
The JWT spec allows an unsigned token via `alg: none` (an "unsecured JWT"). If the server trusts the `alg` header to decide _how_ to verify, an attacker can just set `alg: none`, strip the signature, and the server skips verification entirely.

**Why it works:**\
The server reads `alg` from the token itself - untrusted, attacker-controlled input - instead of deciding server-side which algorithm to expect. Letting the token dictate its own verification method defeats the point of verification.

**How to spot/exploit it:**

1. Decode the JWT, change `alg` to `none` (or `None`, `NONE`, `nOnE` - case tricks bypass naive string filters).
2. Delete the signature, but keep the trailing dot: `header.payload.`
3. Edit any claims you want (`role`, `sub`, etc.).
4. Send it. If the server accepts it, verification was skipped.

**Bypass tricks if `none` is filtered:**

* Mixed case: `None`, `nOnE`
* Unicode/encoding tricks in the header
* Basically: filters doing string matching on `alg` are fragile - try variations

**Fix:**\
Whitelist allowed algorithms server-side (e.g., only `HS256`), and explicitly reject `none` - don't rely on the token's own `alg` claim to choose the verification path.

> Even if the token is unsigned, the payload part must still be terminated with a trailing dot.

{% content-ref url="jwt-labs/lab-2-jwt-authentication-bypass-via-flawed-signature-verification.md" %}
[lab-2-jwt-authentication-bypass-via-flawed-signature-verification.md](jwt-labs/lab-2-jwt-authentication-bypass-via-flawed-signature-verification.md)
{% endcontent-ref %}
