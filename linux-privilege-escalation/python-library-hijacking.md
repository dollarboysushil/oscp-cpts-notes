# Python Library Hijacking

**Python Library Hijacking** is a security vulnerability that allows an attacker to execute arbitrary code by manipulating the Python environment to load a malicious library instead of the intended one. This type of hijacking can lead to privilege escalation or unauthorized access, particularly when running applications or scripts with elevated permissions.

**How Python Library Hijacking Works**

1. **Dynamic Module Loading**:
   * Python allows dynamic loading of modules and packages using the `import` statement. This means that Python searches for modules in specific directories based on the `PYTHONPATH` environment variable and the installation directory of Python libraries.
2. **Module Search Path**:
   * When a Python script imports a module, it searches through several directories:
     * The directory containing the script being executed.
     * Directories specified in the `PYTHONPATH`.
     * Standard library directories and site-packages.
3. **Exploiting the Search Order**:
   * An attacker can create a malicious Python module with the same name as a legitimate module that the target application imports. If the malicious module is placed in a directory that takes precedence in the search order, the Python interpreter will load the malicious module instead of the legitimate one.

## Wrong Write Permission

idea is, if any suid python file imports library and we have write permission on that library, we can add simple malicious code (reverse shell in that library) to get shell us privileged user.

## Library PATH

idea here is, python searches and imports modules in priority order, meaning paths with higher on the list are searched first and then moves to priority with lower on this list.

example.

Copy

```
dollarboysushil@kali:~$ python3 -c 'import sys; print("\n".join(sys.path))'

/usr/lib/python3.5.zip
/usr/lib/python3.5
/usr/lib/python3.5/lib-dynload
/usr/local/lib/python3.5/dist-packages
/usr/lib/python3/dist-packages
```

this shows the order in which modules are searched and imported.

lets say, a suid python file uses numpy module.

Copy

```
dollarboysushil@kali:~$ pip3 show numpy
...SNIP...
Location: /usr/local/lib/python3.5/dist-packages

...SNIP...
```

we can use the above cmd to see numpy is installed in the path `/usr/local/lib/python3.5/dist-packages`.

While importing, python searches in `/usr/lib/python3.5.zip` → `/usr/lib/python3.5` ……. and then goes to `/usr/local/lib/python3.5/dist-packages`

so, what we can do is, create a malicious `numpy.py` in the folder `/usr/lib/python3.5` (if we have write permission) so then when psutil is imported, our malicious file gets executed.

## PYTHONPATH Environment Variable

```
dollarboysushil@kali:~$ sudo -l 

Matching Defaults entries for dollarboysushil on kali:
    env_reset, mail_badpass, .......................................:/snap/bin

User dollarboysushil may run the following commands on kali:
    (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python
```

If we have `SETENV` permission then we can set PYTHONPATH environment variable to somewhere we have write permission and put the respective file in that folder.



```
dollarboysushil@kali:~$ sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./pythoncode.py

uid=0(root) gid=0(root) groups=0(root)
...SNIP...
```

So, here we can put malicious `numpy.py` inside the /tmp file and set the env variable.

## IMPORTANT

sometimes we donot have write permission to these library location, if the suid set python file are present in the directory where we have write permission then we can create malicious python script in this directory (as the current directory **always** comes first) this malicious script gets executed first.
