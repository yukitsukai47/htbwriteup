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

# Netmon
![コメント 2020-07-27 150137.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8449e15c-cc54-5d35-e667-81f96351dd3c.png)


HackTheBox公式より
Netmon is an easy difficulty Windows box with simple enumeration and exploitation. PRTG is running, and an FTP server with anonymous access allows reading of PRTG Network Monitor configuration files. The version of PRTG is vulnerable to RCE which can be exploited to gain a SYSTEM shell.

Netmonは簡単な羅列と搾取の簡単な難易度のWindowsボックスです。PRTGが動作しており、匿名アクセスのFTPサーバでPRTG Network Monitorの設定ファイルを読み込めるようになっています。PRTGのバージョンは、悪用されてSYSTEMシェルを取得することができるRCEに脆弱性があります。

# スキャン
Netmon(10.10.10.152)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN netmon.nmap 10.10.10.152
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Mon Jul 27 01:23:19 2020 as: nmap -sC -sV -oN netmon.nmap 10.10.10.152
Nmap scan report for 10.10.10.152
Host is up (0.23s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_02-25-19  11:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 3m18s, deviation: 0s, median: 3m17s
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-07-27T05:27:27
|_  start_date: 2020-07-27T05:25:40

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 27 01:24:17 2020 -- 1 IP address (1 host up) scanned in 58.42 seconds
```
21番ポートではFTPがanonymousログインできる状態となっております。
80番ポートでPRTG Network Monitorというものが動作しています。
今回の動作しているPRTGにはsearshsploitの結果、RCEの脆弱性が存在するようです。

![スクリーンショット 2020-07-27 15.51.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/292dd805-00d2-1068-34b5-ee0f5b44fc72.png)


それぞれ見ていきましょう。

# 侵入
いきなりですが、FTPにanonymousログインをするとそのままuser.txtを取ることができてしまいます。
![スクリーンショット 2020-07-24 17.09.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9c6484e2-7c81-372b-788b-266e13bf2b3d.png)

ですので、ここからはroot権限を目指してもう一つの手がかりであるPRTGについて調べていきましょう。

# 特権エスカレーション
Netmonの80番ポートにアクセスしてみると、管理画面のようなものが現れました。
![スクリーンショット 2020-07-22 17.51.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6c5cef84-5fd2-46c2-b399-fe826f1db7b4.png)

ひとまずログインするために、デフォルトパスワードをインターネットで調べて入力してみます。
![スクリーンショット 2020-07-24 17.39.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/34045564-06d3-be96-340e-aeb4bc76b0b9.png)
検索したところデフォルトパスワードは
user:prtgadmin
password:prtgadmin
でしたが、これではログインすることができませんでした。
![スクリーンショット 2020-07-24 17.40.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d095746f-7aaa-49d7-f669-43b413f0daa4.png)

次にFTP経由でユーザ権限はあるので、PRTGに関するデータが保管されているディレクトリについて検索します。
![スクリーンショット 2020-07-24 18.04.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ef5f81b2-c580-aacf-8071-288056dd4491.png)

調べてみた結果、C:\Users\All Users\Application Data\Paessler\PRTG Network Monitorにデータがありそうです。
該当ディレクトリを確認したところ、「PRTG Configuration.dat」「PRTG Configuration.old」、「PRTG Configuration.old.bak」というファイルがあったのでそれぞれ読み解いていきます。

そうすると、PRTG Configuration.old.bakにパスワードらしきものを確認することができました。
パスワード:PrTg@dmin2018
![スクリーンショット 2020-07-24 18.15.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/28c987ff-be31-efab-4261-52c1008121f5.png)

この情報を使ってPRTG Network Monitorにアクセスしてみましょう。

![スクリーンショット 2020-07-24 17.40.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/79becffa-4c74-bfaa-779f-863b7c2f576a.png)

この情報では、ログインすることができませんでした。ここでパスワード推測を行い、「パスワード：PrTg@dmin2018」の2018を2019に変更してログインしてみます。そうすると無事ログインすることができます。（パスワード推測ってこういう単純なものでも難しいですよね...苦手です）

ログインに成功すると、このような画面が現れます。
![スクリーンショット 2020-07-24 18.19.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e28f69e2-2185-ffaa-1970-8e8e21a2d010.png)

次に最初のスキャンの段階において、見つけてあったRCEの脆弱性を利用します。
使用方法としてCookieの情報が必要なようです。
![コメント 2020-07-27 143907.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2d1f3aa1-5b31-0f17-c2c1-12069613a932.png)

ここでは、BurpSuiteを使ってCookie情報を取得しました。別にブラウザなどから確認していただいても大丈夫です。
![コメント 2020-07-27 143944.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/45afee1e-eccc-d2cf-e5bd-6f2a3649774a.png)

このCookie情報を使ってスクリプトを動かします。
![コメント 2020-07-27 144830.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/980ed13b-76ff-30eb-9f6d-9cf85cb9479a.png)

完了後スクリプトによって新たに、
user:pentest
password:P3nT2st!
というユーザが作成されています。

impacketのpsexec.pyを使って新たに作成されたユーザに接続します。
![コメント 2020-07-27 145136.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3ad7f4c6-b1e2-6030-49be-3f805519bad7.png)

root権限を取得しました。
お疲れ様でした。