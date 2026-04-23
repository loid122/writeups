# Tempus Fugit: 3 Walkthrough
# Description
```bash
Tempus Fugit is a Latin phrase that roughly translated as “time flies”.

This is an hard, real life box, created by @4nqr34z and @theart42 to be used as a CTF challenge on Bsides Newcastle 23. november 2019 and released on Vulnhub the same day.

In Tempus Fugit 3, the idea is still, like in the first two challenges; to create something “out of the ordinary”.

The vm contains 5 flags. If you don’t see them, you are not looking in the right place...

Need any hints? Feel free to contact us on Twitter: @4nqr34z or @theart42

DHCP-Client.

Tested both on Virtualbox and vmware

Health warning: For external use only

This works better with VirtualBox rather than VMware
```

# Exploitation
Let's start with network scan
```bash
Currently scanning: 192.168.0.1/24   |   Screen View: Unique Hosts                                                                                                                                               
                                                                                                                                                                                                                  
 11 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 660                                                                                                                                                 
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.100   58:11:22:85:0d:41      8     480  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.110   08:00:27:07:f8:19      1      60  PCS Systemtechnik GmbH
```
\
NExt , Open ports on the target machine
```bash
80/tcp http  nginx 1.14.2
```
\
Accessing port 80 on burpsuite
\
<img width="1712" height="642" alt="image" src="https://github.com/user-attachments/assets/bd46e24c-3820-49cf-9c9e-c4bec93fd60f" />
\
Nothing much on this page , just a login functionality
\
<img width="817" height="181" alt="image" src="https://github.com/user-attachments/assets/0e52ca58-be96-4a4f-a396-7221827779e9" />
\
Got pranked when i entered random credentials
\
<img width="1646" height="745" alt="image" src="https://github.com/user-attachments/assets/b9a67cd0-b55a-4c71-93bc-12a70c36ff60" />
\
When i entered a random endpoint to access
\
<img width="1707" height="762" alt="image" src="https://github.com/user-attachments/assets/250977ec-6337-45ee-b5f4-3611ceba9363" />
\
Since the page was displaying the "path i gave after /" i thought it could be SSTI and tested it
\
And it was Jinja ssti 
\
<img width="569" height="761" alt="image" src="https://github.com/user-attachments/assets/f8046543-7173-446a-a859-4107156f6f79" />
\
And i followed "https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md" for further payloads
\
When i first checked if i could access the builtin functions and see which all were supported
\
<img width="519" height="124" alt="image" src="https://github.com/user-attachments/assets/5befdb30-17a4-42fe-98d2-8edd116ecd64" />
\
There were a lot 
\
<img width="1693" height="677" alt="image" src="https://github.com/user-attachments/assets/5a07465e-42c3-4d34-a628-16f5b9f38a80" />
\
So i copied it to a notepad file and searched for "__import"" 
\
<img width="1076" height="507" alt="image" src="https://github.com/user-attachments/assets/516af5a8-bbaf-413f-bd42-baaa6682150e" />
\
And it was there so i could use the next payload i found this from the cheatsheet and used the payload <img width="802" height="77" alt="image" src="https://github.com/user-attachments/assets/3368a570-6abf-4b5f-928e-9af752cc5e6b" />

\
<img width="1240" height="774" alt="image" src="https://github.com/user-attachments/assets/9c0efaca-45af-4fb2-8842-f57899ab7ce3" />
\
