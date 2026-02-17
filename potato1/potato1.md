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

次に`nmap`を実行していきます。

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