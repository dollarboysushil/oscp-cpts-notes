---
description: CopyPasta IDOR via Collection Slug Enumeration
---

# CopyPaste - Slug

**Level:** Easy | **Points:** 10 | **Type:** Daily Challenge

**Overview**

The application allows users to create and manage collections of code snippets. An insecure API endpoint exposes collection metadata without authorization checks, enabling an attacker to enumerate private collections and access their contents using leaked slugs.



<figure><img src="../../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

Navigating to `/collections`, authenticated users can create collections and group snippets inside them.

<figure><img src="../../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

**Identifying the Vulnerability**

The endpoint `/api/collections/:id` returns metadata for any collection by its integer ID  including collections owned by other users.

This immediately raises an **IDOR (Insecure Direct Object Reference)** flag, since the IDs are sequential integers with no authorization check.

Querying `collection/2` reveals a collection owned by **admin**:

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

**Exploitation**

Two key observations make exploitation possible:

**1. The `is_public` field is set to `1`**\
For public collections, the application uses the collection's `slug` to serve its data, meaning knowing the slug is enough to view the contents.

**2. The slug is leaked by the IDOR**\
Even though this is the admin's collection, the unauthenticated metadata endpoint freely returns its slug.

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

To understand the share URL format, we first inspect our own collection's public link:

```
https://lab[id].labs-app.bugforge.io/collections/share/<slug>
```

Substituting the admin's leaked slug into this URL:&#x20;

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

This grants full access to the admin's collection, and reveals the flag.



**Root Cause**

The vulnerability is a combination of two weaknesses:

* **IDOR:** `/api/collections/:id` exposes metadata of all collections with no ownership or authorization check
* **Security through Obscurity failure:** the app treats the slug as an access control mechanism, but leaks it via the IDOR endpoint

***

**Remediation**

* Enforce **authorization checks** on `/api/collections/:id`  users should only be able to query collections they own or that are explicitly shared with them
* Do not rely on slugs or tokens as the sole access control mechanism for private resources

