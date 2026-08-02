# JWT header parameter injections (jwk injection)

**Why these headers exist**

You already know the JOSE header carries `alg`. But the spec allows several _optional_ headers that tell the verifier **where or what key to use**. Three of them matter a lot for attackers because they shift control of key selection from "server decides" to "token says, server obeys":

* **`jwk`** - the actual public key, embedded directly in the header, as a JSON object.
* **`jku`** - a URL. The server is expected to fetch a JWK Set (a list of public keys) from that URL and pick the right one.
* **`kid`** - an identifier (like a filename or index) telling the server which key, out of potentially many it has stored, to use.

All three exist for legitimate reasons - e.g. supporting key rotation, multiple issuers, or federated key sets. The problem is when the server **trusts these fields blindly instead of validating them against a whitelist it controls.**

This is the same root pattern you've now seen three times: _the verifier lets attacker-controlled input decide how verification happens._ First it was `alg`. Now it's "which key."

***

**The `jwk` attack in detail**

**Normal/intended use:** Some systems embed the _signer's_ public key directly in the token via `jwk`, so the recipient doesn't need to look it up elsewhere. The recipient is supposed to check: "Is this public key one I actually trust (e.g., is it in my whitelist / does it match a known issuer)?" before using it to verify.

**The vulnerability:** A misconfigured server skips that trust check. It sees a `jwk` header, extracts the embedded key, and uses _that same key_ to verify the signature - without asking whether it should trust that key at all.

**Why that's catastrophic:** Signature verification only proves "this was signed by whoever holds the private key matching this public key." If the attacker gets to supply _both_ the private key (to sign) _and_ the public key (via `jwk`, for the server to verify with), then the check becomes meaningless - it's just proving the attacker signed their own forged token, which was never in question.

**Concrete flow:**

1. Attacker generates their own RSA key pair locally (private + public).
2. Attacker edits the JWT payload however they like (e.g. `sub: administrator`).
3. Attacker signs the new payload using their **own private key**.
4. Attacker embeds their own **public key** into the `jwk` header field of the token.
5. Server receives the token, sees `jwk`, extracts the key from it, and verifies the signature _using the attacker's own embedded public key_.
6. Verification "succeeds" - of course it does, the attacker signed it with the exact matching private key. The server just never checked whether it should trust that public key in the first place.

Note the header also usually needs a matching `kid` so the server's key-lookup logic (if any) points at the embedded key rather than an internal one - this is a detail the JWT Editor extension automates for you.

**In Burp with JWT Editor extension:**

1. Generate a new RSA key pair in the **JWT Editor Keys** tab.
2. Send the JWT-bearing request to Repeater.
3. Edit the payload claims as desired.
4. Click **Attack → Embedded JWK**, select your generated key.
5. The extension signs the token with your private key, embeds your public key + matching `kid` into the header automatically, and sends it.

***

**`jku` - the URL variant (brief context, since it's covered separately later)**

Instead of embedding the key directly, `jku` points the server to a URL where it should fetch a JWK Set. If the server fetches whatever URL the token specifies - without restricting it to a trusted domain/whitelist - an attacker can host their own JWK Set (containing their public key) on a server they control, point `jku` at it, and get the same outcome as the `jwk` attack: the server verifies using a key the attacker chose.

**`kid` - the lookup variant (brief context)**

If the server uses `kid` to look up a key from disk/database without sanitizing it, this opens totally different bugs: path traversal (`kid: ../../../dev/null` to point at an empty/predictable file, sometimes making the "key" an empty string you can trivially sign with) or even SQL injection if `kid` is used unsafely in a query.

***

**The unifying principle**

All three exploit the same trust failure: **verification key selection must be controlled by the server, never by the token.** A secure implementation would:

* Ignore `jwk` entirely, or validate the embedded key against a fixed, server-side whitelist before use.
* Restrict `jku` fetches to an allowlist of trusted domains (and ideally never trust it at all).
* Sanitize `kid` rigorously if used for lookups, treating it as untrusted input - never as raw a file path or query parameter.

{% content-ref url="jwt-attack-labs/lab-4-jwt-authentication-bypass-via-jwk-header-injection.md" %}
[lab-4-jwt-authentication-bypass-via-jwk-header-injection.md](jwt-attack-labs/lab-4-jwt-authentication-bypass-via-jwk-header-injection.md)
{% endcontent-ref %}
