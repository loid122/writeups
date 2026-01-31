# Prime-1 Writeup

Lets start with a network scan
```bash
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                
                                                                                                                                                                                                              
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                             
 192.168.0.123   00:0c:29:da:37:ed      1      60  VMware, Inc.
```
\
Next ,Open Ports 
```bash
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 00:0C:29:DA:37:ED (VMware)
```
\
Accessing port 80 , we get
\
<img width="1452" height="670" alt="image" src="https://github.com/user-attachments/assets/e8d71dcd-e897-40cb-a046-026f5500260d" />
\
Nothing on the homepage , so i run ffuf
```bash
ffuf -u http://192.168.0.123:80/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -r

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.123:80/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

# directory-list-2.3-medium.txt [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 1ms]
# Priority ordered case-sensitive list, where entries were found [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 3ms]
#                       [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 11ms]
# on at least 2 different hosts [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 10ms]
#                       [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 12ms]
# Copyright 2007 James Fisher [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 24ms]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 95ms]
# This work is licensed under the Creative Commons [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 124ms]
dev                     [Status: 200, Size: 131, Words: 24, Lines: 8, Duration: 2ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 207ms]
#                       [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 207ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 212ms]
javascript              [Status: 403, Size: 299, Words: 22, Lines: 12, Duration: 1ms]
# or send a letter to Creative Commons, 171 Second Street, [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 309ms]
                        [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 310ms]
#                       [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 427ms]
wordpress               [Status: 200, Size: 11019, Words: 464, Lines: 131, Duration: 1631ms]
                        [Status: 200, Size: 136, Words: 8, Lines: 8, Duration: 0ms]
server-status           [Status: 403, Size: 301, Words: 22, Lines: 12, Duration: 0ms]
```
\
Visiting "/dev" endpoint
\
<img width="634" height="260" alt="image" src="https://github.com/user-attachments/assets/18c120c5-f52e-41ff-9648-091100479375" />
\
Nothing again , so next "/wordpress" endpoint
\
<img width="1334" height="774" alt="image" src="https://github.com/user-attachments/assets/5f36f123-e03c-4913-b4a7-422abddd765a" />
\
We now have a wordpress site , so lets use wpscan to check for vuln 
```bash
wpscan --url http://192.168.0.123/wordpress/ -e                                               
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
 
[+] URL: http://192.168.0.123/wordpress/ [192.168.0.123]
[+] Started: Sat Jan 31 01:46:12 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.0.123/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.0.123/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://192.168.0.123/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.0.123/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.2.2 identified (Insecure, released on 2019-06-18).
 | Found By: Rss Generator (Passive Detection)
 |  - http://192.168.0.123/wordpress/?feed=rss2, <generator>https://wordpress.org/?v=5.2.2</generator>
 |  - http://192.168.0.123/wordpress/?feed=comments-rss2, <generator>https://wordpress.org/?v=5.2.2</generator>

[+] WordPress theme in use: twentynineteen
 | Location: http://192.168.0.123/wordpress/wp-content/themes/twentynineteen/
 | Last Updated: 2025-04-15T00:00:00.000Z
 | Readme: http://192.168.0.123/wordpress/wp-content/themes/twentynineteen/readme.txt
 | [!] The version is out of date, the latest version is 3.1
 | Style URL: http://192.168.0.123/wordpress/wp-content/themes/twentynineteen/style.css?ver=1.4
 | Style Name: Twenty Nineteen
 | Style URI: https://wordpress.org/themes/twentynineteen/
 | Description: Our 2019 default theme is designed to show off the power of the block editor. It features custom sty...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.4 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.0.123/wordpress/wp-content/themes/twentynineteen/style.css?ver=1.4, Match: 'Version: 1.4'

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:00 <===============================================================================================================================> (652 / 652) 100.00% Time: 00:00:00
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:02 <=============================================================================================================================> (2575 / 2575) 100.00% Time: 00:00:02

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <================================================================================================================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:00 <======================================================================================================================================> (75 / 75) 100.00% Time: 00:00:00

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:01 <===========================================================================================================================> (100 / 100) 100.00% Time: 00:00:01

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <=================================================================================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] victor
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Requests Done: 3593
[+] Cached Requests: 8
[+] Data Sent: 1.032 MB
[+] Data Received: 999.1 KB
[+] Memory used: 304.766 MB
[+] Elapsed time: 00:00:09
```
\
Next , i bruteforce vulnerable plugin detection , but nothing important is found
```bash
[i] Plugin(s) Identified:

[+] akismet
 | Location: http://192.168.0.123/wordpress/wp-content/plugins/akismet/
 | Last Updated: 2025-07-15T18:17:00.000Z
 | Readme: http://192.168.0.123/wordpress/wp-content/plugins/akismet/readme.txt
 | [!] The version is out of date, the latest version is 5.5
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.0.123/wordpress/wp-content/plugins/akismet/, status: 200
 |
 | Version: 4.1.2 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.0.123/wordpress/wp-content/plugins/akismet/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://192.168.0.123/wordpress/wp-content/plugins/akismet/readme.txt
```
\
So i tried bruteforcing the password for user "victor" on wordpress for some time , still no use
\
So i return back and start enumerating with gobuster
```bash
gobuster dir -t 100 -u http://192.168.0.123/  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x txt,php,html,zip,php.bak -r       
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.123/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,zip,php.bak,txt,php
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/image.php            (Status: 200) [Size: 147]
/dev                  (Status: 200) [Size: 131]
/index.php            (Status: 200) [Size: 136]
/wordpress            (Status: 200) [Size: 11019]
/javascript           (Status: 403) [Size: 299]
/secret.txt           (Status: 200) [Size: 412]
/server-status        (Status: 403) [Size: 301]
Progress: 1323342 / 1323342 (100.00%)
===============================================================
Finished
===============================================================
```
\
We missed a file "secret.txt" in our initial scan!!! 
\
<img width="646" height="298" alt="image" src="https://github.com/user-attachments/assets/0ef6b633-2de6-4abe-923e-9b323f2a8763" />
\
