<img width="1707" height="762" alt="image" src="https://github.com/user-attachments/assets/343e58eb-4ab1-484f-8fbb-833e25985399" /><img width="1646" height="745" alt="image" src="https://github.com/user-attachments/assets/38ccd26b-aaa3-4177-a759-5045eb0c0506" /><img width="1646" height="745" alt="image" src="https://github.com/user-attachments/assets/26f7d9d5-b7e5-46c6-9d88-02ea6798733d" /># Tempus Fugit: 3 Walkthrough
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
Using dirb i found another endpoint
\
<img width="473" height="335" alt="image" src="https://github.com/user-attachments/assets/1b2538b5-cfca-4b7e-bf6a-f7089a1defb8" />
\
Since the page was displaying the "path i gave after /" i thought it could be SSTI and tested it
\
<img width="926" height="705" alt="image" src="https://github.com/user-attachments/assets/dcadac6a-4874-4a8c-bcc0-15499bc6e489" />
\
And it was Jinja ssti and i followed "https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md"
\
<img width="1702" height="690" alt="image" src="https://github.com/user-attachments/assets/15504597-75d9-430c-995e-cbb3cb6f9950" />
\
Then i found this from the cheatsheet and used the payload <img width="802" height="77" alt="image" src="https://github.com/user-attachments/assets/3368a570-6abf-4b5f-928e-9af752cc5e6b" />

\
<img width="1240" height="774" alt="image" src="https://github.com/user-attachments/assets/9c0efaca-45af-4fb2-8842-f57899ab7ce3" />
\
