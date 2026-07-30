# JWT algorithm confusion Vulnerabilities

### Background: two different trust models

| x           | Symmetric (HS256)     | Asymmetric (RS256)                 |
| ----------- | --------------------- | ---------------------------------- |
| Sign with   | Secret key            | Private key                        |
| Verify with | **Same** secret key   | **Different**, matching public key |
| Key secrecy | Must stay 100% secret | Public key is _meant_ to be shared |

### **Root cause**

Some servers support **both** symmetric and asymmetric algorithms in their JWT verification logic, and the developer writes code like:

```python
public_key = get_verification_key()  # server's known RSA public key
jwt.decode(token, key=public_key, algorithms=["RS256", "HS256"])
```

**Developer's intent:** "Use my public key to verify RS256 tokens."

**The actual bug:** The same `key` variable and the same `decode()` call are reachable via **either** algorithm - and which one runs is decided by the `alg` field in the token header, which is **attacker-controlled, untrusted input**.

* If `alg: RS256` → library correctly treats `public_key` as an RSA key, runs RSA verify. Works as intended.
* If `alg: HS256` → library instead treats `public_key`'s raw bytes as an **HMAC secret**, and computes `HMAC-SHA256(public_key_bytes, signing_input)` to check the signature.

The problem: **a value that is safe to expose as a public key is not safe to use as an HMAC secret** - HMAC's entire security model depends on the key staying confidential. The public key was never meant to be secret, so treating it as one collapses the security guarantee entirely.

### **The attack, step by step**

1. **Obtain the server's RSA public key** - often exposed deliberately (JWKS endpoint, client-side code, TLS cert, etc.), since public keys are meant to be shareable.
2. **Forge the payload** - e.g. change `sub` to `administrator`.
3. **Change header `alg` from `RS256` to `HS256`.**
4. **Sign the token** using HMAC-SHA256, with the **public key's raw bytes/PEM string as the HMAC secret**:

```
   signature = HMAC-SHA256(public_key_bytes, signing_input)
```

5. **Send the forged token.** Server reads `alg: HS256` from the header, calls its verify function with the same `public_key` variable it always uses, which now gets used as an HMAC key - running the exact same computation the attacker just performed.
6. **Signatures match → token accepted**, despite the attacker never having access to the server's actual private key.

### **Why it works - no crypto is broken**

RSA and HMAC-SHA256 are both cryptographically sound. The failure is entirely architectural:

* The server let the **token's own header** decide which algorithm/key-type interpretation to apply, instead of enforcing this server-side.
* A single key was reachable through two verification paths that carry incompatible secrecy requirements.

### **How this differs from `jwk`/`jku`/`kid` attacks**

| x                               | Attacker supplies...                                                               |
| ------------------------------- | ---------------------------------------------------------------------------------- |
| `jwk` / `jku` / `kid` injection | A brand-new key of their own choosing                                              |
| Algorithm confusion             | No key at all - reuses the **server's own legitimate public key**, just misapplied |

### **Fix**

```python
public_key = get_verification_key()
jwt.decode(token, key=public_key, algorithms=["RS256"])  # only one algorithm allowed
```

* **Explicitly whitelist exactly one algorithm family per key/endpoint**, server-side - never let the token's `alg` header pick the verification routine.
* **Never let a single key variable be reachable by both an HMAC path and an RSA path.** Bind key type to algorithm type strictly.
* The algorithm whitelist should reflect only what the server actually issues - if it only ever signs with RS256, only RS256 should ever be accepted.

### **One-line summary**

> The server trusted the attacker-controlled `alg` header to decide _how_ to use a key it already trusted - turning a safely-public RSA key into an exploitable HMAC secret.

## Algorithm Confusion Attack, STEPS

### **Step 1 - Obtain the server's public key**

Check standard endpoints: `/jwks.json` or `/.well-known/jwks.json`.

**Example response:**

json

