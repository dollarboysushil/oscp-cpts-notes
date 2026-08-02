# Tanuki

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

In `/profile` we have optiont to edit our profile

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

request to update profile

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

with this request, the first thing that comes in mind is if we can edit the profile of other user\
so, to test this, i created new account `test@gmail.com`

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

then I tried to edit the email and password of `test@gmail.com` and got error `Email already exists or invalid data`

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

Key thing is username is passed in the request `/api/profile/{username}`

so, I tried editing the username and it worked

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

There is no ownership check on `/api/profile/{usernname}`, meaning anyone with a valid token can edit the details of anyother account.
