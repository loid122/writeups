# Symphonos2 writeup

Let's start with a network scan 
```bash
 Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.103   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.113   00:0c:29:c1:ea:17      1      60  VMware, Inc.                                                                                                                                                   
 192.168.0.1     f0:a7:31:e9:bb:f8      1      60  TP-Link Systems Inc
```
\
Next , finding Open Ports
```bash
PORT    STATE SERVICE      REASON
21/tcp  open  ftp          syn-ack ttl 64
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```
\
