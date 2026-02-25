# CopyPasta

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Key Feature: We have option to change password.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Its respective request is

```
PUT /api/profile/password HTTP/2
Host: lab-1772045117373-w7zhpl.labs-app.bugforge.io
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJzdXNoaWwiLCJpYXQiOjE3NzIwNDUxNjh9.x24WzsblCATJ4Z6ADjqzkB349kp3OiJ6nOODyGS3EuU
Content-Length: 33
Origin: https://lab-1772045117373-w7zhpl.labs-app.bugforge.io
Referer: https://lab-1772045117373-w7zhpl.labs-app.bugforge.io/profile/sushil
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{
    "password":"sushil",
    "user_id":5
}
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Key thing to look here is `user_id` value. Next step here would be to change `user_id` to differet value hoping we can change password of different user.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Got 200 Ok. Lets verify this by changing the password of different user.\
From `/public` we can find various users's username

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Getting stats of user `coder123` . `id= 2`

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Changing password of user 2

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Successfull login

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Flag on dashboard

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
