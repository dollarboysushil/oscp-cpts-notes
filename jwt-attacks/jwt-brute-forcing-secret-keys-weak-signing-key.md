# JWT Brute-forcing secret keys (weak signing key)

**What it is:**\
HS256 uses a plain string as the secret key. If it's weak, default, or copy-pasted from example code/docs, an attacker can brute-force it offline.

**Why it works:**\
Anyone with the secret can generate valid signatures for any payload they want - no server interaction needed, since signing is just a local math operation.

**How to exploit:**

1. Grab a valid JWT from the target.
2. Run it against a wordlist of common/known secrets:

```
   hashcat -a 0 -m 16500 <jwt> <wordlist>
```

3. Hashcat re-signs the header+payload with each wordlist entry and checks for a match against the token's signature.
4. Match found → secret recovered → forge any token (e.g. `sub: administrator`) and sign it yourself.

**Note:** Runs fully offline/locally, so it's fast even with large wordlists.

**Fix:**\
Use a long, random, high-entropy secret - never a default, placeholder, or dictionary word. Rotate secrets if ever suspected leaked.

{% content-ref url="jwt-attack-labs/lab-3-jwt-authentication-bypass-via-weak-signing-key.md" %}
[lab-3-jwt-authentication-bypass-via-weak-signing-key.md](jwt-attack-labs/lab-3-jwt-authentication-bypass-via-weak-signing-key.md)
{% endcontent-ref %}
