# Cheesy Does It  (forgot\_password flaw)

Level: Easy\
Points: 10\
Type: Daily Challenge

During register/login we can see forgot password feature.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Forgot-password takes only one arguement i.e `username` , we can pass any username here.

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Once username is passed, OTP is sent to account's email address. The ui doesnot takes the value more than 4 digit.\
Meaning we can try to bruteforce the OTP.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

There is no any rate limiting system and we can successfully bruteforce the OTP.\
After successfull OTP bruteforce, we get the reset\_token

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

From the js file,we can get idea on how to use the reset\_token to change the password.

POST request to /api/verify-token with values of&#x20;

* `username`&#x20;
* `reset_token`&#x20;
* `new_password`

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Password successfully changed.

Now, we can login as admin and get the flag.

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
