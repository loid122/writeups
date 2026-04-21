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
