# Cheesy Does It

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Vulnerable Request:

```http
POST /api/orders HTTP/2
Host: lab-1771857680915-k4ozgp.labs-app.bugforge.io
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJzdXNoaWwiLCJpYXQiOjE3NzE4NTc5NTJ9.N9MP8DhXJMJ6Akg2hi7A3Re26BQXWIuKbZIC9rmURfA
Content-Length: 303
Origin: https://lab-1771857680915-k4ozgp.labs-app.bugforge.io
Referer: https://lab-1771857680915-k4ozgp.labs-app.bugforge.io/checkout
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers

{
  "items": [
    {
      "pizza_name": "Pepperoni Classic",
      "base_name": "Hand Tossed",
      "sauce_name": "Classic Tomato",
      "size": "Medium",
      "toppings": [
        "Pepperoni",
        "Extra Mozzarella"
      ],
      "quantity": 1,
      "unit_price": 12.99,
      "total_price": 12.99,
      "id": 1771857957248
    }
  ],
  "delivery_address": "a",
  "phone": "a",
  "payment_method": "card",
  "notes": ""
}
```

Set the `total_price` to zero and forward the request, we will be able to buy the order for free.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

