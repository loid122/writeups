# CengBox: 3 Walkthrough

# Description
```bash
Goal : Get user and root flag

Difficulty : Intermediate / Hard

Description : Some of us hold on to poems, songs, movies, books. I guess people can't hold onto people anymore. -- Oguz Atay

You know what you have to do. If you get stuck, you can get in touch with me on Twitter. @arslanblcn_

This machine works properly on Virtualbox.

Happy hacking :)

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
 192.168.0.103   58:11:22:85:0d:41      2     120  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.111   08:00:27:18:94:41      1      60  PCS Systemtechnik GmbH                                                                                                                                         

```
\
scanning for Open ports 
```bash
80/tcp  open   http     Apache httpd 2.4.18 ((Ubuntu))
443/tcp open   ssl/http Apache httpd 2.4.18 ((Ubuntu))
```
\
Lets open port 80 on home page 
\
<img width="1705" height="653" alt="image" src="https://github.com/user-attachments/assets/93818221-aa7c-4d65-b823-d3486ef9182e" />
\
WE notice that there is a https port 443, so we check the certificate and add it to /etc/hosts
\
<img width="546" height="666" alt="image" src="https://github.com/user-attachments/assets/ad1a7959-fc0c-44c9-91a1-661c838e5945" />
\
```bash
ffuf -u http://ceng-company.vm/ -H "Host: FUZZ.ceng-company.vm" \
-w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --fs 36015

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://ceng-company.vm/
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.ceng-company.vm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 36015
________________________________________________

dev                     [Status: 200, Size: 4764, Words: 124, Lines: 99, Duration: 5ms]
www.dev                 [Status: 200, Size: 4764, Words: 124, Lines: 99, Duration: 8ms]
:: Progress: [19966/19966] :: Job [1/1] :: 1257 req/sec :: Duration: [0:00:17] :: Errors: 0 ::
```
We check for subdomains and get 2 of them , lets add to our hosts file
\
<img width="1457" height="618" alt="image" src="https://github.com/user-attachments/assets/c9ae8a28-a522-4e68-a954-1711457ece78" />
\
We ahve a logiin page here , but no credentials , so we go back to main website to check for clues
\
From the home page , we get 2 links from profiles of 2 employees
\
<img width="1177" height="735" alt="image" src="https://github.com/user-attachments/assets/0ed0efd6-ba8c-4f6e-aa81-a161d9daf4c1" />
\
which gives a pretty useful hint
```bash
Hint for CengBox3 -- namelastname@ceng-company.vm
```
\
And there is a code.php
\
<img width="1685" height="741" alt="image" src="https://github.com/user-attachments/assets/f5d068a6-160f-4663-9bfe-d3bd88b92994" />
\
And also a reddit link from elizabeth
\
<img width="1204" height="487" alt="image" src="https://github.com/user-attachments/assets/600074de-62f1-4d19-87de-a144f872be12" />
\
So im assuming , that we need to use the email of elizabeth , then we can create a wordlist from the reddit post and bruteforce 
\
So i create a wordlist and then send the login reqeuest to burpsuite intruder
\
<img width="1211" height="343" alt="image" src="https://github.com/user-attachments/assets/993abce0-9af1-4741-95d2-1a8e7f89f75c" />
\
MAke sure the email consists of all lowercase letters 
\
Here we can see that this password gave a different response
\
<img width="1040" height="165" alt="image" src="https://github.com/user-attachments/assets/cd7506dc-2f49-41c8-967a-cb9479aab619" />
\
After signing in , we get this
\
<img width="1344" height="227" alt="image" src="https://github.com/user-attachments/assets/6b3899b4-b267-46c3-87b0-1d622cdf4ed9" />
\
I click on add poem , and it gives some fields to enter data
\
<img width="1182" height="575" alt="image" src="https://github.com/user-attachments/assets/113aa183-2b98-4c12-9de0-ace69dc36f14" />
\
Then i remember the github repo having the "poem" class, which was unserializing user given data 
```bash
"unserialize(urldecode($_REQUEST['data']));"
```
\
So i think we can get Remote Code Execution using this deserialization vulnerability since it calls the magic function "__destruct"
