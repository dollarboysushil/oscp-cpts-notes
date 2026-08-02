# Shady Oaks Financial

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

After login, we get JWT as:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJzdXNoaWwiLCJyb2xlIjoidXNlciIsImlhdCI6MTc3MjgxMzY1M30.DlQBKe4708GVJ1jMIkpWTStnsIcxDaZXc4WJMnzN9hU
```

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

edit the \
`algo` to `none` \
`id` to `1`\
`role` to `admin`

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Then send the request, we now have access to the admin panel

To get flag: GET request to /api/admin/flag

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

