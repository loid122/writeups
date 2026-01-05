# Fristileaks 1.3 writeup
```bash
rustscan -a 192.168.1.13
```
Home page
<img width="1281" height="813" alt="image" src="https://github.com/user-attachments/assets/ae8dc793-97f8-4b8a-bb79-a0763b79b389" />
\
after visiting robots.txt file
\
<img width="437" height="157" alt="image" src="https://github.com/user-attachments/assets/6587dea4-3097-40f5-abec-aabf8221d9d3" />
\
all of the endpoints troll us, so we think , what is cola , beer and sisi  
\
they are drinks and when we see the picture on home page it says to drink "fristi" , so lets go to /fristi endpoint
\
<img width="511" height="471" alt="image" src="https://github.com/user-attachments/assets/68126261-6728-42ce-a77b-06acdaacca70" />
\
<img width="1157" height="715" alt="image" src="https://github.com/user-attachments/assets/ac405541-71e1-4000-a197-96b6e540eb6c" />
\
viewing source , we see a comment with a username "eezeepz"
\
<img width="744" height="93" alt="image" src="https://github.com/user-attachments/assets/a9dc173f-a49c-49d4-8bd7-1203c9ee5597" />
\
and if we scroll down it shows another base64 encoded content in comment
\
<img width="648" height="370" alt="image" src="https://github.com/user-attachments/assets/4d5b3f3c-dc35-441b-ab03-edefa28c8094" />
\
so we decode it and we get an png image which has text , so it probably is the password 
\
<img width="391" height="310" alt="image" src="https://github.com/user-attachments/assets/703b9f03-6269-423d-9d25-c9f451ad8a44" />
\
Now usign the creds eezeepz:keKkeKKeKKeKkEkkEk , we login
\
<img width="511" height="142" alt="image" src="https://github.com/user-attachments/assets/e67b27cc-8509-45e4-802d-7830ce5aed1a" />
\
we see a upload page, after trying different payloads , i used this double extension and changing content type
\
<img width="654" height="436" alt="image" src="https://github.com/user-attachments/assets/ec726be8-1eed-47aa-a343-4d9c9b5c01a8" />
\
<img width="1362" height="452" alt="image" src="https://github.com/user-attachments/assets/3255d116-6437-4d8a-a137-2d0d8c088a44" />
\
after uploading, we setup a listener and execute it by accessing the file at "http://192.168.1.13/fristi/uploads/1.php.gif"
\
we get shell as apache, aftter enumerating a bit , in /var/www , we have a notes.txt
\
<img width="562" height="251" alt="image" src="https://github.com/user-attachments/assets/fa3c5ca6-f4af-4205-bd9f-2d239675c765" />
\
which say that , adding any command to /tmp/runthis will be run as user "admin" , so we can either get an admin shell or take make the permission of /home/admin as world readable
with 
\
```bash
echo "/home/admin/chmod -R 777 /home/admin/" > /tmp/runthis
```
\
and now we can access /home/admin, we see a file cryptpass.py in the home directory
\
<img width="468" height="178" alt="image" src="https://github.com/user-attachments/assets/e1cc8749-830d-4c98-a70d-ac2e352bf51b" />
\
it's just basic rot13 of a flipped string , so after decoding we get "LetThereBeFristi!" 
\
now we can switch to user fristigod with creds fristigod:LetThereBeFristi!
\
after using sudo -l , we can see that fristi can run this file with this perm
\
<img width="642" height="247" alt="image" src="https://github.com/user-attachments/assets/71e15d51-f168-4ae0-b071-303c74ec4338" />
\
so we run this and become root
\
<img width="666" height="104" alt="image" src="https://github.com/user-attachments/assets/1c97d33b-6ebc-47fd-bedf-756f7c120ffe" />
\
<img width="575" height="437" alt="image" src="https://github.com/user-attachments/assets/45e222da-47bf-4e13-8f7c-c07c8ae8a332" />
\
The End , Thank you for readng till the end
