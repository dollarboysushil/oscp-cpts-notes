# Sokudo

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

After signup, we get a token.

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Provided token looks interesting.

```
20260312181120
```

which is in this form

```
2026 03 12 18 11 20
YYYY MM DD HH MM SS
```

To check if it is what I thought. I created a new account and got new token

```
20260312180943
```

Next, I tried to bruteforce last 4 digits hoping new account is created in last 60 mins.\
I also edit the request to /api/admin/users which requires admin token to get 200 response.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>