```json
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "e512b374-b9a5-4d00-9082-4428f8d693ff",
      "alg": "RS256",
      "n": "04OO_LBCpf3HDelo8q7h3CcuUN1R5-kRdcXKnvhRX90To_ywKUDDVS2MmsOEMe7FwiXjB6mBCEX-HCdMKsY_PfMsUvYsrLgKX1C5C1ZE6FjHnUD3EaSK0sOS4u00zJikZszu0fNa3e2ZIkl0G8YrjBoicEE5uuQbOsblctpmedItJKtT0LKMlWS6xBCUEZapHPSZBihLUlZY1peSanp6lLLBLyw8nwB4L5KvCNFyXMnSifcuZy7xDQPfs5T6IlpiBNB5WubtyrN_a-sPTF3mSlXhVrRoWo0Va8ToEQLsd0btd3QwI0wUYy2spV_SNv-9B1IKJ9wWmcDRfwI2KQ40ew"
    }
  ]
}
```

No private key needed - you're borrowing the public key, not stealing the private one.

***

### **Step 2 - Convert to the format the server uses internally**

#### **a) Import the JWK into JWT Editor**

Go to **JWT Editor Keys** tab → **New RSA Key**. A dialog opens with a text box. Paste **the entire single key object** (not the whole `keys` array - just one entry) from Step 1:

json

```json
{
    "kty": "RSA",
    "e": "AQAB",
    "use": "sig",
    "kid": "e512b374-b9a5-4d00-9082-4428f8d693ff",
    "alg": "RS256",
    "n": "04OO_LBCpf3HDelo8q7h3CcuUN1R5-kRdcXKnvhRX90To_ywKUDDVS2MmsOEMe7FwiXjB6mBCEX-HCdMKsY_PfMsUvYsrLgKX1C5C1ZE6FjHnUD3EaSK0sOS4u00zJikZszu0fNa3e2ZIkl0G8YrjBoicEE5uuQbOsblctpmedItJKtT0LKMlWS6xBCUEZapHPSZBihLUlZY1peSanp6lLLBLyw8nwB4L5KvCNFyXMnSifcuZy7xDQPfs5T6IlpiBNB5WubtyrN_a-sPTF3mSlXhVrRoWo0Va8ToEQLsd0btd3QwI0wUYy2spV_SNv-9B1IKJ9wWmcDRfwI2KQ40ew"
}
```

Click **OK** - the key now appears in your keys list.

#### **b) Get it as PEM**

