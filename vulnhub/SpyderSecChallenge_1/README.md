# SpyderSecChallenge_1 writeup
After booting up the virtualbox machine , I start initial netdiscover scan to get it's ip
since im using the vm on bridged network adapter, i use 
sudo netdiscover -r 192.168.1.1/24
<img width="586" height="16" alt="image" src="https://github.com/user-attachments/assets/a43ff88e-0eee-463d-b854-acc8186b0c4d" />

next to find open ports on 192.168.1.8 , i use rustscan ( it is very fast compared to nmap )
( insert rustscan report here )
we find port 80 open , so lets launch burpsuite and from the proxy tab , open the chromium browser
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8c08409a-bb37-49a7-9f87-ef48fb644390" />
