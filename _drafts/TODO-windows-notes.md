
# Windows notes

## Services

### services.exe

services.exe (aka. Service Control Manager or SCM for short) is a system process that starts at system boot and is
responsible for running and managing services on the system. It keeps track of all the services that are installed via
a key in the registry called the SCM database, which is located in `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services`.

When the SCM starts at system boot, it launches all the services that are marked with "auto-start" and all other
dependencies needed to run the services.


All Windows services are spawned as children of the "services.exe" process and when looking at the "services.exe"
process we'll see two kinds of processes:
- processes that are hosting their own services
- "svchost.exe" processes.

### svchost.exe

> From [Microsoft Docs](https://docs.microsoft.com/en-us/windows/application-management/svchost-service-refactoring):
>
> The Service Host (svchost.exe) is a shared-service process that serves as a shell for loading services from DLL files.

svchost.exe (aka. the Service Host process) is a system process that can host from one or more Windows services. Svchost
is an implementation where a number of services can share a process in order to reduce resource consumption.

When using the task manager to see what processes are opened, you'll get a bunch of "svchost.exe" processes with the
description "Host Process for Windows Services". Without any information about the services that are hosted in it.
In this context, **as the old task manager is concerned, a malware could be named "svchost.exe" with the description
"Host Process for Windows Services" and it will make it legitimate and indistinguishable from the legitimate
"svchost.exe" process**.

Inspecting one of the multiple Svchost processes with a tool like Process Explorer could reveal the instance is hosting
one, two or more services.

So the question to answer is how does  simple command line arguments reference multiple services. To explain this let us take a look at the arguments and what they mean.

#### Command line arguments

**The "k" Flag**

When "svchost.exe" uses the "-k" flag, a request will be made to the registry key `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost` where it will try to locate the value corresponding to the one sent via the "-k" flag. This registry key defines values for Service Host Groups. Each value will contain a string that contains references to the services to launch once the "-k" flag is used.

Once its reads these values and no other flag was specified, it will go and load each service referenced inside form its corresponding registry key `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\[Service Name]` and that's how you get multiple services on a single "svchost.exe" process.

**The "s" Flag**

When the "-s" flag is used, this will tell the "svchost.exe" process to load only the service specified from the selected already specified with "-k".

For example, "svchost.exe-k UnistackSvcGroup -s CDPUserSvc" will be requesting the "CDPUserSvc" service inside the "UnistackSvcGroup" registry key and loading that.

### Listing services in CMD

TaskList displays all running applications and services with their Process ID (PID).

**List the services running under each process**

```batch
tasklist /svc
```

**List the services running under each SvcHost process**

```batch
tasklist /fi "imagename eq svchost.exe" /svc
```

**List the services running now**

```batch
tasklist /v /fi "STATUS eq running"
```

**List the services with an ImageName that starts with "C" - notice that a wildcard can only be used at the end of the string**

```batch
tasklist /fi "IMAGENAME eq c*"
```

**List the services running under a specific user account**

```batch
tasklist /v /fi "username eq SERVICE_ACCT05"
```
