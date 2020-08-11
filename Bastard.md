# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。
主に記録用として記しています。
現在のランクはHackerです。
間違っていることも多いかと思いますが、よろしくお願いします。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">  
チートシートも公開しておりますが知識がまだまだ不足しているため、学習経過とともに身につけた内容などを随時更新していきます。
GitHub(ペネトレーションテスト用チートシート):
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet
Twitter:@yukitsukai1731

# Bastard
![コメント 2020-08-09 202902.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/cd3415d1-ed05-fdb0-5c4c-a7817ac608a1.png)


HackTheBox公式より
Bastard is not overly challenging, however it requires some knowledge of PHP in order to modify and use the proof of concept required for initial entry. This machine demonstrates the potential severity of vulnerabilities in content management systems.

Bastardは過度に挑戦的なものではありませんが、最初のエントリに必要な概念実証を修正して使用するためには、PHPのある程度の知識を必要とします。本機は、コンテンツ管理システムにおける脆弱性の潜在的な深刻度を実証します。

# スキャン
Bastard(10.10.10.9)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN bastard.nmap 10.10.10.9
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Fri Aug  7 01:15:02 2020 as: nmap -sC -sV -oN bastard.nmap 10.10.10.9
Nmap scan report for 10.10.10.9
Host is up (0.26s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods:
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Aug  7 01:16:27 2020 -- 1 IP address (1 host up) scanned in 84.55 seconds
```

この結果から80番ポートが空いていることが分かります。またコンテンツ管理システムとしてDrupal 7が使われていることが分かります。  
アクセスしてみると、以下のページが表示されました。
![コメント 2020-08-09 213027.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/99d7686b-b1d7-b1f5-de65-8510e9ddfcb6.png)


nmapの表示された80番ポートの結果からCHANGELOG.txtが検出されているので中身を読んでみます。
![コメント 2020-08-07 144646.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/56af06b7-8455-6b01-dfd6-4a793d8d6cbb.png)
このテキストからDrupal 7.54で動いていることが分かります。
バージョンが分かったので使えるエクスプロイトがないか検索してみます。
# 侵入
![コメント 2020-08-07 153227.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/133303ac-1f05-e30f-f740-878462c31ff3.png)
今回はこのコードを使いました。
https://github.com/pimps/CVE-2018-7600

```
git clone https://github.com/pimps/CVE-2018-7600.git
```
これを使用すると相手サーバでコマンドが実行できるようです。  
シェルを確立したかったため、nc.exeを送り込むのに使えるコマンドはないか検索してみるとcertutil.exeが利用できることが分かります。
![コメント 2020-08-07 170644.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/37a5e0da-1126-abd6-1b23-c00628a4a35a.png)
サーバを立てて、nc.exeを渡す準備をしてからcertutil.exeを使用してncを受け渡します。
![コメント 2020-08-07 170953.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/aa78a34b-c051-ea14-6e03-0807babecde9.png)
これでサーバにnetcatを送り込むことに成功しました。  
続いて、送り込んだnetcatを使ってシェルを確立します。
![コメント 2020-08-07 171328.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/578319a9-64f2-97a8-3d22-d061b11ea327.png)
これでユーザ権限のシェルを取得することに成功しました。

# 特権エスカレーション
手がかりを探すためにWindows-Exploit-Suggesterを使います。
まずはsysteminfoの情報をテキストファイルにコピーしておきます。
![コメント 2020-08-11 165816.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/770ddb28-b903-489c-c1bd-618155e8d7ad.png)

```
./windows-exploit-suggester.py --update
```
出力されたxlsファイルと自身がコピーしておいたsysteminfoの内容が記載されたテキストファイルを指定します。

```
./windows-exploit-suggester.py --database <出力されたファイル> --systeminfo <systeminfoの内容が記載されたテキストファイル>
```

結果はこのとおりになりました。
https://github.com/AonCyberLabs/Windows-Exploit-Suggester
![コメント 2020-08-07 175407.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8cae23e9-635e-a7e4-49e6-67ce401805b1.png)
このうち、順にカーネルエクスプロイトを試して見た結果、MS10-059を使用すると権限昇格することができました。  
カーネルエクスプロイトには、以下のものを使用しました。
https://github.com/SecWiki/windows-kernel-exploits
![コメント 2020-08-11 165431.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/402abf1a-e814-4f48-d6e3-5ff872b2d424.png)
権限昇格させることができました。  
これでrootフラグを取得することができます。  
お疲れさまでした。