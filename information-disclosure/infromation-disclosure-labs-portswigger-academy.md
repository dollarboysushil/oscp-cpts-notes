# Infromation Disclosure (Labs: Portswigger Academy)

### Labs from PortSwigger Academy

#### Lab 1: Information disclosure in error messages

```
GET /product?productId=1 --> Request to load product

add ' , we will get verbose error with framework's version number
GET /product?productId=1'
```

#### Lab 2: Information disclosure on debug page

This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the `SECRET_KEY` environment variable.

```
Using Burpsuite's Engagegement tools > Discover Contents, we find /cgi-bin
inside /cgi-bin there is /phpinfo.php which leaks SECRET_KEY.
```

#### Lab 3: Source code disclosure via backup files

This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code.

```
Using Burpsuite's Engagegement tools > Discover Contents, we find /backup 
/backup contains ProductTemplate.java.bak

```

#### Lab 4: Authentication bypass via information disclosure

This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

```
visiting GET /admin shows Admin interface only available to local users

TRACE /admin shows X-Custom-IP-Authorization: OUR IP

in GET /admin add X-Custom-IP-Authorization: 127.0.0.1 
(we are now admin)

```

#### Lab 5: Information disclosure in version control history

This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the `administrator` user then log in and delete the user `carlos`.

```
there exists .git folder,
wget the .git into your device, and check the commits
```
