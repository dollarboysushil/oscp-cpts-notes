# Shady Oaks Financial

Level: Easy\
Points: 10\
Type: Daily Challenge

After Sign-up , login the interesting feature is ability to exchange currency.\
At the beginning, we have €1000

First thing that comes in my mind looking at the conversion feature is possibility of Race Condition.\
For this, I tried to convert all €1000 to USD.

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

Intercept request,

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

Send request to repeater, Drop request from Intercept tab

In repeater tab, create multiple duplicate tab, add all them to group and select send group (parallel)

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

Send request as group (parallel), triggers Race conditions and gives us flag.

