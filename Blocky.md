# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。
主に記録用として記しています。
現在のランクはHackerです。
間違っていることも多いかと思いますが、よろしくお願いします。
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
Twitter:@yukitsukai1731

# Blocky
HackTheBox公式より
<img width="354" alt="スクリーンショット 2020-06-06 1.50.11.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/beaeb91b-1dc4-ef7b-879a-c8a7bc56bc7d.png">

Blocky is fairly simple overall, and was based on a real-world machine. It demonstrates the risks of bad password practices as well as exposing internal files on a public facing system. On top of
this, it exposes a massive potential attack vector: Minecraft. Tens of thousands of servers exist that are publicly accessible, with the vast majority being set up and configured by young and inexperienced system administrators.

Blockyは全体的にかなりシンプルで、実機をベースにしています。これは、公開されているシステム上の内部ファイルを公開することと同様に、悪いパスワードの実践のリスクを示しています。その上で
これは大規模な潜在的な攻撃ベクトルを 露呈しています マインクラフト 何万台ものサーバーが存在していますが、その大部分は若くて経験の浅いシステム管理者によって設定・設定されています。

# スキャン
Blocky(10.10.10.37)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -Pn -oN blocky.nmap 10.10.10.37
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
# Nmap 7.80 scan initiated Fri Jun  5 12:12:52 2020 as: nmap -sC -sV -Pn -oN blocky.nmap 10.10.10.37
Nmap scan report for 10.10.10.37
Host is up (0.25s latency).
Not shown: 996 filtered ports
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jun  5 12:13:26 2020 -- 1 IP address (1 host up) scanned in 33.55 seconds
```

80番ポートが空いているのでブラウザで開いてみてみましょう。
<img width="1153" alt="スクリーンショット 2020-06-06 1.22.58.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/111a56c5-79c8-663e-50ea-2599a5631647.png">

マインクラフトのようなトップ画面が出てきました。
メッセージを読むとまだサイトを構築中ということが分かります。

<img width="922" alt="スクリーンショット 2020-06-06 12.40.00.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/21f84a00-8164-8df6-3f16-8bd039915cc9.png">

**********************************
日本語訳：
皆さん、ようこそ。サイトとサーバーはまだ建設中ですので、今はあまり期待しないでください。

我々は現在、サーバーのためのwikiシステムと、プレイヤーの統計情報などを追跡するためのコアプラグインを開発しています。将来的には素晴らしいものがたくさん計画されています 🙂
**********************************

Webサーバが動いているということなのでgobusterで何かディレクトリがないか検査しましょう。

## gobuster

```
kali@kali:~/htb/Blocky$ gobuster dir -u http://10.10.10.37 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o blocky_gobuster
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.37
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/05 12:21:22 Starting gobuster
===============================================================
/wiki (Status: 301)
/wp-content (Status: 301)
/plugins (Status: 301)
/wp-includes (Status: 301)
/javascript (Status: 301)
/wp-admin (Status: 301)
/phpmyadmin (Status: 301)
===============================================================
2020/06/05 12:58:43 Finished
===============================================================
```

スキャン中の結果からWordPressを用いてサイトを構築していることがわかります。
wpscanも使用して、並行してスキャンを進めていきましょう。

## wpscan
```
kali@kali:~$ wpscan --url 10.10.10.37 -e u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.1
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.10.37/ [10.10.10.37]
[+] Started: Fri Jun  5 12:43:51 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.37/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://10.10.10.37/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.10.10.37/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.37/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.8 identified (Insecure, released on 2017-06-08).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.10.37/index.php/feed/, <generator>https://wordpress.org/?v=4.8</generator>
 |  - http://10.10.10.37/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.8</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
 | Last Updated: 2020-03-31T00:00:00.000Z
 | Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.3
 | Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8, Match: 'Version: 1.3'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <===================================================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] notch
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.10.10.37/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Notch
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Finished: Fri Jun  5 12:43:58 2020
[+] Requests Done: 24
[+] Cached Requests: 36
[+] Data Sent: 5.749 KB
[+] Data Received: 122.907 KB
[+] Memory used: 145.109 MB
[+] Elapsed time: 00:00:07
```

この結果からnotch,Notchというユーザが見つかります。

# 侵入
先ほどのgobusterの結果から/pluginをブラウザから閲覧してみます。
するとこのような2つのjarファイルが見つかります。
<img width="1157" alt="スクリーンショット 2020-06-06 1.38.56.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ca36949b-5bce-81d7-205f-5fc36f85272f.png">

見つけたファイルと解答するとclassファイルが見つかります。

```
kali@kali:~/htb/Blocky$ jar -xvf BlockyCore.jar 
  inflated: META-INF/MANIFEST.MF
  inflated: com/myfirstplugin/BlockyCore.class
```

このclassファイルをデコンパイルしてみましょう。
jadもありますが、手元の環境でインストールされていなかったためオンラインでデコンパイルしてくれるものを使用しました。

<img width="1239" alt="スクリーンショット 2020-06-06 11.51.02.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/38317137-3153-0571-a8b6-8bc182c7cd2b.png">

プレーンテキストでrootとそのpassを見つけることができます。
これをgobusterで見つけたphpmyadminに入力してみると無事アクセスに成功し、notchというユーザが確認できます。

<img width="1268" alt="スクリーンショット 2020-06-06 13.33.58.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/20b7bc5d-85c3-bf1a-18e2-6bcac33e7310.png">

これはwpscanでも出てきたユーザ名なので、notchと先ほど見つけたpassを使用してssh接続してみます。


```
kali@kali:~/htb/Blocky$ ssh notch@10.10.10.37
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Fri Jun  5 21:56:54 2020 from 10.10.14.7
notch@Blocky:~$ ls
minecraft  user.txt
```

ssh接続できました。続いてこのユーザにsudo権限があるか試してみます。
パスワードには先ほどと同じものを使用します。

```
notch@Blocky:~$ sudo -l
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
notch@Blocky:~$ cd ../../
notch@Blocky:/$ ls
bin   dev  home        lib    lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot  etc  initrd.img  lib64  media       opt  root  sbin  srv   tmp  var
notch@Blocky:/$ sudo cat root/root.txt
0a9694a5b4d???????????????????? notch@Blocky:/$
```

sudo権限があったため直接root.txtファイルの中身を見ることができました。
お疲れ様でした。
