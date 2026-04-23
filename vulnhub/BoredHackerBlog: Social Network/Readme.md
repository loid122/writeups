# BoredHackerBlog: Social Network WAlkthrough
# DEscription
```bash
Leave a message is a new anonymous social networking site where users can post messages for each other. They've assigned you to test their set up. They do utilize docker containers. You can conduct attacks against those too. Try to see if you can get root on the host though.

Difficulty: Med

Tasks involved:

port scanning
webapp attacks
code injection
pivoting
exploitation
password cracking
brute forcing
Virtual Machine:

Format: Virtual Machine (Virtualbox OVA)
Operating System: Linux
Networking:

DHCP Service: Enabled
IP Address Automatically assign
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
 192.168.0.100   58:11:22:85:0d:41      2     120  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.107   08:00:27:41:ad:be      1      60  PCS Systemtechnik GmbH
```
\
Next, finding open ports on target machine
```bash
