# 【VuluHub】Potato: 1

# はじめに

CTFのBoot2Root問題を練習したいと思ったので、以前から気になっていたVulnHubのマシンを攻略していきます！今回は「Potato: 1」というマシンを攻略します！

https://www.vulnhub.com/entry/potato-1,529/

攻略するにあたっての初期設定は、以下の記事を参考にさせていただきました。

https://qiita.com/kodai-160/items/c162e4741315dcf737e9

# 偵察

まずは初めに、ターゲットマシンのIPアドレスを特定します。`192.168.56.100`はDHCPのIPアドレスなので、PotatoマシンのIPアドレスは`192.168.56.101`となります。
```
┌──(kali㉿kali)-[~/VuluHub/Potato1]
└─$ sudo netdiscover -i eth1 -r 192.168.56.103/24

Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                                            
                                                                                                                                                                                                                                          
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                                          
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.56.1    0a:00:27:00:00:14      1      60  Unknown vendor                                                                                                                                                                         
 192.168.56.100  08:00:27:ca:c7:0a      1      60  PCS Systemtechnik GmbH                                                                                                                                                                 
 192.168.56.101  08:00:27:b3:6b:30      1      60  PCS Systemtechnik GmbH 

 ```

## ポートスキャン

次に`nmap`を実行していきます。nmapのオプションの説明ははあとで書く。

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS -sC -sV -p- 192.168.56.101     
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-15 12:47 +0900
Nmap scan report for 192.168.56.101
Host is up (0.00037s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
|   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
|_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Potato company
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
MAC Address: 08:00:27:B3:6B:30 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.52 seconds
```

# 列挙と初期侵入

## FTPの調査

FTPのAnonymousログインが有効になっていることがnmapの結果から分かったので、FTPにAnonymousログインし、2つのファイルをダウンロードして中身を見ていきます。

```
┌──(kali㉿kali)-[~/VuluHub/Potato1]
└─$ ftp -P 2112 192.168.56.101
Connected to 192.168.56.101.
220 ProFTPD Server (Debian) [::ffff:192.168.56.101]
Name (192.168.56.101:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230-Welcome, archive user anonymous@192.168.56.103 !
230-
230-The local time is: Sun Feb 15 12:52:43 2026
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> get index.php.bak
local: index.php.bak remote: index.php.bak
229 Entering Extended Passive Mode (|||44330|)
150 Opening BINARY mode data connection for index.php.bak (901 bytes)
   901       75.42 KiB/s 
226 Transfer complete
901 bytes received in 00:00 (62.23 KiB/s)
ftp> get welcome.msg
local: welcome.msg remote: welcome.msg
229 Entering Extended Passive Mode (|||29870|)
150 Opening BINARY mode data connection for welcome.msg (54 bytes)
    54       45.38 KiB/s 
226 Transfer complete
54 bytes received in 00:00 (8.49 KiB/s)
ftp> exit
221 Goodbye.
```

`welcome.msg`には特にめぼしい情報はなかったため、`index.php.bak`を覗いていきます。

```
┌──(kali㉿kali)-[~/VuluHub/Potato1]
└─$ cat index.php.bak
<html>
<head></head>
<body>

<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>


  <form action="index.php?login=1" method="POST">
                <h1>Login</h1>
                <label><b>User:</b></label>
                <input type="text" name="username" required>
                </br>
                <label><b>Password:</b></label>
                <input type="password" name="password" required>
                </br>
                <input type="submit" id='submit' value='Login' >
  </form>
</body>
</html>
```

`username : admin`, `password: potato`でログインができそうです...

## HTTPの調査

webサービスにアクセスします。(しゃしんをはる)

`gobuster`を実行します。

```
┌──(kali㉿kali)-[~/VuluHub/Potato1]
└─$ gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.101
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 279]
.htaccess            (Status: 403) [Size: 279]
admin                (Status: 301) [Size: 316] [--> http://192.168.56.101/admin/]
.hta                 (Status: 403) [Size: 279]
index.php            (Status: 200) [Size: 245]
server-status        (Status: 403) [Size: 279]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
```

`/admin`にアクセスすると、ログインフォームが表示されたので、先ほどのusernameとpasswordでログインしようとしたところ、以下が表示されました。おそらくpasswordが変更されていると考えられます。(しゃしんはる)