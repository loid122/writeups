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
