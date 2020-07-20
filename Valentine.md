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

# Valentine

![スクリーンショット 2020-07-20 16.58.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ef62d399-4a47-060d-4cfd-14a16e578132.png)


HackTheBox公式より
Valentine is a very unique medium difficulty machine which focuses on the Heartbleed
vulnerability, which had devastating impact on systems across the globe.

バレンタインは、世界中のシステムに壊滅的な影響を与えた「Heartbleed」の脆弱性に焦点を当てた、非常にユニークな中難易度のマシンです。

# スキャン
Valentine(10.10.10.79)に対して、スキャンを行います。
まずnmapからスキャンを開始します。
## nmap 

```
nmap -sC -sV -oN valentine.nmap 10.10.10.79
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -oN...通常出力
- -Pn...ping送信をせずにスキャンを行う

```
# Nmap 7.80 scan initiated Thu Jul 16 21:55:17 2020 as: nmap -sC -sV -oN valentine.nmap 10.10.10.79
Nmap scan report for 10.10.10.79
Host is up (0.24s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2020-07-17T01:59:08+00:00; +3m16s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 3m15s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jul 16 21:55:54 2020 -- 1 IP address (1 host up) scanned in 37.08 seconds
```

この結果から22、80、443番ポートが空いていることが分かります。ディレクトリスキャナーをかけて何か手がかりになる情報を見つけましょう。

<img width="1057" alt="1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/68b24bdf-5e3b-19f1-0f1f-8cd2bdc069d0.png">


## Gobuster

```
gobuster dir -t 25 -u http://10.10.10.79 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -o valentine.gobuster
```

```
kali@kali:~/htb/Valentine$ cat valentine.gobuster 
/index (Status: 200)
/index.php (Status: 200)
/dev (Status: 301)
/encode (Status: 200)
/encode.php (Status: 200)
/decode (Status: 200)
/decode.php (Status: 200)
/omg (Status: 200)
/server-status (Status: 403)
```

/devには2つのファイルが保存されていました。

<img width="1058" alt="2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3678e319-e079-9ef8-562c-c8ebd7f1db7d.png">

hype_key
![スクリーンショット 2020-07-20 16.45.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/615ae509-c53a-0b82-c0b2-83d8950b6220.png)

notes.txt
<img width="1058" alt="3.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/5558cddf-ed36-ff26-228d-ed5593181caf.png">

他にも/encode、/decodeというディレクトリがあるので、さきほど見つけたhype_keyを/decodeに入力してみました。
<img width="1058" alt="スクリーンショット 2020-07-19 22.49.10.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/fad438b6-ed64-6c82-f9a6-939bebb3c429.png">
...別のデコード方法が必要なようです。

そもそもhype_keyが16進数となっていたので、16進数を文字列変換してみました。
<img width="1058" alt="スクリーンショット 2020-07-19 22.49.27.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/57df0f95-6110-8724-2467-1616900f2e95.png">
すると、暗号化されたRSAキーが見つかりました。

## nmap_vlun

```
kali@kali:~/htb/Valentine$ nmap --script vuln -oN valentine.nmapvuln 10.10.10.79
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-19 09:45 EDT
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|_  Hosts are all up (not vulnerable).
Nmap scan report for 10.10.10.79
Host is up (0.25s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
80/tcp  open  http
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
|_  /index/: Potentially interesting folder
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
443/tcp open  https
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
|_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
|_  /index/: Potentially interesting folder
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2014-3704: ERROR: Script execution failed (use -d to debug)
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
| ssl-ccs-injection: 
|   VULNERABLE:
|   SSL/TLS MITM vulnerability (CCS Injection)
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h
|       does not properly restrict processing of ChangeCipherSpec messages,
|       which allows man-in-the-middle attackers to trigger use of a zero
|       length master key in certain OpenSSL-to-OpenSSL communications, and
|       consequently hijack sessions or obtain sensitive information, via
|       a crafted TLS handshake, aka the "CCS Injection" vulnerability.
|           
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0224
|       http://www.openssl.org/news/secadv_20140605.txt
|_      http://www.cvedetails.com/cve/2014-0224
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://www.openssl.org/news/secadv_20140407.txt 
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|_      http://cvedetails.com/cve/2014-0160/
| ssl-poodle: 
|   VULNERABLE:
|   SS POODLE information leak
|     State: VULNERABLE
|     IDs:  CVE:CVE-2014-3566  BID:70574
|           The SSL protocol 3.0, as used in OpenSSL through 1.0.1i and other
|           products, uses nondeterministic CBC padding, which makes it easier
|           for man-in-the-middle attackers to obtain cleartext data via a
|           padding-oracle attack, aka the "POODLE" issue.
|     Disclosure date: 2014-10-14
|     Check results:
|       TLS_RSA_WITH_AES_128_CBC_SHA
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
|       https://www.securityfocus.com/bid/70574
|       https://www.imperialviolet.org/2014/10/14/poodle.html
|_      https://www.openssl.org/~bodo/ssl-poodle.pdf
|_sslv2-drown: 

Nmap done: 1 IP address (1 host up) scanned in 120.25 seconds

```

この結果からHeartBleedの脆弱性があることが分かります。
HeartBleedとはOpenSSLの脆弱性であり、HeartBleedの脆弱性を抱えるサーバが攻撃を受けると、攻撃者は痕跡を残すことなく、PCのメモリから一度に64KBまでの情報を読み出すことができてしまうものです。
今回はこれらの情報を使って侵入を行っていきます。

# 侵入
まずはsearchsploitでHeartBleedに使えるものがないか探します。

<img width="995" alt="スクリーンショット 2020-07-19 22.57.10.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/bd1abe95-eb5e-fe16-3ad4-0ec0b9f73526.png">

今回はmultiple/remote/32764.pyを落としてきて使用します。

```
kali@kali:~/htb/Valentine$ rm 32764.py exploit.py
```

<img width="714" alt="スクリーンショット 2020-07-19 23.12.35.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/a37ccc0b-e38e-0efe-c456-48c4f3f67fa8.png">


```
$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
```
この文字列を/decodeに入れます。
base64なので、手元でデコードしても大丈夫でした。

<img width="1058" alt="スクリーンショット 2020-07-19 23.12.19.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/f928f2b5-89d6-7eab-df85-54f9f98494e2.png">

<img width="1058" alt="スクリーンショット 2020-07-19 23.12.58.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/532e767a-2d1f-2e05-f7ff-e385eb78dfbc.png">

このheartbleedbelievethehypeと言う文字列を使って先ほど見つけたrsaキーを複合します。

```
openssl rsa -in ssh_key -out d_ssh_key
```
![スクリーンショット 2020-07-20 17.12.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6d269075-4ab2-9a07-dfc8-998951386a59.png)

```
ssh -i d_ssh_key hype@10.10.10.79
```

<img width="625" alt="スクリーンショット 2020-07-20 0.00.01.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/3c769a41-c25f-5f59-c7b4-29ba94480f08.png">
これでユーザ権限をゲットすることができました。

# 特権エスカレーション
![スクリーンショット 2020-07-20 17.16.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/ee931e0c-f1f2-a0e0-6959-408a3d3e7ff5.png)

この情報から古いカーネルで動作していることが分かり、カーネルエクスプロイトが使えることが分かります。
このバージョンにはDirtyCowという有名なものが存在します。
![スクリーンショット 2020-07-20 17.29.21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2bcd1e77-2fc8-462d-3612-268f2d8aef07.png)

searchsploitで落としてきて、使用します。
![スクリーンショット 2020-07-20 17.31.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/6ea9c82d-69b1-563f-2f64-61539b5bc931.png)


```
mv 40838.c dirtycow.c
```

<img width="1030" alt="スクリーンショット 2020-07-20 0.30.39.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/1fe2ac3e-a9fe-915f-ef72-40b88637e02e.png">

ソースコードを見てみると、

```
gcc -pthread dirty.c -o dirty -lcrypt
```
のように、コンパイルを行なって使うようです。
pythonでサーバを立てて、valentineでdirtycow.cを受け取ります。
![スクリーンショット 2020-07-20 17.42.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/d1efd95b-e917-61f9-1e84-661e3719a8c2.png)
受け取ったdirtycow.cをコンパイルして実行します。
![スクリーンショット 2020-07-20 17.44.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/db0d2ecb-bcbe-0a9d-584c-4a05e96dfb00.png)

お疲れ様でした。
root権限を取得することができました。