# Warzone: 1 Writeup
# Description 
```bash
Info : Created and Tested in Virtual Box, maybe you need to write code
Based on : Crypto
Scenario : You are trying to gain access to the enemy system
Mission : Your mission is to get the silver and the gold trophy (user.txt, root.txt)
Hints : java decompiler
Twitter : @AL1ENUM
```

# Exploitation
Network scan
```bash
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 180                                                                                                                                                  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.100   58:11:22:85:0d:41      2     120  ASUSTek COMPUTER INC.                                                                                                                                          
 192.168.0.126   00:0c:29:11:ef:07      1      60  VMware, Inc.                                                                                                                                                   

```
\
Lets find open ports on this machine
```bash
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
5000/tcp open  http    Werkzeug httpd 1.0.1 (Python 3.7.3)
```
\
Enumerating port 21 we get 2 files note.txt and warzone-encrypt.jar
```bash
ftp 192.168.0.126                                                                                                                        
Connected to 192.168.0.126.
220 (vsFTPd 3.0.3)
Name (192.168.0.126:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||10859|)
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Oct 22  2020 .
drwxr-xr-x    3 ftp      ftp          4096 Oct 22  2020 ..
dr-xr-xr-x    2 ftp      ftp          4096 Oct 22  2020 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||40474|)
150 Here comes the directory listing.
dr-xr-xr-x    2 ftp      ftp          4096 Oct 22  2020 .
drwxr-xr-x    3 ftp      ftp          4096 Oct 22  2020 ..
-r--r--r--    1 ftp      ftp            77 Oct 22  2020 note.txt
-r--r--r--    1 ftp      ftp          5155 Oct 22  2020 warzone-encrypt.jar
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
229 Entering Extended Passive Mode (|||57946|)
150 Opening BINARY mode data connection for note.txt (77 bytes).
100% |**********************************************************************************************************************************************************************|    77      104.87 KiB/s    00:00 ETA
226 Transfer complete.
77 bytes received in 00:00 (59.53 KiB/s)
ftp> get warzone-encrypt.jar
local: warzone-encrypt.jar remote: warzone-encrypt.jar
229 Entering Extended Passive Mode (|||37311|)
150 Opening BINARY mode data connection for warzone-encrypt.jar (5155 bytes).
100% |**********************************************************************************************************************************************************************|  5155        6.84 MiB/s    00:00 ETA
226 Transfer complete.
5155 bytes received in 00:00 (3.51 MiB/s)
ftp> exit
221 Goodbye.
```
```bash
cat note.txt  
Attention, please encrypt always your password using the warzone-encrypt.jar
```
\
Then i decompiled the .jar file 
```bash
unzip warzone-encrypt.jar_Decompiler.com.zip 
Archive:  warzone-encrypt.jar_Decompiler.com.zip
  inflating: Other/I2.java           
  inflating: Other/K1.java           
  inflating: Other/I1.java           
  inflating: Other/Obfuscated.java   
  inflating: Other/K2.java           
  inflating: encrypt/Main.java       
  inflating: META-INF/MANIFEST.MF    
  inflating: crypto/AES.java
```
\
I first checked the Main.java file
```bash
cat encrypt/Main.java    
package encrypt;

import crypto.AES;
import java.util.Scanner;

public class Main {
   public static void main(String[] args) {
      System.out.println("Symmetric Encryption by Alienum");
      Scanner in = new Scanner(System.in);
      System.out.print("enter the password to encrypt : ");
      String password = in.nextLine();
      System.out.println("encrypted password : " + AES.encryptString(password));
      System.exit(0);
   }
}
```
\
Then i checked the AES.java file
```bash
cat crypto/AES.java 
package crypto;

import Other.Obfuscated;
import java.security.Key;
import java.security.MessageDigest;
import java.util.Base64;
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

public class AES {
   private static final IvParameterSpec DEFAULT_IV = new IvParameterSpec(new byte[19]);
   private static final String ALGORITHM = "AES";
   private static final String TRANSFORMATION = "AES/CBC/PKCS5Padding";
   private Key key;
   private IvParameterSpec iv;
   private Cipher cipher;

   public AES(String key) {
      this(key, 128);
   }

   public AES(String key, int bit) {
      this(key, bit, (String)null);
   }

   public AES(String key, int bit, String iv) {
      if (bit == 256) {
         this.key = new SecretKeySpec(getHash("SHA-256", key), "AES");
      } else {
         this.key = new SecretKeySpec(getHash("MD5", key), "AES");
      }

      if (iv != null) {
         this.iv = new IvParameterSpec(getHash("MD5", iv));
      } else {
         this.iv = DEFAULT_IV;
      }

      this.init();
   }

   private static byte[] getHash(String algorithm, String text) {
      try {
         return getHash(algorithm, text.getBytes("UTF-8"));
      } catch (Exception var3) {
         throw new RuntimeException(var3.getMessage());
      }
   }

   private static byte[] getHash(String algorithm, byte[] data) {
      try {
         MessageDigest digest = MessageDigest.getInstance(algorithm);
         digest.update(data);
         return digest.digest();
      } catch (Exception var3) {
         throw new RuntimeException(var3.getMessage());
      }
   }

   private void init() {
      try {
         this.cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
      } catch (Exception var2) {
         throw new RuntimeException(var2.getMessage());
      }
   }

   public String encrypt(String str) {
      try {
         return this.encrypt(str.getBytes("UTF-8"));
      } catch (Exception var3) {
         throw new RuntimeException(var3.getMessage());
      }
   }

   public String encrypt(byte[] data) {
      try {
         this.cipher.init(1, this.key, this.iv);
         byte[] encryptData = this.cipher.doFinal(data);
         return new String(Base64.getEncoder().encode(encryptData));
      } catch (Exception var3) {
         throw new RuntimeException(var3.getMessage());
      }
   }

   public static String encryptString(String content) {
      Obfuscated obs = new Obfuscated();
      AES ea = new AES(obs.getIV(), 128, obs.getKey());
      return ea.encrypt(content);
   }
}
```
\
Then i realised it was importing Other.Obfuscated , so i checked that file
```bash
cat Other/Obfuscated.java 
package Other;

public class Obfuscated {
   public String getIV() {
      return "w4rz0n3s3cur31vv";
   }

   public String getKey() {
      return "w4rz0n3s3cur3k3y";
   }
}
```
\
Then on port 5000 , i found a webapp
\
<img width="444" height="265" alt="image" src="https://github.com/user-attachments/assets/7d48ac52-9703-4aca-8c8e-7b9b5ab323ec" />
\
Then after some research found that its called railfence cipher
\
<img width="1779" height="440" alt="image" src="https://github.com/user-attachments/assets/f01a58f0-a662-40fc-abb0-a413085e326f" />
\
At the end of page source we find 
\
<img width="276" height="95" alt="image" src="https://github.com/user-attachments/assets/efd38aa7-561c-473a-ab5d-888cac1f26e4" />
\
So i try to decode it 
\
<img width="1802" height="443" alt="image" src="https://github.com/user-attachments/assets/fea25b59-cf4b-4390-84f8-dfe96818464e" />
\
Then i ran dirb on the webpage
```bash
dirb http://192.168.0.126:5000/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Apr  2 03:08:57 2026
URL_BASE: http://192.168.0.126:5000/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.126:5000/ ----
+ http://192.168.0.126:5000/console (CODE:200|SIZE:1985)                                                                                                                                                          
                                                                                                                                                                                                                  
-----------------
END_TIME: Thu Apr  2 03:09:10 2026
DOWNLOADED: 4612 - FOUND: 1
```
\
Reached here
\
<img width="1714" height="526" alt="image" src="https://github.com/user-attachments/assets/8b47b673-bcaf-45c9-aa3c-3e9880210904" />
\
I tried entering a random pin
\
<img width="1189" height="292" alt="image" src="https://github.com/user-attachments/assets/81036bff-19c3-43c8-ae9a-8f333d08eac4" />
\
I think there is a limit to number of attempts ("Exhaust") and its also verifying the auth.
\
This is default debugger of python flask console , so i dont think this is the intended way
\
We have to guess username and password , and the password should be encrypted using the given encrypt.jar, so i am leaving it here for now.
