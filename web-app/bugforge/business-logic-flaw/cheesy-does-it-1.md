# Cheesy Does It

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Discount code `PIZZA-10` is provided.

During order checkout, we have option to add discount code.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Its respective request is

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here, we can edit the discount parameter from

```
"discount":"PIZZA-10"
```

to

```
"discount":["PIZZA-10","PIZZA-10","PIZZA-10","PIZZA-10"]
```

and the system applies the **same coupon multiple times**.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

forward the request and we will get our flag.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>
