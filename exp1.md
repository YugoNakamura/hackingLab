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
   すべてのパケットがpotatoに届いたことが分かった
4. 
 