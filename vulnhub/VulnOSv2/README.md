# VulnOSv2 writeup
after scanning we see 3 ports open 
\
<img width="325" height="72" alt="image" src="https://github.com/user-attachments/assets/b875cea8-9dbe-4124-a9b7-7c27ee8e4023" />
\
accessing home page tells us to access a directory
\
<img width="1307" height="470" alt="image" src="https://github.com/user-attachments/assets/2986b00c-0218-4943-a7cd-cdac690c2fcf" />
\
<img width="1438" height="708" alt="image" src="https://github.com/user-attachments/assets/5c8675e4-2c86-4ff7-9ed3-5ea939b3a377" />
\
Using wappalyzer it told me it is running drupal v7
\
<img width="1399" height="590" alt="image" src="https://github.com/user-attachments/assets/3c5c2a64-5aed-4ee8-959a-60160bb2db35" />
\
So i run msfconsole , and search for exploits
\
<img width="1284" height="663" alt="image" src="https://github.com/user-attachments/assets/5413430c-cfcf-4554-affa-d180179008da" />
\
I use the 7th one, after setting things i got a shell as www-data
\
<img width="833" height="208" alt="image" src="https://github.com/user-attachments/assets/3be7823f-f906-4ad4-bba1-9e3f8b2cb76c" />
\
after enumeration ,we check that kernel is vulnerable to privilege escalation and for exploits in exploitdb
\
<img width="847" height="44" alt="image" src="https://github.com/user-attachments/assets/5fee884e-1dfa-4319-a3cb-bc38496c5847" />
\
```bash
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation                                                                             | linux/local/37292.c
```
\
we download and complie and run the exploit to get root shell
\
<img width="626" height="689" alt="image" src="https://github.com/user-attachments/assets/bf57014b-01dc-4f7d-9b7a-1ff5e1e85567" />
\
The End , thank you for reading till the end
