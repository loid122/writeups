# Lab: 2FA broken logic
First when we login with our own creds , we see that server is settign 2 cookies and we get 2FA on our email server , which proves that we are getting a code
\
<img width="1374" height="584" alt="image" src="https://github.com/user-attachments/assets/d5a404b6-3f5a-496c-92e5-56504d061489" />
\
<img width="1489" height="658" alt="image" src="https://github.com/user-attachments/assets/eb5de0ef-6cf8-4e37-b4db-100dfe7dde0b" />
\
Then intercept the get request of /login2 , and change the verify name to victim name "carlos" .This ensures that a temporary 2FA code is generated for Carlos.
\
Then we send the post login request to intruder and bruteforce the mfa code
<img width="1260" height="380" alt="image" src="https://github.com/user-attachments/assets/e4fd7728-00ea-4406-a551-d7c82eb3bf39" />
\
Then get the response session cookie and then edit the cookie value and verify name and go to ?id=carlos
<img width="868" height="411" alt="image" src="https://github.com/user-attachments/assets/50e56d90-4b6a-407b-99fe-58c9f8a54920" />
