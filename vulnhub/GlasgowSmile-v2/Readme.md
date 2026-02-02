# GlasgowSmile-v2 Writeup
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                         
                                                                                                                                                                                                              
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                             
 192.168.0.129   00:0c:29:a2:5a:01      1      60  VMware, Inc.
```
\
Next, Open Ports
```bash
PORT   STATE SERVICE    REASON
22/tcp   open     ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open     http       Apache httpd 2.4.38 ((Debian))
83/tcp   open     http       Apache httpd 2.4.38 ((Debian))
```
\
I opened both port 80 and 83 , and they were the same page
\
<img width="1443" height="753" alt="image" src="https://github.com/user-attachments/assets/109d7bb0-583b-4007-ac15-b5e618732892" />
\
Then i do some enumeration on both port 80 and 83 , both gave same results
```bash
dirsearch -u http://192.168.0.129:83/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                                                                                                                                                   
 (_||| _) (/_(_|| (_| )                                                                                                                                                                                            
                                                                                                                                                                                                                   
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/reports/http_192.168.0.129_83/__26-02-02_10-27-18.txt

Target: http://192.168.0.129:83/

[10:27:18] Starting:                                                                                                                                                                                               
[10:27:19] 403 -  278B  - /.ht_wsr.txt                                      
[10:27:19] 403 -  278B  - /.htaccess.bak1                                   
[10:27:19] 403 -  278B  - /.htaccess.sample                                 
[10:27:19] 403 -  278B  - /.htaccess.save
[10:27:19] 403 -  278B  - /.htaccess.orig
[10:27:19] 403 -  278B  - /.htaccess_extra                                  
[10:27:19] 403 -  278B  - /.htaccess_sc
[10:27:19] 403 -  278B  - /.htaccessOLD
[10:27:19] 403 -  278B  - /.htaccessBAK
[10:27:19] 403 -  278B  - /.htaccess_orig
[10:27:19] 403 -  278B  - /.htaccessOLD2                                    
[10:27:19] 403 -  278B  - /.htm                                             
[10:27:19] 403 -  278B  - /.html
[10:27:19] 403 -  278B  - /.htpasswd_test                                   
[10:27:19] 403 -  278B  - /.htpasswds
[10:27:19] 403 -  278B  - /.httr-oauth
[10:27:19] 403 -  278B  - /.php                                             
[10:27:35] 301 -  322B  - /javascript  ->  http://192.168.0.129:83/javascript/
[10:27:44] 403 -  278B  - /server-status                                    
[10:27:44] 403 -  278B  - /server-status/
[10:27:48] 200 -  286B  - /todo.txt                                         
                                                                             
Task Completed
```
\
<img width="1105" height="609" alt="image" src="https://github.com/user-attachments/assets/cced9ea7-fbe8-46ec-a263-97bbcbefd815" />
\

