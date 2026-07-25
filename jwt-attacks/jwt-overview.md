# JWT Overview

1. What a JWT actually is

A JWT (JSON Web Token) is a compact way to represent claims between two parties, signed (and optionally encrypted) so the receiver can verify they haven't been tampered with. It's **not encryption by default** - it's signed, meaning anyone can read the contents, but only the holder of the secret/key can produce a valid signature.

Structure: three Base64URL-encoded parts separated by dots:

```
header.payload.signature
```

**Header** - algorithm and token type:

```json
{ "alg": "HS256", "typ": "JWT" }
```

**Payload** - the claims:

```json
{ "sub": "1234567890", "role": "user", "iat": 1690000000, "exp": 1690003600 }
```

Standard registered claims worth knowing: `iss` (issuer), `sub` (subject), `aud` (audience), `exp` (expiry), `nbf` (not before), `iat` (issued at), `jti` (JWT ID, for revocation/replay prevention).

**Signature** - computed over `base64url(header) + "." + base64url(payload)`, using the algorithm named in the header.

## 2. The two families of algorithms - this is the crux of most attacks

**Symmetric (HMAC)** - `HS256`, `HS384`, `HS512`

* Same secret key used to sign _and_ verify.
* If you can guess/brute-force/leak that secret, you can forge any token.

**Asymmetric** - `RS256`, `ES256`, `PS256`, etc.

* Private key signs, public key verifies.
* Server holds private key; public key can be distributed freely (it has to be, for verification) without letting anyone forge tokens - _in theory_.

**`none`** - an explicitly defined "no signature" algorithm in the JWA spec, meant for cases where integrity is handled elsewhere. Many libraries historically accepted it even when a signature was expected. This is the root of one of the most classic JWT bugs.

## 3. JWT vs JWS vs JWE

#### The relationship

**JWT (JSON Web Token)** is not a cryptographic mechanism itself - it's a _claims format_, a standardized way of representing a set of claims as a JSON object, intended to be transferred as one of the two structures below. JWT is the abstract concept; JWS and JWE are the concrete envelope formats that actually carry it.

Think of it like: "PDF" is the concept of a portable document; JWS and JWE are like "signed PDF" and "encrypted PDF" - the actual container formats.

#### JWS - JSON Web Signature (RFC 7515)

This is what people mean 95% of the time when they say "JWT." It provides **integrity and authenticity**, not confidentiality.

```
header.payload.signature
```

* Payload is just base64url - readable by anyone who intercepts it.
* Signature proves it wasn't tampered with and (for asymmetric algs) proves who signed it.
* Algorithms: `HS256/384/512`, `RS256/384/512`, `ES256/384/512`, `PS256/384/512`, `none`.

This is the format basically every JWT auth token, API bearer token, and session token uses. Everything discussed so far (alg confusion, `none` attacks, HS256 secret cracking, `kid`/`jku`/`jwk` injection) is a **JWS attack** - you're attacking the signature verification step.

#### JWE - JSON Web Encryption (RFC 7516)

This provides **confidentiality** (and integrity) - the payload is actually encrypted, not just base64-encoded plaintext.

Five dot-separated parts instead of three:

```
header.encrypted_key.iv.ciphertext.auth_tag
```

* **header** - specifies `alg` (key management algorithm, e.g. `RSA-OAEP`, `dir`, `A256KW`) and `enc` (content encryption algorithm, e.g. `A256GCM`, `A128CBC-HS256`)
* **encrypted\_key** - a randomly generated Content Encryption Key (CEK), itself encrypted with the recipient's key using the `alg` method. Empty if `alg: dir` (key agreement/direct use).
* **iv** - initialization vector for the content cipher
* **ciphertext** - the actual encrypted payload
* **auth\_tag** - authentication tag (for AEAD ciphers like GCM) proving integrity

You'll see this far less often in bug bounty work because it's much rarer in the wild - most apps just need "don't let it be tampered with," not "hide the contents from the bearer," since the bearer already has legitimate access to their own claims. When you do see JWE, attacks look different: padding oracle attacks against CBC modes (similar to classic padding oracle crypto attacks), key-confusion between `alg` and `enc`, or `alg: dir` misuse.

#### Quick comparison table

| x                     | JWS                          | JWE                                            |
| --------------------- | ---------------------------- | ---------------------------------------------- |
| Parts                 | 3 (`header.payload.sig`)     | 5 (`header.key.iv.ciphertext.tag`)             |
| Payload visibility    | Plaintext (base64 only)      | Encrypted                                      |
| Guarantees            | Integrity + authenticity     | Confidentiality + integrity                    |
| Typical use           | Auth tokens, session tokens  | Sensitive data transport, some OIDC id\_tokens |
| What attackers target | Signature verification logic | Encryption/key management logic                |

#### Nested case: JWS inside JWE

Sometimes you'll see both combined - a JWT is signed (JWS) _then_ the whole signed token is encrypted (JWE), giving you both properties. Header will have `"cty": "JWT"` to signal "the decrypted payload is itself another JWT." If you ever see a `cty` header, that's your cue you might be dealing with nested tokens.

For Portswigger's JWT track specifically - every lab is a **JWS** attack (that's basically the entirety of the "JWT attacks" topic there; JWE doesn't come up). So everything in your refresher so far - alg confusion, `none`, `kid` injection, `jwk`/`jku` header injection, weak secrets - is squarely JWS territory. Good to know JWE exists so you're not confused if a bug bounty target throws a 5-part token at you, but you won't need it for the labs you're starting.
