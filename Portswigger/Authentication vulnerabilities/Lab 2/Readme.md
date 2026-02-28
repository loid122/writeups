# Lab: Username enumeration via subtly different responses

similar basic bruteforcing , but when checking for usernames , we do a negative regex search with the exact failure response "Invalid username or password." 
\
<img width="437" height="96" alt="image" src="https://github.com/user-attachments/assets/6ba2b867-5a61-48bf-a3cf-ddb2a98ed319" />
\
The correct username wont have a "." at the end. \n
We just do a normal pw bruteforce
\
<img width="1237" height="418" alt="image" src="https://github.com/user-attachments/assets/28af8909-f674-45f9-8f49-648b907af913" />
\
