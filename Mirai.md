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

# Mirai
![スクリーンショット 2020-06-17 17.09.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e975a633-e0bd-017f-9f22-df0d6be89821.png)


HackTheBox公式より
Mirai demonstrates one of the fastest-growing attack vectors in modern times; improperly configured IoT devices. This attack vector is constantly on the rise as more and more IoT devices are being created and deployed around the globe, and is actively being exploited by a wide variety of botnets. Internal IoT devices are also being used for long-term persistence by malicious actors.

Miraiは、現代で最も急速に成長している攻撃ベクトルの1つである、不適切に設定されたIoTデバイスを実証しています。この攻撃ベクトルは、世界中でより多くのIoTデバイスが作成され、配備されているため、常に増加しており、さまざまなボットネットによって積極的に悪用されています。内部のIoTデバイスも、悪意のある行為者による長期的な持続性のために利用されています。


# スキャン
Mirai(10.10.10.48)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -Pn -oN mirai.nmap 10.10.10.48
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
┌─[✗]─[yukitsukai@parrot]─[~/htb/Mirai]
└──╼ $nmap -sC -sV -Pn -oN mirai.nmap 10.10.10.48
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 17:36 JST
Nmap scan report for 10.10.10.48
Host is up (0.25s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp   open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp   open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
1095/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.55 seconds
```
nmapの結果、22,53,80,1095番ポートが空いていることが分かります。
まずはブラウザからHTTPにアクセスしてみると、何も表示されず、何も見つかりません。
![スクリーンショット 2020-06-17 15.42.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9ee07271-c8a4-9e67-6041-843523ee5c25.png)

何か攻略の手がかりとなるディレクトリスキャナーをかけて何か情報が落ちていないか見てみましょう。

##Gobuster
```
gobuster dir -t 100 -u http://10.10.10.48 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,py -o mirai_gobuster.txt
```

```
┌─[yukitsukai@parrot]─[~/htb/Mirai]
└──╼ $gobuster dir -t 100 -u http://10.10.10.48 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,py -o mirai_gobuster.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.48
[+] Threads:        100
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt,py,php
[+] Timeout:        10s
===============================================================
2020/06/15 17:41:11 Starting gobuster
===============================================================
/admin (Status: 301)
/versions (Status: 200)
===============================================================
2020/06/15 18:19:09 Finished
===============================================================
```

この結果から/adminにアクセスできることが分かります。
ブラウザからアクセスしてみましょう。
![スクリーンショット 2020-06-17 15.46.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4a02dad6-8484-7751-28a5-b28f883dc586.png)

ログイン画面が見つかりますが、パスワードが分かっていないためログインできません。  
マシン名がmiraiという名前からデフォルトのパスワードが設定されていると予測し、デフォルトパスワードについて調べてみます。

![スクリーンショット 2020-06-17 15.33.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/567158a3-0512-7549-766c-abb22134a9fe.png)

いろいろ調べてみたのですが、パスワードの確認を行うことはできず、シェルを取得した後変更をすることは可能のようです。（間違っていたらすみません）  

次にnmapの結果から、sshのポートが空いていることが分かっているのでRaspberry Piのデフォルトパスワードを調べてみます。

![スクリーンショット 2020-06-17 15.17.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/cb3efce2-8e6f-321b-9b68-27aa70ace7c3.png)

ユーザ名がpiでパスワードがraspberryと言うことが分かります。
この情報を使ってsshでログインしてみましょう。

# 侵入
```
ssh pi@10.10.10.48
```

![スクリーンショット 2020-06-17 15.55.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/98bf8dd7-2b17-6e30-54ba-afeafbf2acab.png)

ログインできました。  
miraiと言う名前からsshに何らかのデフォルトパスワードでログインするのではないか？という推測ができれば比較的に簡単に攻略できるものでした。  
https://monoist.atmarkit.co.jp/mn/articles/2003/13/news027.html  
この記事でもsshは第3位の標的とされています。

# 特権エスカレーション

```
sudo -l
```
- -l...sudoを実行するユーザーに許可されているコマンドを一覧表示する。
![スクリーンショット 2020-06-17 15.57.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/713e1aaf-b6d9-3b61-7dd1-04ca35a1dd00.png)

このことから全てのユーザがsudoを許可されていることが分かります。  
sudo suを使って権限を昇格させましょう。  
![スクリーンショット 2020-06-17 16.18.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a90d64f2-bbb3-a9ad-e91c-e39bed6a632f.png)

rootも取得できたので、これで終わりかと思いきや、実は続きがあります。  
root.txtの中身を見ると、メッセージが残させていることが分かります。  

![スクリーンショット 2020-06-17 16.20.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/00006652-86c0-c1a7-f9ed-29f70986b8ca.png)

USB stickにroot.txtをバックアップさせているとメッセージが残されているのでUSB stickをチェックします。

```
df
```
dfコマンドはディスクの使用状況を表示するコマンドです。

![スクリーンショット 2020-06-17 16.30.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3777949c-5a8d-121e-63e2-69d872d526fb.png)

/media/usbstickというそれらしきものが見つかります。
ディレクトリを移動して確認してみましょう。

![スクリーンショット 2020-06-17 16.33.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/45d2c384-e558-f6db-caf1-c353384ce79e.png)

damnit.txtと言うものが見つかります。  
確認してみると、今度はUSBメモリからデータを削除してしまった。というメッセージが残されています。  
ちなみにdamnitは「くそっ！」「ちくしょう！」を表すスラングのようです。  
 
/dev/sdbに/media/usbstickがあったのでddコマンドを用いてUSBドライブのコピーをしてから中身を見れるように作業を行います。

```
dd if=/dev/sdb of=/home/usbstick.txt
```

stringsコマンドで確認してみます。
stringsコマンドはバイナリファイルやデータファイルから“文字列”として読める箇所を表示するコマンドなので、読めるところを抽出してくれます。

![スクリーンショット 2020-06-17 17.02.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fdd15bf4-c39b-3e21-4a24-335ddf8b2313.png)
rootフラグを見つけることができました。
ちなみにcatでもフラグは確認できました。
![スクリーンショット 2020-06-17 16.41.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/0abe102d-1371-a5a6-3d8a-9d713c59ff92.png)
お疲れ様でした。
