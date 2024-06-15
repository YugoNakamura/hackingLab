# EXPERIMENT1 potatoのハッキング
1. parrot OS(攻撃者側)のipアドレスを調べる   
   ```
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:19:83:cf brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.101/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s3
       valid_lft 361sec preferred_lft 361sec
    inet6 fe80::ce56:abf7:2f3a:7cc3/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    ```
    自分のipアドレスは**192.168.56.101/24**であることが判明した
2. potato(被害者側サーバー)のipアドレスを調べる
   ```
   >sudo netdiscover -i enp0s3 -r 192.168.56.0/24
   
     Currently scanning: 192.168.56.0/24   |   Screen View: Unique Hosts           
                                                                               
    3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180               
    _____________________________________________________________________________
    IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
    -----------------------------------------------------------------------------
    192.168.56.1    0a:00:27:00:00:12      1      60  Unknown vendor              
    192.168.56.100  08:00:27:64:23:cf      1      60  PCS Systemtechnik GmbH      
    192.168.56.103  08:00:27:25:34:e4      1      60  PCS Systemtechnik GmbH
   ```
   iオプションはネットワークデバイス名，rオプションはネットワークアドレス(探索範囲)  
   Virtual Boxのホストオンリーアダプタ上の結果なので上の2つのipアドレスの正体は以下の通り
   - 192.168.56.1はVirtual Box上でゲストOS(parrot OSやpotato)から見たホストOS
   - 192.168.56.100はDHCPサーバー  
    →消去法で192.168.56.103がpotatoのipアドレス
3. potatoと疎通確認する  
   pingを送って通信できることを確認する
   ```
    └──╼ $ping 192.168.56.103 -c 5
    PING 192.168.56.103 (192.168.56.103) 56(84) bytes of data.
    64 bytes from 192.168.56.103: icmp_seq=1 ttl=64 time=0.822 ms
    64 bytes from 192.168.56.103: icmp_seq=2 ttl=64 time=1.68 ms
    64 bytes from 192.168.56.103: icmp_seq=3 ttl=64 time=1.53 ms
    64 bytes from 192.168.56.103: icmp_seq=4 ttl=64 time=1.69 ms
    64 bytes from 192.168.56.103: icmp_seq=5 ttl=64 time=0.567 ms

    --- 192.168.56.103 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4052ms
    rtt min/avg/max/mdev = 0.567/1.255/1.687/0.468 ms
   ```
   cオプションんは送信するパケット数  
   potatoにアクセスが可能であることが分かった
4. ポートスキャンする  
   potatoで提供しているサービスとそのポート番号の一覧を取得する
   ```
      └──╼ $sudo nmap -sC -sV -Pn -p- 192.168.56.103
      Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-15 22:07 JST
      Nmap scan report for 192.168.56.103
      Host is up (0.00028s latency).
      Not shown: 65532 closed tcp ports (reset)
      PORT     STATE SERVICE VERSION
      22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
      | ssh-hostkey: 
      |   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
      |   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
      |_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
      80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
      |_http-title: Potato company
      |_http-server-header: Apache/2.4.41 (Ubuntu)
      2112/tcp open  ftp     ProFTPD
      | ftp-anon: Anonymous FTP login allowed (FTP code 230)
      | -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
      |_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
      MAC Address: 08:00:27:25:34:E4 (Oracle VirtualBox virtual NIC)
      Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

      Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
      Nmap done: 1 IP address (1 host up) scanned in 32.92 seconds
   ```
   - sCオプション：デフォルトカテゴリーのスクリプトでスキャン
   - sVオプション：ソフトウェアのバージョン名を教示
   - Pnオプション：スキャン前にそつうかくにんをしない
   - -p-オプション：全ポートをスキャン
  
5. 
 