Right-click the key entry → **Copy Public Key as PEM** (or select the **PEM** radio button in its view and copy). You'll get something like:

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDTg478sEKl/ccN6WjyruHcJy5Q
0VHn6RF1xcqe+FFf3ROj/LApQMNVLYyaw4Qx7sXCJeMHqYEIRf4cJ0wqxj898yxS
9iysuApfULkLVkToWMedQPcRpIrSw5Li7TTMmKRmzO7R81rd7ZkiSXQbxiuMGiJw
QTm65Bs6xuVy2mZ50i0kq1PQsoyVZLrEEJQRlqkc9JkGKEtSVljWl5Jqenqk...
-----END PUBLIC KEY-----
```

#### **c) Base64-encode this entire PEM string** (including the `-----BEGIN/END-----` lines and line breaks)

Go to Burp's **Decoder** tab → paste the PEM → **Encode as Base64**. Example result:

```
LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlHZk1BMEdDU3FHU0liM0RRRUJBUVVBQTRHTkFEQ0JpUUtCZ1FEVGc0NzhzRUtsL2NjTjZXanlydUhjSnk1UQowVkhuNlJGMXhjcWUrRkZmM1JPai9MQXBRTU5WTFl5YXc0UXg3c1hDSmVNSHFZRUlSZjRjSjB3cXhqODk4eXhTCjlpeXN1QXBmVUxrTFZrVG9XTWVkUVBjUnBJclN3NUxpN1RUTW1LUm16TzdSODFyZDdaa2lTWFFieGl1TUdpSncKUVRtNjVCczZ4dVZ5Mm1aNTBpMGtxMVBRc295VlpMckVFSlFSbHFrYzlKa0dLRXRTVmxqV2w1SnFlbnFrLi4uCi0tLS0tRU5EIFBVQkxJQyBLRVktLS0tLQ==
```

#### **d) Create a symmetric key using this as the secret**

Go back to **JWT Editor Keys** → **New Symmetric Key** → click **Generate** (creates a placeholder JWK like below):

```json
{
    "kty": "oct",
    "kid": "a1b2c3d4-...",
    "k": "<auto-generated-placeholder>"
}
```

**Replace the `k` value** with the Base64-encoded PEM string from step (c):

```json
{
    "kty": "oct",
    "kid": "a1b2c3d4-...",
    "k": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlHZk1BMEdDU3FHU0liM0RRRUJBUVVBQTRHTkFEQ0JpUUtCZ1FEVGc0NzhzRUtsL2NjTjZXanlydUhjSnk1UQowVkhuNlJGMXhjcWUrRkZmM1JPai9MQXBRTU5WTFl5YXc0UXg3c1hDSmVNSHFZRUlSZjRjSjB3cXhqODk4eXhTCjlpeXN1QXBmVUxrTFZrVG9XTWVkUVBjUnBJclN3NUxpN1RUTW1LUm16TzdSODFyZDdaa2lTWFFieGl1TUdpSncKUVRtNjVCczZ4dVZ5Mm1aNTBpMGtxMVBRc295VlpMckVFSlFSbHFrYzlKa0dLRXRTVmxqV2w1SnFlbnFrLi4uCi0tLS0tRU5EIFBVQkxJQyBLRVktLS0tLQ=="
}
```

Click **OK** to save.

***

### **Step 3 - Modify the JWT**

* Change payload claim: `"sub": "administrator"`
* Change header: `"alg": "HS256"`

***

### **Step 4 - Sign the JWT**

* Click **Sign** on the JWT (in Repeater's JSON Web Token tab).
* Choose the **symmetric key** you just created in Step 2(d).
* Send the request.

{% content-ref url="jwt-attack-labs/lab-7-jwt-authentication-bypass-via-algorithm-confusion.md" %}
[lab-7-jwt-authentication-bypass-via-algorithm-confusion.md](jwt-attack-labs/lab-7-jwt-authentication-bypass-via-algorithm-confusion.md)
{% endcontent-ref %}

## Deriving public keys from existing tokens

**When to use this**

If the server doesn't expose its public key anywhere (no `/jwks.json`, nothing embedded in client code), you can still attempt algorithm confusion - by **mathematically deriving the RSA public key** from two valid JWTs the server has signed.

**The tool**

Use `jwt_forgery.py` (or similar) from the **`rsa_sign2n`** GitHub repo - or the simplified Docker version:

```
docker run --rm -it portswigger/sig2n <token1> <token2>
```

_(Requires Docker CLI installed. First run pulls the image, may take a few minutes.)_

**What it does**

* Takes **two valid, signed JWTs** from the server (same signing key, different content - e.g. two session tokens for different logins).
* Uses them to mathematically calculate one or more **candidate values of `n`** (the RSA modulus) - the core value needed to reconstruct the public key.
* Don't worry about the underlying math - just know: **usually more than one candidate `n` is produced, and only one of them is the server's actual key.**

**Output**

For each candidate `n`, the tool generates:

* A Base64-encoded **PEM key** (both X.509 and PKCS1 formats)
* A **pre-forged JWT**, already signed using that candidate key as the HS256 secret

**How to find the correct key**

1. Take each forged JWT the tool outputs.
2. Send each one to the server via Burp Repeater (e.g. to `/admin` or wherever the check applies).
3. **Only one will be accepted** - that confirms which candidate `n` (and matching PEM) is the server's real public key.
