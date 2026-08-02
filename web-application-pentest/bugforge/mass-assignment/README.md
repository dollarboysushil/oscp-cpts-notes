# Mass Assignment

**Mass Assignment** is a vulnerability where a web application **automatically binds user input to internal object fields without proper restrictions**.

Because of this, an attacker can **modify hidden or sensitive parameters** (like `role`, `is_admin`, `price`, etc.) that the application did not intend users to control.

Example:

Normal signup request:

```
{
  "email": "user@test.com",
  "password": "Password123"
}
```

Attacker modifies it to:

```
{
  "email": "user@test.com",
  "password": "Password123",
  "role": "admin"
}
```

If the backend accepts all parameters, the attacker can **create an admin account or escalate privileges**.
