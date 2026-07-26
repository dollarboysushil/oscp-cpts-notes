# OAuth Attacks

## What OAuth 2.0 Actually Is

<figure><img src="../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

An **authorization** framework (not authentication) that lets a user grant a third-party app limited access to their resources on another service, without sharing their password.

## The 4 Roles

* **Resource Owner** — the user who owns the data
* **Client** — the application requesting access
* **Authorization Server** — issues access tokens after authenticating the user and getting consent
* **Resource Server** — hosts the protected data, accepts access tokens to serve it

## Key Terms

* **Client ID** — public identifier for the app
* **Client Secret** — private credential, known only to client + auth server
* **Redirect URI** — where the auth server sends the user back after approval
* **Scope** — what level of access is being requested (e.g., `read:email`)
* **Authorization Code** — short-lived, single-use code exchanged for a token
* **Access Token** — credential used to call the resource server's API
* **Refresh Token** — long-lived credential used to get new access tokens without re-login

## The Standard Flow (Authorization Code Grant)

1. Client redirects user to the **authorization endpoint** with `client_id`, `redirect_uri`, `scope`, `state`, `response_type=code`
2. User logs in (if needed) and approves the requested scopes
3. Auth server redirects back to `redirect_uri` with an **authorization code**
4. Client's backend exchanges the code + `client_secret` for an **access token** (and often a refresh token) at the **token endpoint**
5. Client uses the access token to call the resource server's API

## Other Grant Types (for context)

{% content-ref url="oauth-grant-types.md" %}
[oauth-grant-types.md](oauth-grant-types.md)
{% endcontent-ref %}

| Grant Type            | Use Case                                          |
| --------------------- | ------------------------------------------------- |
| Authorization Code    | Standard - server-side apps                       |
| PKCE (extension)      | Mobile/SPA apps - no client secret                |
| Client Credentials    | Machine-to-machine, no user involved              |
| Implicit (deprecated) | Old browser-based flow, token in URL fragment     |
| Device Code           | TVs, CLI tools - user approves on a second device |

## OAuth vs OIDC

* **OAuth 2.0** = authorization ("can this app access this data?")
* **OpenID Connect (OIDC)** = authentication layer on top of OAuth, adds `id_token` (a JWT) to actually verify _who_ the user is

## Quick Mental Model

> The client never sees the user's password. It gets a token that proves "this user allowed me to do X" - scoped, revocable, and time-limited.

## Vulnerabilities in the OAuth client application

### Improper implementation of the implicit grant type

## Key Insight

Even if the **OAuth service** (Google, Facebook, etc.) is secure, the **client application's own implementation** is often the weak link. OAuth spec is loosely defined → lots of optional parameters/configs → lots of room for misconfiguration.

## The Flaw: Improper Implicit Grant Implementation

**Why it happens:**

* Implicit flow sends `access_token` via browser (URL fragment)
* Client JS extracts it
* To persist the session (survive page close), the app sends user data + token to its own backend via `POST`, which then sets a session cookie

**The problem:**

* This `POST` request is **visible and editable by the attacker** (it's just a browser request)
* Server has **no secret/password to independently verify** the submitted data against - no `client_secret` exchange happens in this flow
* So the server **implicitly trusts** whatever fields are sent, _unless_ it separately re-validates the token

**The vulnerability:**

> If the server doesn't check that the `access_token` actually corresponds to the `email`/`username`/user ID also sent in that request, an attacker can simply **edit those fields** and impersonate any user - while using their own valid token.

## One-Line Takeaway

> Implicit flow moves everything through the browser - including the final "log me in as X" request - so if the server trusts client-supplied identity fields instead of deriving identity from the token itself, it's game over.

{% content-ref url="oauth-attacks-labs-portswigger-academy/lab-1-authentication-bypass-via-oauth-implicit-flow.md" %}
[lab-1-authentication-bypass-via-oauth-implicit-flow.md](oauth-attacks-labs-portswigger-academy/lab-1-authentication-bypass-via-oauth-implicit-flow.md)
{% endcontent-ref %}

### Flawed CSRF protection

{% content-ref url="oauth-attacks-labs-portswigger-academy/lab-2-forced-oauth-profile-linking.md" %}
[lab-2-forced-oauth-profile-linking.md](oauth-attacks-labs-portswigger-academy/lab-2-forced-oauth-profile-linking.md)
{% endcontent-ref %}

### Leaking authorization codes and access tokens

{% content-ref url="oauth-attacks-labs-portswigger-academy/lab-3-oauth-account-hijacking-via-redirect_uri.md" %}
[lab-3-oauth-account-hijacking-via-redirect\_uri.md](oauth-attacks-labs-portswigger-academy/lab-3-oauth-account-hijacking-via-redirect_uri.md)
{% endcontent-ref %}

### Stealing codes and access tokens via a proxy page

{% content-ref url="oauth-attacks-labs-portswigger-academy/lab-4-stealing-oauth-access-tokens-via-an-open-redirect.md" %}
[lab-4-stealing-oauth-access-tokens-via-an-open-redirect.md](oauth-attacks-labs-portswigger-academy/lab-4-stealing-oauth-access-tokens-via-an-open-redirect.md)
{% endcontent-ref %}

***



{% content-ref url="openid-connect.md" %}
[openid-connect.md](openid-connect.md)
{% endcontent-ref %}

{% content-ref url="oauth-attacks-labs-portswigger-academy/lab-6-ssrf-via-openid-dynamic-client-registration.md" %}
[lab-6-ssrf-via-openid-dynamic-client-registration.md](oauth-attacks-labs-portswigger-academy/lab-6-ssrf-via-openid-dynamic-client-registration.md)
{% endcontent-ref %}
